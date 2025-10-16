# Infrastructure Repository - Copilot Instructions

This is an infrastructure operations repository for managing ArgoCD itself and all infrastructure components through GitOps.

## Repository Context

This repository contains:
1. **ArgoCD Installation & Configuration**: The ArgoCD platform itself (bootstrap manifests, configs, RBAC)
2. **ArgoCD Applications**: Application definitions that ArgoCD manages
3. **Infrastructure Components**: Kubernetes manifests for infrastructure services

This is a bootstrap repository - ArgoCD manages itself and all other infrastructure through GitOps.

## Key Principles

- **GitOps Workflow**: All infrastructure changes are version-controlled and deployed via ArgoCD
- **Declarative Configuration**: Use Kubernetes manifests (YAML) to define desired state
- **Immutable Infrastructure**: Update versions/configurations rather than modifying running resources
- **Environment Separation**: Maintain clear separation between dev, staging, and production environments

## Common Tasks

### Managing ArgoCD Itself
- **Installation Manifests**: Update ArgoCD version, components, and core configuration
- **ArgoCD ConfigMaps**: Modify argocd-cm, argocd-rbac-cm, argocd-cmd-params-cm
- **RBAC Policies**: Configure user/group permissions and project access
- **Repository Credentials**: Manage repository connections (via secrets or credential templates)
- **SSO/Auth Configuration**: Update OIDC, SAML, or other authentication settings
- **App of Apps Pattern**: Maintain the root Application that manages all other apps

### ArgoCD Applications
- Create ArgoCD Application manifests in the appropriate directory
- Include proper labels for organization (e.g., `environment`, `team`, `component`)
- Set sync policies (automated vs manual) based on environment criticality
- Configure health checks and sync waves for proper deployment ordering

### Kubernetes Manifests
- Follow Kubernetes best practices for resource definitions
- Always specify resource limits and requests
- Use ConfigMaps and Secrets for configuration management
- Implement proper RBAC policies

### Helm Charts
- When using Helm, maintain values files per environment
- Pin chart versions explicitly for reproducibility
- Document any custom values or overrides

## Standards

### Commit Messages
- Follow [Conventional Commits](https://www.conventionalcommits.org/) specification
- Format: `<type>[optional scope]: <description>`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`, `perf`
- Examples:
  - `feat(nginx): add nginx deployment to argocd namespace`
  - `fix(argocd): correct sync policy configuration`
  - `chore(apps): update repository URLs`
  - `docs: update README with bootstrap instructions`

### YAML Format
- Use 2 spaces for indentation
- Include meaningful metadata (labels, annotations)
- Add comments for non-obvious configurations

### Naming Conventions
- Use kebab-case for resource names
- Include environment prefix where applicable (e.g., `prod-app-name`)
- Keep names descriptive but concise

### Security
- Never commit secrets in plain text
- Use sealed-secrets, external-secrets, or vault integration for repository credentials and sensitive configs
- Apply principle of least privilege for service accounts and ArgoCD RBAC
- Protect ArgoCD admin credentials and consider SSO for user access
- Use AppProjects to isolate and restrict application permissions

### Validation
- Validate YAML syntax before committing
- Use `kubectl dry-run` or kubeval for manifest validation
- Check ArgoCD application definitions for correctness

## Directory Structure Expectations

Typical structure includes:
- `argocd/` or `bootstrap/` - ArgoCD installation and core configuration
  - `install/` - ArgoCD deployment manifests
  - `config/` - ConfigMaps (argocd-cm, argocd-rbac-cm, etc.)
  - `projects/` - AppProject definitions
- `apps/` or `applications/` - ArgoCD Application definitions (app-of-apps)
- `base/` - Base Kubernetes manifests for infrastructure services
- `overlays/` or `environments/` - Environment-specific configurations (kustomize)
- `charts/` - Helm chart values and definitions

## When Making Changes

### Changes to ArgoCD Itself
1. **Critical**: Understand that changes affect the entire GitOps platform
2. Test ArgoCD upgrades in non-production first
3. Review breaking changes in ArgoCD release notes
4. Backup ArgoCD configuration before major changes
5. Consider impact on all managed applications
6. Use sync waves to control upgrade order of ArgoCD components

### Changes to Applications
1. Understand the impact scope (which environments/apps are affected)
2. Validate manifests before committing
3. Consider sync policies and whether manual intervention is needed
4. Document significant changes in commit messages
5. Be cautious with production changes - suggest manual sync when appropriate

## Helpful Commands

```bash
# Validate Kubernetes manifests
kubectl apply --dry-run=client -f <file>

# Check ArgoCD status
kubectl get pods -n argocd
argocd version

# Check ArgoCD application status
argocd app get <app-name>

# Preview changes before sync
argocd app diff <app-name>

# Sync application
argocd app sync <app-name>

# View ArgoCD configuration
kubectl get configmap argocd-cm -n argocd -o yaml
kubectl get configmap argocd-rbac-cm -n argocd -o yaml

# Restart ArgoCD components after config changes
kubectl rollout restart deployment argocd-server -n argocd
kubectl rollout restart statefulset argocd-application-controller -n argocd
```

## Code Review Focus

When reviewing or suggesting changes:
- **For ArgoCD changes**: Verify version compatibility, breaking changes, and impact on all managed apps
- Verify resource specifications are appropriate
- Check for security implications (especially RBAC and secrets)
- Ensure compatibility with existing infrastructure
- Validate ArgoCD sync behavior and health checks
- Confirm environment-specific values are correct
- Review AppProject permissions and restrictions

## Bootstrap & Self-Management

This repository uses the "App of Apps" pattern where:
1. ArgoCD is installed via initial bootstrap (typically `kubectl apply`)
2. A root Application points to this repository
3. ArgoCD then manages itself and all other applications from this repo
4. Changes to ArgoCD configuration are deployed through GitOps like any other application
