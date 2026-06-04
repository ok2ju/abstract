# Web App Delivery & SRE — A Beginner's Mental Model

> **Goal:** connect the dots between what you already know (Docker, Docker Compose, nginx)
> and the words you keep hearing — *k8s, Terraform, Ansible, Helm, Atlantis, AWS, Kong,
> Istio, ArgoCD, Prometheus* — into **one clear picture**.

---

## 0. The one thing to remember

It is **not** a single chain. It is **two worlds + glue**:

| Layer | Question it answers | Tools |
|-------|---------------------|-------|
| **World 1 — Infrastructure** | *Where will my app live?* | AWS, Terraform, Ansible, Atlantis |
| **World 2 — Orchestration** | *How does it stay alive & scale?* | Kubernetes, Helm |
| **The Glue — CI/CD** | *How does my code get there?* | GitLab CI, Docker registry, GitOps |

> 🪤 **Most common beginner trap:** Terraform and Helm look alike (both are "files that describe stuff"),
> but they live in different worlds.
> **Terraform builds the cluster** (World 1). **Helm installs apps into it** (World 2).
> *Build the house* vs. *arrange the furniture.*

---

## 1. Foundation you already have

| Tool | Role |
|------|------|
| **Docker** | Package an app + all its dependencies into a portable container. Runs the same everywhere. |
| **Docker Compose** | Run several containers together **on one machine**. Perfect for local development. |
| **nginx** | Reverse proxy / load balancer / TLS termination / serve static files. Sits at the front. |

---

## 2. Where Docker Compose runs out

Compose lives on **one machine** and cannot answer these production questions:

1. A container died at 3 AM — **who restarts it?**
2. Traffic spikes — **who spreads 10 copies across machines?**
3. A machine burns down — **where does prod run now?**
4. Who created the machine, network, and database **in the first place?**

> Every "scary term" below is simply an **answer to one of these four questions.**

---

## 3. World 1 — Provisioning Infrastructure ("where it lives")

### AWS — the rented data center

You rent building blocks instead of buying physical servers.

| Block | Plain meaning |
|-------|---------------|
| **EC2** | A virtual machine (your "server") |
| **VPC** | A private network for your machines |
| **RDS** | Managed database (Postgres/MySQL) — AWS handles backups & updates |
| **S3** | Infinite disk for files (images, backups, static assets) |
| **EKS** | Managed Kubernetes — AWS runs the k8s "brain" for you |
| **IAM** | Access control: who can do what |
| **ALB / ELB** | Cloud load balancer |

### Terraform — Infrastructure as Code

- Describe the **desired** cloud resources in text files.
- `terraform plan` → shows a **diff** ("create 1 cluster, change 1 DB"). Like `git diff` for the cloud.
- `terraform apply` → makes reality match your description.
- **Declarative:** you write *what you want*, not *step-by-step how*.
- ✅ Infra now lives in **Git** → reviewable, versioned, repeatable (clone identical staging/prod).

### Ansible — Configuration Management

- Configures servers **after** they exist: install packages, write configs, start services.
- **Terraform** = "let 5 servers exist." **Ansible** = "now set Docker up on them."
- ⚠️ Its role **shrinks** in the container/k8s world (the image already contains the configured app).

### Atlantis — a GitOps gate for Terraform

- You don't run Terraform from your laptop. You **comment on a Merge Request**:

```
atlantis plan    → Atlantis runs `terraform plan`, posts the diff into the MR
                 → a teammate reviews & approves
atlantis apply   → only now the change is applied to AWS
```

- ✅ Every infra change goes through **code review**; AWS keys live only with Atlantis, not every human.

**World 1 in one line:**

```
Terraform code in Git ──(Atlantis: plan → review → apply)──▶ AWS creates VPC + EKS + RDS + S3
                                                              (Ansible fine-tunes machines if needed)
```

---

## 4. World 2 — Orchestration ("how it stays alive")

### Kubernetes (k8s) — "Docker Compose for a fleet of machines"

> **Docker Compose** manages containers on **one** machine.
> **Kubernetes** manages containers across **many** machines — and self-heals, scales, and reschedules them.

```
Cluster  (the whole Kubernetes "organism")
│
├── Control Plane ──── the "brain": keeps reality == desired state
│
├── Node 1  (one machine = one AWS EC2)
│    ├── Pod ── wraps your container (api)
│    └── Pod ── (api)
│
└── Node 2  (another machine)
     ├── Pod ── (redis)
     └── Pod ── (api)
```

| Term | What it is |
|------|------------|
| **Cluster** | The entire k8s system: brain + all machines |
| **Node** | One machine in the cluster (an EC2 instance) |
| **Pod** | Smallest unit: wraps **one** container (sometimes a few). **Disposable** by design. |
| **Deployment** | Your *desired state*: "always keep 3 live pods of api v1.2." k8s reconciles it. |
| **Service** | A **stable network name** for a group of pods + load balancing between them |
| **Ingress** | HTTP routing **from the outside world into** the cluster |

> 🔌 **Your nginx is still here!** The most popular Ingress is the **nginx-ingress-controller** —
> the familiar reverse proxy, now acting as the cluster's "gatekeeper."

### Helm — package manager for Kubernetes

- Writing k8s YAML by hand for 30 services × 3 environments = hundreds of near-identical files.
- **Chart** = a reusable, templated bundle of k8s manifests with "holes" for parameters.
- **values.yaml** = the parameters for one environment.

```
            ┌── values-staging.yaml   (replicas: 2,  image: v1.2)
One Chart ──┤
            └── values-prod.yaml      (replicas: 10, image: v1.1)
```

- Analogy: a Chart is an **npm package for your cluster**; `values.yaml` is its config.

---

## 5. The Edge — API Gateway (Kong)

**Kong = an API Gateway: a "smart front door" to your backends.** It is the Ingress gatekeeper *with a brain*.

### The idea: don't repeat cross-cutting logic in every service

Auth, rate-limiting, logging, request transforms — do them **once at the gateway**
instead of 30 times inside every microservice. Backends stay "dumb and clean."

| Gateway job | Why it helps |
|-------------|--------------|
| **Authentication** (API key / JWT / OAuth2) | Services trust that the caller is already verified |
| **Rate limiting** | Protection from overload & abuse |
| **Logging / metrics** | Centralized observability |
| **Transformations** | Bridge old & new API versions on the fly |
| **Plugins** | All of the above are pluggable — extend without touching services |

```
User ──HTTPS──▶ ┌─ Kong (API Gateway) ─┐ ──▶ Service: auth
                │ ✔ check token        │ ──▶ Service: orders
                │ ✔ rate limit         │ ──▶ Service: users
                │ ✔ log + route        │
                └──────────────────────┘
```

> 🔧 **Kong is built on nginx** (nginx + Lua plugins). It's "nginx on steroids for APIs," not a rival.
> If you understand nginx, you already get ~70% of Kong.

**Deployment:** runs inside k8s as the **Kong Ingress Controller**, installed via **Helm**,
configured declaratively (Git-friendly, same spirit as Atlantis).

*Other API gateways you may hear: AWS API Gateway, Traefik, Tyk, Apigee — same role, different vendor.*

---

## 6. Service Mesh — Istio ("traffic *between* services")

The API Gateway guards the **door** (outside ↔ cluster). But inside, your 30 microservices
also talk **to each other** constantly. That internal traffic has its own problems:

- How do services securely find & call each other?
- How do you **encrypt** every service-to-service call (mTLS)?
- How do you add retries, timeouts, and canary/traffic-splitting **without changing app code**?
- How do you **see** the call graph between services (tracing)?

Coding this into every service (in every language) is the same duplication trap Kong solved at the edge.
A **service mesh** solves it for internal traffic using the **sidecar pattern**.

### The sidecar pattern (the key idea)

A tiny proxy (**Envoy**) is injected **next to** every app container, inside the same pod.
All traffic in/out of the app goes through its sidecar. A central **control plane (Istio)**
configures every sidecar at once — so your app code does nothing special.

```
Without a mesh:                    With a mesh (sidecars):

Service A ───▶ Service B           Service A → [Envoy] ──mTLS──▶ [Envoy] → Service B
  (retries, TLS, metrics                          ▲                ▲
   coded inside each app)                          └─ Istio control plane ─┘
                                                      (configures all sidecars centrally)
```

| Part | Role |
|------|------|
| **Data plane** (Envoy sidecars) | Do the actual work — intercept & route every call |
| **Control plane** (Istio) | Tells all sidecars what to do, centrally |

**What you get for free (no app changes):** mTLS encryption, retries/timeouts/circuit-breaking,
canary & A/B traffic splitting, and automatic metrics + traces for every call.

> 🧭 **Gateway vs. Mesh — the clean split:**
> **API Gateway (Kong)** = *north-south* traffic — the **door into the house**.
> **Service Mesh (Istio)** = *east-west* traffic — the **hallways between rooms**.
> Big systems run both at once.

> ⚠️ **Trade-off:** a mesh is powerful but adds real complexity and overhead (a proxy in every pod).
> Small systems don't need one. Reach for it when you have many services and need *uniform*
> security/observability/traffic control. *Linkerd* is a lighter alternative to the heavyweight Istio.

---

## 7. The Full Chain: `git push` → user

```
┌──────────────── ONE-TIME / RARE (World 1) ─────────────────┐
│ SRE writes Terraform → Atlantis (plan→apply) → AWS builds   │
│ EKS cluster + network + DB. The cluster now waits for apps. │
└─────────────────────────────────────────────────────────────┘
                              ║  (cluster ready)
                              ▼
1. You write code (FE/BE) and push to Git
2. CI (GitLab CI) builds a Docker image
3. Image is pushed to a Docker Registry           (tag: api:v1.2)
4. CD updates Helm values: "now use api:v1.2"
5. Helm applies it to the Kubernetes cluster
6. k8s rolling-update: start new pods (v1.2), drain old ones  (no downtime)
7. Ingress / Kong routes user traffic to the new pods
8. User sees the update. Monitoring watches health.
```

> The **top block (World 1) runs rarely** — the cluster is built once and lives on.
> **Steps 1–8 run on every deploy.** That's why it all looked like chaos:
> you were seeing tools from *both rhythms* at once.

> ➡️ Modern setups **invert steps 4–6** with GitOps (next section): instead of CI *pushing*
> Helm into the cluster, an agent *inside* the cluster *pulls* the change from Git.

---

## 8. GitOps — ArgoCD / Flux ("Git is the source of truth")

You already saw **Atlantis = GitOps for infrastructure** (Terraform).
**ArgoCD / Flux = GitOps for application deployment** (k8s / Helm). Same philosophy, different target.

### Core principle

> **Git describes exactly what should be running.** A controller continuously compares
> *"what Git says"* vs *"what's actually in the cluster"* and **reconciles** the difference.

### Push vs. Pull — the mental shift

| Model | How it works | Downside |
|-------|--------------|----------|
| **Push** (classic CI/CD) | CI holds cluster credentials and runs `helm upgrade` **into** the cluster | CI can reach into prod; creds leak risk |
| **Pull** (GitOps) | An agent **inside** the cluster watches Git and **pulls** changes in | CI only commits to Git; never touches the cluster |

### Flow

```
1. CI builds image api:v1.2, pushes to registry
2. CI updates the Helm values in a Git "config repo":  image tag → v1.2
3. ArgoCD (running inside the cluster) notices Git changed
4. ArgoCD pulls & applies it → k8s rolls out v1.2
5. ArgoCD keeps watching: cluster state is forced to == Git  (self-healing)
```

| Benefit | Why it matters |
|---------|----------------|
| **Git = single source of truth** | The repo always describes live state; full audit via git history |
| **Auto-sync & self-heal** | A manual `kubectl` tweak drifts → the controller reverts it to match Git |
| **No cluster creds in CI** | More secure — CI just commits to Git |
| **Easy rollback** | `git revert` → the cluster rolls back automatically |

> 🔁 **Two GitOps gates, two scopes:**
> **Atlantis →** GitOps for **infrastructure** (Terraform).
> **ArgoCD / Flux →** GitOps for **applications** (k8s / Helm).
> In both, a Git review gates everything that reaches production.

---

## 9. SRE — the reliability discipline

SRE (Site Reliability Engineering) isn't a tool — it's the **engineering approach to reliability**.

| Term | Meaning |
|------|---------|
| **SLO** | Reliability target, e.g. "99.9% availability per month" |
| **SLI** | The metric that measures it (e.g. % successful requests) |
| **Error Budget** | Allowed failure. 99.9% ≈ 43 min/month of downtime. Out of budget → freeze features, fix reliability. |
| **Observability** | Logs, metrics, traces — seeing *inside* the system (see next section) |
| **Incident response** | Alerts, on-call rotations, postmortems |

> Motto: *"Reliability is a feature — you design for it, you don't hope for it."*

---

## 10. Observability — Prometheus + Grafana ("how SRE measures everything")

SRE can't manage reliability it can't see. **Observability = seeing inside a running system.**
It rests on **three pillars**:

| Pillar | Answers the question | Typical tool |
|--------|----------------------|--------------|
| **Metrics** | *How many / how fast / how often?* (numbers over time) | **Prometheus** |
| **Logs** | *What exactly happened in this event?* (text records) | Loki / ELK |
| **Traces** | *Where did this request spend time across services?* | Jaeger / Tempo |

### The classic stack

- **Prometheus** — collects & stores **metrics** in a time-series database.
  **Pull model:** it periodically scrapes each service's `/metrics` endpoint ("give me your numbers").
  Ships with **Alertmanager** for firing alerts.
- **Grafana** — the **dashboards & graphs** layer. Reads from Prometheus (and others) and visualizes it.
  This is the "pretty screen full of charts" you've seen on big monitors.

```
Your services + Kong + Istio ──expose /metrics──▶ Prometheus ──▶ Grafana (dashboards)
                                                       │
                                                       └──▶ Alertmanager ──▶ on-call alert 📟
```

### How it ties everything together

- Your **SLIs** (§9) are computed **from** Prometheus metrics.
- Alerts fire when metrics breach thresholds → wake up **on-call** (incident response).
- **Kong and Istio emit metrics automatically** → so adding them gives you observability for free.

> 📐 **Rule of thumb:** *Metrics tell you **something** is wrong. Logs & traces tell you **why**.*
> *OpenTelemetry* is the emerging vendor-neutral standard for emitting all three pillars.

---

## 11. What you touch as a FE/BE developer

| Zone | Tools | Owner |
|------|-------|-------|
| 🟢 **Daily** | code, `Dockerfile`, sometimes `values.yaml` (bump version / replicas) | **you** |
| 🟡 **Read & understand** | Helm chart, k8s manifests, CI/CD pipeline, Grafana dashboards | you + DevOps |
| 🔴 **Know what it is** | Terraform, Atlantis, AWS, the cluster, Istio, ArgoCD config | SRE / DevOps |

> You don't need to write Terraform or run Istio to be a great developer.
> But now "we deployed via ArgoCD to EKS, Atlantis applied Terraform for the new RDS,
> and Grafana shows the latency spike" is a sentence you fully understand.

---

## 12. Quick-reference glossary

| Term | One-liner | World |
|------|-----------|-------|
| **Docker** | Package app + deps into a container | Foundation |
| **Docker Compose** | Run multiple containers on one machine (local dev) | Foundation |
| **nginx** | Reverse proxy / load balancer / TLS | Foundation |
| **AWS** | Cloud provider — rent compute, storage, network, DBs | World 1 |
| **EC2 / VPC / RDS / S3** | VM / private network / managed DB / file storage | World 1 |
| **EKS** | AWS-managed Kubernetes | World 1 |
| **Terraform** | Infrastructure as Code — declare & create cloud resources | World 1 |
| **Ansible** | Configure servers after creation (shrinking role in k8s) | World 1 |
| **Atlantis** | GitOps gate for **infra**: run Terraform via MR comments + review | World 1 |
| **Kubernetes (k8s)** | Orchestrate containers across many machines; self-heal & scale | World 2 |
| **Cluster** | The whole k8s system (brain + nodes) | World 2 |
| **Node** | One machine in the cluster | World 2 |
| **Pod** | Smallest unit; wraps a container; disposable | World 2 |
| **Deployment** | Desired state ("keep N replicas") | World 2 |
| **Service** | Stable network name + load balancing for pods | World 2 |
| **Ingress** | HTTP routing into the cluster (often nginx-based) | World 2 |
| **Helm / Chart** | Package manager + templates for k8s manifests | World 2 |
| **Kong** | API Gateway — smart entry: auth + rate-limit + logging + routing | Edge |
| **API Gateway** | Single smart front door for incoming API traffic (north-south) | Edge |
| **Service Mesh** | Manages service↔service traffic (mTLS, retries, tracing) via sidecars | Edge |
| **Istio** | The heavyweight service mesh: control plane + Envoy sidecars (east-west) | Edge |
| **Envoy** | The sidecar proxy doing the mesh's actual work (data plane) | Edge |
| **Sidecar** | A helper container injected next to your app container in a pod | World 2 |
| **CI/CD** | Build/test code → ship it to the cluster | Glue |
| **Docker Registry** | Storage for built container images | Glue |
| **GitOps** | Git is the source of truth; a controller reconciles the cluster to match | Glue |
| **ArgoCD / Flux** | GitOps controllers for **app** deployment (pull model) | Glue |
| **Prometheus** | Metrics collection & time-series storage (pull-based) | Observability |
| **Grafana** | Dashboards & visualization on top of metrics | Observability |
| **Metrics / Logs / Traces** | The three pillars of observability | Observability |
| **OpenTelemetry** | Vendor-neutral standard for emitting telemetry | Observability |
| **SRE** | Discipline of engineering for reliability (SLO/SLI/error budget) | Cross-cutting |

---

## 13. TL;DR

> **Terraform / Atlantis / AWS build the house (the cluster). Helm arranges the furniture (your apps).
> Kubernetes is the building manager that keeps everything running and self-healing.
> Kong is the smart front door (auth, rate-limiting, logging) — really just nginx with a brain.
> SRE is the engineer responsible for it all, measured by reliability targets.**

> **Going deeper:** **Istio (service mesh)** secures and observes traffic *between* services
> (east-west) using sidecars — the gateway guards the door, the mesh runs the hallways.
> **ArgoCD / Flux** deploy apps the GitOps way: Git is the source of truth and the cluster
> *pulls* changes (Atlantis does the same for infrastructure).
> **Prometheus + Grafana** measure everything — turning raw metrics into the dashboards
> and alerts that SRE lives by.
