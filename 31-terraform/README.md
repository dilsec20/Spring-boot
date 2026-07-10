# 🌍 Terraform — Complete In-Depth Guide

> **"Terraform is an Infrastructure as Code (IaC) tool. Instead of clicking through the AWS/GCP console to create servers and databases, you write code to define them. Version controlled, repeatable, and scalable infrastructure."**

---

## 📑 Table of Contents

1. [What is Infrastructure as Code (IaC)?](#1-what-is-infrastructure-as-code-iac)
2. [Why Terraform?](#2-why-terraform)
3. [Terraform Architecture & Core Concepts](#3-terraform-architecture--core-concepts)
4. [Installation & Setup](#4-installation--setup)
5. [The HCL Syntax (HashiCorp Configuration Language)](#5-the-hcl-syntax)
6. [Basic Workflow (Init, Plan, Apply)](#6-basic-workflow-init-plan-apply)
7. [Providers (AWS, GCP, Azure)](#7-providers-aws-gcp-azure)
8. [Variables and Outputs](#8-variables-and-outputs)
9. [State Management (The `.tfstate` file)](#9-state-management)
10. [Modules (Reusable Code)](#10-modules-reusable-code)
11. [Provisioners (Running scripts on boot)](#11-provisioners)
12. [Terraform + Spring Boot Example (AWS EC2 + RDS)](#12-terraform--spring-boot-example)
13. [Best Practices](#13-best-practices)
14. [Interview Questions & Answers (50+)](#14-interview-questions--answers-50)

---

## 1. What is Infrastructure as Code (IaC)?

```
The Old Way (Manual Provisioning):
  1. Log into AWS Console.
  2. Click "Launch EC2".
  3. Select Ubuntu, 2GB RAM.
  4. Create Security Group.
  5. Hope you remember exactly what you clicked when you need a staging environment 6 months later. 😬

The New Way (IaC):
  1. Write a `.tf` file describing the server (Ubuntu, 2GB RAM, Security Group).
  2. Run `terraform apply`.
  3. Commit the file to Git.
  4. Need a staging environment? Change one variable and run it again. ✅
```

---

## 2. Why Terraform?

1. **Declarative**: You say *WHAT* you want (e.g., "I want 3 EC2 instances"). Terraform figures out *HOW* to get there. (If you currently have 2, it adds 1. If you have 4, it destroys 1).
2. **Cloud Agnostic**: Works with AWS, GCP, Azure, DigitalOcean, Kubernetes, and 1000+ others. (Note: The *code* isn't write-once-run-anywhere, but the *tool* and *workflow* are exactly the same).
3. **State Tracking**: Terraform remembers what it built, so it only makes necessary changes.

---

## 3. Terraform Architecture & Core Concepts

*   **Provider**: A plugin that understands API interactions with a specific cloud (e.g., AWS Provider).
*   **Resource**: A piece of infrastructure you want to create (e.g., an EC2 instance, a VPC, an S3 bucket).
*   **Data Source**: A way to query information *already existing* in the cloud (e.g., finding the latest Ubuntu AMI ID).
*   **State**: A file (`terraform.tfstate`) where Terraform maps your code to the real-world resources it created.

---

## 4. Installation & Setup

```bash
# Ubuntu / Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify
terraform version
```

*Prerequisite: You must configure your cloud credentials on your machine (e.g., running `aws configure` to set up your access keys).*

---

## 5. The HCL Syntax

HCL (HashiCorp Configuration Language) is similar to JSON but much easier for humans to read and write.

```hcl
# BLOCK_TYPE "RESOURCE_TYPE" "LOCAL_NAME" {
#   CONFIG = VALUE
# }

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2
  instance_type = "t2.micro"

  tags = {
    Name = "MySpringAppServer"
  }
}
```

*   `aws_instance` is the **Resource Type** (defined by the AWS provider).
*   `web_server` is the **Local Name** (used only within Terraform to reference this block).

---

## 6. Basic Workflow (Init, Plan, Apply)

Create a file named `main.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-spring-app-bucket-2025"
}
```

**The 4 Core Commands:**

```bash
# 1. INITIALIZE (Downloads the AWS provider plugin)
terraform init

# 2. PLAN (Dry run. Shows exactly what will be created/modified/destroyed)
terraform plan

# 3. APPLY (Actually builds the infrastructure in the cloud. Prompts for 'yes')
terraform apply

# 4. DESTROY (Tears down everything managed by this Terraform code)
terraform destroy
```

---

## 7. Providers (AWS, GCP, Azure)

You must tell Terraform which plugins to download in the `terraform { required_providers {} }` block.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Configure the AWS Provider
provider "aws" {
  region = "us-west-2"
  # Authentication is usually picked up automatically from ~/.aws/credentials
}
```

---

## 8. Variables and Outputs

Hardcoding values is bad. Use variables.

**`variables.tf`** (Define the variables)
```hcl
variable "instance_type" {
  description = "The type of EC2 instance"
  type        = string
  default     = "t2.micro"
}

variable "db_password" {
  description = "Database admin password"
  type        = string
  sensitive   = true   # Hides value from CLI output
}
```

**`main.tf`** (Use the variables)
```hcl
resource "aws_instance" "app" {
  ami           = "ami-12345"
  instance_type = var.instance_type
}
```

**`terraform.tfvars`** (Pass values into the variables)
```hcl
instance_type = "t3.medium"
db_password   = "SuperSecret123!"
```

**`outputs.tf`** (Print useful info after apply)
```hcl
output "server_public_ip" {
  description = "The public IP address of the web server"
  value       = aws_instance.app.public_ip
}
# Output after 'terraform apply':
# Outputs:
# server_public_ip = "203.0.113.42"
```

---

## 9. State Management

*   When you run `terraform apply`, it creates a `terraform.tfstate` file (a giant JSON file).
*   **CRITICAL RULE:** Never edit the state file manually.
*   **CRITICAL RULE:** Never commit the state file to Git (it contains unencrypted passwords and secrets!).
*   **Solution: Remote State.** Store the state file in a centralized, secure location like AWS S3 or Terraform Cloud.

**Configuring S3 Remote State (in `main.tf`):**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks" # Prevents two devs from running apply at the same time!
  }
}
```

---

## 10. Modules (Reusable Code)

Modules are like functions. Instead of writing raw EC2 and VPC code every time, you can call a pre-written module.

```hcl
# Calling a community module to create an entire VPC (Networking)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-spring-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
}
```

---

## 11. Provisioners (Running scripts on boot)

You can run scripts *after* an instance is created using `user_data` (Cloud-Init) or Terraform provisioners.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c7217cdde317cfec" # Ubuntu
  instance_type = "t2.micro"

  # user_data runs ONCE when the instance boots up
  user_data = <<-EOF
              #!/bin/bash
              sudo apt update
              sudo apt install -y openjdk-21-jre
              echo "Java is installed!" > /tmp/status.txt
              EOF
}
```
*(Note: It is generally better to use Ansible/Packer for complex configuration, and let Terraform handle just the infrastructure).*

---

## 12. Terraform + Spring Boot Example

**Goal:** Create a Postgres Database (RDS) and an EC2 Server to run a Spring Boot app.

```hcl
# 1. Provide AWS Credentials (assumes profile is set up)
provider "aws" {
  region = "us-east-1"
}

# 2. Security Group for Database (Allow only EC2 to connect)
resource "aws_security_group" "db_sg" {
  name = "db_security_group"
  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    security_groups = [aws_security_group.app_sg.id]
  }
}

# 3. Create Postgres RDS
resource "aws_db_instance" "postgres_db" {
  allocated_storage    = 20
  engine               = "postgres"
  engine_version       = "15.3"
  instance_class       = "db.t3.micro"
  db_name              = "springdb"
  username             = "postgres"
  password             = "SuperSecretPass123"
  skip_final_snapshot  = true
  vpc_security_group_ids = [aws_security_group.db_sg.id]
}

# 4. Security Group for App (Allow HTTP 8080 and SSH 22)
resource "aws_security_group" "app_sg" {
  name = "app_security_group"
  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# 5. Create EC2 Instance
resource "aws_instance" "spring_app" {
  ami           = "ami-0c7217cdde317cfec" # Ubuntu 22.04
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.app_sg.id]
  
  tags = {
    Name = "SpringBoot-Prod"
  }
}

# 6. Output the connection strings!
output "app_url" {
  value = "http://${aws_instance.spring_app.public_ip}:8080"
}
output "db_endpoint" {
  value = aws_db_instance.postgres_db.endpoint
}
```

---

## 13. Best Practices

1. **Use Remote State**: Always store `.tfstate` in S3/GCS with state locking (DynamoDB). Never in Git.
2. **Modularize**: Don't write a 1000-line `main.tf`. Use `variables.tf`, `outputs.tf`, and custom modules.
3. **Use Workspaces or Folders for Environments**: Separate Dev, Staging, and Prod state files. Do not mix them.
4. **Pin Provider Versions**: Always specify `version = "~> 5.0"` for providers so an update doesn't break your code.
5. **Format your code**: Run `terraform fmt` before committing. It automatically aligns and cleans up your HCL.
6. **Validate**: Run `terraform validate` to check syntax errors before planning.

---

## 14. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Terraform?** An open-source Infrastructure as Code (IaC) tool by HashiCorp. It allows you to define, provision, and manage cloud infrastructure using declarative configuration files.

**Q2: What is Infrastructure as Code (IaC)?** Managing and provisioning computing infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools (like web consoles).

**Q3: What language does Terraform use?** HCL (HashiCorp Configuration Language). It is JSON-compatible but designed to be human-readable.

**Q4: Name the 4 core Terraform commands.** `init` (initialize), `plan` (preview changes), `apply` (execute changes), `destroy` (tear down).

**Q5: What is the purpose of `terraform init`?** It initializes the working directory by downloading necessary provider plugins (like AWS, GCP) and setting up the backend for state storage.

**Q6: What does `terraform plan` do?** It compares the current state of infrastructure with your configuration code and shows an execution plan of what will be added, modified, or destroyed. It does *not* make changes.

**Q7: What is the `.tfstate` file?** Terraform's memory. It maps the resources defined in your `.tf` files to the real-world resources existing in the cloud.

**Q8: Should you commit `.tfstate` to Git?** NO. It can contain sensitive data (passwords, private keys) in plain text, and multiple developers modifying a local state file simultaneously will cause conflicts and corruption.

---

### Intermediate

**Q9: Terraform vs Ansible?** Terraform is an orchestration/provisioning tool (best for creating VMs, VPCs, Databases). Ansible is a configuration management tool (best for installing software *inside* those VMs). They are often used together.

**Q10: What is a Terraform Provider?** A plugin that interacts with the API of a specific cloud or service (AWS, Azure, GitHub, Kubernetes).

**Q11: What is a Terraform Module?** A container for multiple resources that are used together. It allows you to package and reuse infrastructure code (like a function in programming).

**Q12: How do you handle secrets in Terraform?** 1. Mark variables as `sensitive = true`. 2. Use a secret manager (AWS Secrets Manager, HashiCorp Vault) as a data source. 3. Pass secrets via environment variables (`TF_VAR_db_password=...`), never hardcode them.

**Q13: What is State Locking?** A mechanism to prevent multiple users from running `terraform apply` concurrently on the same state file, which would corrupt it. In AWS, this is typically done using a DynamoDB table.

**Q14: What is a Data Source?** (`data "..." "{}"`) Allows Terraform to fetch data computed elsewhere or created outside of Terraform (e.g., querying AWS for the latest Ubuntu AMI ID, or finding an existing VPC).

**Q15: What is a Resource?** (`resource "..." "{}"`) The most important block in Terraform. It defines an infrastructure object you want to create (e.g., `aws_instance`, `aws_s3_bucket`).

---

### Rapid-Fire (Q16–Q50)

**Q16: How do you destroy infrastructure?** `terraform destroy`.

**Q17: Can you destroy just one specific resource?** Yes, `terraform destroy -target=aws_instance.my_server`.

**Q18: What is `terraform fmt`?** A command that rewrites Terraform configuration files to a canonical format and style (fixes indentation).

**Q19: What is `terraform validate`?** Checks whether a configuration is syntactically valid and internally consistent, without accessing the cloud API.

**Q20: How do you output a value after apply?** Use the `output` block.

**Q21: How do you declare a variable?** Use the `variable "name" {}` block.

**Q22: How do you reference a variable?** `var.name`.

**Q23: How do you pass variables via CLI?** `terraform apply -var="instance_type=t2.micro"`.

**Q24: What file does Terraform automatically read for variables?** `terraform.tfvars`.

**Q25: What is a Remote Backend?** Storing the state file remotely (S3, Terraform Cloud, Azure Blob) rather than locally, enabling team collaboration.

**Q26: What happens if you delete the state file manually?** Terraform forgets about all the resources it created. Running `apply` again will attempt to create *new* resources, potentially causing conflicts (like "bucket name already exists").

**Q27: How can you bring an existing, manually created resource into Terraform?** Use the `terraform import` command.

**Q28: What is `terraform show`?** Reads and outputs the current state or a saved plan file in a human-readable format.

**Q29: What is `terraform refresh`?** Queries the cloud provider to update the local state file with the real-world status of resources. (Note: `terraform plan` does this automatically now).

**Q30: What is a provisioner?** Executes scripts on a local or remote machine as part of resource creation or destruction (e.g., `local-exec`, `remote-exec`).

**Q31: Why are provisioners considered a "last resort"?** They are not declarative, not idempotent, and failures can leave infrastructure in an unknown state. Use user_data (cloud-init) or tools like Ansible/Packer instead.

**Q32: What does `depends_on` do?** Explicitly defines a dependency between resources, forcing Terraform to create one before the other. (Terraform usually infers dependencies automatically).

**Q33: What is the `count` meta-argument?** Creates multiple identical copies of a resource (e.g., `count = 3` creates 3 EC2 instances).

**Q34: What is the `for_each` meta-argument?** Iterates over a map or set of strings to create multiple instances of a resource, allowing each instance to have unique configurations based on the map keys/values.

**Q35: `count` vs `for_each`?** If an item is removed from the middle of a `count` list, Terraform destroys and recreates subsequent resources. `for_each` uses explicit keys, avoiding this destructive behavior. `for_each` is generally preferred.

**Q36: What is a Workspace in Terraform CLI?** A way to maintain multiple distinct state files from the same working directory (e.g., `default`, `dev`, `prod`).

**Q37: Why do some people prefer distinct folders over Workspaces?** Distinct folders allow for different code configurations per environment, whereas Workspaces use the exact same code but different state files, which can be harder to untangle mentally.

**Q38: What does `lifecycle { create_before_destroy = true }` do?** By default, Terraform destroys a resource before creating its replacement. This rule reverses that, ensuring zero downtime (if the resource supports it).

**Q39: What does `lifecycle { prevent_destroy = true }` do?** Prevents Terraform from accidentally destroying a critical resource (like a production database). `apply` will throw an error if a destroy is attempted.

**Q40: How do you upgrade a provider plugin?** `terraform init -upgrade`.

**Q41: What is a `.terraform.lock.hcl` file?** It locks the specific versions of providers used, ensuring every developer on the team uses the exact same provider plugin versions. Should be committed to Git.

**Q42: What is the `local` block?** Assigns a name to an expression or value so it can be reused multiple times within a module (like a local variable). Referenced as `local.name`.

**Q43: What is `terraform taint`?** (Deprecated in favor of `terraform apply -replace=...`). Manually marks a resource as degraded/broken, forcing Terraform to destroy and recreate it on the next apply.

**Q44: What is the Null Resource (`null_resource`)?** A resource that does nothing. Often used with triggers or provisioners to run scripts when a specific value changes.

**Q45: How can you write an if/else statement in Terraform?** Using the ternary operator: `condition ? true_val : false_val`.

**Q46: What is `terraform plan -out=plan.tfplan`?** Saves the execution plan to a file. You can then run `terraform apply plan.tfplan` to guarantee only those exact changes are applied.

**Q47: What is dynamic block?** Constructs repeatable nested configuration blocks within a resource (e.g., multiple `ingress` rules in a security group) dynamically based on a variable list.

**Q48: How do you handle circular dependencies?** You must refactor the code to break the cycle, often by splitting a resource into two (e.g., separating a Security Group from its Security Group Rules).

**Q49: What is Terraform Cloud/Enterprise?** HashiCorp's managed service offering remote state storage, secure variable management, RBAC, and private module registries.

**Q50: Can Terraform manage Kubernetes?** Yes, using the Kubernetes provider, Terraform can create pods, services, and deployments, though many prefer Helm or ArgoCD for K8s-specific workloads.

---

## 📚 References

- [Terraform Official Documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform Registry (Providers & Modules)](https://registry.terraform.io/)
- [HashiCorp Learn](https://developer.hashicorp.com/terraform/tutorials)

---

> **Previous Topic:** [← 30 - Jenkins](../30-jenkins/README.md)  
> **Back to Root:** [Spring Boot Mastery 🚀](../../README.md)
