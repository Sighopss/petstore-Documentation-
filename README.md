# Algonquin Pet Store (On Steroids) - Cloud Native Application

A microservices-based e-commerce application for a pet store, built with cloud-native principles and deployed on Kubernetes.

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed architecture diagram and component descriptions.

**Note**: The architecture diagram in ARCHITECTURE.md is provided in text format. For a visual diagram, you can:
1. Use the text diagram as a reference to create a Draw.io diagram
2. Export the diagram as PNG/SVG and save as `architecture-diagram.png` in the root directory
3. The diagram should show all 5 microservices, MongoDB, and their interconnections

### Microservices Overview

1. **Store-Front**: Customer-facing web application for browsing and purchasing pet products
2. **Store-Admin**: Employee web application for managing products, orders, and inventory
3. **Order-Service**: RESTful API for processing and managing customer orders
4. **Product-Service**: RESTful API for product catalog management
5. **Makeline-Service**: Background worker service for processing and fulfilling orders

### Technology Stack

- **Frontend**: React with TypeScript
- **Backend**: Node.js with Express
- **Database**: MongoDB (StatefulSet)
- **Containerization**: Docker
- **Orchestration**: Kubernetes (AKS)
- **CI/CD**: GitHub Actions

## Repository Links

| Service | Repository | Docker Hub Image |
|---------|-----------|------------------|
| Store-Front | [Link](#) | [Link](#) |
| Store-Admin | [Link](#) | [Link](#) |
| Order-Service | [Link](#) | [Link](#) |
| Product-Service | [Link](#) | [Link](#) |
| Makeline-Service | [Link](#) | [Link](#) |

## Deployment Instructions

For detailed deployment instructions, see [DEPLOYMENT.md](DEPLOYMENT.md).

### Quick Start

1. **Update Docker image references:**
   - Replace `trevorkutto` in `deployment-files/services/*.yaml` with your Docker Hub username

2. **Build and push Docker images:**
   ```bash
   docker build -t trevorkutto/store-front:latest ./store-front
   docker push trevorkutto/store-front:latest
   # Repeat for all services
   ```

3. **Deploy to Kubernetes:**
   ```bash
   kubectl apply -f deployment-files/
   ```

4. **Verify deployment:**
   ```bash
   kubectl get pods -n pet-store
   kubectl get services -n pet-store
   ```

## Local Development

### Running Services Locally

Each service can be run independently:

```bash
# Store-Front
cd store-front
npm install
npm run dev

# Store-Admin
cd store-admin
npm install
npm run dev

# Order-Service
cd order-service
npm install
npm start

# Product-Service
cd product-service
npm install
npm start

# Makeline-Service
cd makeline-service
npm install
npm start
```

### Docker Compose (Local Testing)

```bash
docker-compose up
```

## CI/CD Pipeline

Each microservice has a GitHub Actions workflow that:
1. Builds the Docker image on push to main
2. Pushes to Docker Hub
3. Updates Kubernetes deployment (optional)

Configure the following secrets in GitHub:
- `DOCKER_USERNAME`
- `DOCKER_PASSWORD`
- `KUBECONFIG` (for automated deployments)

## Monitoring and Logging

- Application logs: `kubectl logs -f <pod-name>`
- Service health: Check `/health` endpoints on each service
- MongoDB status: `kubectl get statefulset mongodb`

## Contributing

1. Create a feature branch
2. Make your changes
3. Commit with meaningful messages
4. Push and create a pull request

## License

MIT License

