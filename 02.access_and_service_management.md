# Access & Service Account Management

A simple guide to managing who and what can access our AWS resources.

---

## Architecture Diagram

![Access & Service Account Management](https://drive.google.com/uc?export=view&id=1a7ql-QIMduz9p6uxDm83hrXoVAn3tIYI)

---

## 1. Identity & Authentication

- **Single Sign-On**  
  Use AWS SSO (or our corporate SAML/OIDC) so everyone logs in the same way.  
- **Multi-Factor Authentication (MFA)**  
  Require MFA for all user sign-ins.  
- **Short-Lived Credentials**  
  Give temporary AWS tokens via STS instead of long-lived access keys.

---

## 2. User Roles & Permissions

- **Role-Based Access**  
  Create IAM roles for each job function (e.g. “DevOps-Engineer”, “Security-Auditor”).  
- **Least Privilege**  
  Grant only the exact permissions each role needs.  
- **Separation of Duties**  
  Keep automation tools (Jenkins, Terraform) in their own roles—separate from human admins.  
- **Just-In-Time Elevation**  
  Let users request extra permissions when needed; they expire automatically.

---

## 3. Service Accounts

- **EKS IRSA**  
  Map Kubernetes ServiceAccounts to specific IAM roles.  
- **Dedicated Automation Roles**  
  Give Jenkins, Terraform, and other tools their own IAM roles.  
- **Secrets Storage**  
  Keep passwords, API keys, and certificates in Vault or AWS Secrets Manager.

---

## 4. Auditing & Reviews

- **CloudTrail Everywhere**  
  Turn on CloudTrail in all accounts; send logs to a central, write-only S3 bucket.  
- **Access Analyzer**  
  Regularly scan IAM policies to catch overly broad permissions.  
- **Periodic Reviews**  
  Every quarter, clean up unused users and roles.  
- **Alerts**  
  Notify on unusual activity (e.g. many AssumeRole calls or failed logins).

---

## 5. Onboarding & Self-Service

- **Infrastructure as Code**  
  Define all IAM roles and policies in Terraform, versioned in GitHub.  
- **Self-Service Portal**  
  Provide a simple UI or scripts to request access or rotate keys.  
- **Automated Offboarding**  
  Tie user removal to HR or GitHub team changes so access is revoked quickly.

---

**Why this matters**  
- Keeps our AWS accounts secure and easy to audit  
- Ensures people and services have only the permissions they actually need  
- Makes it simple for developers to get the access they need—and for security to stay in control  
