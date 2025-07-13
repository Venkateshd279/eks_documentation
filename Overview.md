# Deploying Microservice Applications on Amazon EKS via a CI/CD Pipeline

_Venkatesh Dhanapalraj_

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Architecture Diagram](#architecture-diagram)  
3. [Key Components](#key-components)  
4. [Workflow Overview](#workflow-overview)  
5. [Networking & Security](#networking--security)  
6. [Scalability & High Availability](#scalability--high-availability)  
7. [Monitoring & Logging](#monitoring--logging)  
8. [Getting Started](#getting-started)  
9. [Future Enhancements](#future-enhancements)  
10. [License](#license)  

---

## Introduction

This document describes a robust, multi-stage CI/CD pipeline for deploying containerized microservices to Amazon EKS across multiple Availability Zones. The solution leverages Terraform for infra provisioning, GitHub for source control, and Jenkins for build, test, and deploy automation. It’s designed for high availability, security, and observability.

---

## Architecture Diagram

![Deploy Microservice Applications Using CICD Pipeline](https://drive.google.com/uc?export=view&id=11fz_JL3OmFHR-QG-baWkUPTGy2J8vXcN)

> **Figure:** High-level architecture for CI/CD-driven microservice deployment on AWS EKS.

---

## Key Components

| Layer                | Service / Tool           | Purpose                                                                 |
|----------------------|--------------------------|-------------------------------------------------------------------------|
| **Source Control**   | GitHub                   | Hosts application code and Terraform configuration.                     |
| **CI/CD**            | Jenkins                  | Automates build, test, and deploy stages.                               |
| **Infrastructure**   | Terraform                | Provision VPC, subnets, EKS cluster, ALBs, RDS, DynamoDB, IAM, etc.     |
| **Edge & Delivery**  | Route 53, CloudFront     | DNS routing, CDN caching for static assets in S3.                       |
| **Load Balancing**   | Application Load Balancer| Distributes traffic to frontend and backend services.                   |
| **Compute**          | Amazon EKS               | Managed Kubernetes for container orchestration.                         |
| **Data Stores**      | Amazon RDS, DynamoDB     | Relational and NoSQL databases with cross-AZ replication.               |
| **Security**         | Security Groups, Secrets | Network isolation; AWS Secrets Manager for sensitive credentials.       |
| **Monitoring**       | CloudWatch               | Collects metrics, logs, and sets up alarms for proactive alerting.      |

---

## Workflow Overview

1. **Commit & Push**  
   Developers push code and Terraform HCL to GitHub.

2. **Terraform Apply**  
   Jenkins triggers a Terraform job to provision or update AWS resources.

3. **Build & Test**  
   - **Build Stage**: Container images are built and tagged.  
   - **Test Stage**: Unit, integration, and security tests run.  

4. **Deploy to EKS**  
   - **Deploy Stage**: Images are pushed to ECR and Kubernetes manifests (or Helm charts) are applied to the EKS cluster.  
   - **Rolling Update**: Ensures zero-downtime deployments across node groups.

5. **Traffic Routing**  
   Route 53 directs client requests to CloudFront; CloudFront caches static assets from S3. ALB distributes API traffic to EKS pods.

6. **Data Synchronization**  
   RDS instances replicate across AZs; DynamoDB provides a globally available key-value store.

7. **Monitoring & Alerts**  
   CloudWatch collects metrics and logs; alarms notify on threshold breaches.

---

## Networking & Security

- **VPC Configuration**  
  - Public Subnets (`10.0.1.0/24`) host the ALB and NAT gateways.  
  - Private Subnets (`10.0.2.0/24`) host EKS worker nodes and databases.  

- **Secure Access**  
  - Jump server in public subnet for bastion access.  
  - Kubernetes RBAC and IAM roles scoped per service.  
  - Secrets Manager for database credentials and API keys.

---

## Scalability & High Availability

- **Auto Scaling Groups**  
  Worker nodes scale across ≥2 AZs for fault tolerance.

- **Stateless Services**  
  Horizontal Pod Autoscaler (HPA) adapts to traffic patterns.

- **Stateful Data**  
  RDS Multi-AZ and DynamoDB global tables ensure data durability.

---

## Monitoring & Logging

- **Metrics**  
  - EKS cluster health, pod CPU/memory, ALB request count, RDS CPU, DynamoDB throughput  
- **Logs**  
  - Application logs aggregated via Fluentd to CloudWatch Logs.  
- **Alerts**  
  - Proactive notifications for CPU, memory, HTTP 5xx errors, and database failover events.

---

## Getting Started

1. **Clone Repository**  
   ```bash
   git clone https://github.com/your-org/your-repo.git
   cd your-repo
