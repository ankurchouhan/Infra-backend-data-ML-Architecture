# 🎬 Cloud-Native Streaming Platform (Netflix / Apple TV+ Style)

This project demonstrates a **scalable, production-grade streaming architecture** built on **Google Cloud Platform (GCP)** using a mix of **serverless and compute services**.

## 🚀 Overview

The system simulates a video-on-demand (VOD) platform — similar to Netflix or Apple TV+ — with:
- User authentication and profiles
- Video catalog and metadata APIs
- Media upload, transcoding, and delivery via CDN
- Real-time analytics and recommendations

## 🧩 Architecture Diagram
streaming-platform-gcp-architecture/
│
├── README.md
├── architecture/
│   ├── high-level-diagram.png
│   ├── serverless-vs-compute.png
│   └── gcp-service-map.md
│
├── backend/
│   ├── auth-service/
│   ├── catalog-service/
│   ├── playback-service/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── infra/
│   ├── terraform/
│   ├── gcp-setup.md
│   └── ci-cd-pipeline.yaml
│
├── data/
│   ├── firestore-schema.json
│   ├── pubsub-topics.yaml
│   └── bigquery-dataset.sql
│
└── notebooks/
    ├── recommendation_model.ipynb
    └── analytics_demo.ipynb

![architecture-diagram](architecture/high-level-diagram.png)

### Core Components

| Layer | GCP Service | Purpose |
|--------|--------------|----------|
| Auth / API | Cloud Run | Stateless microservices |
| Metadata | Firestore | NoSQL metadata store |
| Billing | Cloud SQL | Relational transactions |
| Transcoding | Transcoder API | Media processing |
| Delivery | Media CDN + Cloud Storage | Global streaming |
| Analytics | Pub/Sub + BigQuery | Event pipeline |
| Recommendations | Vertex AI | ML-driven personalization |

## 🏗️ Infrastructure

- **Terraform** for IaC  
- **Cloud Build** for CI/CD  
- **Artifact Registry** for containers  
- **Cloud Logging / Monitoring** for observability  
- **IAM / Secrets Manager** for security

## 🧠 Key Design Principles

- Event-driven architecture (Pub/Sub)
- Serverless for stateless microservices
- Compute Engine / GKE for stateful heavy workloads
- Multi-region failover design
- Data-driven personalization (Vertex AI)

## 📊 Data Flow

1. User logs in → Cloud Run Auth Service → Firestore
2. User starts playback → Playback Service → signed Media CDN URL
3. Player emits events → Pub/Sub → BigQuery
4. ML model in Vertex AI updates recommendations

## 🧰 Tech Stack

- GCP (Cloud Run, Firestore, BigQuery, Pub/Sub, Media CDN, Transcoder API)
- Python / Go / Node.js (backend)
- React / Next.js (frontend)
- Terraform / Cloud Build (infra + CI/CD)
- Vertex AI (machine learning)

## 🧑‍💻 Author

*Ankur Chouhan* — Cloud Architect / Backend Engineer  
📫 *[LinkedIn / Website / Email]*  
