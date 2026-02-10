#### argo workflows 
- argo workflow fundamentals 
- generating and consuming artifacts
- argo workflow templates
- argo workflow spec
- work with DAG( directed-acyclic graphs)
- data processing jobs with argo workflow

#### argo cd 
- understand argo cd fundamentals
- sync app using argo cd
- use argo cd app
- config argo cd with helm and kustomize
- identify common recon pattern

#### argo rollouts
- argo rollout fundamentals
- common progressive rollout strategies 
- describe analysis template and analysis run

#### argo events
- argo events fundamentals
- argo event components and architecture 

# Certified Argo Project Associate (CAPA) Learning Plan
## Linux Foundation Certified Argo Project Associate Exam Preparation

**Current Status:** Ready to begin after CGOA success (90% - Jan 2026) ✅  
**Target Date:** End of Q1 2026 (February-March 2026)  
**Study Duration:** 6-8 weeks  
**Daily Commitment:** 1.5-2 hours  
**Exam Type:** Multiple choice, online proctored  
**Exam Duration:** 90 minutes  
**Passing Score:** 66%  

---

## Prerequisites ✅
- ✅ Complete [DevOps and Workflow Management with Argo (LFS256)](https://trainingportal.linuxfoundation.org/courses/devops-and-workflow-management-with-argo-lfs256)
- ✅ Basic Kubernetes knowledge (CKAD passed Sept 2025, CKA passed Dec 2025)
- ✅ Understanding of GitOps principles (CGOA passed Jan 2026 - 90%)
- ✅ YAML and container fundamentals
- ✅ CI/CD concepts

---

## Your Advantages

**Strong Foundation:**
- ✅ **CGOA passed (90%)**: Deep GitOps principles understanding
- ✅ **CKA passed**: Kubernetes cluster operations expertise
- ✅ **CKAD passed**: Application development and deployment skills
- ✅ **Flux knowledge**: Can compare and contrast with Argo CD

**What This Means:**
- Skip basic K8s and GitOps review
- Focus on Argo-specific implementations
- Leverage Flux vs Argo CD comparisons
- Build on existing CI/CD knowledge
- Faster progression through material

---

## Week 1-2: Argo Workflows (Building on CKA/CKAD)

### Argo Workflow Fundamentals
**Focus Areas** (you already know K8s CRDs from CKA):
- Workflow as Kubernetes CRD structure
- Workflow phases: Pending → Running → Succeeded/Failed
- Workflow Controller reconciliation loop
- Step vs DAG templates (new concept)
- Retry and timeout strategies (similar to Job/CronJob)
- Exit handlers for cleanup
- Workflow lifecycle vs Deployment lifecycle

### Generating and Consuming Artifacts
**New Concepts** (beyond CKA Volumes):
- Output artifacts from workflow steps
- Input artifacts to subsequent steps
- Artifact passing between steps (similar to ConfigMap/Secret mounting)
- S3, GCS, MinIO storage configuration
- Artifact repository ConfigMap setup
- Global artifacts vs step artifacts
- Artifact garbage collection policies

### Argo Workflow Templates
**Template Types** (know when to use each):
- Container template (like Pod spec)
- Script template (inline script execution)
- Resource template (create K8s resources)
- Suspend template (manual approval gates)
- WorkflowTemplate CRD (reusable, namespace-scoped)
- ClusterWorkflowTemplate (cluster-wide, like ClusterRole)
- Parameters and arguments
- Input/output parameters
- Default values and overrides

### Argo Workflow Spec
**Leverage CKA Knowledge:**
- `entrypoint` definition (entry point template)
- `templates` section (like containers in Pod)
- `volumes` and `volumeMounts` (you know this!)
- `imagePullSecrets` (same as Pod)
- `serviceAccountName` (RBAC knowledge applies)
- `activeDeadlineSeconds` (like Job)
- `podGC` garbage collection (new)
- TTL for completed workflows
- `priorityClassName` (from CKA)
- Node selectors and affinity (you're familiar)

### Practice Labs
- [ ] Install Argo Workflows via kubectl (use CKA skills)
- [ ] Create workflow with container steps
- [ ] Implement artifact passing with MinIO
- [ ] Create reusable WorkflowTemplates
- [ ] Configure artifact repository in ConfigMap
- [ ] Test retry strategies (compare to Job backoffLimit)
- [ ] Practice CLI: `argo submit`, `argo list`, `argo logs`, `argo get`

**Key CLI Commands:**
```bash
argo submit workflow.yaml -n argo --watch
argo list -n argo
argo get <workflow-name> -n argo
argo logs <workflow-name> -n argo
argo delete <workflow-name> -n argo
argo retry <workflow-name>
argo terminate <workflow-name>
```

---

## Week 3: Advanced Workflows (New Patterns)

### Work with DAG (Directed Acyclic Graphs)
**New Paradigm** (different from sequential Steps):
- DAG vs Steps comparison
- Explicit task dependencies: `dependencies: [task1, task2]`
- Parallel execution by default
- Conditional dependencies based on task status
- Dynamic task generation with parameters
- Task result passing between dependencies
- Complex dependency patterns (fan-out, fan-in)

**Compare to CKA:**
- Similar to Job parallelism but more flexible
- Like Init Containers but non-sequential
- Dependencies instead of sequential order

### Data Processing Jobs
**Batch Processing Patterns:**
- Parallel processing with loops
- Map-reduce workflows (new pattern)
- Dynamic parallelism:
  - `withItems`: static list iteration
  - `withParam`: dynamic JSON array
  - `withSequence`: numeric sequences
- ETL pipeline patterns
- Resource optimization strategies
- Pod GC for cleanup

### Practice Labs
- [ ] Convert Steps workflow to DAG
- [ ] Build DAG with parallel tasks (e.g., parallel tests)
- [ ] Implement map-reduce data processing
- [ ] Create dynamic parallel workflows with `withParam`
- [ ] Test complex dependencies (fan-out → process → fan-in)
- [ ] Optimize workflow resource usage (requests/limits)
- [ ] Build CI/CD pipeline DAG (checkout → parallel tests → build → deploy)

---

## Week 4-5: Argo CD (Leverage CGOA Knowledge)

### Understand Argo CD Fundamentals
**Architecture Components** (compare to Flux from CGOA):
- **API Server**: UI, CLI, webhooks (Flux has no UI)
- **Repository Server**: Git caching (like Flux Source Controller)
- **Application Controller**: Reconciliation (like Flux Kustomize Controller)
- **Redis**: Caching layer
- **Dex**: SSO integration (optional)

**Core Concepts** (you know GitOps already):
- **Application CRD** (vs Flux's GitRepository + Kustomization)
- **Project**: Logical grouping with RBAC (Flux uses namespaces)
- **Sync vs Refresh**: Manual vs automated reconciliation
- **Health status**: Resource health assessment
- **Sync waves**: Ordered deployment (Flux uses depends-on)
- **Out of sync detection**: Drift detection (like Flux)

### Sync App Using Argo CD
**Sync Strategies** (compare to Flux automation):
- Manual sync (explicit `argocd app sync`)
- Automated sync (`syncPolicy.automated`)
- Sync options:
  - `prune: true` (delete removed resources - like Flux prune)
  - `selfHeal: true` (auto-correct drift - like Flux force)
- Selective sync (sync specific resources)
- Sync waves for ordering (0, 1, 2...)
- Pre/post sync hooks (like Helm hooks)
- Sync timeout configuration

### Use Argo CD App
**Application CRD** (you understand CRDs from CKA):
- Create via UI (Argo has UI, Flux doesn't)
- Create via CLI: `argocd app create`
- Create via YAML (declarative GitOps way)
- **Source**: Git repo, path, revision
- **Destination**: cluster URL, namespace
- **App of Apps pattern** (bootstrap multiple apps)
- **ApplicationSet**: Template-based generation (like Flux with multiple sources)
- Multi-environment management
- Project configuration and RBAC

### Config Argo CD with Helm and Kustomize
**Helm Integration** (you used Helm in CKAD):
- Helm repository configuration
- Chart version selection
- Values file management (`values.yaml`)
- Value overrides in Application CRD
- Multiple values files support
- Parameters vs values
- Helm hooks and weights

**Kustomize Integration** (compare to Flux Kustomization):
- Base and overlay structure (same as Flux)
- `namePrefix`/`nameSuffix`
- `commonLabels`/`commonAnnotations`
- Image tag updates (Argo requires manual/webhook, Flux has automation)
- Patches and strategic merge
- Kustomize build options

### Identify Common Reconciliation Patterns
**Reconciliation** (compare to Flux from CGOA):
- Pull-based reconciliation with polling interval
- Webhook-based refresh (faster than polling)
- Automated sync with self-heal (like Flux force)
- Manual approval gates (Projects can enforce)
- Progressive sync strategies (with Rollouts)
- Multi-cluster patterns (Argo CD can manage multiple clusters)
- Monorepo vs multi-repo (same debate as Flux)

**Key Differences from Flux:**
| Feature | Argo CD | Flux (from CGOA) |
|---------|---------|------------------|
| UI | Built-in Web UI | No UI (external tools) |
| CRD | Application | GitRepository + Kustomization |
| Multi-cluster | Native support | Requires multiple Flux installations |
| Image updates | Webhook/manual | Native Image Automation Controllers |
| RBAC | Project-based | Namespace-based |

### Practice Labs
- [ ] Install Argo CD: `kubectl apply -n argocd -f install.yaml`
- [ ] Access UI: `kubectl port-forward svc/argocd-server -n argocd 8080:443`
- [ ] Deploy app via UI (compare experience to Flux CLI-only)
- [ ] Deploy app via CLI: `argocd app create`
- [ ] Deploy app via YAML (GitOps way)
- [ ] Configure Helm chart deployment
- [ ] Set up Kustomize overlay-based application
- [ ] Test automated sync with self-heal (create drift manually)
- [ ] Create App of Apps pattern
- [ ] Configure webhook from GitHub/GitLab
- [ ] Compare Argo CD vs Flux workflows

**Key CLI Commands:**
```bash
argocd login <server>
argocd app create <name> --repo <url> --path <path> --dest-server <cluster> --dest-namespace <ns>
argocd app list
argocd app get <name>
argocd app sync <name>
argocd app diff <name>  # Show pending changes
argocd app delete <name>
argocd cluster add <context>
argocd proj create <name>
```

---

## Week 6: Argo Rollouts (New Advanced Deployment Patterns)

### Argo Rollout Fundamentals
**New Concepts** (beyond Deployment from CKAD):
- Purpose: Progressive delivery strategies
- Rollout CRD vs Deployment (superset of Deployment)
- Rollout controller watches Rollouts
- Rollout phases: Healthy, Progressing, Degraded, Paused
- Manual operations: promote, abort, pause, resume
- Rollout history and revisions (like Deployment)
- Integration with service mesh (Istio, Linkerd)

### Common Progressive Rollout Strategies

**Canary Deployment:**
- Gradual traffic shifting (20% → 40% → 60% → 80% → 100%)
- Pause steps: duration-based or manual approval
- Analysis runs between steps (metrics-based decisions)
- Manual promotion: `kubectl argo rollouts promote`
- Automatic promotion based on analysis success
- Abort on failure: `kubectl argo rollouts abort`
- Service mesh integration for traffic split (Istio VirtualService)

**Blue-Green Deployment:**
- Active service (current production - "blue")
- Preview service (new version - "green")
- Instant traffic switching (no gradual shift)
- Zero-downtime deployments
- Quick rollback: switch back to blue
- Resource cost: runs 2x replicas temporarily
- Auto-promotion configuration: manual vs automatic

**Traffic Management:**
- SMI (Service Mesh Interface) standard
- Istio VirtualService manipulation
- Linkerd TrafficSplit CRD
- NGINX Ingress with canary annotations
- ALB (AWS) with TargetGroup weights
- Ambassador integration

### Describe Analysis Template and Analysis Run

**AnalysisTemplate:**
- Define metrics to query (Prometheus, Datadog, New Relic, CloudWatch)
- Success condition: e.g., `error_rate < 0.01`
- Failure condition: e.g., `error_rate > 0.05`
- Query configuration (PromQL, Datadog query, etc.)
- Interval: how often to measure (e.g., every 30s)
- Count: how many measurements (e.g., 10 times)
- Initial delay before first measurement
- Arguments passed from Rollout

**AnalysisRun:**
- Created automatically during Rollout
- Can also create manually for testing
- Inline analysis (defined in Rollout spec)
- Template-based analysis (reference AnalysisTemplate)
- Background analysis (runs throughout)
- Foreground analysis (blocks next step)
- Success/failure thresholds
- Inconclusive handling

**Automated Decisions:**
- Promote on analysis success
- Abort on analysis failure
- Automated rollback triggers
- Manual intervention override
- Dry-run mode for testing

### Practice Labs
- [ ] Install Argo Rollouts: `kubectl create ns argo-rollouts && kubectl apply -f install.yaml`
- [ ] Install Rollouts kubectl plugin
- [ ] Convert Deployment to Rollout (change `kind: Deployment` to `kind: Rollout`)
- [ ] Create canary Rollout with manual promotion
- [ ] Implement blue-green deployment
- [ ] Install Prometheus for metrics
- [ ] Configure AnalysisTemplate with Prometheus query
- [ ] Test automated rollback on metric failure
- [ ] Integrate with NGINX Ingress for traffic management
- [ ] Practice manual operations: promote, abort, pause, resume

**Key CLI Commands:**
```bash
kubectl argo rollouts list rollouts
kubectl argo rollouts get rollout <name> --watch
kubectl argo rollouts status <name>
kubectl argo rollouts promote <name>
kubectl argo rollouts abort <name>
kubectl argo rollouts retry <name>
kubectl argo rollouts set image <name> <container>=<image>
kubectl argo rollouts dashboard  # Optional UI
```

---

## Week 7: Argo Events (Event-Driven Automation)

### Argo Events Fundamentals
**New Paradigm** (Event-driven vs GitOps pull):
- Event-driven automation for Kubernetes
- Use cases:
  - CI/CD triggers (Git push → build workflow)
  - Incident response (alert → remediation workflow)
  - Scheduled jobs (cron → cleanup workflow)
- Trigger Argo Workflows automatically
- Webhook handling (GitHub, GitLab, Bitbucket, generic)
- Calendar/cron events (scheduled automation)
- Resource change detection (K8s events)
- Multi-source event aggregation (combine multiple events)

### Argo Event Components and Architecture

**Core Components:**
1. **EventSource CRD**: Defines where events come from
2. **Sensor CRD**: Listens to events and triggers actions
3. **EventBus**: NATS messaging backbone (transport layer)
4. **Triggers**: Actions to execute (create Workflow, K8s resource, HTTP request)

**EventSource Types:**
- **Webhook**: Generic HTTP endpoint, GitHub, GitLab, Bitbucket, Slack
- **Calendar**: Cron-like scheduling
- **Resource**: Kubernetes object changes (Pod, Deployment, etc.)
- **AWS**: SNS, SQS, S3 events
- **Kafka**: Kafka topic messages
- **NATS**: NATS messages
- **Redis**: Redis pub/sub
- **MQTT**: IoT device events
- **AMQP**: RabbitMQ messages
- **File**: File system changes

**Sensor Components:**
- **Dependencies**: Event filters and conditions
- **Triggers**: Action definitions (what to do when event occurs)
- **Template**: Trigger specification (Workflow spec, K8s resource spec)
- **Circuit breaker**: Prevent cascading failures
- **Rate limiting**: Control trigger frequency

**Trigger Types:**
- **Argo Workflow**: Create and submit Workflow
- **Kubernetes Resource**: Create/update/delete K8s resources
- **HTTP**: Send HTTP requests to external systems
- **Slack**: Send Slack notifications
- **AWS Lambda**: Invoke Lambda functions
- **Apache OpenWhisk**: Invoke OpenWhisk actions
- **Log**: Output to logs

**EventBus:**
- NATS Streaming as default backend
- Jetstream configuration (NATS 2.0+)
- Persistence options (in-memory vs persistent)
- Multi-tenant isolation (namespace-based)
- Scalability configuration

### Use Cases (Real-World Examples)
- **CI/CD**: Git push → EventSource webhook → Sensor triggers Workflow → Build & test
- **Infrastructure Automation**: Pod crash → Resource EventSource → Sensor triggers remediation Workflow
- **Scheduled Maintenance**: Calendar EventSource (daily 2 AM) → Sensor triggers cleanup Workflow
- **Alert Response**: Prometheus alert → Webhook EventSource → Sensor triggers incident Workflow
- **Multi-step Workflows**: Event A + Event B → Sensor triggers combined Workflow

### Practice Labs
- [ ] Install Argo Events: `kubectl create ns argo-events && kubectl apply -f install.yaml`
- [ ] Create NATS EventBus: `kubectl apply -f eventbus.yaml`
- [ ] Create webhook EventSource
- [ ] Build Sensor to trigger Workflow on webhook
- [ ] Test GitHub push → Workflow automation
- [ ] Configure calendar-based scheduled Workflow
- [ ] Create resource EventSource (watch Pod events)
- [ ] Set up multi-source event aggregation (event A + event B)
- [ ] Send Slack notification on Workflow completion
- [ ] Build complete event-driven CI/CD pipeline

---

## Week 8: Integration & Exam Prep

### Complete Argo Pipeline Integration
**End-to-End Flow** (All 4 Argo projects together):

1. **Developer Action**: Push code to GitHub
2. **Argo Events**: 
   - GitHub webhook → EventSource receives push event
   - Sensor detects event and triggers Argo Workflow
3. **Argo Workflows**:
   - Checkout code from Git
   - Build Docker image
   - Run unit tests
   - Run integration tests
   - Push image to registry
   - Update K8s manifest in Git (new image tag)
4. **Argo CD**:
   - Detects Git change (polling or webhook)
   - Syncs updated manifest to cluster
   - Creates/updates Argo Rollout
5. **Argo Rollouts**:
   - Starts canary deployment (20% traffic)
   - Runs AnalysisTemplate (queries Prometheus metrics)
   - Analysis success → Promote to 40%, 60%, 80%, 100%
   - Analysis failure → Abort and rollback
6. **Notification**: Argo Events sends Slack notification on completion

**Integration Points:**
- Events → Workflows: Trigger workflow execution
- Workflows → Git: Update manifests (image tags)
- Git → CD: GitOps sync
- CD → Rollouts: Deploy with progressive strategy
- Rollouts → Metrics: Analysis for decisions
- All → Notifications: Status updates

### Key Concepts Summary (Exam Focus)

**Argo Workflows:**
- [ ] Template types: Container, Script, Resource, Suspend
- [ ] Steps vs DAG patterns
- [ ] Artifact management (inputs/outputs, storage)
- [ ] WorkflowTemplate vs ClusterWorkflowTemplate
- [ ] Dynamic parallelism: withItems, withParam, withSequence
- [ ] Retry strategies and timeout handling
- [ ] Exit handlers for cleanup

**Argo CD:**
- [ ] Application CRD structure (source, destination, syncPolicy)
- [ ] Sync policies: manual, automated, prune, selfHeal
- [ ] Source types: Git directory, Helm chart, Kustomize
- [ ] Project-based multi-tenancy and RBAC
- [ ] App of Apps pattern
- [ ] ApplicationSet generators
- [ ] Sync waves and hooks
- [ ] Health assessment
- [ ] Compare to Flux (from CGOA)

**Argo Rollouts:**
- [ ] Canary vs Blue-Green strategies
- [ ] Traffic management: Service mesh (Istio, Linkerd), Ingress (NGINX, ALB)
- [ ] AnalysisTemplate: metrics, providers, conditions
- [ ] AnalysisRun: inline vs template-based, background vs foreground
- [ ] Manual operations: promote, abort, pause, resume
- [ ] Integration with Argo CD for GitOps

**Argo Events:**
- [ ] EventSource types (webhook, calendar, resource, AWS, Kafka, etc.)
- [ ] Sensor: dependencies, triggers, template
- [ ] EventBus: NATS architecture
- [ ] Trigger types: Workflow, K8s resource, HTTP, Slack, Lambda
- [ ] Event filtering and conditions
- [ ] Multi-source aggregation

### Tool Comparisons (Exam May Ask)

**Argo CD vs Flux:**
| Feature | Argo CD | Flux (CGOA) |
|---------|---------|-------------|
| UI | Built-in Web UI ✅ | CLI only (external UIs available) |
| CRD | Application | GitRepository + Kustomization |
| Multi-cluster | Native support ✅ | Multiple Flux installations needed |
| Image automation | Webhooks/manual | Native Image Automation Controllers ✅ |
| Multi-tenancy | Projects with RBAC ✅ | Namespace-based |
| Notification | External (Argo Events) | Built-in Notification Controller ✅ |
| Helm support | Native ✅ | HelmRelease CRD ✅ |
| Community | CNCF Incubating | CNCF Graduated ✅ |

**When to Use Argo Ecosystem:**
- Need Web UI for visibility
- Advanced deployment strategies (Rollouts)
- Event-driven workflows (Events)
- Data processing pipelines (Workflows)
- Complex DAG workflows
- Multi-cluster from single control plane

**When to Use Flux (from CGOA):**
- Prefer CLI and GitOps purity
- Need automated image updates
- Want graduated CNCF project
- Simpler architecture (fewer components)
- Built-in notifications

### Exam Strategy

**Format:**
- 90 minutes
- Multiple choice questions
- 66% passing score (lower than CKA/CKAD!)
- Online proctored

**Must Know:**
- [ ] All 4 Argo project architectures
- [ ] When to use each tool
- [ ] CRD structures and key fields
- [ ] CLI commands for each tool
- [ ] Integration patterns
- [ ] Troubleshooting approaches
- [ ] Comparison to other tools (Flux, Tekton, Jenkins)

**Common Question Patterns:**
- "What command would you use to...?"
- "Which strategy is best for scenario X?"
- "What happens when you set syncPolicy.automated.prune to true?"
- "How do you configure...?"
- "What's the difference between X and Y?"
- "Which EventSource type for scenario Z?"

**Time Management:**
- 90 minutes / ~60 questions = ~1.5 min per question
- Flag difficult questions, return later
- Read questions carefully (multiple choice can be tricky)
- Eliminate obviously wrong answers first

### Final Week Checklist

**Technical:**
- [ ] Hands-on with all 4 Argo projects
- [ ] Build complete CI/CD pipeline (Events → Workflows → CD → Rollouts)
- [ ] Practice troubleshooting: workflow failures, sync issues, rollout problems
- [ ] Review official documentation (skim, not deep read)
- [ ] Understand all CRD structures
- [ ] Know all CLI commands by heart
- [ ] Practice time management with mock questions

**Mental Prep:**
- [ ] Get good sleep before exam
- [ ] Prepare quiet exam environment
- [ ] Test webcam and internet connection
- [ ] Have government ID ready
- [ ] Clear desk (proctored exam rules)
- [ ] Stay calm - 66% is achievable! (You got 90% on CGOA!)

---

## Study Resources

### Official Documentation
- [Argo Workflows Docs](https://argoproj.github.io/argo-workflows/)
- [Argo CD Docs](https://argo-cd.readthedocs.io/)
- [Argo Rollouts Docs](https://argoproj.github.io/argo-rollouts/)
- [Argo Events Docs](https://argoproj.github.io/argo-events/)

### Training
- [LFS256: DevOps and Workflow Management with Argo](https://trainingportal.linuxfoundation.org/courses/devops-and-workflow-management-with-argo-lfs256) - **Complete this first!**

### GitHub Examples
- [Argo Workflows Examples](https://github.com/argoproj/argo-workflows/tree/master/examples)
- [Argo CD Example Apps](https://github.com/argoproj/argocd-example-apps)
- [Argo Rollouts Demo](https://github.com/argoproj/rollouts-demo)
- [Argo Events Examples](https://github.com/argoproj/argo-events/tree/master/examples)

### Community
- [Argo Slack](https://argoproj.github.io/community/join-slack)
- CNCF Slack #argo-workflows, #argo-cd, #argo-rollouts, #argo-events
- GitHub Discussions

### Lab Setup Requirements
- Kubernetes cluster (Kind, Minikube, K3s) - **You know this from CKA!**
- Install all 4 Argo projects
- Prometheus for Rollouts metrics
- Git repository (GitHub/GitLab)
- Container registry (Docker Hub, Harbor)
- CLI tools: kubectl, helm, kustomize, argo CLI, argocd CLI

---

## Progress Tracking

| Week | Topic | Status | Notes |
|------|-------|--------|-------|
| 1-2  | Argo Workflows | ⬜ | Leverage CKA CRD knowledge |
| 3    | DAGs & Data Processing | ⬜ | New patterns beyond CKAD |
| 4-5  | Argo CD | ⬜ | Compare to Flux from CGOA |
| 6    | Argo Rollouts | ⬜ | Advanced deployment strategies |
| 7    | Argo Events | ⬜ | Event-driven automation |
| 8    | Integration & Exam Prep | ⬜ | Full pipeline + review |

---

## Your Certification Journey

**Completed:**
- ✅ CKAD (September 2025) - Application development
- ✅ CKA (December 2025) - Cluster administration
- ✅ CGOA (January 2026 - 90%) - GitOps principles

**Current Focus:**
- 🎯 CAPA (Q1 2026) - Argo ecosystem

**Next in Q1 2026:**
- LFCS - Linux system administration

**Strategy Alignment:**
- **Delivery Track (Q1 2026)**: CAPA + CGOA ✅
- Strong foundation for future certs
- CAPA complements CGOA (Argo vs Flux)
- Building towards Kubestronaut! 🚀

---

**Last Updated:** January 2026  
**Target Exam Date:** End of Q1 2026 (February-March 2026)  
**Study Partner:** CGOA (Completed ✅) - Leverage Flux knowledge for comparison