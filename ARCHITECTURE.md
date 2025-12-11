# Architecture Diagram

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Kubernetes Cluster (AKS)                 │
│                                                                   │
│  ┌──────────────────┐      ┌──────────────────┐                │
│  │   Store-Front    │      │   Store-Admin    │                │
│  │   (Next.js)      │      │   (Next.js)      │                │
│  │   Port: 3000     │      │   Port: 3003     │                │
│  └────────┬─────────┘      └────────┬─────────┘                │
│           │                         │                           │
│           │                         │                           │
│  ┌────────▼─────────┐      ┌───────▼──────────┐               │
│  │  Product-Service │      │  Order-Service   │               │
│  │  (Express API)   │      │  (Express API)   │               │
│  │  Port: 3001      │      │  Port: 3002      │               │
│  └────────┬─────────┘      └────────┬─────────┘               │
│           │                         │                           │
│           │                         │                           │
│  ┌────────▼─────────────────────────▼──────────┐              │
│  │         MongoDB (StatefulSet)                │              │
│  │         Port: 27017                          │              │
│  └──────────────────────────────────────────────┘              │
│                                                                   │
│  ┌─────────────────────────────────────────────────┐           │
│  │         Makeline-Service (Worker)               │           │
│  │         Background Order Processing             │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    External Services                            │
│                                                                   │
│  ┌──────────────────┐      ┌──────────────────┐                │
│  │   Docker Hub     │      │  GitHub Actions  │                │
│  │   (Registry)     │      │  (CI/CD)         │                │
│  └──────────────────┘      └──────────────────┘                │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Component Details

### Frontend Services

1. **Store-Front** (Customer Portal)
   - Technology: Next.js 14 with React and TypeScript
   - Purpose: Customer-facing web application
   - Features: Product browsing, cart management, order placement
   - Exposed via: LoadBalancer service

2. **Store-Admin** (Admin Portal)
   - Technology: Next.js 14 with React and TypeScript
   - Purpose: Employee/admin web application
   - Features: Product management, order management, inventory control
   - Exposed via: LoadBalancer service

### Backend Services

3. **Product-Service**
   - Technology: Node.js with Express
   - Purpose: Product catalog management API
   - Endpoints:
     - GET /api/products - List all products
     - GET /api/products/:id - Get product by ID
     - POST /api/products - Create product
     - PUT /api/products/:id - Update product
     - DELETE /api/products/:id - Delete product
   - Database: MongoDB (products collection)

4. **Order-Service**
   - Technology: Node.js with Express
   - Purpose: Order processing API
   - Endpoints:
     - GET /api/orders - List all orders
     - GET /api/orders/:id - Get order by ID
     - POST /api/orders - Create order
     - PATCH /api/orders/:id - Update order status
     - DELETE /api/orders/:id - Delete order
   - Database: MongoDB (orders collection)

5. **Makeline-Service**
   - Technology: Node.js (Background Worker)
   - Purpose: Automated order fulfillment
   - Functionality:
     - Polls for pending orders every 10 seconds
     - Checks product availability
     - Updates product stock
     - Updates order status (pending → processing → completed)
   - Database: MongoDB (reads orders, updates products)

### Data Layer

6. **MongoDB**
   - Deployment: StatefulSet (for persistent storage)
   - Purpose: Primary database for all services
   - Collections:
     - products: Product catalog
     - orders: Order records
   - Storage: Persistent Volume Claims (10Gi)

## Data Flow

1. **Customer Order Flow:**
   - Customer browses products (Store-Front → Product-Service)
   - Customer adds to cart (Store-Front → Order-Service)
   - Order created with status "pending"
   - Makeline-Service picks up pending order
   - Makeline-Service checks stock (Product-Service)
   - Makeline-Service updates order status and stock

2. **Admin Management Flow:**
   - Admin views products/orders (Store-Admin → Product-Service/Order-Service)
   - Admin creates/updates products (Store-Admin → Product-Service)
   - Admin updates order status (Store-Admin → Order-Service)

## Configuration Management

- **ConfigMaps**: Application configuration (service URLs, MongoDB URI)
- **Secrets**: Sensitive data (MongoDB credentials)
- **Environment Variables**: Service-specific settings

## CI/CD Pipeline

Each service has its own GitHub Actions workflow:
1. Triggered on push to main/master branch
2. Builds Docker image
3. Pushes to Docker Hub
4. (Optional) Updates Kubernetes deployment

## Networking

- **ClusterIP**: Internal services (Product-Service, Order-Service)
- **LoadBalancer**: External access (Store-Front, Store-Admin)
- **Headless Service**: MongoDB StatefulSet

## Scalability

- All services support horizontal scaling (except MongoDB StatefulSet)
- Replicas configured: 2 for frontend/backend services, 1 for MongoDB and Makeline-Service
- Resource limits defined for all containers

