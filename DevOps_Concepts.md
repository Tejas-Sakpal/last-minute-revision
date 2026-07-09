# DevOps Concepts — Last-Minute Interview Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Example/Config** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Covers **core DevOps concepts** — CI/CD, Docker, Kubernetes, IaC, monitoring, and culture — at the level asked of data/software engineers and junior–mid DevOps roles. Built for a 2–4 hour final revision.

---

## Table of Contents

1. [What is DevOps? (Culture & Principles)](#1-what-is-devops)
2. [The DevOps Lifecycle & CALMS/DORA](#2-the-devops-lifecycle--calmsdora)
3. [Version Control (Git) in DevOps](#3-version-control-git-in-devops)
4. [CI/CD — Continuous Integration & Delivery/Deployment](#4-cicd)
5. [Build Tools & Artifacts](#5-build-tools--artifacts)
6. [Containers & Docker](#6-containers--docker)
7. [Container Orchestration & Kubernetes](#7-container-orchestration--kubernetes)
8. [Infrastructure as Code (IaC)](#8-infrastructure-as-code-iac)
9. [Configuration Management](#9-configuration-management)
10. [Monitoring, Logging & Observability](#10-monitoring-logging--observability)
11. [Cloud & Networking Basics](#11-cloud--networking-basics)
12. [Deployment Strategies](#12-deployment-strategies)
13. [Security in DevOps (DevSecOps)](#13-security-in-devops-devsecops)
14. [Common Scenarios (with answers)](#14-common-scenarios)
15. [Tricky / Gotcha Questions](#15-tricky--gotcha-questions)
16. [Rapid-Fire One-Liners](#16-rapid-fire-one-liners)
17. [Last-Minute Checklist](#17-last-minute-checklist)

---

## 1. What is DevOps?

### Theory

**DevOps** is a **culture, set of practices, and tools** that unifies software **Development (Dev)** and **IT Operations (Ops)** to shorten the delivery lifecycle and deliver software **faster, more reliably, and continuously**. It's not a single tool or role — it's a way of working.

**Core goals:** break down silos between Dev and Ops, **automate** everything repeatable, get **fast feedback**, and ship **small changes frequently** with high quality.

**Key principles:**
- **Collaboration** — shared ownership between Dev and Ops.
- **Automation** — build, test, deploy, infrastructure.
- **Continuous Improvement** — measure, learn, iterate.
- **Customer-centric, fast feedback loops.**
- **Shift left** — test and secure earlier in the lifecycle.

**Why it matters:** faster releases, fewer failures, quicker recovery, and better collaboration.

### Interview Q&A

**Q: What is DevOps?**
A **culture and set of practices** that bring **development and operations together** to deliver software **faster and more reliably** through **automation, collaboration, continuous integration/delivery, and feedback loops**. It's about breaking down silos, not just using tools.

**Q: Why do organizations adopt DevOps / what are the benefits?**
**Faster time to market**, **more frequent and reliable releases**, **faster recovery** from failures, **better collaboration**, and **higher quality** through automation and early testing. It reduces the friction between writing code and running it in production.

**Q: Is DevOps a tool, a role, or a culture?**
Primarily a **culture and methodology** — supported by tools and practices. Tools (Jenkins, Docker, Kubernetes) enable it, but the core is **shared ownership, automation, and continuous improvement**.

**Q: What does "shift left" mean?**
Moving activities like **testing and security earlier** ("left") in the development lifecycle, so issues are caught **sooner and cheaper** rather than in production.

---

## 2. The DevOps Lifecycle & CALMS/DORA

### Theory

**The DevOps lifecycle** (often drawn as an infinite loop): **Plan → Code → Build → Test → Release → Deploy → Operate → Monitor** → (feedback back to Plan).

**CALMS** — a framework for assessing DevOps maturity:
- **C**ulture, **A**utomation, **L**ean, **M**easurement, **S**haring.

**DORA metrics** — the four key metrics for measuring DevOps performance:
1. **Deployment Frequency** — how often you deploy.
2. **Lead Time for Changes** — commit → production time.
3. **Change Failure Rate** — % of deployments causing failures.
4. **MTTR (Mean Time to Recovery)** — how fast you recover from failures.

### Interview Q&A

**Q: Walk me through the DevOps lifecycle.**
**Plan → Code → Build → Test → Release → Deploy → Operate → Monitor**, then feedback loops back to planning. It's continuous and iterative — each stage is automated and monitored so improvements flow back in.

**Q: How do you measure DevOps success?**
Using **DORA metrics**: **deployment frequency**, **lead time for changes**, **change failure rate**, and **mean time to recovery (MTTR)**. High performers deploy often, with short lead times, low failure rates, and fast recovery.

**Q: What is CALMS?**
A maturity model: **Culture, Automation, Lean, Measurement, Sharing** — the five pillars that indicate how well an organization has adopted DevOps.

---

## 3. Version Control (Git) in DevOps

### Theory

**Version control** is the foundation of DevOps — all code (and increasingly **infrastructure and config**) lives in Git. It enables collaboration, history, rollback, and triggers CI/CD pipelines.

- **Branching strategies:** feature-branch + PR, Gitflow, **trunk-based development** (favored for CI/CD).
- **Pull/Merge Requests** — the gate for code review + automated checks.
- **GitOps** — using Git as the **single source of truth** for declarative infrastructure and deployments (changes via PRs, auto-synced by tools like Argo CD/Flux).

### Interview Q&A

**Q: Why is version control central to DevOps?**
Because **everything-as-code** (app code, infrastructure, config, pipelines) lives in Git — enabling **collaboration, history, rollback, peer review, and automated triggers** for CI/CD. It's the single source of truth.

**Q: What is GitOps?**
A practice where **Git is the source of truth** for declarative infrastructure and application state. You change infra/deployments via **Git commits/PRs**, and an operator (Argo CD, Flux) **automatically syncs** the cluster to match Git — giving auditability, easy rollback, and consistency.

**Q: What branching strategy suits CI/CD?**
**Trunk-based development** (short-lived branches, frequent merges to main behind feature flags) or a **lightweight feature-branch + PR** flow — both keep integration continuous and avoid long-lived divergent branches.

---

## 4. CI/CD

### Theory

CI/CD is the **backbone of DevOps** — automating the path from code commit to production.

- **CI (Continuous Integration)** — developers **merge code frequently**; each merge triggers an automated pipeline: **checkout → build → unit tests → static analysis**. Goal: catch integration issues early.
- **CD (Continuous Delivery)** — every change that passes CI is **automatically prepared and deployable** to production, but the final **production deploy is a manual approval**.
- **CD (Continuous Deployment)** — goes one step further: **every passing change auto-deploys to production** with no manual gate.

**Typical pipeline stages:** Source → Build → Test → (Security scan) → Deploy to staging → Integration tests → Deploy to production.

**Tools:** Jenkins (Jenkinsfile), GitHub Actions, GitLab CI, Azure DevOps Pipelines, CircleCI, Argo CD.

### Example (a simple pipeline)

```yaml
# GitHub Actions example
name: CI
on: [push]
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: pytest                 # unit tests
      - run: flake8 .               # static analysis
```

### Interview Q&A

**Q: What is CI/CD?**
**CI (Continuous Integration)** automatically **builds and tests** every code change merged to the repo, catching issues early. **CD** is **Continuous Delivery** (changes are always **deployable**, deploy on approval) or **Continuous Deployment** (every passing change **auto-deploys** to production). Together they automate code → production.

**Q: Continuous Delivery vs Continuous Deployment?**
Both automate up to production-readiness. **Continuous Delivery** stops at a **manual approval** before the production deploy. **Continuous Deployment** has **no manual gate** — every change that passes the pipeline goes live automatically.

**Q: What are the typical stages of a CI/CD pipeline?**
**Source (checkout) → Build → Unit tests → Static code/security analysis → Deploy to staging → Integration tests → Deploy to production**, with **rollback** capability. Each stage gates the next.

**Q: What is a Jenkinsfile?**
A text file (in the repo) that **defines a Jenkins pipeline as code** — its stages, steps, and configuration — so the pipeline is version-controlled and reproducible.

**Q: What makes a good pipeline?**
**Fast feedback** (quick tests first), **fail-fast**, **idempotent/repeatable**, **automated tests and security scans**, **artifact versioning**, **environment parity**, and **easy rollback**.

---

## 5. Build Tools & Artifacts

### Theory

- **Build** — compiling/packaging source into a runnable **artifact** (JAR, wheel, Docker image, binary).
- **Artifact** — the versioned, deployable output of a build.
- **Artifact repository** — stores build outputs: **JFrog Artifactory, Nexus, Docker registries (Docker Hub, ACR, ECR, GCR)**.
- **Build tools:** Maven/Gradle (Java), npm (JS), pip/Poetry (Python), Make.

**Key principle: build once, deploy everywhere** — produce a single immutable artifact and promote the *same* artifact through environments (dev → staging → prod), rather than rebuilding per environment.

### Interview Q&A

**Q: What is an artifact and why store it in a repository?**
An **artifact** is the **deployable output of a build** (image, JAR, package). An **artifact repository** stores it **versioned and immutable**, so the exact same tested build is promoted across environments — ensuring consistency and traceability.

**Q: What does "build once, deploy everywhere" mean?**
Build a **single immutable artifact once**, then **promote that same artifact** through dev → staging → prod. This guarantees what you tested is exactly what you ship — avoiding environment-specific build drift.

---

## 6. Containers & Docker

### Theory

**Containers** package an application with **all its dependencies** into a single, portable, isolated unit that runs consistently across environments — solving "it works on my machine."

**Docker** is the most popular containerization platform.

- **Image** — a read-only **blueprint/template** (app + dependencies + config), built from a **Dockerfile** in layers.
- **Container** — a **running instance** of an image.
- **Dockerfile** — instructions to build an image.
- **Registry** — stores/shares images (Docker Hub, ACR, ECR).
- **Docker Compose** — define and run **multi-container** apps via a YAML file.

**Container vs VM:** Containers share the **host OS kernel** and virtualize at the OS level — they're **lightweight, fast to start, and dense**. VMs include a **full guest OS** via a hypervisor — heavier, slower, more isolated.

### Example (Dockerfile)

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

```bash
docker build -t myapp:1.0 .       # build image
docker run -d -p 8000:8000 myapp:1.0   # run container
docker ps                         # list running containers
docker push registry/myapp:1.0    # push to registry
```

### Interview Q&A

**Q: What is Docker and how is it different from a VM?**
**Docker** packages an app and its dependencies into a **container** that runs consistently anywhere. Unlike a **VM** (which bundles a **full guest OS** on a hypervisor), containers **share the host OS kernel** and isolate at the OS level — so they're **far lighter, start in seconds, and pack more densely**, at the cost of weaker isolation than VMs.

**Q: Image vs container?**
An **image** is a **read-only blueprint** (app + dependencies), built from a Dockerfile. A **container** is a **running instance** of that image. One image → many containers.

**Q: What is a Dockerfile?**
A script of instructions (`FROM`, `COPY`, `RUN`, `CMD`, etc.) that **builds a Docker image** in **layers** — version-controlled and reproducible.

**Q: Why are Docker layers important?**
Each Dockerfile instruction creates a **cached layer**. Reusing unchanged layers makes builds **faster** and images **smaller**. Ordering matters — put rarely-changing steps (dependency install) before frequently-changing ones (copying code) to maximize cache hits.

**Q: What is Docker Compose?**
A tool to **define and run multi-container applications** with a single YAML file (`docker-compose.yml`) — e.g., app + database + cache — and start them all with `docker compose up`.

**Q: How do you reduce Docker image size?**
Use a **slim/alpine base image**, **multi-stage builds** (build in one stage, copy only artifacts to a clean final stage), combine `RUN` commands, clean caches, and use a **`.dockerignore`** to exclude unneeded files.

---

## 7. Container Orchestration & Kubernetes

### Theory

When you run **many containers across many machines**, you need **orchestration** — automating deployment, scaling, networking, and healing. **Kubernetes (K8s)** is the standard.

**Core objects:**
- **Pod** — the **smallest deployable unit**; one or more containers sharing network/storage.
- **ReplicaSet** — maintains a desired number of pod replicas.
- **Deployment** — manages ReplicaSets; enables **rolling updates and rollbacks**.
- **Service** — stable network endpoint to reach a set of pods (load-balances). Types: ClusterIP, NodePort, LoadBalancer.
- **Ingress** — HTTP routing into the cluster.
- **ConfigMap / Secret** — externalized config and sensitive data.
- **Namespace** — logical partition of cluster resources.

**Architecture:** **Control plane** (API server, scheduler, controller manager, **etcd** key-value store) + **worker nodes** (kubelet, kube-proxy, container runtime).

**Self-healing & desired state:** you declare the **desired state**; Kubernetes continuously **reconciles** actual to desired (restarting failed pods, rescheduling, scaling).

### Example (Deployment YAML)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels: { app: myapp }
  template:
    metadata:
      labels: { app: myapp }
    spec:
      containers:
        - name: myapp
          image: registry/myapp:1.0
          ports: [{ containerPort: 8000 }]
```

### Interview Q&A

**Q: What is Kubernetes and what problem does it solve?**
**Kubernetes** is a **container orchestration** platform that automates **deploying, scaling, networking, and self-healing** of containerized applications across a cluster of machines. It solves the problem of managing **many containers in production** reliably.

**Q: What is a Pod?**
The **smallest deployable unit** in Kubernetes — one or more **tightly coupled containers** that share the same network namespace and storage. You usually run one main container per pod.

**Q: Deployment vs Pod vs ReplicaSet?**
A **Pod** runs containers. A **ReplicaSet** keeps a desired **number of identical pods** running. A **Deployment** manages ReplicaSets and adds **rolling updates, rollbacks, and version control** — you almost always create Deployments, not bare pods.

**Q: What is a Service and why is it needed?**
Pods are **ephemeral** (IPs change as they restart). A **Service** provides a **stable virtual IP/DNS name** and **load-balances** traffic across the matching pods, so clients have a reliable endpoint.

**Q: ConfigMap vs Secret?**
Both externalize configuration from images. **ConfigMap** holds **non-sensitive** config (env vars, files). **Secret** holds **sensitive** data (passwords, tokens, keys), base64-encoded and handled more carefully (and can be encrypted at rest).

**Q: How does Kubernetes self-heal?**
It works on **desired state** — you declare what you want (e.g., 3 replicas), and the **controllers continuously reconcile** actual vs desired: restarting crashed pods, rescheduling pods off dead nodes, and maintaining replica counts automatically.

**Q: What is etcd?**
A distributed **key-value store** that holds the **entire cluster state** (the source of truth). The API server reads/writes cluster state to etcd; losing it means losing cluster state, so it's backed up carefully.

---

## 8. Infrastructure as Code (IaC)

### Theory

**Infrastructure as Code (IaC)** is managing and provisioning infrastructure (servers, networks, databases) through **machine-readable definition files** instead of manual setup. Infrastructure becomes **version-controlled, repeatable, and automated**.

- **Declarative** (what you want — Terraform, CloudFormation, ARM/Bicep) vs **Imperative** (step-by-step — scripts).
- **Idempotent** — applying the same config repeatedly yields the same result.
- **Terraform** — the popular **cloud-agnostic** IaC tool; uses **state** to track managed resources; workflow: `init → plan → apply`.

**Benefits:** consistency, repeatability, versioning, faster provisioning, disaster recovery, no config drift.

### Example (Terraform)

```hcl
resource "azurerm_storage_account" "data" {
  name                     = "mydatalake"
  resource_group_name      = "rg-data"
  location                 = "eastus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

```bash
terraform init      # initialize providers
terraform plan      # preview changes
terraform apply     # provision
terraform destroy   # tear down
```

### Interview Q&A

**Q: What is Infrastructure as Code and why use it?**
**IaC** manages infrastructure through **code/config files** instead of manual clicks. Benefits: **repeatable, version-controlled, consistent** provisioning; no **config drift**; easy disaster recovery; and infrastructure changes go through the **same review/CI process** as app code.

**Q: Declarative vs imperative IaC?**
**Declarative** (Terraform, CloudFormation) — you describe the **desired end state** and the tool figures out how to reach it. **Imperative** — you write the **step-by-step commands**. Declarative is preferred because it's idempotent and self-documenting.

**Q: What is Terraform state and why does it matter?**
Terraform keeps a **state file** mapping your config to **real provisioned resources**. It uses state to know what exists and compute the **diff** on `plan`/`apply`. State must be **stored securely and shared** (e.g., remote backend with locking) so teams don't corrupt it or drift.

**Q: What does idempotent mean in IaC?**
Applying the **same configuration multiple times produces the same result** — no duplicate resources, no unintended changes. This makes runs safe and predictable.

---

## 9. Configuration Management

### Theory

**Configuration management** automates the setup and maintenance of servers/software to a **consistent, desired state**. Tools: **Ansible** (agentless, push, YAML playbooks), **Chef/Puppet** (agent-based, pull), **SaltStack**.

- **Ansible** — agentless (uses SSH), **declarative playbooks** in YAML, idempotent tasks.
- Distinct from IaC provisioning: IaC **creates** infrastructure; config management **configures** what's on it (though they overlap).

### Interview Q&A

**Q: What is configuration management?**
Automating the **configuration of servers/software to a consistent desired state** — installing packages, setting files, managing services — so all machines are identical and reproducible. Tools include **Ansible, Puppet, Chef**.

**Q: Ansible vs Terraform?**
**Terraform** is for **provisioning infrastructure** (declarative, creates cloud resources). **Ansible** is for **configuration management** (setting up software/config on existing machines), though it can do some provisioning. They're often used **together** — Terraform builds the servers, Ansible configures them.

**Q: Why is Ansible "agentless" an advantage?**
It connects over **SSH** with no software to install on managed nodes — simpler setup, less maintenance, and nothing extra to secure on each host.

---

## 10. Monitoring, Logging & Observability

### Theory

You can't operate what you can't see. **Observability** = understanding system internal state from its outputs, built on **three pillars**:

1. **Metrics** — numeric time-series (CPU, latency, request rate). Tools: **Prometheus** (collect/store), **Grafana** (visualize).
2. **Logs** — discrete event records. Tools: **ELK/Elastic Stack** (Elasticsearch, Logstash, Kibana), Loki, Splunk.
3. **Traces** — request paths across distributed services. Tools: Jaeger, OpenTelemetry, Zipkin.

- **Monitoring vs observability:** monitoring tells you **when something is wrong** (known issues, alerts); observability helps you **explore why** (including unknown issues).
- **Alerting** — notify on thresholds/anomalies (Alertmanager, PagerDuty).
- **SLI / SLO / SLA** — Indicator (measured), Objective (target), Agreement (promise).

### Interview Q&A

**Q: What are the three pillars of observability?**
**Metrics** (numeric time-series like CPU/latency), **Logs** (discrete event records), and **Traces** (the path of a request across services). Together they let you understand system health and debug issues.

**Q: Monitoring vs observability?**
**Monitoring** watches for **known problems** and alerts (is the system up? is latency high?). **Observability** is the broader ability to **ask arbitrary questions** about system behavior — including unforeseen issues — using metrics, logs, and traces.

**Q: What's a common monitoring stack?**
**Prometheus + Grafana** for metrics and dashboards, and the **ELK stack** (Elasticsearch, Logstash, Kibana) for logs. Prometheus scrapes metrics; Grafana visualizes; Alertmanager/PagerDuty handle alerts.

**Q: SLI vs SLO vs SLA?**
**SLI** = a **measured indicator** (e.g., 99.95% of requests succeed). **SLO** = the **internal target** (e.g., 99.9% success). **SLA** = the **external contractual promise** to customers (with consequences if breached).

---

## 11. Cloud & Networking Basics

### Theory

DevOps is largely cloud-based. Know the basics:

- **Cloud models:** **IaaS** (VMs/networking — you manage the OS up), **PaaS** (managed platform — you manage the app), **SaaS** (fully managed software).
- **Major providers:** AWS, Azure, GCP.
- **Key concepts:** **regions/availability zones**, **autoscaling**, **load balancers**, **VPC/VNet** (private networks), **IAM** (identity & access), **managed services** (databases, queues).
- **Networking basics:** DNS, HTTP(S)/ports, firewalls/security groups, public vs private subnets, reverse proxy/load balancer.

### Interview Q&A

**Q: IaaS vs PaaS vs SaaS?**
**IaaS** gives you raw infrastructure (VMs, storage, networking) — you manage the OS and up. **PaaS** gives a managed platform — you just deploy your app. **SaaS** is fully managed software you just use. More managed = less control but less ops burden.

**Q: What is a load balancer?**
A component that **distributes incoming traffic across multiple servers/instances** to improve **availability, scalability, and reliability** — and to route around unhealthy instances.

**Q: What is autoscaling?**
Automatically **adding or removing compute** based on load (CPU, requests, queue depth), so you handle traffic spikes without over-provisioning — balancing **performance and cost**.

**Q: What is IAM?**
**Identity and Access Management** — controls **who can do what** on cloud resources via users, roles, and policies, following the **least-privilege** principle.

---

## 12. Deployment Strategies

### Theory

How you roll out new versions safely:

- **Recreate** — stop old, start new (downtime).
- **Rolling update** — gradually replace instances (default in Kubernetes); no downtime, but mixed versions during rollout.
- **Blue/Green** — run two identical environments (blue = current, green = new); **switch traffic** to green once verified; instant **rollback** by switching back.
- **Canary** — release to a **small % of users** first, monitor, then gradually expand. Limits blast radius.
- **Feature flags** — toggle features on/off at runtime without redeploying.

### Interview Q&A

**Q: Blue/green vs canary deployment?**
**Blue/green** runs **two full environments** and switches all traffic from old (blue) to new (green) at once — instant rollback by switching back, but needs double resources. **Canary** gradually shifts a **small percentage** of traffic to the new version, monitoring before increasing — lower risk and resource cost, but slower rollout.

**Q: What is a rolling update?**
Gradually replacing old pods/instances with new ones **a few at a time**, keeping the service available throughout. It's Kubernetes' default Deployment strategy; rollback re-deploys the previous version.

**Q: How do you roll back a bad deployment?**
With **blue/green**, switch traffic back to the old environment instantly. In **Kubernetes**, `kubectl rollout undo` reverts to the previous ReplicaSet. The key is having a **fast, tested rollback path** and good monitoring to detect the problem quickly.

**Q: What are feature flags?**
Runtime **toggles** that enable/disable features without a redeploy — allowing you to **ship code dark**, do gradual rollouts, A/B test, and instantly **turn off** a problematic feature.

---

## 13. Security in DevOps (DevSecOps)

### Theory

**DevSecOps** integrates **security into every stage** of the DevOps pipeline ("shift security left") rather than bolting it on at the end.

**Key practices:**
- **Secrets management** — never hardcode credentials; use **Vault, cloud Key Vaults, K8s Secrets**.
- **Scanning** — **SAST** (static code analysis), **DAST** (running-app scanning), **dependency/SCA** scanning, **container image scanning** (Trivy, Clair).
- **Least privilege & RBAC** — minimal access for users and services.
- **Policy as code** — enforce security rules automatically (OPA).
- **Immutable infrastructure** — replace rather than patch in place.

### Interview Q&A

**Q: What is DevSecOps?**
Integrating **security into the DevOps pipeline from the start** ("shift left") — automated security scanning, secrets management, and policy checks at every stage — instead of treating security as a final gate. Security becomes a shared responsibility.

**Q: How do you manage secrets in a pipeline?**
Never hardcode them — store secrets in a **secrets manager** (HashiCorp Vault, Azure Key Vault, AWS Secrets Manager) or **Kubernetes Secrets**, inject them at runtime, rotate regularly, and keep them out of code and logs.

**Q: What security scans belong in CI/CD?**
**SAST** (static analysis of source), **SCA/dependency scanning** (vulnerable libraries), **container image scanning**, **secret detection**, and **DAST** (testing the running app). Failing these can block the pipeline.

**Q: What is immutable infrastructure?**
Instead of patching servers in place, you **replace them with new, freshly built instances** from a known-good image. This eliminates configuration drift and makes deployments predictable and easily reversible.

---

## 14. Common Scenarios

**Q: A deployment to production broke things. How do you respond?**
"**Roll back first** (blue/green switch or `kubectl rollout undo`) to restore service, then investigate. I'd check **monitoring/logs** to find the cause, fix it, add a **test** to catch it, and improve the pipeline (e.g., better staging tests or a canary) so it can't recur. Communicate status throughout and do a blameless **post-mortem**."

**Q: How would you design a CI/CD pipeline for a microservice?**
"On each commit: **checkout → build → unit tests → static + security scan → build Docker image → push to registry → deploy to staging → integration tests → (approval) → deploy to prod** via rolling/canary, with **automated rollback** on failure and **monitoring/alerting** post-deploy. Everything **as code** in the repo."

**Q: Your container works locally but fails in Kubernetes. How do you debug?**
"Check `kubectl get pods` and `kubectl describe pod` for events (image pull errors, crash loops, resource limits), then `kubectl logs` for app errors. Common causes: **missing env/config (ConfigMap/Secret), wrong image tag, resource limits, failing health probes, or networking**. Reproduce with the same image/config locally."

**Q: How do you handle configuration that differs per environment?**
"**Externalize config** from the image — use **environment variables, ConfigMaps/Secrets**, or parameterized IaC. The **same artifact** runs everywhere; only the injected config changes per environment (dev/staging/prod). Secrets come from a vault, never the image."

**Q: How do you achieve zero-downtime deployments?**
"Use **rolling updates** (or blue/green), with **readiness/liveness probes** so traffic only goes to healthy pods, plus a **load balancer**. New version comes up and is verified before old instances are removed; rollback is instant if health checks fail."

---

## 15. Tricky / Gotcha Questions

**1. DevOps is a culture, not a tool/role** — lead with collaboration + automation, not "Jenkins."

**2. Container ≠ VM** — containers share the host kernel (lightweight); VMs run a full guest OS.

**3. Image vs container** — image = blueprint (static); container = running instance.

**4. Continuous Delivery has a manual prod gate; Continuous Deployment doesn't.**

**5. Pods are ephemeral** — never rely on a pod's IP; use a Service.

**6. Don't deploy bare Pods** — use Deployments for rolling updates/rollback.

**7. ConfigMap = non-sensitive; Secret = sensitive** (and Secrets are only base64, not encrypted by default — enable encryption at rest).

**8. Declarative IaC is idempotent** — re-applying doesn't duplicate resources.

**9. Terraform state is critical** — store remotely with locking; never edit by hand.

**10. "Build once, deploy everywhere"** — don't rebuild per environment; promote the same artifact.

**11. Monitoring (known issues) vs observability (explore unknowns).**

**12. Blue/green = instant switch + double resources; canary = gradual + lower risk.**

---

## 16. Rapid-Fire One-Liners

- **DevOps** = culture + automation uniting Dev and Ops for fast, reliable delivery.
- **Lifecycle:** Plan → Code → Build → Test → Release → Deploy → Operate → Monitor.
- **DORA metrics:** deploy frequency, lead time, change failure rate, MTTR.
- **CI** = auto build + test on every merge; **CD** = always deployable / auto-deploy.
- **Pipeline:** source → build → test → scan → staging → integration → prod (+ rollback).
- **Artifact** = versioned build output; build once, deploy everywhere.
- **Container** shares host kernel (light); **VM** = full guest OS (heavy).
- **Image** = blueprint; **container** = running instance; **Dockerfile** builds it in layers.
- **Kubernetes** = container orchestration (deploy, scale, heal).
- **Pod** = smallest unit; **Deployment** manages replicas + rolling updates; **Service** = stable endpoint.
- **ConfigMap** (non-secret) vs **Secret** (sensitive); **etcd** = cluster state store.
- **Self-healing** = reconcile actual to declared desired state.
- **IaC** = infrastructure via code; declarative + idempotent (Terraform: init→plan→apply).
- **Config mgmt** (Ansible) configures servers; IaC (Terraform) provisions them.
- **Observability** = metrics + logs + traces (Prometheus/Grafana + ELK + Jaeger).
- **Monitoring** = known issues; **observability** = explore unknowns.
- **SLI** (measured) < **SLO** (target) < **SLA** (promise).
- **IaaS/PaaS/SaaS** = increasing levels of managed service.
- **Deploy strategies:** rolling, blue/green (instant switch), canary (gradual), feature flags.
- **DevSecOps** = shift security left; secrets in a vault; SAST/DAST/image scans.
- **GitOps** = Git as source of truth, auto-synced to the cluster.

---

## 17. Last-Minute Checklist

The hour before:

- [ ] DevOps = culture + automation; benefits; shift-left.
- [ ] Lifecycle + **DORA metrics** + CALMS.
- [ ] **CI vs CD (delivery vs deployment)** and pipeline stages.
- [ ] Artifacts + "build once, deploy everywhere."
- [ ] **Docker:** image vs container, Dockerfile/layers, container vs VM, Compose, image-size tips.
- [ ] **Kubernetes:** Pod, Deployment, ReplicaSet, Service, ConfigMap/Secret, self-healing, etcd, control plane vs workers.
- [ ] **IaC:** declarative vs imperative, idempotency, Terraform state (init/plan/apply).
- [ ] **Config management** (Ansible) vs provisioning (Terraform).
- [ ] **Observability:** metrics/logs/traces; Prometheus+Grafana, ELK; SLI/SLO/SLA.
- [ ] **Cloud basics:** IaaS/PaaS/SaaS, load balancer, autoscaling, IAM.
- [ ] **Deployment strategies:** rolling, blue/green, canary, feature flags; rollback.
- [ ] **DevSecOps:** secrets management, SAST/DAST/SCA, least privilege.
- [ ] Practice the **scenario answers** in §14 (broken deploy, debug pod, design a pipeline).

**Interview tips:** For basics, always frame DevOps as **culture + automation**, not just tools. For Docker/K8s, nail **image vs container** and **Pod/Deployment/Service**. For any "design a pipeline" or "deployment broke" scenario, structure your answer around **automation, testing, monitoring, and fast rollback**. Tie answers to real experience — you've used **Docker, Git, CI/CD, and Jira** — so describe an actual pipeline you worked with to make it concrete.

---

*Good luck — read it top-to-bottom once, then drill CI/CD, Docker (image vs container, vs VM), Kubernetes basics (Pod/Deployment/Service), and deployment strategies. Those are the most-asked DevOps fundamentals.*
