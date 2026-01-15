### curriculum

Study Resources Summary
Official Resources
LFS169: Introduction to GitOps - Free
LFS269: GitOps: Continuous Delivery on Kubernetes with Flux - Paid
https://trainingportal.linuxfoundation.org/courses/gitops-continuous-delivery-on-kubernetes-with-flux-lfs269


#### GitOps terminology 
- continous
- declarative description
- desired state
- state drift
- state recognition
- gitops managed software system
- state store
- feedback loop 
- rollback 

#### GitOps principles
- declarative 
- versioned and immutable
- pulled automatically
- continuous reconciled

#### related practices 
- config as code (CaC)
- Infra as Code (IaC)
- DevOps and DevSecOps
- CICD

#### GitOps patterns
- deployment and release pattern
- progressive delivery patterns
- pull vs event-driven
- architecture pattern( in-cluster and external reconcile, state store mgmt)

#### tooling

- manifest format and packaging
- state store systems( git and alternatives)
- reconciliation engine( argoCD, Flux...)
- interoperability with noti, observability, and continous integration tool

# Certified GitOps Associate (CGOA) Learning Plan
## Linux Foundation Certified GitOps Associate Exam Preparation

**Target Date:** Q1 2026 (Alongside CAPA)  
**Study Duration:** 6-8 weeks  
**Daily Commitment:** 1.5-2 hours  
**Exam Type:** Multiple choice, online proctored  
**Exam Duration:** 90 minutes  
**Passing Score:** 66%  

---

## Prerequisites
- Complete [Introduction to GitOps (LFS169)](https://trainingportal.linuxfoundation.org/courses/introduction-to-gitops-lfs169) - Free
- Complete [GitOps: Continuous Delivery on Kubernetes with Flux (LFS269)](https://trainingportal.linuxfoundation.org/courses/gitops-continuous-delivery-on-kubernetes-with-flux-lfs269) - Paid
- Basic Kubernetes knowledge
- Git fundamentals
- YAML syntax familiarity


---

## LFS269

### k8s for gitops
why we need k8s for gitops

### git workflow

- fork and clone the app repo 

- auto generate and commit manifests 

- git branching and merging 
- branching model and trunkbase dev 
  - git flow 
  - github flow 
  - trunk-based dev 

https://nvie.com/posts/a-successful-git-branching-model/
https://docs.github.com/en/get-started/using-github/github-flow
https://trunkbaseddevelopment.com/


- enforcing branching policies 
  - code need to be tested before go to trunk branch 
  - "require pull request review before merging"
  - "require status check to pass before merging" - need CI setup
  - "include administrator"

- tagging and sematic versioning
  - sematic versioning 
  - x.y.z - major(major release backward incompatible).minor(minor feature backward compatible).patch(bug fixes)
  - how to tag: -a annotate 
    - git tag -a v0.1.0 -m 'initial release' 
    - git push origin v0.1.0

### fluxCD

https://fluxcd.io/flux/get-started/
- install flux cli 
  - create personal access token on github
    - only the full repo access, readonly repo hook, make sure no delete access
  - install flux cli 
  - prepare working k8s cluster 
  - boostraping flux infra 
---

## Week 1-2: GitOps Fundamentals & Terminology

### GitOps Terminology
- **Continuous Concepts**
  - Continuous Integration (CI)
  - Continuous Delivery (CD)
  - Continuous Deployment
  - Continuous Reconciliation
  - Difference between delivery and deployment

- **Core Concepts**
  - Declarative description vs imperative commands
  - Desired state vs actual state
  - State drift detection and handling
  - State reconciliation mechanisms
  - GitOps managed software systems
  - State store (Git as single source of truth)
  - Feedback loops and observability
  - Rollback strategies and procedures

### GitOps Principles (The 4 Principles)
1. **Declarative**
   - System state described declaratively
   - YAML/JSON manifests
   - No imperative commands

2. **Versioned and Immutable**
   - Git as version control
   - Immutable infrastructure
   - Audit trail through Git history
   - Branching strategies

3. **Pulled Automatically**
   - Agents pull desired state from Git
   - No push to production
   - Security benefits
   - Reduced blast radius

4. **Continuously Reconciled**
   - Automated drift detection
   - Self-healing systems
   - Reconciliation loops
   - Sync strategies

### Practice Labs
- [ ] Set up Git repository for infrastructure code
- [ ] Create declarative Kubernetes manifests
- [ ] Practice Git operations (branch, merge, revert)
- [ ] Document desired state vs actual state scenarios
- [ ] Implement simple reconciliation workflow

---

## Week 3-4: Related Practices & Patterns

### Related Practices

**Configuration as Code (CaC)**
- Application configuration management
- ConfigMaps and Secrets in K8s
- Environment-specific configurations
- Secret management strategies

**Infrastructure as Code (IaC)**
- Terraform for infrastructure provisioning
- Kubernetes cluster setup as code
- Cloud provider integrations
- State management

**DevOps and DevSecOps**
- DevOps culture and practices
- Security shift-left approach
- Policy as Code
- Compliance automation

**CI/CD Integration**
- CI responsibilities (build, test, publish)
- CD responsibilities (deploy, verify)
- GitOps in CI/CD pipeline
- Separation of concerns

### GitOps Patterns

**Deployment and Release Patterns**
- Environment promotion (Dev → Staging → Production)
- Git branch strategies
- Pull request workflows
- Approval gates
- Multi-cluster deployments
- Fleet management
- Disaster recovery

**Progressive Delivery Patterns**
- Canary deployments
  - Gradual traffic shifting
  - Metrics-based promotion
  - Automated rollback
- Blue-green deployments
  - Zero-downtime deployments
  - Quick rollback capability
- A/B testing
  - Feature flags
  - User-based routing

**Architecture Patterns**
- Pull vs Event-Driven
  - Pull-based reconciliation (polling)
  - Event-driven (webhooks, notifications)
  - Hybrid approaches
  - Trade-offs and use cases

- In-Cluster vs External Reconciliation
  - Agent deployment models
  - Security implications
  - Network requirements
  - Scalability considerations

- State Store Management
  - Monorepo vs multi-repo
  - Repository structure
  - Git branching strategies
  - Access control models

### Practice Labs
- [ ] Implement Terraform for K8s cluster setup
- [ ] Create multi-environment Git repository structure
- [ ] Set up CI pipeline (build, test, push image)
- [ ] Implement promotion workflow (PR-based)
- [ ] Practice canary deployment pattern
- [ ] Configure webhook-based notifications

---

## Week 5-6: GitOps Tooling

### Manifest Format and Packaging

**Kubernetes Native Formats**
- Plain YAML manifests
- Kustomize overlays and bases
- Directory structure patterns
- Strategic merge patches

**Helm**
- Helm charts structure
- Values.yaml templating
- Chart dependencies
- Helm in GitOps context (templating only)
- HelmRelease CRD

**Other Formats**
- Jsonnet
- CUE language
- OCI artifacts

### State Store Systems

**Git as State Store**
- Git repository structure
- Commit message conventions
- Tag and release management
- Git submodules vs separate repos

**Git Alternatives**
- OCI registries
- S3-compatible storage
- Trade-offs vs Git
- Use cases

### Reconciliation Engines

**Flux CD**
- Architecture
  - Source Controller
  - Kustomize Controller
  - Helm Controller
  - Notification Controller
  - Image Reflector/Automation Controllers

- Core Resources
  - GitRepository
  - Kustomization
  - HelmRepository
  - HelmRelease
  - ImageRepository
  - ImagePolicy

- Flux CLI
  - `flux bootstrap`
  - `flux create source git`
  - `flux create kustomization`
  - `flux reconcile`
  - `flux logs`

- Configuration
  - Multi-tenancy setup
  - RBAC configuration
  - Notification providers
  - Image automation

**ArgoCD (Overview)**
- Architecture components
- Application CRD
- Projects and AppProjects
- Sync policies
- CLI basics

**Comparison: Flux vs ArgoCD**
- Agent-based vs API-based
- CLI vs UI preferences
- RBAC models
- Resource consumption
- Community and ecosystem

### Interoperability

**Notifications**
- Slack, Discord, Teams integrations
- Git commit status updates
- Alert manager integration
- Custom webhooks

**Observability**
- Prometheus metrics export
- Grafana dashboards
- Logging (Loki integration)
- Tracing (OpenTelemetry)

**CI Integration**
- GitHub Actions
- GitLab CI
- Jenkins
- Tekton
- Image update automation

### Practice Labs
- [ ] Create Kustomize overlays for multiple environments
- [ ] Package application with Helm chart
- [ ] Install Flux in Kubernetes cluster
- [ ] Bootstrap Flux with GitHub repository
- [ ] Create Flux GitRepository and Kustomization resources
- [ ] Set up HelmRelease with Flux
- [ ] Configure image automation pipeline
- [ ] Implement Slack notifications
- [ ] Set up Prometheus monitoring for Flux

---

## Week 7: Practical Implementation

### Complete GitOps Setup Project

**Requirements:**
- Application repository in GitHub
- CI pipeline (build and publish to Docker Hub)
- Helm charts for packaging
- Flux for deployment
- Automated image updates

**Implementation Steps:**
1. Application repo with GitHub Actions CI
2. GitOps repo with proper structure
3. CI pipeline (build, test, push)
4. Flux bootstrap to cluster
5. HelmRelease configuration
6. Image automation setup
7. Monitoring and notifications

**Testing & Validation:**
- Trigger CI pipeline
- Verify Flux reconciliation
- Test rollback scenarios
- Validate state drift handling

### Practice Labs
- [ ] Set up complete CI pipeline with GitHub Actions
- [ ] Bootstrap Flux in local Kubernetes cluster
- [ ] Implement HelmRelease with Flux
- [ ] Configure image reflector and automation
- [ ] Test end-to-end deployment flow
- [ ] Implement rollback procedure
- [ ] Add monitoring and notifications

---

## Week 8: Exam Preparation

### Mock Scenarios & Practice

**Scenario 1: Multi-Environment Deployment**
- Set up Git repo for dev/staging/prod
- Implement environment-specific configs
- Configure promotion workflow via PRs
- Test deployment across environments

**Scenario 2: Disaster Recovery**
- Simulate cluster failure
- Bootstrap new cluster with Flux
- Restore application state from Git
- Verify complete recovery

**Scenario 3: Security Incident Response**
- Detect unauthorized manual changes (drift)
- Trigger automated reconciliation
- Audit changes via Git history
- Implement preventive controls

**Scenario 4: Progressive Delivery**
- Implement canary deployment
- Configure automated rollback on failure
- Set up metrics-based promotion

### Key Concepts to Master

**Troubleshooting:**
- Flux resource status checking
- Log analysis (`flux logs`)
- Debugging reconciliation failures
- Common error patterns

**Security Best Practices:**
- Secret management (Sealed Secrets, SOPS)
- RBAC configuration
- Git authentication methods
- Least privilege principle

**Performance & Scalability:**
- Repository size optimization
- Sync frequency tuning
- Multi-cluster management
- Resource limits and requests

### Exam Tips
1. Know the 4 GitOps principles thoroughly
2. Be comfortable with Flux CLI commands
3. Master Git operations (branch, merge, revert)
4. Understand Kustomize and Helm patterns
5. Practice troubleshooting common issues
6. Understand architecture trade-offs

### Final Checklist
- [ ] Review all GitOps principles and terminology
- [ ] Practice Flux commands and operations
- [ ] Understand CI/CD integration points
- [ ] Know progressive delivery patterns
- [ ] Master Git-based workflows
- [ ] Review security best practices
- [ ] Complete hands-on scenarios
- [ ] Take practice exam (if available)

---

## Study Resources

### Official
- [LFS169: Introduction to GitOps](https://trainingportal.linuxfoundation.org/courses/introduction-to-gitops-lfs169) - Free
- [LFS269: GitOps with Flux](https://trainingportal.linuxfoundation.org/courses/gitops-continuous-delivery-on-kubernetes-with-flux-lfs269) - Paid
- [CGOA Exam Domains](https://training.linuxfoundation.org/certification/certified-gitops-associate-cgoa/)
- [OpenGitOps Project](https://opengitops.dev/)

### Documentation
- Flux Documentation: https://fluxcd.io/
- ArgoCD Documentation: https://argo-cd.readthedocs.io/
- Kustomize: https://kustomize.io/
- GitOps Working Group: https://github.com/gitops-working-group

### Community
- Flux Slack channel
- ArgoCD Slack channel
- CNCF Slack #gitops channel
- GitHub: Flux and ArgoCD examples

### Lab Setup
- Kubernetes cluster (Kind, Minikube, K3s)
- Git account (GitHub free tier)
- Container registry (Docker Hub free tier)
- Tools: flux CLI, kubectl, helm, kustomize, git

---

## Progress Tracking

| Week | Topic | Status | Notes |
|------|-------|--------|-------|
| 1-2  | GitOps Fundamentals & Terminology | ⬜ |  |
| 3-4  | Related Practices & Patterns | ⬜ |  |
| 5-6  | GitOps Tooling (Flux, ArgoCD) | ⬜ |  |
| 7    | Practical Implementation | ⬜ |  |
| 8    | Exam Preparation | ⬜ |  |

---

## CGOA + CAPA Combined Strategy

**Parallel Study Plan:**
- **Week 1-2**: GitOps principles (CGOA) + Argo Workflows (CAPA)
- **Week 3-4**: GitOps patterns (CGOA) + Advanced Workflows + Argo CD (CAPA)
- **Week 5-6**: Flux deep dive (CGOA) + Argo CD + Rollouts (CAPA)
- **Week 7-8**: Flux implementation (CGOA) + Argo Events + Integration (CAPA)

**Synergies:**
- Both cover GitOps principles
- Complementary tools (Flux vs Argo)
- Progressive delivery patterns
- Event-driven automation
- Complete CI/CD understanding

**Benefits:**
- Comprehensive GitOps ecosystem knowledge
- Tool comparison expertise (Flux vs Argo)
- Better architecture decisions
- Dual certification value
- Increased job market competitiveness

---

**Last Updated:** January 5, 2026  
**Target Exam Date:** Q1 2026  
**Study Partner:** CAPA (Certified Argo Project Associate)
