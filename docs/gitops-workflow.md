# GitOps Workflow for LINQ Platform

This document describes the GitOps workflow used for deploying and managing the LINQ platform across multiple environments.

## Overview

The LINQ platform uses ArgoCD to implement GitOps principles, where all deployments are driven by Git commits to this repository. This ensures:

- **Declarative Configuration**: All infrastructure and application state is declared in Git
- **Version Control**: Complete audit trail of all changes
- **Automated Sync**: Environments automatically sync to desired state
- **Rollback Capability**: Easy rollback to previous known-good states

## Repository Structure

```
├── helm-charts/           # Helm charts for each service
│   ├── linq-workcell-manager/
│   ├── linq-orchestrator/
│   └── ...
├── environments/          # Environment-specific configurations  
│   ├── dev-env-1/
│   │   ├── values.yaml    # Service versions and config
│   │   └── status.yaml    # Current environment status
│   ├── staging-env-1/
│   └── production-env-1/
├── argocd-apps/          # ArgoCD Application manifests
│   ├── dev-env-1-app.yaml
│   ├── staging-env-1-app.yaml
│   └── production-env-1-app.yaml
└── docs/                 # Documentation
```

## Deployment Workflow

### 1. Development Deployment

**Trigger**: Push to main branch with changes to `environments/dev-env-1/`

```bash
# Update service version
vim environments/dev-env-1/values.yaml

# Commit changes
git add environments/dev-env-1/values.yaml
git commit -m "Update dev environment to linq-orchestrator v3.2.2"
git push origin main
```

**ArgoCD Behavior**:
- Detects change within 3 minutes (default polling interval)
- Automatically syncs to dev-cluster
- Updates running pods with new image versions
- Reports sync status and health

### 2. Staging Deployment

**Trigger**: Manual update after development validation

```bash
# Update staging environment
vim environments/staging-env-1/values.yaml

# Update versions from dev to staging versions
# Commit and push
git add environments/staging-env-1/values.yaml
git commit -m "Promote tested versions to staging"
git push origin main
```

**ArgoCD Behavior**:
- Automatically syncs to staging-cluster
- Runs health checks on all services
- Validates service mesh connectivity
- Reports deployment status

### 3. Production Deployment

**Trigger**: Manual approval process

```bash
# Update production environment (requires review)
vim environments/production-env-1/values.yaml

# Create pull request for production changes
git checkout -b production-update-v1.2.1
git add environments/production-env-1/values.yaml
git commit -m "Production update: LINQ Platform v1.2.1"
git push origin production-update-v1.2.1

# Create PR and get approval
gh pr create --title "Production Update v1.2.1" --body "..."
```

**ArgoCD Behavior**:
- **Manual sync required** - production apps do not auto-sync
- Admin must review changes in ArgoCD UI
- Manual trigger of sync operation
- Enhanced monitoring during deployment
- Automatic rollback on health check failures

## Sync Policies

### Development Environment
```yaml
syncPolicy:
  automated:
    prune: true      # Remove resources not in Git
    selfHeal: true   # Fix drift automatically
  syncOptions:
    - CreateNamespace=true
```

### Staging Environment  
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
```

### Production Environment
```yaml
syncPolicy:
  automated:
    prune: false     # Do not auto-remove resources
    selfHeal: false  # Do not auto-fix drift
  syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
```

## Health Checks

ArgoCD monitors application health using:

1. **Kubernetes Resource Status**: Pod, Service, Deployment status
2. **Custom Health Checks**: Application-specific health endpoints
3. **Sync Status**: Git commit alignment with cluster state

### Service Health Endpoints
- **Path**: `/health`
- **Port**: Application port (8080)
- **Timeout**: 30 seconds
- **Retries**: 3 attempts

## Rollback Procedures

### Automatic Rollback
- Production apps automatically rollback on failed health checks
- Rollback to previous successful deployment
- Alerts sent to operations team

### Manual Rollback
```bash
# View deployment history
argocd app history production-env-1-linq-platform

# Rollback to specific revision
argocd app rollback production-env-1-linq-platform --revision 123

# Or rollback via Git
git revert <commit-hash>
git push origin main
```

## Monitoring and Alerting

### ArgoCD UI
- Real-time sync status for all applications
- Visual representation of resource health
- Deployment history and diff view
- Log aggregation for troubleshooting

### Prometheus Metrics
- Application sync duration
- Sync failure rate
- Resource drift detection
- Health check success rate

### Alerting Rules
- Sync failures > 3 consecutive attempts
- Health degradation in production
- Configuration drift detection
- Manual intervention required

## Security and Compliance

### RBAC (Role-Based Access Control)
- **Developers**: Dev environment full access
- **QA Team**: Staging environment access  
- **SRE Team**: Production environment access
- **Viewers**: Read-only across all environments

### Sync Windows
- **Production**: Deployments allowed Mon-Fri 14:00-18:00 UTC
- **Emergency**: Override capability for critical fixes
- **Maintenance**: Blocked during scheduled maintenance windows

### Audit Trail
- All changes tracked in Git history
- ArgoCD operation logs
- Kubernetes audit logs
- Deployment approval records

## Troubleshooting

### Common Issues

#### Sync Failures
```bash
# Check application status
argocd app get <app-name>

# View sync operation details
argocd app sync <app-name> --dry-run

# Check for resource conflicts
kubectl get events -n <namespace>
```

#### Resource Drift
```bash
# Compare desired vs actual state
argocd app diff <app-name>

# Force sync to desired state
argocd app sync <app-name> --force
```

#### Health Check Failures
```bash
# Check pod status
kubectl get pods -n <namespace>

# View pod logs
kubectl logs -n <namespace> <pod-name>

# Check service endpoints
kubectl describe endpoints -n <namespace>
```

## Best Practices

### Git Workflow
- Use feature branches for major changes
- Require PR reviews for production changes
- Use semantic versioning for releases
- Include deployment notes in commit messages

### Configuration Management
- Keep secrets in Kubernetes Secret resources
- Use Helm value overrides for environment differences
- Validate YAML syntax before committing
- Test changes in development first

### Monitoring
- Monitor resource utilization trends
- Set up alerting for deployment failures
- Review ArgoCD sync metrics regularly
- Maintain deployment runbooks

## Emergency Procedures

### Critical Production Issue
1. **Immediate**: Manual rollback via ArgoCD UI
2. **Investigation**: Review logs and metrics
3. **Fix**: Create hotfix branch and emergency deployment
4. **Post-mortem**: Document incident and improve processes

### ArgoCD Outage
1. **Fallback**: Direct kubectl operations (break-glass)
2. **State Sync**: Re-sync ArgoCD when restored
3. **Validation**: Ensure all environments match Git state