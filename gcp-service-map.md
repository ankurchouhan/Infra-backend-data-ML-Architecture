# GCP Service Map — Cloud-Native Streaming Platform (Architecture Blueprint)

This document outlines every **Google Cloud Platform (GCP)** service and its role within the production-grade streaming architecture, modeled after Netflix / Apple TV+.  

It acts as a *textual substitute for diagrams* (`high-level-diagram.png`, `serverless-vs-compute.png`) and captures how **serverless, compute, data, and ML layers** cooperate as one unified system.

---

## 🔹 1. High-Level Architecture Overview

### 🏗️ Conceptual Model
Think of the platform as **three stacked layers** connected by data streams and APIs:

[ Clients ]
↓
[ Edge / API Layer ]
↓
[ Application Logic (Serverless + Compute) ]
↓
[ Data & Intelligence Layer ]

markdown
Copy code

### 💡 Responsibilities by Layer
| Layer | Function | Core GCP Services | Notes |
|--------|-----------|------------------|-------|
| **Client / Edge** | First contact with users; enforces security, latency optimization | Cloud Load Balancing · Cloud Armor · Cloud CDN | Distributes traffic globally and shields backend |
| **Application / API** | Executes business logic; exposes REST / GraphQL APIs | Cloud Run · Cloud Functions · GKE · API Gateway | Mix of stateless microservices and persistent workloads |
| **Data & ML** | Stores, transforms, and learns from data | Firestore · Cloud SQL · BigQuery · Pub/Sub · Vertex AI | Drives analytics and personalization |

### 🧠 Key Idea
> Edge delivers, serverless orchestrates, compute processes, data & ML learn — together forming a continuous intelligence loop.

---

## 🔹 2. `high-level-diagram.png` (Textual Explanation)

This section narrates how requests and data traverse from **user action** to **AI-driven insight**.

1. **User Interaction** – Requests from apps hit the **Global HTTPS Load Balancer**, entering the system through an edge closest to the user.
2. **API Gateway + Cloud Armor** – Authenticate, throttle, and route traffic; protect against abuse and regional surges.
3. **Application Services** – Stateless logic runs on **Cloud Run** or **Functions**. Heavy, long-lived or GPU workloads run on **GKE / Compute Engine**.
4. **Media Pipeline** – Uploaded assets land in **Cloud Storage (Ingest Bucket)**; triggers → **Transcoder API** → encoded variants → **Origin Bucket** → served globally via **Media CDN**.
5. **Data Pipeline** – User and system events are streamed via **Pub/Sub** to **Dataflow**, normalized, then persisted in **BigQuery**.
6. **Machine Learning Layer** – **Vertex AI** consumes curated data, trains models, deploys inference endpoints; **Recommendation Service** queries them in real-time.
7. **Observability & Security** – Unified through **Cloud Logging, Monitoring, Trace, Secret Manager, IAM** — ensuring full visibility and compliance.

---

## 🔹 3. `serverless-vs-compute.png` (Workload Classification)

Serverless handles *elastic, ephemeral* operations; Compute handles *persistent or intensive* ones.

| Workload | GCP Service | Type | Architectural Reasoning |
|-----------|--------------|------|-------------------------|
| **Auth Service** | Cloud Run | Serverless | Burst-friendly, stateless login logic |
| **Catalog Service** | Cloud Run | Serverless | High read, low CPU, instant scale |
| **Notification Service** | Cloud Functions | Serverless | Trigger-based fan-out |
| **Billing API** | Cloud Run + Cloud SQL | Hybrid | ACID transactions need RDBMS |
| **Playback Service** | GKE / Compute Engine | Compute | Stateful connections, GPU/CPU tuning |
| **Transcoding Jobs** | Transcoder API / GKE Batch | Compute | Long-running media encoding |
| **Media Delivery** | Media CDN + Cloud Storage | Managed | Persistent, geo-replicated assets |
| **Analytics Ingestion** | Pub/Sub + Dataflow | Serverless | Elastic, event-driven ETL |
| **ML Training** | Vertex AI / GKE | Compute | Parallel, resource-intensive training |
| **ML Serving** | Vertex AI Endpoint | Managed | Autoscaled prediction APIs |

**Design Guideline →** Use *serverless* when latency ≤ 500 ms and no state retention; choose *compute* for throughput-bound or GPU-dependent tasks.

---

## 🔹 4. Service Roles — Full GCP Catalog with Responsibilities

| Category | Service | Primary Responsibility | Typical Usage in Platform |
|-----------|----------|-----------------------|---------------------------|
| **Networking / Edge** | Cloud Load Balancing | Global ingress & routing | Entry point for HTTPS traffic |
| | Cloud Armor | WAF + DDoS defense | Protect APIs from attacks |
| | Cloud CDN | Edge caching | Delivers media with low latency |
| **Application** | Cloud Run | Stateless microservices | Auth, Catalog, Recommendation APIs |
| | Cloud Functions | Event triggers | Start Transcoder Jobs, send notifications |
| | API Gateway | Unified routing / auth | Single entry for microservices |
| **Data Persistence** | Firestore | NoSQL profiles / metadata | User data, preferences |
| | Cloud SQL | Relational billing | Transactions, subscriptions |
| | Spanner | Global RDBMS | Enterprise-grade consistency |
| | Cloud Storage | Object store for media | Originals + encoded outputs |
| | BigQuery | Analytical warehouse | Event analytics, reporting |
| | Bigtable | Time-series logs | Low-latency activity queries |
| **Streaming & Analytics** | Pub/Sub | Message bus | Decouple services, stream events |
| | Dataflow | Stream / batch ETL | Transform and load data |
| | Looker Studio | BI front-end | Executive dashboards |
| **Machine Learning** | Vertex AI | Train + serve models | Recommendations, forecasting |
| | BigQuery ML | Inline SQL ML | Lightweight models |
| | Vertex Feature Store | Reusable features | Consistent training / serving |
| **Infra / DevOps** | Terraform | IaC automation | Provision all resources |
| | Cloud Build | CI/CD pipeline | Build and deploy containers |
| | Artifact Registry | Image repo | Versioned container storage |
| **Monitoring / Security** | Cloud Logging | Central log store | Unified visibility |
| | Cloud Monitoring | Metrics / alerts | SLO tracking |
| | Cloud Trace | Latency tracing | Root-cause analysis |
| | Secret Manager | Secure secrets | API keys, DB creds |
| | IAM | Access control | Least-privilege policies |

---

## 🔹 5. Data Flow Summary (End-to-End)

The full life-cycle of data — from login to ML-powered recommendations — unfolds as follows.

### 🎬 User Access & Authentication
- Client → Load Balancer → API Gateway → **Auth Service (Cloud Run)**  
- Verifies credentials (Firebase Auth) → persists session (Firestore).  
- Publishes `login_event` → **Pub/Sub**.

### 🎞️ Catalog Browsing
- Client → **Catalog Service (Cloud Run)** → Firestore / BigQuery.  
- Returns titles, metadata, thumbnails from Cloud Storage.  
- Emits interaction events → Pub/Sub for behavior analytics.

### ▶️ Playback & Delivery
- Client → **Playback Service (GKE)** validates subscription (Cloud SQL).  
- Generates signed URL → **Media CDN** serves HLS/DASH streams from Cloud Storage.  
- Player telemetry → **Analytics Service (Cloud Run)** → Pub/Sub.

### ☁️ Event Processing Pipeline
- Pub/Sub topics (`user_activity`, `playback_events`) → **Dataflow** → enrich + write to **BigQuery**.  
- Nightly aggregations produce `user_summary` and `title_engagement`.  
- **Looker Studio** and **BigQuery ML** consume these datasets for reports & model inputs.

### 🧠 Machine Learning Loop
- **Vertex AI Pipelines** train recommendation models from BigQuery features.  
- Models deployed → **Vertex AI Endpoints** queried by **Recommendation Service (Cloud Run)**.  
- Recommendations cached in Firestore / Redis → rendered on client home page.  
- New user interactions re-enter the loop → continuous improvement.

### 🔄 Text Flow Map
Client
↓
Load Balancer + Cloud Armor
↓
API Gateway
↓
Auth / Catalog / Playback Services
↓
Firestore · Cloud SQL · Cloud Storage
↓
Pub/Sub → Dataflow → BigQuery
↓
Vertex AI Training → Endpoint
↓
Recommendation Service → Client UI

yaml
Copy code

---

## 🔹 6. Multi-Region & Fault Tolerance Overview

- **Multi-Region Deployment** — Cloud Run, Firestore, BigQuery, and Pub/Sub span regions (`us-central1`, `europe-west1`, `asia-south1`) with global load-balancing failover.  
- **Media CDN Edge Nodes** automatically serve from nearest location.  
- **Cross-Region Backups** — Cloud Storage replication + Terraform state stored in multi-region buckets.  
- **Disaster Recovery Workflow** — Terraform modules can recreate infra from state; service images pulled from Artifact Registry.  
- **Testing** — Quarterly DR simulations validate RTO/RPO targets.

---

## 🔹 7. Key Architectural Insights

1. **Event First Design** – Everything emits events; systems react asynchronously for loose coupling.  
2. **Observability by Default** – All services log structured JSON; Cloud Monitoring aggregates SLO metrics.  
3. **Cold-Start Mitigation** – Critical Cloud Run services use min-instances > 0 to stay warm.  
4. **Security Fabric** – IAM roles scoped per service account; Secret Manager holds credentials.  
5. **Immutable Infrastructure** – Terraform ensures reproducibility; Cloud Build handles zero-downtime rollouts.  
6. **Cost Optimization** – Serverless used for spiky workloads; sustained compute discounts for GKE nodes.  
7. **Feedback Loop** – User behavior → Analytics → ML → Personalization → New behavior → Repeat.

---

## 🧠 Closing Thought

> *Architecture is not just topology — it’s a sequence of intentional decisions that make change safe and learning continuous.*  

This document evolves with the platform; append new services and regions here to keep it the single source of truth for your GCP ecosystem.

---

**Author:**  
**Ankur Chouhan** — DevOps, Cloud Architect & Solutions / Backend Engineer  



