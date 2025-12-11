# Deployment Guide

## Prerequisites

1. **Azure Kubernetes Service (AKS) Cluster**
   - Create an AKS cluster via Azure Portal or CLI
   - Ensure kubectl is configured: `az aks get-credentials --resource-group <rg> --name <cluster-name>`

2. **Docker Hub Account**
   - Create account at https://hub.docker.com
   - Note your username for image references

3. **GitHub Repository**
   - Fork or create repositories for each service
   - Configure GitHub Actions secrets (see CI/CD section)

## Step 1: Build and Push Docker Images

### Option A: Manual Build and Push

```bash
# Build and push each service
docker build -t trevorkutto/store-front:latest ./store-front
docker push trevorkutto/store-front:latest

docker build -t trevorkutto/store-admin:latest ./store-admin
docker push trevorkutto/store-admin:latest

docker build -t trevorkutto/product-service:latest ./product-service
docker push trevorkutto/product-service:latest

docker build -t trevorkutto/order-service:latest ./order-service
docker push trevorkutto/order-service:latest

docker build -t trevorkutto/makeline-service:latest ./makeline-service
docker push trevorkutto/makeline-service:latest
```

### Option B: Use CI/CD Pipeline

1. Push code to GitHub
2. GitHub Actions will automatically build and push images
3. Ensure `DOCKER_USERNAME` and `DOCKER_PASSWORD` secrets are configured

## Step 2: Update Kubernetes Manifests

1. **Update image references:**
   ```bash
   # If you need to change the Docker username, replace trevorkutto in all service YAML files
   # sed -i 's/trevorkutto/yourusername/g' deployment-files/services/*.yaml
   # Note: Currently configured with trevorkutto
   ```

2. **Update MongoDB credentials (optional):**
   ```bash
   # Edit deployment-files/secrets/app-secrets.yaml
   # Update MONGODB_USERNAME and MONGODB_PASSWORD
   ```

## Step 3: Deploy to Kubernetes

### Quick Deployment

```bash
# Apply all manifests
kubectl apply -f deployment-files/
```

### Verify Deployment

```bash
# Check namespace
kubectl get namespace pet-store

# Check all pods
kubectl get pods -n pet-store

# Check all services
kubectl get services -n pet-store

# Check StatefulSet
kubectl get statefulset -n pet-store
```

### Wait for Services to be Ready

```bash
# Wait for MongoDB
kubectl wait --for=condition=ready pod -l app=mongodb -n pet-store --timeout=300s

# Wait for all deployments
kubectl wait --for=condition=available deployment --all -n pet-store --timeout=300s
```

## Step 4: Access the Application

### Get Service URLs

```bash
# Get Store-Front LoadBalancer IP
kubectl get service store-front -n pet-store

# Get Store-Admin LoadBalancer IP
kubectl get service store-admin -n pet-store
```

### Port Forwarding (Alternative)

```bash
# Store-Front
kubectl port-forward service/store-front 3000:3000 -n pet-store

# Store-Admin
kubectl port-forward service/store-admin 3003:3003 -n pet-store

# Product-Service
kubectl port-forward service/product-service 3001:3001 -n pet-store

# Order-Service
kubectl port-forward service/order-service 3002:3002 -n pet-store
```

## Step 5: Initialize Data (Optional)

### Add Sample Products

```bash
# Port forward to product-service
kubectl port-forward service/product-service 3001:3001 -n pet-store

# In another terminal, create products
curl -X POST http://localhost:3001/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Dog Food Premium",
    "description": "High-quality dog food",
    "price": 29.99,
    "category": "Food",
    "stock": 100
  }'

curl -X POST http://localhost:3001/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Cat Litter",
    "description": "Premium cat litter",
    "price": 19.99,
    "category": "Supplies",
    "stock": 50
  }'
```

## Monitoring

### View Logs

```bash
# View logs for a specific pod
kubectl logs -f <pod-name> -n pet-store

# View logs for all pods of a service
kubectl logs -f -l app=product-service -n pet-store
```

### Check Health Endpoints

```bash
# Product Service
kubectl exec -it <product-service-pod> -n pet-store -- wget -qO- http://localhost:3001/health

# Order Service
kubectl exec -it <order-service-pod> -n pet-store -- wget -qO- http://localhost:3002/health
```

### Resource Usage

```bash
# Check resource usage
kubectl top pods -n pet-store

# Check node resources
kubectl top nodes
```

## Scaling

### Scale Services

```bash
# Scale Product Service to 3 replicas
kubectl scale deployment product-service --replicas=3 -n pet-store

# Scale Order Service to 3 replicas
kubectl scale deployment order-service --replicas=3 -n pet-store
```

## Troubleshooting

### Pods in CrashLoopBackOff

```bash
# Check pod logs
kubectl logs <pod-name> -n pet-store

# Check pod events
kubectl describe pod <pod-name> -n pet-store
```

### Services Not Accessible

```bash
# Check service endpoints
kubectl get endpoints -n pet-store

# Check service selector
kubectl describe service <service-name> -n pet-store
```

### MongoDB Connection Issues

```bash
# Check MongoDB pod status
kubectl get pods -l app=mongodb -n pet-store

# Check MongoDB logs
kubectl logs -l app=mongodb -n pet-store

# Test MongoDB connection
kubectl exec -it <mongodb-pod> -n pet-store -- mongosh
```

## Cleanup

### Remove All Resources

```bash
# Delete all resources in namespace
kubectl delete -f deployment-files/

# Or delete entire namespace
kubectl delete namespace pet-store
```

### Remove Persistent Volumes (if needed)

```bash
# List PVCs
kubectl get pvc -n pet-store

# Delete PVCs (data will be lost!)
kubectl delete pvc -l app=mongodb -n pet-store
```

## CI/CD Configuration

### GitHub Actions Secrets

Configure the following secrets in your GitHub repository:

1. Go to Settings → Secrets and variables → Actions
2. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token

### Automated Deployment (Optional)

To enable automated Kubernetes deployments:

1. Add `KUBECONFIG` secret with your kubeconfig file content
2. Update GitHub Actions workflows to include deployment steps
3. Configure deployment triggers (e.g., on tag creation)

## Production Considerations

1. **Use Ingress Controller** instead of LoadBalancer for cost optimization
2. **Enable TLS/SSL** for all external services
3. **Use managed MongoDB** (Azure Cosmos DB) for production
4. **Implement monitoring** (Azure Monitor, Prometheus)
5. **Set up backup** for MongoDB data
6. **Configure resource quotas** and limits
7. **Use secrets management** (Azure Key Vault)
8. **Enable horizontal pod autoscaling** (HPA)

