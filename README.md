# 🕹️ Nakama + CockroachDB + Prometheus on GKE

This repository contains Kubernetes manifests to deploy a real-time gaming backend stack on Google Kubernetes Engine (GKE), consisting of:

- ⚙️ [CockroachDB](https://www.cockroachlabs.com/) – Distributed SQL database
- 🧠 [Nakama](https://heroiclabs.com/) – Real-time multiplayer game server
- 📊 [Prometheus](https://prometheus.io/) – Monitoring system & time-series database

All services are exposed internally using ClusterIP, and public access is managed via GKE Ingress.

## 📁 File Structure

```
.
├── backendconfig.yaml              # GKE BackendConfig for timeout/session settings
├── cockroachdb-deployment.yaml     # CockroachDB Deployment
├── cockroachdb-pvc.yaml            # Persistent Volume Claim for CockroachDB
├── cockroachdb-service.yaml        # ClusterIP Service for CockroachDB
├── ingress.yaml                    # GKE Ingress for routing traffic to Nakama
├── managed-certificate.yaml        # GKE-managed SSL certificate
├── nakama-deployment.yaml          # Nakama Deployment with socket configs
├── nakama-service.yaml             # ClusterIP Service for Nakama
├── prometheus-config.yaml          # Prometheus ConfigMap to scrape Nakama metrics
├── prometheus-deployment.yaml      # Prometheus Deployment
├── prometheus-service.yaml         # ClusterIP Service for Prometheus
└── README.md                       # This file
```

## 🚀 Step-by-Step Deployment Guide

### 1. Prerequisites

- A GKE cluster is created, and kubectl is configured to access it
- A static IP is reserved for Ingress

  ```
  gcloud compute addresses create nakama-ingress-ip \
  --region=asia-south1 \
  --ip-version=IPV4
  ```

  To retrieve the IP after creation:

  ```
  gcloud compute addresses describe nakama-ingress-ip \
  --region=asia-south1 --format="get(address)"
  ```

- A DNS record (e.g., api.example.com) points to that static IP
- The GKE Ingress controller is enabled(usually enabled by default)

### 2. Clone the Repository

```
git clone https://github.com/madgicaltechdom/nakama-gke.git
cd nakama-gke
```

And replace YOUR_DOMAIN keyword with your Domain.

### 3. Deploy CockroachDB

```
kubectl apply -f cockroachdb-pvc.yaml
kubectl apply -f cockroachdb-deployment.yaml
kubectl apply -f cockroachdb-service.yaml
```

### 4. Deploy Nakama

```
kubectl apply -f nakama-deployment.yaml
kubectl apply -f nakama-service.yaml
```

Key Configs:

--database.address root@cockroachdb:26257

--socket.read_timeout_ms 60000

--socket.idle_timeout_ms 120000

Metrics exposed on port 9100

### 5. Deploy Prometheus

```
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
```

### 6. Set Up Ingress for Nakama

```
kubectl apply -f backendconfig.yaml
kubectl apply -f managed-certificate.yaml
kubectl apply -f ingress.yaml
```

## ✅ Verifying the Setup

```
kubectl get all
kubectl get ingress
kubectl get managedcertificate
kubectl get backendconfig
```

## ⚠️ Notes

- The certificate status will typically change from Provisioning to Active within 6 hours after creating the managed certificate and updating DNS records, but it may complete sooner or later depending on Google's verification process.
- Nakama is configured with a session token expiry of `7200` seconds (2 hours).
- Prometheus scrapes Nakama metrics from `nakama:9100`.
- CockroachDB is running in **insecure mode** for development only.

## 📜 License

MIT
