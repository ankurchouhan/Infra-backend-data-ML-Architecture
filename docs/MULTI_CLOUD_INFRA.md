# 🏗️ Infrastructure as Code (IaC)

## Overview

All infrastructure is provisioned using **Terraform**, with reusable modules for **AWS**, **GCP**, and **Azure**.

This approach enables:
- Consistent resource definitions
- Cross-cloud version control
- Automated environment setup (dev/staging/prod)

---

## Structure

```bash
infra/
├─ terraform/
│  ├─ envs/
│  │  ├─ aws-dev/
│  │  ├─ gcp-staging/
│  │  └─ azure-prod/
│  └─ modules/
│     ├─ aws/
│     ├─ gcp/
│     └─ azure/
