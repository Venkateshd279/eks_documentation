# Continuous Integration & Continuous Delivery with Jenkins

_Venkatesh Dhanapalraj_

---

## Introduction

This guide shows our simple CI/CD pipeline. We use GitHub for source control, Jenkins for automation, and Amazon ECR + EKS for container deployment. The goal is fast, reliable updates with minimal manual steps.

---

## Architecture Diagram

![CI/CD Flow](https://drive.google.com/uc?export=view&id=1GJZSyboV7EyaL4qZlWPuaDRb1uipPwec)

---

## Pipeline Steps

1. **Code & Push**  
   - Developers write code locally and push to GitHub.  
   - A webhook notifies Jenkins of the new commit.

2. **Checkout & Scan**  
   - Jenkins pulls the code.  
   - SonarQube scans for bugs and style issues.  
     - **Fail** → Pipeline stops and team fixes code.  
     - **Pass** → Continue.

3. **Build & Artifact**  
   - Jenkins builds the app (compiles or packages).  
   - The build artifact is stored and tagged (e.g., `v1.2.3`).

4. **Docker Image**  
   - Jenkins runs the Dockerfile to create a container image.  
   - The image is tagged and pushed to Amazon ECR.

5. **Update Config**  
   - Jenkins updates deployment files (e.g., sets the new image tag).

6. **Kubernetes Check**  
   - Jenkins verifies access to the EKS cluster.

7. **Deploy & Rollout**  
   - Jenkins applies Kubernetes manifests or Helm charts.  
   - Kubernetes performs a rolling update for zero downtime.

8. **Verify Deployment**  
   - Jenkins runs quick tests (smoke or health checks).  
   - **Success** → Pipeline ends.  
   - **Failure** → Alerts sent and rollback can occur.

---

## Benefits

- **Quick Feedback**: Scans catch issues early.  
- **Faster Releases**: Automated steps cut manual effort.  
- **High Uptime**: Rolling updates keep the service live.  
- **Clear Tracking**: Every build is versioned from code to deployment.

---

## Getting Started

1. **Install Jenkins Plugins**  
   - Git, Docker, SonarQube, Kubernetes CLI.

2. **Add Credentials**  
   - Store GitHub, ECR, and EKS access in Jenkins Credentials.

3. **Configure Webhook**  
   - Point your GitHub repo to the Jenkins job URL.

4. **Run a Test**  
   - Push a change and watch the pipeline run through each step.

