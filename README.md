# EKS Platform Documentation

Simple guide for building and running applications on Amazon EKS with automated deployments.

## Requirements

### **Functional Requirements (What It Does)**
- **Code Deployment**: Automatically build, test, and deploy applications from GitHub
- **Container Management**: Run applications in Docker containers on Kubernetes
- **User Access Control**: Manage who can access what parts of the platform
- **Infrastructure Provisioning**: Create and manage AWS resources with Terraform
- **Monitoring & Alerting**: Track application health and send notifications
- **Secret Management**: Securely store and distribute passwords and API keys

### **Non-Functional Requirements (How Well It Works)**
- **Availability**: 99.9% uptime with multi-zone deployment
- **Scalability**: Auto-scale from 2 to 50+ pods based on demand
- **Performance**: Deploy new code in under 5 minutes
- **Security**: Multi-factor authentication and automatic vulnerability scanning
- **Reliability**: Zero-downtime deployments with automatic rollback on failure
- **Maintainability**: Infrastructure as Code with version control and documentation

## What This Is

A cloud platform that:
- Automatically builds and deploys your code
- Runs apps in containers on Kubernetes
- Keeps everything secure and monitored
- Scales up and down automatically

## Documentation Files

**📖 Start Here:**
1. **[Overview](./01.Overview.md)** - What the platform does
2. **[AWS Services](./03.aws_services.md)** - What tools we use

**🛠️ For Developers:**
3. **[CI/CD Pipeline](./05.cicd-developer-platform.md)** - How code gets deployed
4. **[Developer Tools](./07.internal-development-platform.md)** - Tools and workflows

**🔧 For Operations:**
5. **[Scaling](./04.scalability.md)** - How the platform grows
6. **[Security](./06.security-access-management.md)** - How we stay safe
7. **[Access Management](./02.access_and_service_management.md)** - Who can do what

## Quick Start

**New to this platform?**
→ Read [Overview](./01.Overview.md) first

**Want to deploy code?**  
→ Check [CI/CD Pipeline](./05.cicd-developer-platform.md)

**Setting up security?**  
→ Go to [Security](./06.security-access-management.md)

## Main Tools We Use

- **Amazon EKS** - Runs containerized apps
- **Jenkins** - Builds and deploys code
- **GitHub** - Stores code
- **Terraform** - Creates infrastructure
- **Datadog** - Monitors everything

## What You Get

✅ **Push code → Auto deploy** - No manual steps  
✅ **Always available** - Multi-zone deployment  
✅ **Secure by default** - Built-in security  
✅ **Scales automatically** - Handles traffic spikes  
✅ **Easy monitoring** - See what's happening  

---



