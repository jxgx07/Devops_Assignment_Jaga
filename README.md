# React App: CI/CD, Kubernetes Deployment & Monitoring

## Table of Contents

- [Project Overview](#project-overview)
- [Requirements](#requirements)
- [CI/CD Pipeline (Jenkins)](#cicd-pipeline-jenkins)
- [Docker Image Build](#docker-image-build)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Monitoring: Prometheus & Grafana](#monitoring-prometheus--grafana)
- [Usage Instructions](#usage-instructions)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Project Overview

This repo provides a complete workflow for deploying a React application in Kubernetes with automated CI/CD via Jenkins, Dockerized builds, and best-practice monitoring using Prometheus & Grafana.

---

## Requirements

- Docker
- Jenkins (NodeJS, Docker, and necessary plugins)
- Kubernetes cluster (local or cloud)
- Nginx Ingress Controller
- Helm v3+
- kubectl
- Trivy (security scanner)
- DockerHub account and Jenkins credentials set as `dockerhub-credentials`

---

## CI/CD Pipeline (Jenkins)

1. **Checkout Code**: Pull from source control.
2. **Install Dependencies**: `npm install` in `/app` folder.
3. **Run Tests**: Executes tests if present.
4. **Build Docker Image**: Uses multi-stage Dockerfile.
5. **Security Scan**: Uses Trivy to block vulnerabilities of severity "CRITICAL."
6. **Docker Login & Push**: Pushes tagged and `latest` image to DockerHub.
7. **Log Out**: Always logs out after pipeline run.

See `Jenkinsfile` for full details.

---

## Docker Image Build

- Multi-stage build for production-ready static React assets.
- Uses [serve](https://github.com/vercel/serve) to host app on port `3000`.

---

## Kubernetes Deployment

Apply using:

kubectl apply -f k8s.yaml:

This creates:

- **Deployment**: 2 replicas, rolling updates.
- **Service**: NodePort, cluster port 80, exposed on 30080.
- **Ingress**: Domain `myapp.local` (requires Ingress controller and `/etc/hosts` update).

**Hosts setup for local dev:**

127.0.0.1 myapp.local

---

## Monitoring: Prometheus & Grafana

### 1. Add Prometheus Community Helm Repository

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update


### 2. Create Namespace

kubectl create namespace monitoring


### 3. Install kube-prometheus-stack

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring


This will deploy Prometheus, Grafana, exporters, and alerting components.

---

### 4. Access Grafana

- **Default credentials:**
  - Username: `admin`
  - Password: `prom-operator`
- **Port-forward to localhost:**

kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

- **Open browser:**

http://localhost:3000


---

## Usage Instructions

### Build & Push Docker Image (Jenkins)

- Configure DockerHub credentials in Jenkins.
- Trigger build via Jenkins or SCM webhook.

### Deploy to Kubernetes

kubectl apply -f k8s.yaml


### Set up Hostname for Local Dev

Add to `/etc/hosts` (Linux/Mac):

127.0.0.1 myapp.local


### Access App

Open [http://myapp.local](http://myapp.local) in your browser.

### Deploy and Access Monitoring Stack

Follow the above instructions for Prometheus & Grafana.

---

## Troubleshooting

- **App not available:** Check pod logs with:
kubectl logs <pod-name>

- **Ingress not working:** Ensure Nginx ingress controller is deployed and running. Verify Service and Ingress mappings.
- **Monitoring stack issues:** Check monitoring namespace pod status:

- **Pipeline failures:** See Jenkins build logs for details, especially Trivy scan results.

---

## References

- [Jenkins](https://jenkins.io)
- [Docker](https://docker.com)
- [Trivy](https://aquasecurity.github.io/trivy/)
- [Kubernetes](https://kubernetes.io)
- [Prometheus Helm Charts](https://github.com/prometheus-community/helm-charts/)
- [Grafana](https://grafana.com)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)

---

