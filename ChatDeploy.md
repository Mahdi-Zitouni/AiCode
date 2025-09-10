# On-Premise Chatbot Deployment Plan

## 1. Physical Server (Recommended Spec)

- **GPU**: 2× NVIDIA L40S (48 GB each)
- **CPU**: 32–48 cores
- **RAM**: 256 GB
- **Storage**: 4–6 TB NVMe SSD (local)
- **Object Storage**: 10 TB+ (NAS/S3-compatible for logs/backups)
- **Network**: Dual 25 GbE NICs (10 GbE min)
- **OS (Host)**: Proxmox VE

## 2. Proxmox VM Resource Dispatch

| VM | Purpose | vCPU | RAM | GPU | Storage | Notes |
|---|---|---|---|---|---|---|
| **VM1 – AI Chatbot** | Qwen 3 8B + Jina + Backend + Frontend + DB + Vector DB | 16 vCPU | 128 GB | 1× L40S (48 GB) | 2 TB NVMe | Main chatbot system |
| **VM2 – Broadcasting** | Streaming / message broadcasting service | 8 vCPU | 32 GB | – | 500 GB | Handles outbound messaging |
| **VM3 – Automation** | Cron jobs, monitoring, scripts | 8 vCPU | 32 GB | – | 500 GB | Lightweight automation tasks |

*(Host retains remaining cores/RAM for Proxmox overhead + future scaling)*

## 3. Deployment Steps – AI Chatbot VM

### Provision VM1 in Proxmox
- Ubuntu 22.04 LTS
- Attach GPU passthrough (1× L40S)
- Assign 16 vCPU, 128 GB RAM, 2 TB NVMe

### Install Dependencies
- Update system (`apt update && apt upgrade`)
- Install Docker + Docker Compose
- Install NVIDIA drivers + CUDA toolkit

### Deploy with Docker Compose
Define services:
- **qwen** (model inference, GPU accelerated)
- **jina** (embedding model)
- **backend** (API layer, orchestrates queries)
- **frontend** (user interface)
- **db** (Postgres/MySQL for users/logs)
- **vector-db** (Milvus/Weaviate/FAISS for embeddings)

Mount volumes for persistent data
Run `docker compose up -d`

### Testing & Access
- Verify backend API and chatbot responses
- Expose frontend via NGINX reverse proxy

## 4. Deployment Steps – Broadcasting VM

### Provision VM2 in Proxmox
- Ubuntu 22.04 LTS
- Assign 8 vCPU, 32 GB RAM, 500 GB storage

### Install & Deploy
- Install Docker/Docker Compose
- Deploy broadcasting services (e.g., message queue, streaming service, notification microservice)

## 5. Deployment Steps – Automation VM

### Provision VM3 in Proxmox
- Ubuntu 22.04 LTS
- Assign 8 vCPU, 32 GB RAM, 500 GB storage

### Install & Configure
- Install scripts & automation tools
- Monitoring, backup tasks, data ingestion, cron jobs

## 6. Scaling Notes

- Add GPU nodes if concurrency grows
- Use object storage for logs and backups
- Scale horizontally with more chatbot VMs if user base expands
