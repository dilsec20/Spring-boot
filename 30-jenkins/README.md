# 🏗️ Jenkins — Complete In-Depth Guide

> **"Jenkins is the most widely adopted open-source CI/CD automation server. It automates the process of building, testing, and deploying software, enabling Continuous Integration and Continuous Delivery."**

---

## 📑 Table of Contents

1. [What is CI/CD?](#1-what-is-cicd)
2. [What is Jenkins?](#2-what-is-jenkins)
3. [Jenkins Architecture](#3-jenkins-architecture)
4. [Installation & Setup](#4-installation--setup)
5. [Freestyle vs Pipeline Jobs](#5-freestyle-vs-pipeline-jobs)
6. [Jenkinsfile (Declarative Pipeline)](#6-jenkinsfile-declarative-pipeline)
7. [Scripted vs Declarative Pipelines](#7-scripted-vs-declarative-pipelines)
8. [Pipeline Syntax & Steps](#8-pipeline-syntax--steps)
9. [Webhooks (Automated Triggers)](#9-webhooks-automated-triggers)
10. [Credentials Management](#10-credentials-management)
11. [Jenkins with Docker](#11-jenkins-with-docker)
12. [Deploying Spring Boot via Jenkins](#12-deploying-spring-boot-via-jenkins)
13. [Jenkins Best Practices](#13-jenkins-best-practices)
14. [Interview Questions & Answers (50+)](#14-interview-questions--answers-50)

---

## 1. What is CI/CD?

```
CI (Continuous Integration):
  Developers merge code changes into a central repository frequently (e.g., GitHub).
  Each merge triggers an automated BUILD and TEST sequence.
  Goal: Catch bugs early and ensure the code compiles.

CD (Continuous Delivery):
  Automates the release of the validated code to a repository/artifact manager (e.g., Nexus, Docker Hub).
  Code is ALWAYS in a deployable state.
  Requires manual approval to deploy to Production.

CD (Continuous Deployment):
  Takes Continuous Delivery one step further.
  Every change that passes all tests is AUTOMATICALLY deployed to Production.
  NO manual intervention.
```

---

## 2. What is Jenkins?

```
Jenkins is an automation server written in Java.

Why use Jenkins?
  ✅ Open-source and free.
  ✅ Massive plugin ecosystem (1500+ plugins) integrating with almost every tool (Git, Maven, Docker, AWS).
  ✅ Supports pipelines as code (Jenkinsfile).
  ✅ Distributed architecture (Master-Slave) allows scaling builds across many machines.
```

---

## 3. Jenkins Architecture

```
┌──────────────────────────────────────────────┐
│                Jenkins Master                 │
│                                              │
│ - Manages configuration                      │
│ - Schedules jobs                             │
│ - Distributes jobs to Slaves (Nodes)         │
│ - Hosts the UI                               │
└──────────────────────┬───────────────────────┘
                       │ TCP / SSH
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
┌────────────┐   ┌────────────┐  ┌────────────┐
│ Node/Slave │   │ Node/Slave │  │ Node/Slave │
│ (Ubuntu)   │   │ (Windows)  │  │ (Docker)   │
│            │   │            │  │            │
│ Executes   │   │ Executes   │  │ Executes   │
│ builds     │   │ builds     │  │ builds     │
└────────────┘   └────────────┘  └────────────┘

* Best practice: Do NOT run builds on the Master node. Only use it for scheduling and UI.
```

---

## 4. Installation & Setup

```bash
# Installing Jenkins on Ubuntu (Requires Java)

# 1. Install Java 21
sudo apt update
sudo apt install openjdk-21-jdk -y

# 2. Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# 3. Install Jenkins
sudo apt update
sudo apt install jenkins -y

# 4. Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# 5. Unlock Jenkins (Get Initial Admin Password)
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Access UI at: http://<your-server-ip>:8080
```

---

## 5. Freestyle vs Pipeline Jobs

```
Freestyle Job (Old way):
  ❌ Configured entirely via the UI (clicking buttons, filling forms).
  ❌ Configuration is NOT version-controlled.
  ❌ Hard to replicate or review changes.

Pipeline Job (Modern way):
  ✅ Configured via code (Jenkinsfile).
  ✅ Code lives in your Git repository alongside the app source code.
  ✅ Version-controlled, reviewable via PRs, auditable.
```

---

## 6. Jenkinsfile (Declarative Pipeline)

A `Jenkinsfile` defines your CI/CD process.

```groovy
// Example: Basic Spring Boot Pipeline

pipeline {
    // Run this pipeline on any available node (slave)
    agent any

    // Define tools needed (Must be configured in Jenkins Global Tool Configuration)
    tools {
        maven 'Maven 3.9'
        jdk 'Java 21'
    }

    // Pipeline stages
    stages {
        stage('Checkout') {
            steps {
                // Check out the code from SCM (Source Control Management)
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Compile the code (skip tests for this stage)
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                // Run Unit Tests
                sh 'mvn test'
            }
            post {
                always {
                    // Always publish JUnit test results, even if tests fail
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                // Create the executable JAR (skipping tests as they already ran)
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                // Save the JAR file in Jenkins for later download/deployment
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    // Actions to perform after pipeline finishes
    post {
        success {
            echo "Pipeline succeeded! ✅"
        }
        failure {
            echo "Pipeline failed! ❌"
            // Example: send email notification
            // mail to: 'dev-team@example.com', subject: "Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]", body: "Please check Jenkins."
        }
    }
}
```

---

## 7. Scripted vs Declarative Pipelines

| Feature | Declarative Pipeline (`pipeline { ... }`) | Scripted Pipeline (`node { ... }`) |
|---------|-------------------------------------------|-------------------------------------|
| Style | Modern, Strict structure, easier to read | Older, Free-form Groovy code |
| Learning Curve | Low (Simpler syntax) | High (Requires knowing Groovy) |
| Post Actions | Built-in (`post { success {} }`) | Manual (`try-catch-finally` blocks) |
| Best For | Most standard CI/CD pipelines | Very complex, highly customized pipelines |

*Recommendation: Always use Declarative Pipelines unless you have extremely complex programming requirements that demand raw Groovy.*

---

## 8. Pipeline Syntax & Steps

```groovy
// ═══ Environment Variables ═══
environment {
    // Custom variables
    APP_NAME = "payment-service"
    
    // Credentials (securely retrieved from Jenkins Credentials Manager)
    DB_PASSWORD = credentials('prod-db-password-id')
}

// ═══ Parameters (Manual input when triggering build) ═══
parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build')
    choice(name: 'ENV', choices: ['dev', 'qa', 'prod'], description: 'Target Env')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
}

// ═══ Conditional Execution (when) ═══
stage('Deploy to Prod') {
    when {
        branch 'main'           // Only run if branch is 'main'
        // expression { params.ENV == 'prod' } 
    }
    steps {
        echo "Deploying to production..."
    }
}

// ═══ Parallel Execution ═══
stage('Testing') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn failsafe:integration-test' }
        }
    }
}

// ═══ Manual Approval ═══
stage('Approval') {
    steps {
        // Pipeline pauses here and waits for human click
        input message: 'Approve deployment to Production?'
    }
}
```

---

## 9. Webhooks (Automated Triggers)

```
How to trigger Jenkins automatically when you push code to GitHub:

1. In Jenkins Job config:
   - Check "GitHub hook trigger for GITScm polling"

2. In GitHub Repository:
   - Settings -> Webhooks -> Add webhook
   - Payload URL: http://<jenkins-ip>:8080/github-webhook/
   - Content type: application/json
   - Events: Just the push event (or pull requests)

Now: Git Push → GitHub sends POST request → Jenkins triggers build automatically! 🚀
```

---

## 10. Credentials Management

```
Never hardcode passwords in Jenkinsfiles!

1. Go to Jenkins Dashboard → Manage Jenkins → Credentials.
2. Add Credentials (e.g., Username/Password, Secret Text, SSH Key).
3. Give it an ID (e.g., 'docker-hub-creds').
4. Use it in Jenkinsfile:

environment {
    // Example 1: Secret Text
    API_KEY = credentials('my-api-key-id')
}

steps {
    // Example 2: Username and Password block
    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        sh 'docker login -u $USERNAME -p $PASSWORD'
    }
}
```

---

## 11. Jenkins with Docker

```groovy
// Using Docker as the execution agent!
// You don't need Maven installed on the Jenkins server; Jenkins spins up a Maven container, runs the build, and destroys it.

pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-21'
            // Mount maven cache so subsequent builds are faster
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                // This executes INSIDE the maven container!
                sh 'mvn clean package'
            }
        }
    }
}
```

---

## 12. Deploying Spring Boot via Jenkins

A complete CI/CD pipeline building a JAR, building a Docker image, and deploying it via SSH to a remote server.

```groovy
pipeline {
    agent any
    environment {
        APP_NAME = "springboot-demo"
        IMAGE_NAME = "dilsec20/${APP_NAME}"
        // ID of SSH credentials stored in Jenkins to access the prod server
        PROD_SSH_CREDS = 'prod-server-ssh-key'
        PROD_IP = '192.168.1.100'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build JAR') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${env.BUILD_ID} ."
                sh "docker tag ${IMAGE_NAME}:${env.BUILD_ID} ${IMAGE_NAME}:latest"
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${env.BUILD_ID}"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Deploy to Prod') {
            when { branch 'main' }
            steps {
                // Use sshagent plugin to execute remote commands
                sshagent([PROD_SSH_CREDS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${PROD_IP} '
                            docker pull ${IMAGE_NAME}:latest
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                            docker run -d -p 8080:8080 --name ${APP_NAME} ${IMAGE_NAME}:latest
                        '
                    """
                }
            }
        }
    }
}
```

---

## 13. Jenkins Best Practices

1. **Pipeline as Code** — Always use `Jenkinsfile` checked into source control. Avoid Freestyle jobs.
2. **Master-Slave Architecture** — Never run heavy builds on the Master node. It causes UI slowness and instability. Use agent nodes.
3. **Use Docker Agents** — Define your build environment as a Docker image instead of installing tools manually on agent nodes. This prevents "it worked on agent-1 but failed on agent-2" issues.
4. **Clean Workspace** — Always start pipelines with a clean workspace (or run `mvn clean`) to prevent stale artifacts from passing a build that should fail.
5. **Credentials Manager** — Never hardcode secrets. Use Jenkins Credentials bindings.
6. **Backup Jenkins** — Backup the `JENKINS_HOME` directory regularly (contains configs and job history). Use plugins like ThinBackup.
7. **Role-Based Access Control (RBAC)** — Don't give Admin rights to everyone. Use the Matrix Authorization plugin to restrict access.

---

## 14. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Jenkins?** An open-source automation server used for CI/CD, automating builds, tests, and deployments.

**Q2: What is CI/CD?** Continuous Integration (frequent merges + automated builds/tests) and Continuous Delivery/Deployment (automated releases/deployments).

**Q3: What is a Jenkinsfile?** A text file that contains the definition of a Jenkins Pipeline, stored alongside source code in version control.

**Q4: Freestyle vs Pipeline job?** Freestyle is UI-configured and hard to track. Pipeline is code-based (Jenkinsfile), version-controlled, and highly flexible.

**Q5: What is the default port for Jenkins?** 8080.

**Q6: What language is Jenkins written in?** Java.

**Q7: How do you start/stop Jenkins on Linux?** `sudo systemctl start/stop jenkins`.

**Q8: What is a Jenkins Plugin?** An extension that adds functionality to Jenkins (e.g., Git plugin, Docker plugin, Maven plugin).

---

### Intermediate

**Q9: Declarative vs Scripted Pipeline?** Declarative uses strict syntax (`pipeline { }`) and is easier to read/write. Scripted uses free-form Groovy code (`node { }`) for complex logic.

**Q10: What is the Jenkins Master-Slave architecture?** Master schedules jobs and handles the UI. Slaves (agents) do the actual heavy lifting (building, testing).

**Q11: Why shouldn't you run builds on the Master node?** Builds consume CPU/Memory. Heavy builds on Master can cause the UI to freeze or crash the Jenkins server.

**Q12: How do you trigger a Jenkins build automatically from GitHub?** Use Webhooks. Configure GitHub to send a payload to Jenkins URL (`/github-webhook/`) on push events.

**Q13: How do you handle secrets in Jenkins?** Store them in the Jenkins Credentials Manager. Use `withCredentials` block or `environment { cred = credentials('id') }` in the pipeline.

**Q14: What is the `agent` directive?** Specifies WHERE the pipeline (or a specific stage) will execute (e.g., `agent any`, `agent { label 'linux' }`, `agent { docker 'maven' }`).

**Q15: How do you run tasks in parallel?** Use the `parallel { }` block within a stage.

---

### Rapid-Fire (Q16–Q50)

**Q16: What is a workspace in Jenkins?** The directory on the agent node where Jenkins checks out the source code and runs the build.

**Q17: What does `archiveArtifacts` do?** Saves files (like a .jar or test reports) generated during the build so they can be downloaded from the Jenkins UI later.

**Q18: What is `JENKINS_HOME`?** The directory where Jenkins stores all its configuration, jobs, workspaces, and plugins (usually `/var/lib/jenkins`).

**Q19: How do you backup Jenkins?** Backup the `JENKINS_HOME` directory, or use the ThinBackup plugin.

**Q20: What is Groovy?** The scripting language used to write Jenkins pipelines.

**Q21: How do you skip a stage based on a condition?** Use the `when { }` block (e.g., `when { branch 'main' }`).

**Q22: How do you pause a pipeline for manual approval?** Use the `input` step.

**Q23: What is the `post` block?** Defines actions to run at the end of a pipeline or stage (e.g., `always`, `success`, `failure`).

**Q24: How do you schedule a job to run periodically?** Use the `cron` syntax in the trigger section (e.g., `triggers { cron('H 4 * * *') }`).

**Q25: What is a Multibranch Pipeline?** Automatically creates a Jenkins job for every branch in your Git repository that contains a Jenkinsfile.

**Q26: How do you clean the workspace?** Use the `cleanWs()` step (requires Workspace Cleanup plugin) or `sh 'rm -rf *'`.

**Q27: What is Blue Ocean?** A modern, user-friendly UI plugin for Jenkins pipelines (visualizes stages clearly).

**Q28: How do you use a specific Maven version in pipeline?** Define it in `Global Tool Configuration` and reference it in the `tools { maven 'mvn3.9' }` block.

**Q29: How do you send an email on build failure?** Use the `mail` step or `emailext` plugin inside the `post { failure { } }` block.

**Q30: What is a downstream job?** A job triggered automatically by the successful completion of an upstream job.

**Q31: How do you trigger another job from a pipeline?** Use the `build job: 'other-job-name'` step.

**Q32: Can a pipeline check out multiple git repositories?** Yes, by using multiple `checkout scm` or `git url: ...` steps in different directories.

**Q33: What is the `stash` and `unstash` step?** `stash` saves files temporarily to pass them between different agents/nodes in the same pipeline. `unstash` retrieves them.

**Q34: How do you pass parameters to a Jenkins job?** Use the `parameters { }` block in Declarative, or configure "This project is parameterized" in UI.

**Q35: How do you access a parameter in a shell script step?** Use `${params.PARAM_NAME}` or just `$PARAM_NAME`.

**Q36: What is RBAC in Jenkins?** Role-Based Access Control. Using Role Strategy Plugin to restrict who can view, run, or configure specific jobs.

**Q37: How do you restart Jenkins from URL?** Navigate to `http://<jenkins-url>/restart` or `/safeRestart`.

**Q38: What does `/safeRestart` do?** Waits for all running jobs to finish before restarting Jenkins. Prevents aborted builds.

**Q39: What is the `sh` step?** Executes a shell command on Linux/Mac agents.

**Q40: What is the `bat` step?** Executes a batch command on Windows agents.

**Q41: How do you handle timeouts in pipelines?** Use the `timeout(time: 1, unit: 'HOURS') { ... }` block.

**Q42: How do you handle retries in pipelines?** Use the `retry(3) { ... }` block to retry a failing step.

**Q43: What is "Pipeline script from SCM"?** A configuration where Jenkins fetches the Jenkinsfile from a Git repository rather than pasting the script into the UI.

**Q44: Why use `fingerprint: true` in artifacts?** Tracks which build produced an artifact and where it is being used in other downstream jobs.

**Q45: How to run Jenkins in Docker?** `docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts`.

**Q46: What is port 50000 used for in Jenkins?** JNLP communication port for Jenkins Master to communicate with slave agents.

**Q47: Can Jenkins deploy to Kubernetes?** Yes, using plugins or executing `kubectl` commands from the pipeline.

**Q48: What is a shared library in Jenkins?** A separate Git repo containing reusable Groovy functions (pipeline logic) that multiple different Jenkinsfiles can import and use to reduce code duplication.

**Q49: How do you troubleshoot a failed pipeline?** Click the failed build, view the "Console Output", look for the error stack trace or exit code.

**Q50: Jenkins vs GitLab CI / GitHub Actions?** Jenkins: Older, highly customizable, requires hosting your own server. GitLab/GitHub CI: Modern, YAML-based, deeply integrated into the Git platform, often cloud-hosted (SaaS).

---

## 📚 References

- [Jenkins Official Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Jenkins Tutorial for Beginners](https://www.jenkins.io/doc/tutorials/)

---

> **Previous Topic:** [← 29 - Ansible](../29-ansible/README.md)  
> **Next Topic:** [31 - Terraform →](../31-terraform/README.md)
