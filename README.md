# 🚀 Enterprise DevOps & SecOps End-to-End Pipeline

[![Build Status](https://img.shields.io/badge/Jenkins-CI%2FCD-red?logo=jenkins)](http://localhost:8080)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Minikube-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-Multi--Stage-2496ED?logo=docker)](https://www.docker.com/)
[![Security](https://img.shields.io/badge/Security-Snyk%20%7C%20Trivy-4B0082?logo=snyk)](https://snyk.io/)
[![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%26%20Grafana-E6522C?logo=prometheus)](https://prometheus.io/)

A production-ready **DevSecOps** pipeline for containerized microservices. This project demonstrates automated integration, shift-left security scanning, automated deployment to a local Kubernetes cluster, and full-stack observability with custom metrics scraping.

---

## 🏗️ Architecture Overview

                    +-------------------------------------------------+
                    |                 DEVELOPER WORKFLOW              |
                    +-------------------------------------------------+
                                             |
                                       [ Git Push ]
                                             v
                    +-------------------------------------------------+
                    |                GITHUB REPOSITORY                |
                    +-------------------------------------------------+
                                             |
                                      [ Webhook / Poll ]
                                             v
                    +-------------------------------------------------+
                    |                 JENKINS PIPELINE                |
                    +-------------------------------------------------+
                                             |
       +--------------------+----------------+--------------------+
       |                    |                                     |
       v                    v                                     v
[ SonarQube Analysis ]   [ Snyk Dependency Scan ]     [ Docker Build (Multi-Stage) ]
|                    |                                     |
+--------------------+----------------+--------------------+
|
v
[ Trivy Container Image Scan ]
|
v
[ K8s Deployment Sync ]
|
v
+-------------------------------------------------+
|               KUBERNETES CLUSTER                |
|                   (Minikube)                    |
+-------------------------------------------------+
|                               |
[ App Deployment: Port 30080 ]        [ ServiceMonitor ]
|                               |
+---------------+---------------+
|
v
+---------------------------+
|    PROMETHEUS & GRAFANA   |
|    (Cluster Monitoring)   |
+---------------------------+


---

## ✨ Key Features

* **Node.js/Express App**: Implements `/health` and `/metrics` endpoints using `prom-client`.
* **Shift-Left Security**:
  * **Snyk**: Scans open-source dependencies for security vulnerabilities.
  * **Trivy**: Scans final Docker images for OS and package vulnerabilities.
  * **SonarQube**: Static code quality and security standards analysis.
* **Production Containerization**:
  * Multi-stage build to reduce image size.
  * Non-root user execution (`USER node`) for Linux security compliance.
* **Kubernetes Orchestration**:
  * Deployment and Service specs with health checks (`livenessProbe` & `readinessProbe`).
* **Full Observability Stack**:
  * Deployed via `kube-prometheus-stack` Helm chart.
  * Custom `ServiceMonitor` target discovery for scraping application metrics every 10s.
  * Grafana Dashboard integration (ID: `11159`).

---

## 🛠️ Tech Stack

| Domain | Tool / Technology |
| :--- | :--- |
| **Application** | Node.js, Express, `prom-client` |
| **Source Control** | Git, GitHub |
| **CI/CD Automation** | Jenkins (Dockerized) |
| **Security / SecOps** | Snyk, Trivy, SonarQube |
| **Containerization** | Docker (Multi-stage build) |
| **Orchestration** | Kubernetes (Minikube), Helm |
| **Observability** | Prometheus, Grafana, Alertmanager |

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed locally:
* Linux (Ubuntu 22.04 LTS recommended)
* Docker & Docker Compose
* Minikube & kubectl
* Helm 3
* Jenkins (running inside Docker)

---

### Step 1: Clone Repository

```bash
git clone [https://github.com/ahmadelomdaa/Ultimate-DevOps-Project-End2End.git](https://github.com/ahmadelomdaa/Ultimate-DevOps-Project-End2End.git)
cd Ultimate-DevOps-Project-End2End
Step 2: Deploy Monitoring Stack (Prometheus & Grafana)
Bash
# Add Helm Repository
helm repo add prometheus-community [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)
helm repo update

# Install Stack in 'monitoring' namespace
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set kubeControllerManager.enabled=false \
  --set kubeScheduler.enabled=false \
  --set kubeEtcd.enabled=false
Step 3: Deploy ServiceMonitor for App Scraping
Apply the custom ServiceMonitor manifest so Prometheus can discover the application metrics automatically:

YAML
apiVersion: [monitoring.coreos.com/v1](https://monitoring.coreos.com/v1)
kind: ServiceMonitor
metadata:
  name: ultimate-devops-servicemonitor
  namespace: monitoring
  labels:
    release: prometheus
spec:
  namespaceSelector:
    matchNames:
      - default
  selector:
    matchLabels:
      app: ultimate-devops-app
  endpoints:
  - port: http
    path: /metrics
    interval: 10s
Apply it using:

Bash
kubectl apply -f k8s/servicemonitor.yaml
Step 4: Run the Jenkins CI/CD Pipeline
The Jenkinsfile automates the following steps:

Checkout Code: Clones the latest commit.

Dependency Scan: Runs snyk test.

Build Image: Builds optimized multi-stage Docker image.

Image Security Scan: Runs trivy image.

K8s Load & Deploy: Loads image to Minikube cluster and applies deployment manifests (kubectl apply -f k8s/).

📊 Accessing Observability Dashboards
1. Application Endpoint
Access the deployed Node.js application:

Bash
http://<minikube-ip>:30080/health
http://<minikube-ip>:30080/metrics
2. Grafana Dashboard
Access Grafana dashboard locally:

Bash
kubectl --namespace monitoring port-forward svc/prometheus-grafana 3001:80
URL: http://localhost:3001

Username: admin

Password Query:

Bash
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
Import Dashboard: Import Dashboard ID 11159 for Node.js Application metrics (Heap usage, Event loop lag, active requests).

📁 Repository Structure
Plaintext
├── k8s/
│   ├── deployment.yaml          # Kubernetes Deployment Manifest
│   ├── service.yaml             # Kubernetes Service Manifest
│   └── servicemonitor.yaml      # Prometheus ServiceMonitor CRD
├── Dockerfile                   # Multi-stage Dockerfile
├── Jenkinsfile                  # Complete CI/CD & SecOps pipeline script
├── index.js                     # Node.js Express Server with prom-client
├── package.json                 # Node.js dependencies
└── README.md                    # Project Documentation
👤 Author
Ahmad Elomda - DevOps & Cloud Engineer - GitHub Profile
