# Docker Use Cases

## Virtual Network in Docker

**Date:** October 20, 2025  
**Topic:** Practical Docker Networking Patterns and Architectures

This section explores real-world scenarios where Docker networks solve actual business and technical problems.

---

### Use Case 1: Microservices Architecture

**Scenario:** E-commerce Platform with Multiple Services

An e-commerce platform typically consists of many independent services that need to communicate with each other.

**Implementation:**

```bash
# Create network for the entire application
docker network create ecommerce-net

# Start microservices
docker run -d --name user-service --network ecommerce-net user-service:latest
docker run -d --name product-service --network ecommerce-net product-service:latest
docker run -d --name order-service --network ecommerce-net order-service:latest
docker run -d --name payment-service --network ecommerce-net payment-service:latest
docker run -d --name inventory-service --network ecommerce-net inventory-service:latest
docker run -d --name notification-service --network ecommerce-net notification-service:latest
```

**How Services Communicate:**

```bash
# From order-service container:
curl http://payment-service:8080/api/process-payment

# From product-service container:
curl http://inventory-service:3000/api/check-stock

# From notification-service container:
curl http://user-service:5000/api/user/email
```

**Benefits:**
- ✅ **Service Discovery:** Services find each other by name (no hardcoded IPs)
- ✅ **Isolation:** Entire application isolated from other apps on same host
- ✅ **Scalability:** Easy to add new services to the network
- ✅ **DNS Resolution:** Automatic DNS for all service names
- ✅ **Development/Production Parity:** Same setup in all environments

**Architecture Diagram:**
```
┌──────────────────────────────────────────────────────┐
│           ecommerce-net (172.20.0.0/16)              │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │ user-service │  │product-service│ │order-service│ │
│  │ :5000        │  │ :3000         │ │ :8000      │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │payment-service│ │inventory-svc  │ │notification│ │
│  │ :8080         │ │ :4000         │ │ :9000      │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│                                                      │
│  All services communicate by name via DNS            │
└──────────────────────────────────────────────────────┘
```

**Real-World Example:**
- **Netflix:** Uses this pattern for 700+ microservices
- **Amazon:** Thousands of microservices on internal networks
- **Uber:** Service mesh with isolated networks per service group

---

### Use Case 2: Multi-Tier Application (Frontend/Backend/Database)

**Scenario:** Web Application with Strict Layer Separation

A traditional three-tier architecture where each layer should only communicate with adjacent layers.

**Implementation:**

```bash
# Create separate networks for each tier
docker network create frontend-net
docker network create backend-net
docker network create database-net

# Tier 3: Database (only on database network)
docker run -d --name postgres-db \
  --network database-net \
  -e POSTGRES_PASSWORD=securepass \
  -e POSTGRES_DB=appdb \
  postgres:15

# Tier 2: Backend API (connected to both backend and database networks)
docker run -d --name api-server \
  --network backend-net \
  -e DATABASE_URL=postgres://postgres-db:5432/appdb \
  api:latest

docker network connect database-net api-server

# Tier 1: Frontend (only on frontend network)
docker run -d --name web-ui \
  --network frontend-net \
  nginx:alpine

# API Gateway (bridges frontend and backend)
docker run -d --name api-gateway \
  --network frontend-net \
  -p 443:443 \
  gateway:latest

docker network connect backend-net api-gateway
```

**Security Matrix:**

| Component | frontend-net | backend-net | database-net |
|-----------|--------------|-------------|--------------|
| web-ui | ✅ | ❌ | ❌ |
| api-gateway | ✅ | ✅ | ❌ |
| api-server | ❌ | ✅ | ✅ |
| postgres-db | ❌ | ❌ | ✅ |

**Security Benefits:**
- 🔒 Frontend **CANNOT** directly access database
- 🔒 Frontend **CANNOT** directly access backend APIs
- 🔒 Only API server can reach database
- 🔒 Only API Gateway can reach backend services
- 🔒 Defense in depth strategy (multiple security layers)

**Architecture Diagram:**
```
Internet
   │
   ↓
[443] api-gateway (frontend-net + backend-net)
   │
   ├─→ web-ui (frontend-net only)
   │
   └─→ api-server (backend-net + database-net)
          │
          └─→ postgres-db (database-net only)
```

**Why This Matters:**
- If frontend is compromised → Cannot access backend/database
- If API gateway is compromised → Cannot access database
- If backend is compromised → Cannot affect frontend directly
- Lateral movement is prevented

**Real-World Example:**
- **Banking Apps:** Strict tier separation for PCI-DSS compliance
- **Healthcare Systems:** HIPAA requires network segmentation
- **Enterprise SaaS:** Multi-tenant isolation requirements

---

### Use Case 3: Database Isolation & Security

**Scenario:** Production Database Needs Maximum Protection

Databases are the most valuable asset and need complete isolation from external networks.

**Implementation:**

```bash
# Create INTERNAL network (no external access whatsoever)
docker network create --internal db-internal-net

# Production database on internal network only
docker run -d --name production-db \
  --network db-internal-net \
  -e POSTGRES_PASSWORD=super_secret_password \
  -e POSTGRES_DB=production \
  -v /var/lib/postgresql/data:/var/lib/postgresql/data \
  postgres:15

# Application server with access to database
docker run -d --name app-server \
  --network db-internal-net \
  -e DB_HOST=production-db \
  -e DB_PORT=5432 \
  -e DB_NAME=production \
  myapp:latest

# Web server (separate network for external access)
docker network create web-public-net

docker run -d --name nginx-proxy \
  --network web-public-net \
  -p 443:443 \
  nginx:alpine

# Connect app-server to both networks
docker network connect web-public-net app-server
```

**Network Configuration:**
```bash
# Verify database is truly isolated
docker network inspect db-internal-net

# Output shows:
"Internal": true  # ← This prevents ANY external access
```

**What "Internal" Means:**
- ❌ Database **CANNOT** access internet
- ❌ Internet **CANNOT** access database
- ❌ Database **CANNOT** make outbound connections
- ❌ Even if container is compromised, no data exfiltration possible
- ✅ Only containers on same internal network can communicate

**Security Layers:**

```
┌─────────────────────────────────────────────────┐
│         web-public-net (External Access)        │
│                                                 │
│  [nginx-proxy] ←→ [app-server]                 │
│      ↑                    ↑                     │
│      │                    │                     │
│   Internet          Dual-homed                 │
└──────────────────────────┼──────────────────────┘
                           │
┌──────────────────────────┼──────────────────────┐
│      db-internal-net (INTERNAL - No Internet)   │
│                          ↓                      │
│                   [app-server] ←→ [production-db]│
│                                                 │
│  🔒 Completely isolated from external networks  │
└─────────────────────────────────────────────────┘
```

**Compliance Benefits:**
- ✅ **PCI-DSS Requirement 1.3:** Network segmentation for cardholder data
- ✅ **HIPAA Security Rule:** Protected health information isolation
- ✅ **SOC 2 Type II:** Network isolation controls
- ✅ **GDPR:** Data protection by design

**Real-World Example:**
- **Stripe:** Payment processing databases on isolated networks
- **Healthcare Providers:** Patient data in internal networks only
- **Financial Institutions:** Transaction databases completely isolated

---

### Use Case 4: Development vs Staging vs Production Environments

**Scenario:** Same Application, Different Network Configurations

**Development Environment (docker-compose.dev.yml):**

```yaml
version: '3.8'
services:
  app:
    image: myapp:dev
    networks:
      - dev-net
    ports:
      - "3000:3000"  # Exposed for debugging
    environment:
      - DEBUG=true
      - LOG_LEVEL=debug
  
  db:
    image: postgres:15
    networks:
      - dev-net
    ports:
      - "5432:5432"  # Direct DB access for developers
    environment:
      - POSTGRES_PASSWORD=devpass
  
  redis:
    image: redis:alpine
    networks:
      - dev-net
    ports:
      - "6379:6379"  # Direct access for debugging

networks:
  dev-net:
    driver: bridge
```

**Staging Environment (docker-compose.staging.yml):**

```yaml
version: '3.8'
services:
  app:
    image: myapp:staging
    networks:
      - staging-backend
      - staging-db
    environment:
      - LOG_LEVEL=info
  
  db:
    image: postgres:15
    networks:
      - staging-db
    # No ports exposed to host
    environment:
      - POSTGRES_PASSWORD=${STAGING_DB_PASSWORD}
  
  nginx:
    image: nginx:alpine
    networks:
      - staging-frontend
      - staging-backend
    ports:
      - "443:443"
    volumes:
      - ./nginx-staging.conf:/etc/nginx/nginx.conf

networks:
  staging-frontend:
  staging-backend:
  staging-db:
    internal: true  # Database isolated
```

**Production Environment (docker-compose.prod.yml):**

```yaml
version: '3.8'
services:
  app:
    image: myapp:prod
    networks:
      - prod-backend
      - prod-db
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
    environment:
      - LOG_LEVEL=warn
  
  db:
    image: postgres:15
    networks:
      - prod-db
    # No ports exposed
    volumes:
      - prod-db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${PROD_DB_PASSWORD}
  
  nginx:
    image: nginx:alpine
    networks:
      - prod-frontend
      - prod-backend
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx-prod.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt:ro

networks:
  prod-frontend:
  prod-backend:
  prod-db:
    internal: true  # Production DB completely isolated
    
volumes:
  prod-db-data:
```

**Environment Comparison:**

| Feature | Development | Staging | Production |
|---------|-------------|---------|------------|
| Exposed Ports | All (debugging) | Only HTTPS | HTTPS + HTTP redirect |
| DB Access | Direct | App only | App only |
| Network Type | Single bridge | Multi-tier | Multi-tier + internal |
| Security | Minimal | Medium | Maximum |
| Logging | Debug | Info | Warn/Error |

**Real-World Benefits:**
- Developers can debug locally with full access
- Staging mimics production security
- Production has zero unnecessary exposure
- Same codebase, different configurations

---

### Use Case 5: CI/CD Pipeline with Isolated Testing

**Scenario:** Automated Testing with Complete Environment Isolation

**Jenkins/GitLab CI Pipeline Script:**

```bash
#!/bin/bash
# CI/CD Test Script

BUILD_ID=$(date +%s)
TEST_NETWORK="test-env-${BUILD_ID}"

echo "Creating isolated test environment: ${TEST_NETWORK}"

# Create isolated network for this build
docker network create ${TEST_NETWORK}

# Start test database
docker run -d \
  --name test-db-${BUILD_ID} \
  --network ${TEST_NETWORK} \
  -e POSTGRES_PASSWORD=testpass \
  -e POSTGRES_DB=testdb \
  postgres:15

# Wait for database to be ready
sleep 5

# Run database migrations
docker run --rm \
  --network ${TEST_NETWORK} \
  -e DATABASE_URL=postgres://test-db-${BUILD_ID}:5432/testdb \
  myapp:${GIT_COMMIT} npm run migrate

# Run unit tests
docker run --rm \
  --name unit-tests-${BUILD_ID} \
  --network ${TEST_NETWORK} \
  -e DATABASE_URL=postgres://test-db-${BUILD_ID}:5432/testdb \
  -e NODE_ENV=test \
  myapp:${GIT_COMMIT} npm run test:unit

UNIT_TEST_EXIT_CODE=$?

# Run integration tests
docker run --rm \
  --name integration-tests-${BUILD_ID} \
  --network ${TEST_NETWORK} \
  -e DATABASE_URL=postgres://test-db-${BUILD_ID}:5432/testdb \
  -e NODE_ENV=test \
  myapp:${GIT_COMMIT} npm run test:integration

INTEGRATION_TEST_EXIT_CODE=$?

# Run E2E tests
docker run -d \
  --name app-${BUILD_ID} \
  --network ${TEST_NETWORK} \
  -e DATABASE_URL=postgres://test-db-${BUILD_ID}:5432/testdb \
  myapp:${GIT_COMMIT}

docker run --rm \
  --network ${TEST_NETWORK} \
  -e BASE_URL=http://app-${BUILD_ID}:3000 \
  cypress:latest npm run test:e2e

E2E_TEST_EXIT_CODE=$?

# Cleanup
echo "Cleaning up test environment: ${TEST_NETWORK}"
docker stop test-db-${BUILD_ID} app-${BUILD_ID}
docker rm test-db-${BUILD_ID} app-${BUILD_ID}
docker network rm ${TEST_NETWORK}

# Exit with failure if any test failed
if [ $UNIT_TEST_EXIT_CODE -ne 0 ] || [ $INTEGRATION_TEST_EXIT_CODE -ne 0 ] || [ $E2E_TEST_EXIT_CODE -ne 0 ]; then
  echo "Tests failed!"
  exit 1
fi

echo "All tests passed!"
exit 0
```

**Parallel Build Isolation:**

```
Build #123 (test-env-1698234567)
├── test-db-1698234567
├── app-1698234567
└── Network: 172.20.0.0/16

Build #124 (test-env-1698234589)  ← Running simultaneously
├── test-db-1698234589
├── app-1698234589
└── Network: 172.21.0.0/16

Build #125 (test-env-1698234601)  ← Also running simultaneously
├── test-db-1698234601
├── app-1698234601
└── Network: 172.22.0.0/16
```

**Advantages:**
- ✅ **Parallel Execution:** Multiple builds run simultaneously
- ✅ **Zero Interference:** Each build completely isolated
- ✅ **No Port Conflicts:** Networks handle routing internally
- ✅ **Clean State:** Fresh environment per build
- ✅ **Easy Cleanup:** Remove network = remove all traces
- ✅ **Reproducible:** Same setup every time

**Real-World Example:**
- **Google:** Isolated test environments for every commit
- **Facebook:** Thousands of parallel test runs daily
- **GitHub Actions:** Each job runs in isolated network

---

### Use Case 6: API Gateway / Service Mesh Pattern

**Scenario:** Netflix-Style Architecture with Controlled Entry Point

All external traffic goes through an API Gateway, which routes to internal microservices.

**Implementation:**

```bash
# Create networks
docker network create public-net
docker network create private-services-net

# API Gateway (Kong/Nginx/Traefik) - Entry point
docker run -d --name kong-gateway \
  --network public-net \
  -p 80:8000 \
  -p 443:8443 \
  -p 8001:8001 \
  kong:latest

# Connect gateway to private network
docker network connect private-services-net kong-gateway

# Private microservices (NOT exposed to public)
docker run -d --name auth-service \
  --network private-services-net \
  auth-service:latest

docker run -d --name user-service \
  --network private-services-net \
  user-service:latest

docker run -d --name analytics-service \
  --network private-services-net \
  analytics-service:latest

docker run -d --name billing-service \
  --network private-services-net \
  billing-service:latest
```

**Gateway Configuration (Kong Example):**

```bash
# Configure routes in Kong
# Route /api/auth/* → auth-service
curl -X POST http://localhost:8001/services \
  -d name=auth-service \
  -d url=http://auth-service:3000

curl -X POST http://localhost:8001/services/auth-service/routes \
  -d paths[]=/api/auth

# Route /api/users/* → user-service
curl -X POST http://localhost:8001/services \
  -d name=user-service \
  -d url=http://user-service:4000

curl -X POST http://localhost:8001/services/user-service/routes \
  -d paths[]=/api/users
```

**Traffic Flow:**

```
Internet
   │
   ↓
[Public IP:443]
   │
   ↓
┌──────────────────────────────────┐
│ public-net                       │
│                                  │
│  [Kong Gateway] ← SSL Termination│
│        │                         │
└────────┼──────────────────────────┘
         │ (Dual-homed)
┌────────┼──────────────────────────┐
│  private-services-net            │
│        │                          │
│        ├─→ [auth-service:3000]   │
│        ├─→ [user-service:4000]   │
│        ├─→ [analytics-service]   │
│        └─→ [billing-service]     │
│                                  │
│  Services ONLY accessible via    │
│  Kong Gateway                    │
└──────────────────────────────────┘
```

**Security Features:**
- 🔒 **Single Entry Point:** All traffic through gateway
- 🔒 **SSL Termination:** HTTPS at gateway, HTTP internally
- 🔒 **Rate Limiting:** Gateway enforces limits
- 🔒 **Authentication:** JWT validation at gateway
- 🔒 **Service Protection:** Services not directly accessible
- 🔒 **DDoS Protection:** Gateway handles attack traffic

**Real-World Example:**
- **Netflix:** Zuul API Gateway with internal service mesh
- **Airbnb:** Kong gateway for all API traffic
- **Lyft:** Envoy proxy as service mesh

---

### Use Case 7: WordPress with Database Isolation

**Scenario:** Popular WordPress Deployment Pattern

**Implementation:**

```bash
# Create networks
docker network create wordpress-frontend
docker network create wordpress-backend

# MySQL Database (backend network only)
docker run -d \
  --name wordpress-mysql \
  --network wordpress-backend \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppass \
  -v wordpress-db:/var/lib/mysql \
  mysql:8.0

# WordPress Application (both networks)
docker run -d \
  --name wordpress-app \
  --network wordpress-frontend \
  -e WORDPRESS_DB_HOST=wordpress-mysql \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppass \
  -e WORDPRESS_DB_NAME=wordpress \
  -v wordpress-files:/var/www/html \
  wordpress:latest

docker network connect wordpress-backend wordpress-app

# Nginx Reverse Proxy (frontend network only)
docker run -d \
  --name nginx-proxy \
  --network wordpress-frontend \
  -p 80:80 \
  -p 443:443 \
  -v /etc/letsencrypt:/etc/letsencrypt:ro \
  -v ./nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx:alpine

# Redis Cache (backend network) - Optional but recommended
docker run -d \
  --name wordpress-redis \
  --network wordpress-backend \
  redis:alpine

docker network connect wordpress-backend wordpress-redis
```

**Network Topology:**

```
Internet → [Nginx Proxy] (frontend-net)
              ↓
         [WordPress App] (frontend-net + backend-net)
              ↓
         [MySQL DB] (backend-net only)
         [Redis Cache] (backend-net only)
```

**Security Benefits:**
- MySQL not exposed to internet (even indirectly through WordPress)
- Nginx handles SSL/TLS termination
- Redis cache isolated
- WordPress can't be directly accessed (only through Nginx)

---

### Use Case 8: Multi-Environment on Same Host

**Scenario:** Running Dev, Staging, and Production on Same Server

**Implementation:**

```bash
# Development Environment
docker network create dev-network
docker run -d --name dev-api --network dev-network -p 3001:3000 api:dev
docker run -d --name dev-db --network dev-network -p 5433:5432 postgres:15
docker run -d --name dev-redis --network dev-network -p 6380:6379 redis:alpine

# Staging Environment
docker network create staging-network
docker run -d --name staging-api --network staging-network -p 3002:3000 api:staging
docker run -d --name staging-db --network staging-network postgres:15
docker run -d --name staging-redis --network staging-network redis:alpine

# Production Environment
docker network create prod-network
docker run -d --name prod-api --network prod-network -p 443:3000 api:prod
docker run -d --name prod-db --network prod-network postgres:15
docker run -d --name prod-redis --network prod-network redis:alpine

# Monitoring (shared across all environments)
docker network create monitoring-network
docker run -d --name prometheus --network monitoring-network -p 9090:9090 prom/prometheus
docker run -d --name grafana --network monitoring-network -p 3000:3000 grafana/grafana

# Connect all environments to monitoring
docker network connect monitoring-network dev-api
docker network connect monitoring-network staging-api
docker network connect monitoring-network prod-api
```

**Isolation Matrix:**

```
┌────────────┬────────┬─────────┬──────┬───────────┐
│ Environment│ Network│ App Port│ DB   │ Monitoring│
├────────────┼────────┼─────────┼──────┼───────────┤
│ Development│ dev-net│ :3001   │ :5433│ Connected │
│ Staging    │ stg-net│ :3002   │ N/A  │ Connected │
│ Production │ prd-net│ :443    │ N/A  │ Connected │
└────────────┴────────┴─────────┴──────┴───────────┘
```

**Benefits:**
- ✅ Complete isolation between environments
- ✅ No cross-environment interference
- ✅ Shared monitoring sees all environments
- ✅ Different versions can run simultaneously
- ✅ Easy environment-specific testing

---

### Use Case 9: Message Queue / Event-Driven Architecture

**Scenario:** Microservices with RabbitMQ Event Bus

**Implementation:**

```bash
# Create event bus network
docker network create event-bus-net

# Message Broker (RabbitMQ)
docker run -d --name rabbitmq \
  --network event-bus-net \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=secret \
  rabbitmq:3-management

# Event Producers
docker run -d --name order-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  order-service:latest

docker run -d --name inventory-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  inventory-service:latest

docker run -d --name payment-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  payment-service:latest

# Event Consumers
docker run -d --name email-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  email-service:latest

docker run -d --name notification-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  notification-service:latest

docker run -d --name analytics-service \
  --network event-bus-net \
  -e RABBITMQ_HOST=rabbitmq \
  analytics-service:latest
```

**Event Flow Example:**

```
1. User places order
   ↓
   [order-service] publishes "OrderCreated" event to RabbitMQ
   ↓
   RabbitMQ broadcasts to all subscribers:
   ↓
   ├─→ [inventory-service] ← Reduces stock
   ├─→ [payment-service] ← Processes payment
   ├─→ [email-service] ← Sends confirmation email
   ├─→ [notification-service] ← Sends push notification
   └─→ [analytics-service] ← Logs for reporting
```

**Benefits:**
- ✅ Loose coupling between services
- ✅ Services don't need to know about each other
- ✅ Easy to add new event consumers
- ✅ Asynchronous processing
- ✅ All communication through message broker

---

### Use Case 10: Monitoring & Logging Stack

**Scenario:** Prometheus + Grafana + Loki Observability Stack

**Implementation:**

```bash
# Create monitoring network
docker network create monitoring-net

# Prometheus (metrics collection)
docker run -d --name prometheus \
  --network monitoring-net \
  -p 9090:9090 \
  -v ./prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Grafana (visualization)
docker run -d --name grafana \
  --network monitoring-net \
  -p 3000:3000 \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  grafana/grafana

# Loki (log aggregation)
docker run -d --name loki \
  --network monitoring-net \
  -p 3100:3100 \
  grafana/loki

# Promtail (log collector)
docker run -d --name promtail \
  --network monitoring-net \
  -v /var/log:/var/log \
  -v ./promtail-config.yml:/etc/promtail/config.yml \
  grafana/promtail

# Application containers join monitoring network
docker network connect monitoring-net my-web-app
docker network connect monitoring-net my-api-server
docker network connect monitoring-net my-database
```

**Monitoring Architecture:**

```
┌─────────────────────────────────────────────┐
│         monitoring-net                      │
│                                             │
│  [Prometheus] ←─ Scrapes metrics from apps │
│       ↓                                     │
│  [Grafana] ←──── Visualizes data           │
│       ↑                                     │
│  [Loki] ←─────── Aggregates logs           │
│       ↑                                     │
│  [Promtail] ←─── Collects logs             │
│                                             │
│  Connected Apps:                            │
│  • my-web-app (exposes /metrics)           │
│  • my-api-server (exposes /metrics)        │
│  • my-database (exposes /metrics)          │
└─────────────────────────────────────────────┘
```

**Benefits:**
- ✅ Centralized monitoring for all containers
- ✅ Single network for all observability tools
- ✅ Applications expose metrics internally (not to internet)
- ✅ Easy to add new applications to monitoring

---

## Quick Reference: Network Pattern Selection Guide

### When to Use Which Pattern

| Use Case | Network Pattern | Primary Benefit | Complexity |
|----------|----------------|-----------------|------------|
| **Microservices** | Single custom bridge | Service discovery via DNS | Low |
| **3-Tier App** | Multiple networks | Layer isolation & security | Medium |
| **Database Security** | Internal network | Complete isolation from internet | Low |
| **Dev/Staging/Prod** | Separate networks per env | Environment isolation | Medium |
| **CI/CD Testing** | Ephemeral networks | Test isolation, parallel builds | Medium |
| **API Gateway** | Public + Private networks | Controlled entry point | High |
| **Multi-Tenant** | Network per tenant | Complete tenant isolation | High |
| **Event-Driven** | Shared message bus network | Loose coupling | Medium |
| **Monitoring** | Shared monitoring network | Centralized observability | Low |
| **WordPress/CMS** | Frontend + Backend split | CMS security | Medium |

---

## Network Pattern Decision Tree

```
Start: What are you building?
│
├─ Simple app with 2-3 containers?
│  └─→ Use: Single custom bridge network
│     Example: docker network create myapp-net
│
├─ Multi-tier application?
│  └─→ Use: Multiple networks (one per tier)
│     Example: frontend-net, backend-net, db-net
│
├─ Microservices (5+ services)?
│  └─→ Use: Service mesh pattern
│     Example: public-net + private-services-net
│
├─ Need maximum database security?
│  └─→ Use: Internal network
│     Example: docker network create --internal db-net
│
├─ Running multiple environments?
│  └─→ Use: Separate network per environment
│     Example: dev-net, staging-net, prod-net
│
├─ CI/CD pipeline?
│  └─→ Use: Ephemeral networks (create/destroy per build)
│     Example: test-env-${BUILD_ID}
│
└─ Event-driven architecture?
   └─→ Use: Shared message bus network
      Example: event-bus-net with RabbitMQ/Kafka
```

---

## Best Practices Summary

### ✅ DO:

1. **Use custom networks** instead of default bridge
2. **Name networks descriptively** (e.g., `ecommerce-backend-net` not `net1`)
3. **Use internal networks** for databases
4. **Document network architecture** in README
5. **Use docker-compose** for complex multi-network setups
6. **Leverage DNS** for service discovery (use container names)
7. **Separate concerns** with multiple networks (frontend/backend/database)
8. **Use network aliases** for service groups
9. **Monitor network metrics** (connections, bandwidth)
10. **Plan IP ranges** to avoid conflicts

### ❌ DON'T:

1. **Don't use default bridge** for production
2. **Don't expose database ports** to host (unless absolutely necessary)
3. **Don't put all services** on same network (security risk)
4. **Don't hardcode IP addresses** (use DNS names)
5. **Don't share networks** between unrelated applications
6. **Don't forget cleanup** (remove unused networks)
7. **Don't use host network** unless performance critical
8. **Don't skip network documentation**
9. **Don't mix environments** on same network
10. **Don't ignore network security** in development

---

## Summary

Docker networks enable powerful architectural patterns that solve real business problems:

**Security:** Multi-layer defense with network segmentation  
**Scalability:** Easy to add services without reconfiguration  
**Isolation:** Complete separation between environments and tenants  
**Flexibility:** Same app, different network configs per environment  
**Observability:** Centralized monitoring without exposing metrics publicly  
**Reliability:** Fault isolation prevents cascading failures  

By understanding and applying these patterns, you can build production-ready, secure, scalable containerized applications.

---

