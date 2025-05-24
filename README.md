# Calendar Application

A full-stack calendar application built with React and Node.js, featuring a modern UI and robust backend services.

## Features

- Modern and responsive calendar interface
- Event management and scheduling
- User authentication and authorization
- RESTful API backend
- Docker containerization
- Kubernetes deployment
- Monitoring with Prometheus and Grafana
- SSL/TLS encryption with Let's Encrypt

## Project Structure

```
├── app-chart/           # Helm chart configuration
│   ├── Chart.yaml      # Chart metadata and dependencies
│   └── values.yaml     # Default configuration values
├── backend/            # Node.js backend service
│   ├── Dockerfile      # Backend container configuration
│   ├── package.json    # Node.js dependencies
│   └── server.js       # Main backend application
├── frontend/           # React frontend application
│   ├── Dockerfile      # Frontend container configuration
│   ├── app.js         # Main React application
│   ├── index.html     # Entry point
│   └── style.css      # Frontend styling
├── manifests/          # Kubernetes manifest files
├── terraform/          # Infrastructure as Code configurations
│   ├── aws/           # AWS infrastructure setup
│   └── cluster-resources/ # EKS cluster resources
└── docker-compose.yaml # Local development environment
```

## Prerequisites

- Node.js (v14 or higher)
- Docker and Docker Compose
- Kubernetes cluster
- Helm 3
- MySQL database
- Cloudflare DNS (for Let's Encrypt)
- ECR (Elastic Container Registry) for container images
- AWS CLI configured with credentials
- kubectl configured with EKS cluster context

## Local Development

1. Install dependencies:
```bash
# Backend
cd backend
npm install

# Frontend
cd frontend
npm install
```

2. Start the development environment:
```bash
docker-compose up
```

3. Access the application:
- Frontend: http://localhost
- Backend API: http://localhost:3000/api

## Production Deployment

### AWS Infrastructure Setup

1. **Create secret.tfvars files**
   Create a `secret.tfvars` file in both `aws` and `aws_cluster_resources` directories with your AWS credentials:
   ```hcl
   aws_access_key = "your_access_key"
   aws_secret_key = "your_secret_key"
   region         = "us-east-1"
   ```

2. **Deploy AWS Infrastructure**
   ```bash
   # Deploy AWS resources
   cd terraform/aws
   terraform init
   terraform apply -var-file="secret.tfvars"
   ```

3. **Deploy EKS Cluster Resources**
   ```bash
   # Deploy EKS cluster resources
   cd terraform/aws_cluster_resources
   terraform init
   terraform apply -var-file="secret.tfvars"
   ```

4. **Configure AWS CLI and kubectl**
   ```bash
   aws configure set aws_access_key_id "your_access_key"
   aws configure set aws_secret_access_key "your_secret_key"
   aws configure set default.region us-east-1
   aws eks --region us-east-1 update-kubeconfig --name staging
   ```

5. **Configure DNS Records**
   Get the external IP and configure DNS records in Cloudflare:
   ```bash
   kubectl get svc
   # Copy the external IP and run nslookup
   nslookup <external_ip>
   ```

6. **Configure ArgoCD**
   - Access ArgoCD at: https://argocd.exampledomain.com
   - Default credentials:
     - Username: `admin`
     - Password: `ASDqwe123`
   - Create application using `argo-application.yaml`
   - Sync the application

### Application Deployment

1. Build and push Docker images:
```bash
# Build backend image
docker build -t <ECR_REPO>/trainings:backend -f backend/Dockerfile .

# Build frontend image
docker build -t <ECR_REPO>/trainings:frontendv3 -f frontend/Dockerfile .
```

2. Deploy to Kubernetes via Helm:
`update values.yaml appropriately with correct values`
```bash
cd app-chart/
helm install my-app .
```

## Configuration

### Environment Variables

- Backend:
  - `MYSQL_DATABASE`: Database name (default: calenderDb)
  - `MYSQL_USER`: Database username
  - `MYSQL_PASSWORD`: Database password

- Frontend:
  - `REACT_APP_API_BASE_URL`: Backend API URL (default: https://api.exampledomain.com)

### Database

The application uses MySQL as its database. The database configuration can be found in `values.yaml` under the `mysql` section.

### Monitoring

The application includes Prometheus and Grafana for monitoring:

- Prometheus: For metrics collection and monitoring
- Grafana: For visualization of metrics
- Prometheus ServiceMonitor enabled
- Grafana persistence enabled with 8Gi storage

## Security

- HTTPS enabled through cert-manager
- Let's Encrypt certificates for SSL/TLS
- Resource limits and security context defined
- Pod anti-affinity for high availability
- Read-only root filesystem
- Privilege escalation disabled
- Resource quotas defined

## Cleanup

To destroy the infrastructure, follow this order:

1. EKS Cluster Resources:
```bash
cd terraform/aws_cluster_resources
terraform destroy -var-file="secret.tfvars"
```

2. AWS Infrastructure:
```bash
cd terraform/aws
terraform destroy -var-file="secret.tfvars"
```

[![Build, Push to ECR, and Deploy to EKS](https://github.com/faizananwar532/calendar-app/actions/workflows/cicd.yaml/badge.svg)](https://github.com/faizananwar532/calendar-app/actions/workflows/cicd.yaml)  

