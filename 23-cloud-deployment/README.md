# ☁️ Cloud Deployment — Complete In-Depth Guide

> **"Deploying to the cloud means your app is accessible worldwide, auto-scales, and has high availability. This guide covers AWS, GCP, Azure, and Railway/Render."**

---

## 📑 Table of Contents

1. [Cloud Deployment Overview](#1-cloud-deployment-overview)
2. [Cloud Service Models (IaaS, PaaS, SaaS)](#2-cloud-service-models)
3. [AWS Deployment](#3-aws-deployment)
4. [Google Cloud Platform (GCP)](#4-google-cloud-platform-gcp)
5. [Azure Deployment](#5-azure-deployment)
6. [Simple PaaS (Railway, Render, Fly.io)](#6-simple-paas-railway-render-flyio)
7. [CI/CD Pipeline](#7-cicd-pipeline)
8. [Environment Variables & Secrets](#8-environment-variables--secrets)
9. [Monitoring & Logging in Cloud](#9-monitoring--logging-in-cloud)
10. [Scaling Strategies](#10-scaling-strategies)
11. [Cost Optimization](#11-cost-optimization)
12. [Interview Questions & Answers (50+)](#12-interview-questions--answers-50)

---

## 1. Cloud Deployment Overview

```
LOCAL → CLOUD deployment path:

1. Develop locally (localhost:8080)
2. Build JAR/Docker image
3. Push to registry (Docker Hub / ECR / GCR)
4. Deploy to cloud (AWS ECS, GCP Cloud Run, Azure Container Apps)
5. Configure domain, SSL, monitoring
6. Auto-scale based on traffic

Deployment Options:
┌────────────────────────────────────────────────────┐
│ Simple (PaaS)          │ Advanced (IaaS/Container) │
│ Railway, Render, Heroku│ AWS ECS, GCP GKE, Azure   │
│ Git push → deployed!   │ Full control, complex      │
│ Good for: learning,    │ Good for: production,      │
│ small projects         │ enterprise, microservices   │
└────────────────────────────────────────────────────┘
```

---

## 2. Cloud Service Models

```
┌──────────────────────────────────────────────────────────┐
│                        YOU MANAGE                         │
│                                                           │
│  On-Premise    IaaS          PaaS         SaaS           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ App     │  │ App     │  │ App     │  │         │   │
│  │ Runtime │  │ Runtime │  │         │  │         │   │
│  │ OS      │  │ OS      │  │         │  │         │   │
│  │ Network │  │         │  │         │  │         │   │
│  │ Storage │  │         │  │         │  │         │   │
│  │ Server  │  │         │  │         │  │         │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│  Everything    VM + OS      Just code    Nothing        │
│                                                           │
│  Example:      AWS EC2      Railway      Gmail           │
│  Your server   GCP Compute  Cloud Run    Google Docs     │
└──────────────────────────────────────────────────────────┘
```

---

## 3. AWS Deployment

### Option 1: EC2 (Virtual Machine)

```bash
# 1. Launch EC2 instance (Ubuntu)
# 2. SSH into instance
ssh -i mykey.pem ec2-user@your-ec2-ip

# 3. Install Java
sudo apt update && sudo apt install -y openjdk-21-jre

# 4. Copy JAR to EC2
scp -i mykey.pem target/myapp.jar ec2-user@your-ec2-ip:/home/ec2-user/

# 5. Run application
nohup java -jar myapp.jar --spring.profiles.active=prod &

# 6. Configure security group: allow port 8080
```

### Option 2: ECS (Elastic Container Service)

```bash
# 1. Push Docker image to ECR (Elastic Container Registry)
aws ecr get-login-password | docker login --username AWS --password-stdin 123456.dkr.ecr.region.amazonaws.com
docker tag myapp:latest 123456.dkr.ecr.region.amazonaws.com/myapp:latest
docker push 123456.dkr.ecr.region.amazonaws.com/myapp:latest

# 2. Create ECS task definition (JSON)
# 3. Create ECS service
# 4. Configure ALB (Application Load Balancer)
# 5. Auto-scaling based on CPU/memory
```

### Option 3: Elastic Beanstalk (PaaS)

```bash
# Simplest AWS option — upload JAR, AWS handles everything
eb init myapp --platform java-21 --region us-east-1
eb create production
eb deploy
# AWS provisions: EC2, load balancer, auto-scaling, monitoring
```

### AWS RDS (Managed Database)

```properties
# Connect Spring Boot to AWS RDS
spring.datasource.url=jdbc:mysql://mydb.abc123.us-east-1.rds.amazonaws.com:3306/mydb
spring.datasource.username=${RDS_USERNAME}
spring.datasource.password=${RDS_PASSWORD}
```

---

## 4. Google Cloud Platform (GCP)

### Cloud Run (Recommended for Containers)

```bash
# 1. Build and push to GCR
gcloud builds submit --tag gcr.io/my-project/myapp

# 2. Deploy to Cloud Run
gcloud run deploy myapp \
  --image gcr.io/my-project/myapp \
  --platform managed \
  --region us-central1 \
  --port 8080 \
  --memory 512Mi \
  --set-env-vars SPRING_PROFILES_ACTIVE=prod \
  --allow-unauthenticated

# Cloud Run: serverless containers. Pay only when handling requests!
```

### App Engine

```yaml
# app.yaml
runtime: java21
instance_class: F2
env_variables:
  SPRING_PROFILES_ACTIVE: prod
```

```bash
gcloud app deploy
```

---

## 5. Azure Deployment

### Azure Container Apps

```bash
az containerapp create \
  --name myapp \
  --resource-group mygroup \
  --image myregistry.azurecr.io/myapp:latest \
  --target-port 8080 \
  --ingress external \
  --min-replicas 1 \
  --max-replicas 10
```

### Azure App Service

```bash
az webapp create --name myapp --resource-group mygroup --plan myplan
az webapp config set --name myapp --resource-group mygroup --java-version 21
az webapp deploy --name myapp --src-path target/myapp.jar
```

---

## 6. Simple PaaS (Railway, Render, Fly.io)

### Railway (Easiest)

```bash
# 1. Push code to GitHub
# 2. Go to railway.app → New Project → Deploy from GitHub
# 3. Railway auto-detects Spring Boot, builds, and deploys
# 4. Add MySQL/PostgreSQL/Redis with one click
# 5. Environment variables in Railway dashboard

# Railway Dockerfile deployment:
railway up
```

### Render

```yaml
# render.yaml
services:
  - type: web
    name: myapp
    env: docker
    dockerfilePath: ./Dockerfile
    envVars:
      - key: SPRING_PROFILES_ACTIVE
        value: prod
      - key: DATABASE_URL
        fromDatabase:
          name: mydb
          property: connectionString

databases:
  - name: mydb
    plan: free
```

---

## 7. CI/CD Pipeline

```yaml
# GitHub Actions: Build → Test → Docker → Deploy
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      
      - name: Build & Test
        run: mvn clean verify
      
      - name: Build Docker Image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to Registry
        run: |
          docker tag myapp:${{ github.sha }} ${{ secrets.REGISTRY }}/myapp:latest
          docker push ${{ secrets.REGISTRY }}/myapp:latest
      
      - name: Deploy to Cloud Run
        uses: google-github-actions/deploy-cloudrun@v1
        with:
          service: myapp
          image: ${{ secrets.REGISTRY }}/myapp:latest
```

---

## 8. Environment Variables & Secrets

```bash
# NEVER hardcode secrets in code!

# Spring Boot reads environment variables automatically:
# SPRING_DATASOURCE_URL → spring.datasource.url
# DB_PASSWORD → ${DB_PASSWORD} in properties

# AWS: Systems Manager Parameter Store or Secrets Manager
# GCP: Secret Manager
# Azure: Key Vault
# Railway/Render: Dashboard environment variables
```

---

## 9. Monitoring & Logging in Cloud

```
Spring Boot Actuator + Cloud Monitoring:

AWS:    CloudWatch (metrics + logs)
GCP:    Cloud Monitoring + Cloud Logging
Azure:  Application Insights

Application-level:
  Spring Boot Actuator → /actuator/health, /actuator/metrics
  Prometheus + Grafana → Custom dashboards
  ELK Stack → Centralized logging
```

---

## 10. Scaling Strategies

```
VERTICAL SCALING (Scale Up):
  Bigger machine: 2 CPU → 8 CPU, 4GB → 32GB RAM
  Simple but limited.

HORIZONTAL SCALING (Scale Out):
  More instances: 1 server → 10 servers
  Need: Load balancer, stateless app, shared database
  
Auto-scaling rules:
  IF CPU > 80% for 5 minutes → Add 2 instances
  IF CPU < 30% for 10 minutes → Remove 1 instance
  Min instances: 2 (availability)
  Max instances: 20 (cost control)
```

---

## 11. Cost Optimization

```
1. Right-size instances — Don't over-provision
2. Use spot/preemptible instances — 60-90% cheaper
3. Auto-scale to zero — Serverless (Cloud Run, Lambda)
4. Reserved instances — 30-70% off for committed usage
5. Use free tiers — AWS/GCP/Azure all have free tiers
6. Monitor costs — Set billing alerts
```

---

## 12. Interview Questions & Answers (50+)

### Beginner

**Q1: What is cloud computing?** On-demand computing resources (servers, storage, databases) over the internet. Pay-as-you-go.

**Q2: IaaS vs PaaS vs SaaS?** IaaS: virtual machines (EC2). PaaS: deploy code, platform managed (Elastic Beanstalk). SaaS: use software (Gmail).

**Q3: What is AWS EC2?** Elastic Compute Cloud — virtual servers in AWS. You choose OS, CPU, memory.

**Q4: What is a container registry?** Storage for Docker images. Docker Hub, AWS ECR, GCR, Azure ACR.

**Q5: What is CI/CD?** Continuous Integration (auto-build/test) + Continuous Deployment (auto-deploy). Automated pipeline.

**Q6: What is auto-scaling?** Automatically add/remove instances based on load. More traffic → more servers.

**Q7: What is a load balancer?** Distributes traffic across multiple server instances. Round robin, least connections.

**Q8: What is serverless?** Run code without managing servers. Pay per execution. AWS Lambda, Cloud Run.

---

### Intermediate

**Q9: How to deploy Spring Boot to AWS?** EC2 (JAR), ECS (Docker), Elastic Beanstalk (PaaS), or Lambda (serverless).

**Q10: What is AWS RDS?** Managed relational database. Handles backups, patching, replication. MySQL, PostgreSQL, etc.

**Q11: What is GCP Cloud Run?** Serverless containers. Deploy Docker image, auto-scales to zero. Pay per request.

**Q12: How to manage secrets in cloud?** AWS Secrets Manager, GCP Secret Manager, Azure Key Vault. Never in code.

**Q13: What is a CDN?** Content Delivery Network. Caches static content at edge locations worldwide. CloudFront, Cloud CDN.

**Q14: What is blue-green deployment?** Run two identical environments. Switch traffic from blue (old) to green (new). Instant rollback.

**Q15: What is canary deployment?** Route small % of traffic to new version. If healthy, gradually increase.

---

### Rapid-Fire (Q16–Q50)

**Q16: AWS S3?** Simple Storage Service — object storage for files.

**Q17: AWS Lambda?** Serverless functions. Run code without servers.

**Q18: AWS ECS vs EKS?** ECS: AWS-native container orchestration. EKS: Managed Kubernetes.

**Q19: What is Kubernetes?** Container orchestration platform. Auto-scaling, self-healing, service discovery.

**Q20: What is a VPC?** Virtual Private Cloud — isolated network in the cloud.

**Q21: What is an availability zone?** Physically separate data center in a region.

**Q22: What is a region?** Geographic area with multiple availability zones.

**Q23: What is CloudFormation?** AWS Infrastructure as Code. Define resources in YAML/JSON.

**Q24: What is Terraform?** Cloud-agnostic Infrastructure as Code tool.

**Q25: What is SSL/TLS certificate?** Encryption for HTTPS. AWS Certificate Manager provides free certs.

**Q26: What is Route 53?** AWS DNS service.

**Q27: What is Cloud SQL (GCP)?** Managed relational database on Google Cloud.

**Q28: What is IAM?** Identity and Access Management — controls who can do what in cloud.

**Q29: What is SQS?** AWS Simple Queue Service — managed message queue.

**Q30: What is SNS?** AWS Simple Notification Service — pub/sub messaging.

**Q31: Horizontal vs vertical scaling?** Horizontal: more machines. Vertical: bigger machine.

**Q32: What is 12-factor app?** Methodology for building cloud-native apps. Stateless, config in env, disposable.

**Q33: What is health check?** Endpoint (e.g., /actuator/health) that load balancer checks to verify app is healthy.

**Q34: What is sticky session?** Load balancer routes same user to same server. Avoid if possible (stateless better).

**Q35: What is a reverse proxy?** Server that sits in front of app servers. Nginx, HAProxy. Handles SSL, caching, routing.

**Q36: What is Docker Swarm vs Kubernetes?** Swarm: simpler, built into Docker. K8s: more powerful, industry standard.

**Q37: Cloud Run vs App Engine?** Cloud Run: containers, any language. App Engine: more managed, specific runtimes.

**Q38: What is observability?** Monitoring + logging + tracing. Understanding system behavior.

**Q39: What is distributed tracing?** Tracking requests across microservices. Zipkin, Jaeger, AWS X-Ray.

**Q40: What is feature flag?** Toggle features on/off without deployment. LaunchDarkly, Unleash.

**Q41: What is immutable infrastructure?** Never modify servers. Replace with new ones. Deploy new image, destroy old.

**Q42: What is service mesh?** Infrastructure layer for service-to-service communication. Istio, Linkerd.

**Q43: What is GitOps?** Git as single source of truth for infrastructure and deployments.

**Q44: What is Infrastructure as Code?** Define infrastructure in code files. Terraform, CloudFormation, Pulumi.

**Q45: Free tier services?** AWS: EC2 t2.micro, RDS, S3. GCP: Cloud Run, Firestore. Azure: App Service.

**Q46: What is multi-region deployment?** Deploy to multiple geographic regions for low latency and disaster recovery.

**Q47: What is disaster recovery?** Plan for recovering from failures. RTO (Recovery Time Objective), RPO (Recovery Point Objective).

**Q48: What is chaos engineering?** Deliberately introduce failures to test resilience. Netflix Chaos Monkey.

**Q49: What is a WAF?** Web Application Firewall. Protects against SQL injection, XSS, DDoS.

**Q50: What is cost allocation tags?** Labels on cloud resources to track spending by project/team.

---

## 📚 References

- [AWS Documentation](https://docs.aws.amazon.com/)
- [GCP Documentation](https://cloud.google.com/docs)
- [Azure Documentation](https://learn.microsoft.com/en-us/azure/)
- [12-Factor App](https://12factor.net/)
- [Spring Boot on AWS](https://spring.io/guides/gs/spring-boot-docker)

---

> **Previous Topic:** [← 22 - Docker](../22-docker/README.md)  
> **Next Topic:** [24 - Spring AI →](../24-spring-ai/README.md)
