# URL Shortener Microservices

A multi-service URL shortener platform deployed on Kubernetes with CI/CD pipeline, monitoring, and auto-scaling capabilities.

## 🏗️ Architecture

The system consists of three independent microservices and Redis caching:

| Service | Language | Framework | Port | Description |
|---------|----------|-----------|------|-------------|
| Python Service | Python 3.11 | Flask | 5000 | Dashboard & Click Analytics |
| Go Service | Go 1.24 | Gin | 8000 | URL Shortening & Redirection |
| Node.js Service | Node.js 24 | Express | 3000 | URL Metadata Fetching |
| Redis | - | - | 6379 | Caching & Event Pub/Sub |

## 📋 Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Git](https://git-scm.com/downloads)

## 🚀 Deployment Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/ziadhosain-oss/urlshortner-microservices.git
cd urlshortner-microservices
```

### 2. Start Minikube Cluster
```bash
minikube start --driver=docker --cpus=4 --memory=3600
```

### 3. Enable Required Addons
```bash
minikube addons enable ingress
minikube addons enable metrics-server
```

### 4. Build Docker Images in Minikube
```bash
# Point to Minikube's Docker daemon
minikube docker-env | Invoke-Expression

# Build all images
docker build -t ziad10010/python-service:v1 ./python-service
docker build -t ziad10010/go-service:v1 ./go-service
docker build -t ziad10010/node-service:v1 ./node-service
```

### 5. Deploy All Services
```bash
# Apply all Kubernetes manifests
kubectl apply -f k8s/

# Verify deployment
kubectl get pods
kubectl get svc
```

Expected output: 7 pods running (2 Python, 2 Go, 2 Node, 1 Redis)

### 6. Deploy Monitoring Stack
```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Deploy Prometheus and Grafana
kubectl apply -f prometheus.yaml
kubectl apply -f grafana.yaml

# Verify monitoring pods
kubectl get pods -n monitoring
```

### 7. Access Services
```bash
# Start Minikube tunnel (keep running)
minikube tunnel

# Test endpoints (in another terminal)
curl -H "Host: urlshortner.local" http://localhost/python/
curl -H "Host: urlshortner.local" http://localhost/go/
curl -H "Host: urlshortner.local" http://localhost/node/health
```

### 8. Access Monitoring Dashboards
```bash
# Grafana
kubectl port-forward -n monitoring svc/grafana 3030:3000
# Open: http://localhost:3030 (Username: admin, Password: admin123)

# Prometheus
kubectl port-forward -n monitoring svc/prometheus 9090:9090
# Open: http://localhost:9090
```

## 🐳 Local Development with Docker Compose

```bash
# Start all services locally
docker-compose up -d

# Access services
# Python: http://localhost:5000
# Go: http://localhost:8000
# Node: http://localhost:3000

# Stop services
docker-compose down
```

## 📊 Kubernetes Resources

### Deployments & Auto-scaling
| Service | Replicas | HPA Min | HPA Max | CPU Target |
|---------|----------|---------|---------|------------|
| python-service | 2 | 2 | 10 | 70% |
| go-service | 2 | 2 | 10 | 70% |
| node-service | 2 | 2 | 10 | 70% |
| redis | 1 | - | - | - |

### Services
| Service | Type | Cluster Port | Container Port |
|---------|------|-------------|----------------|
| python-service | ClusterIP | 5000 | 5000 |
| go-service | ClusterIP | 8080 | 8000 |
| node-service | ClusterIP | 3000 | 3000 |
| redis-service | ClusterIP | 6379 | 6379 |

### Ingress Routing
```
Host: urlshortner.local
├── /python → python-service:5000
├── /go     → go-service:8080
└── /node   → node-service:3000
```

### ConfigMaps
- **python-config**: Service URLs and environment configuration
- **go-config**: Python service URL for HTTP event fallback
- **node-config**: Node environment settings

### Secrets
- **python-secret**: Database password (base64 encoded)
- **go-secret**: Service credentials

## 🔄 CI/CD Pipeline

GitHub Actions automatically builds and pushes Docker images on push to main branch.

**Pipeline File:** `.github/workflows/deploy.yml`

### Pipeline Stages:
1. Checkout code
2. Login to DockerHub
3. Build Docker images (Python & Node)
4. Push to DockerHub

### Required GitHub Secrets:
| Secret | Description |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |

## 🔍 SonarQube Code Analysis

Automated code quality analysis via **SonarCloud** on every push and pull request.

| Metric | Rating | Issues |
|--------|--------|--------|
| Security | A | 26 |
| Reliability | E | 19 |
| Maintainability | A | 45 |
| Duplications | - | 2.2% |

**Quality Gate:** Sonar way  
**Organization:** ziadhosain-oss  
**Configuration:** `sonar-project.properties`

**Required GitHub Secrets:**
| Secret | Description |
|--------|-------------|
| `SONAR_TOKEN` | SonarCloud access token |
| `SONAR_HOST_URL` | `https://sonarcloud.io` |

## 🧪 Load Testing

### Simulate Traffic Spike
```powershell
# Run 100 concurrent requests to Go service
1..100 | ForEach-Object {
    $body = '{"long_url":"https://www.example.com"}'
    try {
        $response = Invoke-RestMethod -Uri "http://localhost:8000/api/shorten" `
            -Method Post -Body $body -ContentType "application/json"
        Write-Host "Created: $($response.short_code)"
    } catch {}
}
```

### Monitor HPA Scaling
```bash
# Watch HPA during load test
kubectl get hpa -w

# Watch pods scaling
kubectl get pods -w
```

## 🎯 Traffic Spike Handling

The system is configured to handle daily traffic spikes at 12:00 PM through:
- **HPA**: Automatically scales pods from 2 to 10 based on CPU utilization
- **Redis Caching**: Reduces database load with 1-hour URL cache TTL
- **Multiple Replicas**: Each service runs minimum 2 replicas for high availability

## 📁 Project Structure
```
urlshortner-microservices/
├── .github/workflows/
│   └── deploy.yml              # CI/CD Pipeline
├── k8s/                        # Kubernetes Manifests
│   ├── python-deployment.yaml
│   ├── python-config.yaml
│   ├── python-secret.yaml
│   ├── python-hpa.yaml
│   ├── go-deployment.yaml
│   ├── go-config.yaml
│   ├── go-secret.yaml
│   ├── go-hpa.yaml
│   ├── node-deployment.yaml
│   ├── node-config.yaml
│   ├── node-hpa.yaml
│   ├── redis-deployment.yaml
│   └── ingress.yaml
├── python-service/             # Python Microservice
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│   └── templates/
├── go-service/                 # Go Microservice
│   ├── Dockerfile
│   ├── main.go
│   └── go.mod
├── node-service/               # Node.js Microservice
│   ├── Dockerfile
│   ├── server.js
│   └── package.json
├── monitoring/                 # Monitoring Configs
│   ├── prometheus.yaml
│   ├── grafana.yaml
│   ├── prometheus-rbac.yaml
│   └── prometheus-cm.yaml
├── sonar-project.properties    # SonarQube config
├── docker-compose.yml          # Local development
├── architecture.png            # Architecture diagram
└── README.md                   # This file

## ✅ Verification Checklist

Run these commands to verify the deployment:

```bash
# All pods running (should show 7 pods)
kubectl get pods

# All services available
kubectl get svc

# HPA configured correctly
kubectl get hpa

# Deployments healthy
kubectl get deployments

# Ingress active
kubectl get ingress

# ConfigMaps created
kubectl get configmaps

# Secrets created
kubectl get secrets

# Monitoring running
kubectl get pods -n monitoring
```

## 🔧 Troubleshooting

### Pods stuck in ImagePullBackOff
```bash
# Build images in Minikube's Docker context
minikube docker-env | Invoke-Expression
docker build -t ziad10010/go-service:v1 ./go-service
```

### Go service CrashLoopBackOff
```bash
# Ensure Dockerfile has CGO enabled for SQLite
# Required: apk add --no-cache gcc musl-dev
# Build: CGO_ENABLED=1 go build
```

### Metrics API not available
```bash
# Enable metrics server
minikube addons enable metrics-server
# Wait 1-2 minutes
kubectl top nodes
```

### HPA shows unknown
```bash
# Wait for metrics server to be fully ready
kubectl get pods -n kube-system | findstr metrics
```

## 📄 License

This project is part of a DevOps & Cloud Engineering assignment.

## 👤 Author

**Ziad Hosain**
- GitHub: [@ziadhosain-oss](https://github.com/ziadhosain-oss)
- DockerHub: [@ziad10010](https://hub.docker.com/u/ziad10010)
