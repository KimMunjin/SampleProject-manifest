# SampleProject Kubernetes Manifests

This repository contains Kubernetes manifests for the SampleProject application.

## Repository Structure
```
.
├── dev/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── prod/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── README.md
```

## Environment Differences

### Development (dev/)
- 1 replica
- Resources:
    - Requests: CPU 200m, Memory 256Mi
    - Limits: CPU 500m, Memory 512Mi
- Ingress: dev.${INGRESS_HOST}

### Production (prod/)
- 2 replicas
- Resources:
    - Requests: CPU 500m, Memory 512Mi
    - Limits: CPU 1000m, Memory 1Gi
- Ingress: ${INGRESS_HOST}

## CI/CD Setup

### Prerequisites
1. AWS Setup
    - ECR Repository
    - IAM OIDC Provider
    - IAM Role with ECR policies

2. GitHub Repository Secrets
    - `AWS_ACCOUNT_ID`: AWS Account ID
    - `AWS_REGION`: AWS Region (e.g., ap-northeast-2)
    - `ECR_REPOSITORY`: ECR Repository name
    - `GIT_TOKEN`: GitHub Personal Access Token
    - `INGRESS_HOST`: Base domain for ingress

### Workflow
1. Code push triggers GitHub Actions
    - `main` branch → prod environment
    - `develop` branch → dev environment

2. Actions performed:
    - Build Java application
    - Create Docker image
    - Push to ECR
    - Update environment-specific manifests

## Manual Deployment

For manual deployment (if needed):
```bash
# Development
kubectl apply -f dev/

# Production
kubectl apply -f prod/
```

## Required AWS CLI Commands

```bash
# Create ECR Repository
aws ecr create-repository \
    --repository-name sample-project \
    --region ap-northeast-2

# Create OIDC Provider for GitHub Actions
aws iam create-open-id-connect-provider \
    --url https://token.actions.githubusercontent.com \
    --client-id-list sts.amazonaws.com \
    # Additional configuration required

# Create and attach IAM policies (minimal permissions)
# See policy details in the AWS setup documentation
```

## Maintenance

### Adding New Resources
1. Create the resource YAML in both dev/ and prod/ directories
2. Update GitHub Actions workflow if needed
3. Commit and push changes

### Updating Existing Resources
1. Modify the appropriate YAML files
2. The changes will be applied automatically by ArgoCD when using GitOps

## Notes
- All sensitive information is handled via GitHub Secrets
- Manifest files use placeholders that are replaced during deployment
- Resources are configured differently for dev and prod environments