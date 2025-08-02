# LINQ Platform - ArgoCD Repository

This repository contains the ArgoCD configuration and Helm charts for deploying the LINQ platform across multiple environments using GitOps principles.

## Overview

The LINQ platform consists of multiple microservices deployed across different environments (development, staging, production) with environment-specific configurations and service versions.

### Architecture

```
├── helm-charts/           # Helm charts for each service
├── environments/          # Environment-specific configurations
├── argocd-apps/          # ArgoCD Application manifests
└── docs/                 # Documentation
```

## Services

### Core Services
- **linq-workcell-manager** - Manages workcell operations and state
- **linq-orchestrator** - Orchestrates workflows and service interactions  
- **linq-service-metadata-repository** - Stores and manages service metadata
- **linq-workflow** - Handles workflow execution and management
- **linq-planner** - Planning and scheduling service
- **linq-frontend** - User interface and web frontend
- **linq-aggregator** - Data aggregation and processing service

### Foundation Services
- **platform** - Core platform version (v6.2.1+)

### Workcell Services  
- **linq-maestro** - Workcell-specific service orchestration

## Environments

### Development Environment (`dev-env-1`)
- **Purpose**: Development and testing
- **Cluster**: `dev-cluster`
- **Namespace**: `dev-env-1`
- **Sync Policy**: Automated (prune: true, selfHeal: true)
- **Service Versions**: Latest development versions
- **Resources**: Minimal resource allocation for cost efficiency

**Current Bundle**: Development Bundle v1.0.0
- linq-workcell-manager: 2.1.0
- linq-orchestrator: 3.2.1
- linq-service-metadata-repository: 1.5.0
- linq-workflow: 4.0.0
- linq-planner: 2.3.1
- linq-frontend: 5.1.2
- linq-aggregator: 1.8.3
- platform-version: 6.2.1
- linq-maestro: 2.7.1

### Staging Environment (`staging-env-1`)
- **Purpose**: Pre-production testing and validation
- **Cluster**: `staging-cluster`
- **Namespace**: `staging-env-1`
- **Sync Policy**: Automated (prune: true, selfHeal: true)
- **Service Versions**: Stable staging versions
- **Resources**: Medium resource allocation with 2 replicas

**Current Bundle**: Staging Bundle v1.1.0
- linq-workcell-manager: 2.1.1
- linq-orchestrator: 3.2.2
- linq-service-metadata-repository: 1.5.1
- linq-workflow: 4.0.1
- linq-planner: 2.3.2
- linq-frontend: 5.1.3
- linq-aggregator: 1.8.4
- platform-version: 6.2.2
- linq-maestro: 2.7.2

### Production Environment (`production-env-1`)
- **Purpose**: Live production workloads
- **Cluster**: `production-cluster`
- **Namespace**: `production-env-1`
- **Sync Policy**: Manual (prune: false, selfHeal: false)
- **Service Versions**: Stable production versions
- **Resources**: High resource allocation with 3 replicas

**Current Bundle**: Production Bundle v1.2.0
- linq-workcell-manager: 2.2.0
- linq-orchestrator: 3.3.0
- linq-service-metadata-repository: 1.6.0
- linq-workflow: 4.1.0
- linq-planner: 2.4.0
- linq-frontend: 5.2.0
- linq-aggregator: 1.9.0
- platform-version: 6.3.0
- linq-maestro: 2.8.0

## Bundle Types and Compatibility

The platform follows a strict bundle type hierarchy:

- **development** < **staging** < **production**

### Deployment Rules
- Development bundles can only be deployed to development environments
- Staging bundles can be deployed to development or staging environments  
- Production bundles can be deployed to any environment type

## ArgoCD Applications

Each environment has dedicated ArgoCD Applications for service deployment:

- `dev-env-1-linq-platform` - Main platform services for development
- `staging-env-1-linq-platform` - Main platform services for staging
- `production-env-1-linq-platform` - Main platform services for production

### Application Features
- **Automated Sync**: Dev/Staging environments auto-sync, Production requires manual approval
- **Health Monitoring**: Built-in health checks and monitoring
- **Rollback Support**: Maintains revision history for easy rollbacks
- **Resource Management**: Environment-specific resource allocations

## Environment Status

Each environment maintains status information including:
- Current bundle version and service versions
- Health status of all services
- Deployment information (cluster, namespace, replicas)
- Last update timestamps

Status files are located at: `environments/{env-name}/status.yaml`

## GitOps Workflow

1. **Code Changes**: Developers push changes to service repositories
2. **Image Building**: CI/CD builds and tags new service images
3. **Values Update**: Update environment values files with new image tags
4. **ArgoCD Sync**: ArgoCD detects changes and syncs to Kubernetes
5. **Status Update**: Environment status is updated automatically

## Deployment Process

### Development Deployments
1. Update `environments/dev-env-1/values.yaml` with new service versions
2. Commit changes to repository
3. ArgoCD automatically syncs changes to dev cluster

### Staging Deployments  
1. Update `environments/staging-env-1/values.yaml` with tested versions
2. Commit changes to repository
3. ArgoCD automatically syncs changes to staging cluster

### Production Deployments
1. Update `environments/production-env-1/values.yaml` with stable versions
2. Commit changes to repository
3. **Manual sync required** - ArgoCD waits for manual approval
4. Review changes in ArgoCD UI and manually trigger sync

## Monitoring and Observability

Each service includes:
- **Health Checks**: Liveness and readiness probes
- **Metrics**: Prometheus metrics endpoint on port 9090
- **Logging**: Structured logging with configurable levels
- **Tracing**: Distributed tracing capabilities

## Security

- **RBAC**: Role-based access control for ArgoCD
- **Network Policies**: Environment isolation
- **Secret Management**: Kubernetes secrets for sensitive data
- **Image Security**: Only trusted image repositories

## Maintenance

### Adding New Services
1. Create Helm chart in `helm-charts/{service-name}/`
2. Add service configuration to environment values files
3. Create ArgoCD Application manifest in `argocd-apps/`

### Environment Updates
1. Update values in `environments/{env-name}/values.yaml`
2. Update status in `environments/{env-name}/status.yaml`
3. Commit changes and let ArgoCD sync

### Version Upgrades
1. Test in development environment first
2. Promote to staging after validation
3. Deploy to production with manual approval

## Troubleshooting

### ArgoCD Application Issues
```bash
# Check application status
argocd app get {app-name}

# View application logs
argocd app logs {app-name}

# Manual sync
argocd app sync {app-name}
```

### Service Health Issues
```bash
# Check pod status
kubectl get pods -n {namespace}

# View service logs
kubectl logs -n {namespace} {pod-name}

# Check service health
kubectl get events -n {namespace}
```

## Support

For issues and questions:
- Create issues in this repository
- Contact the LINQ Platform Team
- Check ArgoCD UI for deployment status