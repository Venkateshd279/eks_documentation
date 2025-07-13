# Deploy Microservices with CI/CD and Multi-Region DR on Amazon EKS

This document details the AWS services we use to build a highly available, fault-tolerant microservices platform with CI/CD and multi-region disaster recovery.

---

## 1. CI/CD Pipeline

- **GitHub**  
  Stores source code and Terraform infrastructure configurations.

- **Jenkins**  
  Orchestrates build → test → deploy workflows. Runs on EKS for scalability.

- **Terraform**  
  Provisions VPC, EKS clusters, databases, and all infrastructure in both regions.m Design Overview

This document lists the AWS services we will use to build a highly available, fault-tolerant platform. 
---

## 1. Identity & Access

- **AWS SSO (IAM Identity Center)**  
  Single place for everyone to log in. Works with our company  OIDC (OpenID Connect).

- **IAM & STS**  
  - Create roles for each job (e.g. “DevOps Engineer,” “Auditor”).  
  - Issue short-lived tokens so no one holds long-term keys.

---

## 2. Kubernetes & Compute

- **Amazon EKS**  
  Managed Kubernetes spanning multiple Availability Zones per region.

- **Auto Scaling Groups**  
  Automatically maintains sufficient worker nodes based on demand.

- **AWS Fargate**  
  Serverless containers for handling traffic bursts without pre-provisioning.

---

## 3. Networking & Traffic

- **VPC**  
  Network isolation with public and private subnets across multiple AZs.

- **ALB & NLB**  
  Application and Network Load Balancers for layer-7 and layer-4 traffic distribution.

- **Route 53**  
  DNS with health-check based failover between regions.

- **CloudFront & S3**  
  Global content delivery network for static assets and caching.

---

## 4. Data & Storage

- **RDS Multi-AZ**  
  Primary database with standby replica in separate AZ within each region.

- **Aurora Global DB**  
  Cross-region database replication with fast promotion capabilities.

- **DynamoDB Global Tables**  
  Multi-region NoSQL database with automatic cross-region replication.

- **S3**  
  Durable object storage with cross-region replication for backups and static content.

---

## 5. Monitoring & Alerts

- **Datadog**  
  Centralized application logs and metrics monitoring across both regions.

- **CloudWatch**  
  AWS infrastructure metrics, logs, and native service monitoring.

- **EventBridge → SNS**  
  Routes alarms and events to email, SMS, and other notification channels.

---

## 6. Security & Access

- **MFA & STS**  
  Multi-factor authentication and short-lived token-based access.

- **IAM Access Analyzer**  
  Automatically detects overly broad permissions and access patterns.

- **WAF & Shield**  
  Web Application Firewall and DDoS protection for applications and APIs.

---

## 7. Disaster Recovery

- **Active/Passive Regions**  
  Primary region serves traffic while secondary region stands by for failover.

- **Route 53 Failover**  
  Automatic DNS switch to secondary region on primary failure detection.

- **Immutable Infrastructure**  
  Terraform code can rebuild entire infrastructure in any region quickly.

---

### Summary

With these services and patterns:

1. **High Availability** – Survives AZ or region failures.  
2. **Fast Recovery** – Automatic failover and rebuilds.  
3. **Elastic Scaling** – Adjusts capacity to the workload.  
4. **Strong Security** – Multi-factor auth, encryption, and DDoS protection.  
5. **Easy Management** – Everything defined in code, with repeatable pipelines.
