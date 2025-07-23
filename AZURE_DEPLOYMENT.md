# Azure Deployment Guide for Pickup

This guide explains how to deploy Pickup (by Sentinel Blue) to Azure using Docker containers.

## Prerequisites

- Azure CLI installed
- Docker installed locally
- An Azure subscription
- Azure Container Registry (ACR) or Docker Hub account

## Local Build and Test

1. Build the Docker image locally:
```bash
docker build -t pickup:latest .
```

2. Test locally using docker-compose:
```bash
docker-compose -f docker-compose.azure.yml up
```

The application will be available at http://localhost

## Azure Container Instances Deployment

### Option 1: Using Azure Container Registry (ACR)

1. Create an Azure Container Registry:
```bash
az acr create --resource-group myResourceGroup --name myacr --sku Basic
```

2. Log in to ACR:
```bash
az acr login --name myacr
```

3. Tag and push the image:
```bash
docker tag pickup:latest myacr.azurecr.io/pickup:latest
docker push myacr.azurecr.io/pickup:latest
```

4. Deploy to Azure Container Instances:
```bash
az container create \
  --resource-group myResourceGroup \
  --name pickup-app \
  --image myacr.azurecr.io/pickup:latest \
  --registry-login-server myacr.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --dns-name-label pickup-app \
  --ports 80 \
  --environment-variables \
    STORAGE_TYPE=redis \
    SECRET_EXPIRY=604800
```

### Option 2: Using Docker Hub

1. Tag and push to Docker Hub:
```bash
docker tag pickup:latest yourusername/pickup:latest
docker push yourusername/pickup:latest
```

2. Deploy to Azure Container Instances:
```bash
az container create \
  --resource-group myResourceGroup \
  --name pickup-app \
  --image yourusername/pickup:latest \
  --dns-name-label pickup-app \
  --ports 80
```

## Azure App Service Deployment

1. Create an App Service Plan:
```bash
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux
```

2. Create a Web App for Containers:
```bash
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name pickup-webapp \
  --deployment-container-image-name myacr.azurecr.io/pickup:latest
```

3. Configure continuous deployment (optional):
```bash
az webapp deployment container config \
  --name pickup-webapp \
  --resource-group myResourceGroup \
  --enable-cd true
```

## Azure Kubernetes Service (AKS) Deployment

For production deployments with Redis persistence, consider using AKS:

1. Create an AKS cluster:
```bash
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys
```

2. Get credentials:
```bash
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

3. Apply Kubernetes manifests (create these based on docker-compose.azure.yml)

## Environment Variables

Key environment variables for Azure deployment:

- `STORAGE_TYPE`: Set to "redis" for production
- `REDIS_URL`: Redis connection string (if using external Redis)
- `SECRET_EXPIRY`: Default expiry time in seconds (604800 = 7 days)
- `CUSTOMIZE`: Path to customization file (optional)

## SSL/TLS Configuration

For production use, ensure HTTPS is enabled:

- Azure Container Instances: Use Azure Application Gateway
- App Service: Enable HTTPS in the portal
- AKS: Use an Ingress controller with cert-manager

## Monitoring

- Enable Application Insights for monitoring
- Configure alerts for high memory/CPU usage
- Monitor Redis memory usage if using persistent storage

## Security Considerations

1. Use managed identities where possible
2. Store sensitive configuration in Azure Key Vault
3. Enable network security groups to restrict access
4. Regularly update the base images
5. Enable Azure Defender for container security scanning