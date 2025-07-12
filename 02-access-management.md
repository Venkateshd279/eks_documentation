# Access and Service Account Management

## Overview

This document outlines the comprehensive approach to managing user access, roles, and service accounts to ensure secure access to AWS resources while maintaining operational efficiency and compliance.

## Identity and Access Management Strategy

### 1. Identity Provider Integration

#### AWS IAM Identity Center (SSO)
- **Primary Identity Source**: Centralized identity management
- **Multi-Account Access**: Single sign-on across all AWS accounts
- **External IdP Integration**: Connect with corporate Active Directory/LDAP
- **MFA Enforcement**: Mandatory multi-factor authentication

```yaml
# Example SSO Configuration
IdentityCenter:
  InstanceArn: "arn:aws:sso:::instance/ins-xxxxxxxxx"
  ExternalIdP:
    Type: "ActiveDirectory"
    Domain: "corp.company.com"
  MFASettings:
    Enabled: true
    Methods: ["TOTP", "WebAuthn"]
```

#### Federated Access
- **SAML 2.0 Integration**: Enterprise identity providers
- **OIDC Providers**: Modern authentication flows
- **Temporary Credentials**: Short-lived access tokens
- **Cross-Account Roles**: Secure access across AWS accounts

### 2. Account Structure and Organization

#### AWS Organizations Setup
```
Root Organization
├── Security Account (Log Archive, Audit)
├── Shared Services Account (DNS, Monitoring)
├── Production Accounts
│   ├── Production-EKS-US-East
│   ├── Production-EKS-US-West
│   └── Production-EKS-EU-West
├── Non-Production Accounts
│   ├── Development
│   ├── Staging
│   └── Testing
└── Sandbox Accounts
    ├── Developer-Sandbox-1
    └── Developer-Sandbox-N
```

#### Service Control Policies (SCPs)
- **Preventive Controls**: Block unauthorized actions
- **Regional Restrictions**: Limit operations to approved regions
- **Service Restrictions**: Control which AWS services can be used
- **Cost Controls**: Prevent expensive resource creation

### 3. Role-Based Access Control (RBAC)

#### Platform Roles

**Platform Administrator**
- Full access to platform infrastructure
- Ability to modify security policies
- Cross-account access management
- Emergency access capabilities

**Platform Operator**
- Day-to-day platform operations
- Monitoring and troubleshooting
- Non-critical configuration changes
- Incident response access

**Developer**
- Application deployment and management
- Read access to logs and metrics
- Limited infrastructure provisioning
- Namespace-scoped access in Kubernetes

**Security Auditor**
- Read-only access to security logs
- Compliance reporting access
- Security configuration review
- Audit trail access

#### AWS IAM Roles Structure

```json
{
  "PlatformRoles": {
    "PlatformAdmin": {
      "AssumeRolePolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "AWS": "arn:aws:iam::SECURITY-ACCOUNT:root"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
              "Bool": {
                "aws:MultiFactorAuthPresent": "true"
              }
            }
          }
        ]
      },
      "Policies": ["PlatformAdminPolicy"]
    }
  }
}
```

### 4. Service Account Management

#### Kubernetes Service Accounts with IRSA
```yaml
# Service Account with IAM Role for Service Accounts (IRSA)
apiVersion: v1
kind: ServiceAccount
metadata:
  name: application-service-account
  namespace: application-namespace
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/ApplicationRole
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: application
spec:
  template:
    spec:
      serviceAccountName: application-service-account
```

#### Service Account Lifecycle Management
- **Automated Provisioning**: GitOps-based service account creation
- **Rotation Strategy**: Regular credential rotation
- **Least Privilege**: Minimal required permissions
- **Audit Trail**: Complete service account activity logging

### 5. Access Patterns and Authentication Flow

#### Human User Access Flow
1. User authenticates via corporate SSO
2. AWS IAM Identity Center validates credentials
3. MFA challenge if required
4. Temporary credentials issued
5. Role assumption based on group membership
6. Access to AWS resources or Kubernetes clusters

#### Service-to-Service Authentication
1. Pod starts with IRSA-enabled service account
2. AWS SDK automatically assumes IAM role
3. Temporary credentials obtained from IMDS
4. Service accesses AWS resources with assumed role
5. Credentials automatically refreshed

#### CI/CD Pipeline Authentication
```yaml
# GitHub Actions OIDC Configuration
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::ACCOUNT:role/GitHubActionsRole
          aws-region: us-east-1
```

### 6. Security Controls and Monitoring

#### Access Controls
- **JIT Access**: Just-in-time privileged access for sensitive operations
- **Break Glass**: Emergency access procedures with full audit trail
- **Session Recording**: Record privileged access sessions
- **IP Restrictions**: Limit access from approved networks

#### Monitoring and Auditing
- **CloudTrail**: Comprehensive API call logging
- **AWS Config**: Configuration change tracking
- **Access Analyzer**: Identify overly permissive policies
- **GuardDuty**: Anomalous access pattern detection

```yaml
# CloudTrail Configuration
CloudTrail:
  Name: "PlatformAuditTrail"
  S3BucketName: "platform-audit-logs"
  IncludeGlobalServiceEvents: true
  IsMultiRegionTrail: true
  EnableLogFileValidation: true
  EventSelectors:
    - ReadWriteType: "All"
      IncludeManagementEvents: true
      DataResources:
        - Type: "AWS::S3::Object"
          Values: ["arn:aws:s3:::sensitive-bucket/*"]
```

### 7. Secrets Management

#### AWS Secrets Manager Integration
- **Database Credentials**: Automatic rotation for RDS passwords
- **API Keys**: Secure storage and retrieval of third-party API keys
- **Certificates**: SSL/TLS certificate management
- **Application Secrets**: Environment-specific configuration

#### External Secrets Operator
```yaml
# External Secrets Operator Configuration
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        serviceAccount:
          name: external-secrets-sa
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
    - secretKey: username
      remoteRef:
        key: prod/database/credentials
        property: username
```

### 8. Compliance and Governance

#### Policy Management
- **Centralized Policies**: Manage IAM policies through code
- **Policy Validation**: Automated policy analysis and validation
- **Compliance Scanning**: Regular access review and compliance checks
- **Drift Detection**: Identify unauthorized access changes

#### Access Review Process
- **Quarterly Reviews**: Regular access certification process
- **Automated Reports**: Generate access reports for managers
- **Risk Scoring**: Assess risk based on access patterns
- **Remediation**: Automated removal of stale access

#### Implementation Tools
- **Terraform/OpenTofu**: Infrastructure as Code for IAM resources
- **AWS Config Rules**: Automated compliance checking
- **AWS IAM Access Analyzer**: Policy validation and optimization
- **Custom Lambda Functions**: Automated access governance workflows
