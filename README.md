# 🛒 E-Commerce Microservices Platform

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2.0-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Angular](https://img.shields.io/badge/Angular-16+-red.svg)](https://angular.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-blue.svg)](https://kubernetes.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A production-ready, cloud-native e-commerce application built using microservices architecture with Spring Boot backend, Angular frontend, and deployed on Kubernetes.

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Microservices](#microservices)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [Deployment](#deployment)
- [Monitoring](#monitoring)
- [Security](#security)
- [Contributing](#contributing)
- [License](#license)

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Angular Frontend                         │
│                    (Port: 4200)                              │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   API Gateway                                │
│              (Spring Cloud Gateway)                          │
│                    (Port: 8080)                              │
└──────┬────────┬────────┬────────┬────────┬─────────┬────────┘
       │        │        │        │        │         │
       ▼        ▼        ▼        ▼        ▼         ▼
┌──────────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────┐
│   Auth   │ │Product│ │ Cart │ │Order │ │Payment│ │Inventory │
│ Service  │ │Service│ │Service│ │Service│ │Service│ │ Service  │
│  :8081   │ │ :8082 │ │ :8083│ │ :8084│ │ :8085 │ │  :8086   │
└────┬─────┘ └───┬───┘ └───┬──┘ └───┬──┘ └───┬───┘ └────┬─────┘
     │           │         │        │        │          │
     ▼           ▼         ▼        ▼        ▼          ▼
┌──────────┐ ┌──────────┐ ┌─────┐ ┌──────────┐        │
│PostgreSQL│ │PostgreSQL│ │Redis│ │PostgreSQL│        │
└──────────┘ └──────────┘ └─────┘ └──────────┘        │
                                    │                  │
                                    └──────┬───────────┘
                                           ▼
                                  ┌─────────────────┐
                                  │ Kafka / RabbitMQ│
                                  └─────────────────┘
```

### Key Architectural Patterns

- **API Gateway Pattern**: Single entry point for all client requests
- **Database per Service**: Each microservice has its own database
- **Event-Driven Architecture**: Async communication using Kafka/RabbitMQ
- **Service Discovery**: Eureka for service registration and discovery
- **Circuit Breaker**: Resilience4j for fault tolerance
- **CQRS**: Command Query Responsibility Segregation for Order service

---

## ✨ Features

### User Features
- ✅ User registration and authentication (JWT-based)
- ✅ Browse products with search and filtering
- ✅ Shopping cart management
- ✅ Order placement and tracking
- ✅ Payment processing integration
- ✅ Order history and status updates

### Admin Features
- ✅ Product management (CRUD operations)
- ✅ Inventory management
- ✅ Order management
- ✅ User management

### Technical Features
- ✅ Microservices architecture
- ✅ RESTful APIs
- ✅ JWT authentication & authorization
- ✅ API rate limiting
- ✅ Distributed tracing
- ✅ Centralized logging
- ✅ Health checks and monitoring
- ✅ Auto-scaling capabilities
- ✅ CI/CD pipeline

---

## 🛠️ Tech Stack

### Backend
- **Framework**: Spring Boot 3.2.0
- **Language**: Java 17
- **Security**: Spring Security, OAuth2, JWT
- **API Gateway**: Spring Cloud Gateway
- **Service Discovery**: Netflix Eureka
- **Config Server**: Spring Cloud Config
- **Resilience**: Resilience4j (Circuit Breaker, Retry, Rate Limiter)
- **API Documentation**: SpringDoc OpenAPI (Swagger)

### Frontend
- **Framework**: Angular 16+
- **UI Library**: Angular Material
- **State Management**: RxJS
- **HTTP Client**: Angular HttpClient
- **Routing**: Angular Router

### Databases
- **Relational**: PostgreSQL (User, Product, Order, Inventory services)
- **Cache**: Redis (Cart service)
- **Search**: Elasticsearch (Optional for product search)

### Message Broker
- **Primary**: Apache Kafka
- **Alternative**: RabbitMQ

### DevOps & Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes (EKS/AKS/GKE)
- **CI/CD**: Jenkins / GitHub Actions / GitLab CI
- **GitOps**: ArgoCD / Flux
- **IaC**: Terraform / Helm Charts
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana) / Loki
- **Tracing**: Jaeger / Zipkin
- **Service Mesh**: Istio (Optional)

---

## 🔧 Microservices

### 1. API Gateway (Port: 8080)
**Responsibilities:**
- Route requests to appropriate services
- JWT token validation
- Rate limiting
- Load balancing

**Endpoints:**
- `/api/auth/**` → Auth Service
- `/api/products/**` → Product Service
- `/api/cart/**` → Cart Service
- `/api/orders/**` → Order Service
- `/api/payments/**` → Payment Service

### 2. Auth Service (Port: 8081)
**Responsibilities:**
- User registration and login
- JWT token generation and validation
- Role-based access control (RBAC)

**Database:** PostgreSQL

**Key Endpoints:**
```
POST   /api/auth/register
POST   /api/auth/login
GET    /api/auth/validate
POST   /api/auth/refresh
```

### 3. Product Service (Port: 8082)
**Responsibilities:**
- Product catalog management
- Category management
- Product search and filtering
- Inventory checking

**Database:** PostgreSQL

**Key Endpoints:**
```
GET    /api/products
GET    /api/products/{id}
POST   /api/products          [ADMIN]
PUT    /api/products/{id}     [ADMIN]
DELETE /api/products/{id}     [ADMIN]
GET    /api/products/search?q={query}
```

### 4. Cart Service (Port: 8083)
**Responsibilities:**
- Shopping cart management
- Add/remove/update cart items
- Cart persistence

**Database:** Redis (In-memory cache)

**Key Endpoints:**
```
GET    /api/cart
POST   /api/cart/items
PUT    /api/cart/items/{productId}
DELETE /api/cart/items/{productId}
DELETE /api/cart
```

### 5. Order Service (Port: 8084)
**Responsibilities:**
- Order placement
- Order status management
- Order history
- Emit order events to Kafka

**Database:** PostgreSQL

**Key Endpoints:**
```
POST   /api/orders
GET    /api/orders
GET    /api/orders/{id}
PUT    /api/orders/{id}/status  [ADMIN]
GET    /api/orders/user/{userId}
```

**Order Status Flow:**
```
PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED
                    ↓
                 CANCELLED
```

### 6. Payment Service (Port: 8085)
**Responsibilities:**
- Payment processing
- Payment gateway integration (Stripe/Razorpay)
- Payment confirmation
- Refund handling

**Database:** PostgreSQL

**Key Endpoints:**
```
POST   /api/payments/process
GET    /api/payments/{orderId}
POST   /api/payments/{id}/refund  [ADMIN]
```

### 7. Inventory Service (Port: 8086)
**Responsibilities:**
- Stock management
- Reserve/release inventory
- Consume order events from Kafka
- Low stock notifications

**Database:** PostgreSQL

**Key Endpoints:**
```
GET    /api/inventory/{productId}
PUT    /api/inventory/{productId}  [ADMIN]
POST   /api/inventory/reserve
POST   /api/inventory/release
```

---

## 📦 Prerequisites

### Required Software
- **Java**: 17 or higher
- **Node.js**: 18.x or higher
- **npm**: 9.x or higher
- **Maven**: 3.8+
- **Docker**: 20.x+
- **Docker Compose**: 2.x+
- **Kubernetes**: 1.28+ (kubectl CLI)
- **Helm**: 3.x+

### Optional Tools
- **Postman**: For API testing
- **Lens**: Kubernetes IDE
- **k9s**: Terminal-based Kubernetes UI
- **IntelliJ IDEA / VS Code**: IDEs

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/ecommerce-microservices.git
cd ecommerce-microservices
```

### 2. Setup Infrastructure with Docker Compose

```bash
# Start all infrastructure services (databases, message brokers, etc.)
docker-compose -f docker-compose-infra.yml up -d

# Verify all containers are running
docker-compose -f docker-compose-infra.yml ps
```

This will start:
- PostgreSQL (ports: 5432-5436)
- Redis (port: 6379)
- Kafka + Zookeeper (port: 9092)
- Eureka Server (port: 8761)

### 3. Build and Run Backend Services

#### Option A: Run Individually (Development)

```bash
# Auth Service
cd auth-service
./mvnw clean install
./mvnw spring-boot:run

# Product Service
cd ../product-service
./mvnw clean install
./mvnw spring-boot:run

# Repeat for other services...
```

#### Option B: Run All Services with Docker Compose

```bash
# Build all services
./build-all.sh

# Start all services
docker-compose up -d

# Check logs
docker-compose logs -f
```

### 4. Run Frontend Application

```bash
cd frontend-angular
npm install
ng serve

# Application will be available at http://localhost:4200
```

### 5. Access Applications

| Service | URL | Credentials |
|---------|-----|-------------|
| Frontend | http://localhost:4200 | - |
| API Gateway | http://localhost:8080 | - |
| Eureka Dashboard | http://localhost:8761 | - |
| Swagger UI | http://localhost:8080/swagger-ui.html | - |

---

## 📁 Project Structure

```
ecommerce-microservices/
│
├── api-gateway/                 # API Gateway service
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── auth-service/                # Authentication service
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/ecommerce/auth/
│   │   │   │       ├── controller/
│   │   │   │       ├── service/
│   │   │   │       ├── repository/
│   │   │   │       ├── entity/
│   │   │   │       ├── dto/
│   │   │   │       └── config/
│   │   │   └── resources/
│   │   │       └── application.yml
│   │   └── test/
│   ├── pom.xml
│   └── Dockerfile
│
├── product-service/             # Product catalog service
├── cart-service/                # Shopping cart service
├── order-service/               # Order management service
├── payment-service/             # Payment processing service
├── inventory-service/           # Inventory management service
│
├── frontend-angular/            # Angular frontend
│   ├── src/
│   │   ├── app/
│   │   │   ├── core/           # Core modules (guards, interceptors)
│   │   │   ├── modules/
│   │   │   │   ├── auth/
│   │   │   │   ├── product/
│   │   │   │   ├── cart/
│   │   │   │   └── order/
│   │   │   ├── shared/         # Shared components
│   │   │   └── app.module.ts
│   │   ├── assets/
│   │   └── environments/
│   ├── angular.json
│   ├── package.json
│   └── Dockerfile
│
├── infrastructure/
│   ├── docker/
│   │   ├── docker-compose-infra.yml
│   │   └── docker-compose.yml
│   ├── kubernetes/
│   │   ├── namespace.yaml
│   │   ├── configmaps/
│   │   ├── secrets/
│   │   ├── deployments/
│   │   ├── services/
│   │   ├── ingress/
│   │   └── hpa/
│   ├── helm/
│   │   └── ecommerce/
│   │       ├── Chart.yaml
│   │       ├── values.yaml
│   │       └── templates/
│   └── terraform/
│       ├── main.tf
│       ├── variables.tf
│       └── modules/
│
├── ci-cd/
│   ├── jenkins/
│   │   └── Jenkinsfile
│   ├── github-actions/
│   │   └── .github/workflows/
│   └── gitlab-ci/
│       └── .gitlab-ci.yml
│
├── monitoring/
│   ├── prometheus/
│   │   └── prometheus.yml
│   ├── grafana/
│   │   └── dashboards/
│   └── elk/
│       └── logstash.conf
│
├── docs/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   └── CONTRIBUTING.md
│
├── scripts/
│   ├── build-all.sh
│   ├── deploy-k8s.sh
│   └── cleanup.sh
│
├── .gitignore
├── README.md
└── LICENSE
```

---

## ⚙️ Configuration

### Environment Variables

Each microservice requires the following environment variables:

```bash
# Database Configuration
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/dbname
SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=password

# Service Discovery
EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://localhost:8761/eureka/

# Kafka Configuration
SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092

# JWT Configuration
JWT_SECRET=your-secret-key-here-min-256-bits
JWT_EXPIRATION=86400000

# Redis Configuration (Cart Service)
SPRING_REDIS_HOST=localhost
SPRING_REDIS_PORT=6379
```

### Application Properties

**Example: auth-service/src/main/resources/application.yml**

```yaml
server:
  port: 8081

spring:
  application:
    name: auth-service
  
  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/auth_db}
    username: ${SPRING_DATASOURCE_USERNAME:postgres}
    password: ${SPRING_DATASOURCE_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8081

eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_CLIENT_SERVICEURL_DEFAULTZONE:http://localhost:8761/eureka/}
  instance:
    preferIpAddress: true

jwt:
  secret: ${JWT_SECRET:your-secret-key-here-min-256-bits}
  expiration: ${JWT_EXPIRATION:86400000}

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

---

## 📚 API Documentation

### Authentication

All protected endpoints require a JWT token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

### Sample API Requests

#### 1. User Registration

```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123",
    "firstName": "John",
    "lastName": "Doe"
  }'
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "email": "user@example.com",
  "role": "USER"
}
```

#### 2. User Login

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "password123"
  }'
```

#### 3. Get Products

```bash
curl -X GET "http://localhost:8080/api/products?page=0&size=10" \
  -H "Authorization: Bearer <token>"
```

#### 4. Add to Cart

```bash
curl -X POST http://localhost:8080/api/cart/items \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": 1,
    "quantity": 2
  }'
```

#### 5. Place Order

```bash
curl -X POST http://localhost:8080/api/orders \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [
      {
        "productId": 1,
        "quantity": 2,
        "price": 29.99
      }
    ],
    "shippingAddress": {
      "street": "123 Main St",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001",
      "country": "USA"
    }
  }'
```

### Swagger Documentation

Access interactive API documentation at:
- **API Gateway Swagger**: http://localhost:8080/swagger-ui.html
- **Individual Services**: http://localhost:{port}/swagger-ui.html

---

## 🚢 Deployment

### Docker Deployment

#### 1. Build Docker Images

```bash
# Build all services
./scripts/build-all.sh

# Or build individually
cd auth-service
docker build -t ecommerce/auth-service:latest .
```

#### 2. Push to Registry

```bash
# Tag images
docker tag ecommerce/auth-service:latest your-registry/auth-service:latest

# Push to registry
docker push your-registry/auth-service:latest
```

#### 3. Run with Docker Compose

```bash
docker-compose up -d
```

### Kubernetes Deployment

#### 1. Setup Namespace

```bash
kubectl create namespace ecommerce
```

#### 2. Deploy with Kubectl

```bash
# Apply all configurations
kubectl apply -f infrastructure/kubernetes/ -n ecommerce

# Check deployment status
kubectl get pods -n ecommerce
kubectl get svc -n ecommerce
```

#### 3. Deploy with Helm

```bash
# Install the chart
helm install ecommerce infrastructure/helm/ecommerce -n ecommerce

# Upgrade
helm upgrade ecommerce infrastructure/helm/ecommerce -n ecommerce

# Uninstall
helm uninstall ecommerce -n ecommerce
```

#### 4. Access Application

```bash
# Get LoadBalancer IP
kubectl get svc api-gateway -n ecommerce

# Port forward (for local testing)
kubectl port-forward svc/api-gateway 8080:8080 -n ecommerce
```

### Terraform Infrastructure

```bash
cd infrastructure/terraform

# Initialize Terraform
terraform init

# Plan deployment
terraform plan

# Apply infrastructure
terraform apply

# Destroy infrastructure
terraform destroy
```

---

## 📊 Monitoring

### Prometheus & Grafana

#### 1. Deploy Monitoring Stack

```bash
# Using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

# Access Grafana
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
```

**Grafana Credentials:**
- Username: `admin`
- Password: Get from secret: `kubectl get secret prometheus-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode`

#### 2. Import Dashboards

Pre-configured dashboards available in `monitoring/grafana/dashboards/`:
- **Spring Boot Metrics**: JVM, HTTP, database metrics
- **Kafka Metrics**: Message throughput, lag
- **Kubernetes Metrics**: Pod, node, cluster metrics

### Health Checks

All services expose health endpoints:

```bash
# Check service health
curl http://localhost:8081/actuator/health

# Check all actuator endpoints
curl http://localhost:8081/actuator
```

### Distributed Tracing

#### Jaeger Setup

```bash
# Deploy Jaeger
kubectl apply -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/main/deploy/crds/jaegertracing.io_jaegers_crd.yaml

# Access Jaeger UI
kubectl port-forward svc/jaeger-query 16686:16686 -n monitoring
```

Visit: http://localhost:16686

---

## 🔒 Security

### Authentication & Authorization

- **JWT Tokens**: Stateless authentication
- **Token Expiration**: 24 hours (configurable)
- **Role-Based Access Control (RBAC)**:
  - `USER`: Access to shopping features
  - `ADMIN`: Full access including product/inventory management

### API Security

- **Rate Limiting**: Configured in API Gateway
- **CORS**: Configurable origins
- **HTTPS**: TLS termination at Ingress
- **SQL Injection Protection**: Prepared statements via JPA
- **XSS Protection**: Content Security Policy headers

### Secrets Management

```bash
# Create secrets in Kubernetes
kubectl create secret generic jwt-secret \
  --from-literal=JWT_SECRET=your-secret-key \
  -n ecommerce

kubectl create secret generic db-credentials \
  --from-literal=username=postgres \
  --from-literal=password=your-db-password \
  -n ecommerce
```

### Security Scanning

```bash
# Scan Docker images with Trivy
trivy image ecommerce/auth-service:latest

# Dependency check with OWASP
mvn org.owasp:dependency-check-maven:check
```

---

## 🧪 Testing

### Unit Tests

```bash
# Run tests for all services
./mvnw test

# Run tests for specific service
cd auth-service
./mvnw test
```

### Integration Tests

```bash
# Run integration tests
./mvnw verify -P integration-tests
```

### Load Testing

```bash
# Using Apache Bench
ab -n 1000 -c 10 http://localhost:8080/api/products

# Using K6
k6 run scripts/load-test.js
```

### Frontend Tests

```bash
cd frontend-angular

# Unit tests
ng test

# E2E tests
ng e2e
```

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build with Maven
        run: mvn clean install
      - name: Run Tests
        run: mvn test
      - name: Build Docker Images
        run: ./scripts/build-all.sh
      - name: Push to Registry
        run: ./scripts/push-images.sh
      
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: kubectl apply -f infrastructure/kubernetes/
```

### Pipeline Stages

1. **Build**: Compile code and create artifacts
2. **Test**: Run unit and integration tests
3. **Security Scan**: Trivy, SonarQube, OWASP
4. **Docker Build**: Create container images
5. **Push**: Push images to registry
6. **Deploy**: Deploy to Kubernetes cluster
7. **Smoke Tests**: Verify deployment

---

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/amazing-feature`
3. **Commit your changes**: `git commit -m 'Add amazing feature'`
4. **Push to branch**: `git push origin feature/amazing-feature`
5. **Open a Pull Request**

### Coding Standards

- Follow Java code conventions
- Write unit tests for new features
- Update documentation
- Use meaningful commit messages

### Pull Request Process

1. Update the README.md with details of changes
2. Update the API documentation if needed
3. Ensure all tests pass
4. Request review from maintainers

---

## 📖 Additional Documentation

- [Architecture Details](docs/ARCHITECTURE.md)
- [API Specification](docs/API.md)
- [Deployment Guide](docs/DEPLOYMENT.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)

---

## 📞 Support

For support and questions:

- **Issues**: [GitHub Issues](https://github.com/yourusername/ecommerce-microservices/issues)
- **Email**: support@example.com
- **Documentation**: [Wiki](https://github.com/yourusername/ecommerce-microservices/wiki)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Spring Boot Team
- Angular Team
- Kubernetes Community
- All contributors

---

## 🗺️ Roadmap

- [ ] Add GraphQL support
- [ ] Implement recommendation engine
- [ ] Add multi-language support
- [ ] Implement advanced analytics
- [ ] Add mobile app (React Native)
- [ ] Implement AI-powered chatbot
- [ ] Add social media integration
- [ ] Implement advanced search with Elasticsearch

---

**Made with ❤️ by [Your Team Name]**

**⭐ Star this repo if you find it helpful!**
