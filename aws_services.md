# AWS Platform Design Overview

This document lists the AWS services we will use to build a highly available, fault-tolerant platform. It’s written in simple, everyday language so it’s easy to share with customers and architects.

---

## 1. Identity & Access

- **AWS SSO (IAM Identity Center)**  
  Single place for everyone to log in. Works with our company SAML or OIDC.

- **IAM & STS**  
  - Create roles for each job (e.g. “DevOps Engineer,” “Auditor”).  
  - Issue short-lived tokens so no one holds long-term keys.

---

## 2. Compute & Containers

- **Amazon EKS (Kubernetes)**  
  Runs our containers in multiple Availability Zones. If one AZ or node fails, pods restart elsewhere.

- **EC2 Auto Scaling**  
  Adds or removes worker nodes automatically based on load.

- **AWS Fargate**  
  Serverless containers. Good for sudden spikes—no EC2 to manage.

---

## 3. Networking & Traffic

- **Amazon VPC**  
  Isolates resources in public and private subnets.

- **Elastic Load Balancers (ALB/NLB)**  
  Distributes traffic evenly across healthy nodes.

- **Route 53**  
  DNS with health checks. Can fail over to another region if needed.

---

## 4. Data & Storage

- **RDS Multi-AZ**  
  Primary database in one AZ, standby in another. Automatic failover on failure.

- **Aurora Global DB**  
  Keeps a copy in a second region. Fast promotion if primary region goes down.

- **DynamoDB Global Tables**  
  NoSQL data stored in multiple regions at once.

- **S3**  
  Durable object store with cross-region replication for backups.

---

## 5. CI/CD & Automation

- **Jenkins (on EC2 or EKS)**  
  Runs build and deploy pipelines. Agents scale up or down as needed.

- **AWS CodePipeline**  
  An alternative AWS-managed option for build, test, deploy.

- **Systems Manager**  
  Automates routine tasks (patching, run commands).

---

## 6. Monitoring & Alerts

- **CloudWatch**  
  Collects metrics and logs from EC2, EKS, RDS, etc.

- **X-Ray**  
  Traces requests across services to find slow spots.

- **OpenSearch**  
  Central log search and dashboards.

---

## 7. Security & Compliance

- **WAF & Shield**  
  Protect web apps from common attacks and DDoS.

- **Config & Audit Manager**  
  Tracks changes and checks against company rules.

- **Secrets Manager**  
  Stores and automatically rotates passwords and API keys.

- **AWS Backup**  
  Central backup for databases, file systems, and volumes.

---

## 8. Disaster Recovery

- **Multi-Region Setup**  
  Primary in one region, DR in another. Route 53 switches on failure.

- **Immutable Infrastructure**  
  Everything is code (Terraform). We can rebuild in any region quickly.

- **Cross-Region Pipelines**  
  Pipelines mirrored in DR region.

---

## 9. Scalability Patterns

- **Auto Scaling**  
  EC2 and EKS grow or shrink based on CPU, memory, or custom metrics.

- **Serverless Bursting**  
  Lambda or Fargate handle sudden spikes without pre-provisioning.

- **Database Replicas**  
  RDS read replicas and DynamoDB partitions share the load.

---

### Summary

With these services and patterns:

1. **High Availability** – Survives AZ or region failures.  
2. **Fast Recovery** – Automatic failover and rebuilds.  
3. **Elastic Scaling** – Adjusts capacity to the workload.  
4. **Strong Security** – Least-privilege, encrypted data, and audits.  
5. **Easy Management** – Everything defined in code, with repeatable pipelines.
