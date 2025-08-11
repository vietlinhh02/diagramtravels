# 04 — Deployment & Infrastructure TravelSense v2

## Cloud Architecture

```mermaid
graph TB
    subgraph "CDN & Edge"
        cdn["CloudFlare CDN"]
        edge["Edge Locations"]
        waf["Web Application Firewall"]
    end

    subgraph "Load Balancing"
        lb["Load Balancer<br/>(ALB/nginx)"]
        ssl["SSL Termination"]
    end

    subgraph "Application Tier"
        app1["App Server 1<br/>(Node.js/Docker)"]
        app2["App Server 2<br/>(Node.js/Docker)"]
        app3["App Server 3<br/>(Node.js/Docker)"]
    end

    subgraph "Microservices"
        auth["Auth Service"]
        trip["Trip Service"]
        ai["AI Service"]
        export["Export Service"]
    end

    subgraph "Data Tier"
        pg["PostgreSQL<br/>(RDS/Managed)"]
        redis["Redis Cluster<br/>(ElastiCache)"]
        s3["Object Storage<br/>(S3/R2)"]
        vector["Vector DB<br/>(Pinecone/Weaviate)"]
    end

    subgraph "External Services"
        llm["LLM APIs<br/>(OpenAI/Anthropic)"]
        maps["Maps APIs"]
        places["Places APIs"]
        weather["Weather APIs"]
    end

    subgraph "Background Processing"
        queue["Message Queue<br/>(SQS/RabbitMQ)"]
        worker1["Worker 1"]
        worker2["Worker 2"]
        scheduler["Cron Scheduler"]
    end

    subgraph "Monitoring"
        logs["Centralized Logging<br/>(ELK/DataDog)"]
        metrics["Metrics<br/>(Prometheus/CloudWatch)"]
        alerts["Alerting<br/>(PagerDuty)"]
    end

    cdn --> edge --> waf --> lb
    lb --> ssl --> app1
    lb --> app2
    lb --> app3

    app1 --> auth
    app1 --> trip
    app1 --> ai
    app1 --> export

    auth --> pg
    trip --> pg
    ai --> vector
    export --> s3

    app1 --> redis
    app2 --> redis
    app3 --> redis

    ai --> llm
    trip --> maps
    trip --> places
    trip --> weather

    app1 --> queue
    queue --> worker1
    queue --> worker2
    scheduler --> queue

    app1 --> logs
    app1 --> metrics
    metrics --> alerts

    %% Styling
    classDef edge fill:#e3f2fd
    classDef app fill:#e8f5e8
    classDef data fill:#fff3e0
    classDef external fill:#fce4ec
    classDef bg fill:#f3e5f5
    classDef monitor fill:#e0f2f1

    class cdn,edge,waf,lb,ssl edge
    class app1,app2,app3,auth,trip,ai,export app
    class pg,redis,s3,vector data
    class llm,maps,places,weather external
    class queue,worker1,worker2,scheduler bg
    class logs,metrics,alerts monitor
```

## Container Strategy

### Docker Configuration
```mermaid
flowchart LR
    subgraph "Base Images"
        A[node:18-alpine]
        B[postgres:15]
        C[redis:7-alpine]
        D[nginx:alpine]
    end

    subgraph "Application Images"
        E[travel-sense-api]
        F[travel-sense-worker]
        G[travel-sense-proxy]
    end

    subgraph "Development"
        H[docker-compose.dev.yml]
        I[Hot Reload]
        J[Debug Mode]
    end

    subgraph "Production"
        K[docker-compose.prod.yml]
        L[Multi-stage Build]
        M[Health Checks]
    end

    A --> E
    A --> F
    D --> G
    B --> H
    C --> H

    E --> K
    F --> K
    G --> K

    H --> I --> J
    K --> L --> M
```

### Multi-stage Dockerfile
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Runtime stage
FROM node:18-alpine AS runtime
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .
USER nextjs
EXPOSE 3000
CMD ["npm", "start"]
```

## Kubernetes Deployment

### Service Mesh Architecture
```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Namespace: travel-sense"
            subgraph "Frontend"
                fe1[web-app-1]
                fe2[web-app-2]
                fe-svc[web-service]
            end

            subgraph "API Gateway"
                gw1[api-gateway-1]
                gw2[api-gateway-2]
                gw-svc[gateway-service]
            end

            subgraph "Microservices"
                auth-pod[auth-service]
                trip-pod[trip-service]
                ai-pod[ai-service]
                export-pod[export-service]
            end

            subgraph "Data Services"
                pg-pod[postgresql]
                redis-pod[redis-cluster]
            end

            subgraph "Workers"
                worker1[worker-1]
                worker2[worker-2]
                queue-pod[message-queue]
            end
        end

        subgraph "System Services"
            ingress[Ingress Controller]
            service-mesh[Istio/Linkerd]
            monitoring[Prometheus]
        end
    end

    fe1 --> fe-svc
    fe2 --> fe-svc
    gw1 --> gw-svc
    gw2 --> gw-svc

    fe-svc --> gw-svc
    gw-svc --> auth-pod
    gw-svc --> trip-pod
    gw-svc --> ai-pod
    gw-svc --> export-pod

    trip-pod --> pg-pod
    auth-pod --> redis-pod
    ai-pod --> redis-pod

    trip-pod --> queue-pod
    queue-pod --> worker1
    queue-pod --> worker2

    ingress --> fe-svc
    service-mesh -.-> fe-svc
    service-mesh -.-> gw-svc
    monitoring -.-> ingress
```

### Resource Allocation
```yaml
# Kubernetes Resource Limits
apiVersion: v1
kind: ResourceQuota
metadata:
  name: travel-sense-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
    persistentvolumeclaims: "5"
```

## CI/CD Pipeline

```mermaid
flowchart LR
    subgraph "Source Control"
        A[Git Push]
        B[Pull Request]
        C[Main Branch]
    end

    subgraph "CI Pipeline"
        D[Code Quality<br/>ESLint, Prettier]
        E[Unit Tests<br/>Jest, Vitest]
        F[Integration Tests<br/>Playwright]
        G[Security Scan<br/>Snyk, SonarQube]
    end

    subgraph "Build & Package"
        H[Docker Build]
        I[Image Scan<br/>Trivy, Clair]
        J[Push to Registry<br/>ECR, DockerHub]
    end

    subgraph "Deploy Stages"
        K[Dev Environment]
        L[Staging Environment]
        M[Production Environment]
    end

    subgraph "Post-Deploy"
        N[Health Checks]
        O[Smoke Tests]
        P[Performance Tests]
        Q[Rollback if Needed]
    end

    A --> D
    B --> D
    C --> D

    D --> E --> F --> G
    G --> H --> I --> J

    J --> K
    K -->|Auto| L
    L -->|Manual Approval| M

    M --> N --> O --> P
    P -->|Failed| Q
    Q --> L
```

### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test
      - run: npm run test:e2e

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t travel-sense:${{ github.sha }} .
      - name: Push to registry
        run: docker push travel-sense:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to Kubernetes
        run: kubectl set image deployment/api api=travel-sense:${{ github.sha }}
```

## Environment Configuration

### Environment Separation
```mermaid
graph TB
    subgraph "Development"
        dev-db[(Dev DB)]
        dev-redis[(Dev Redis)]
        dev-api[Dev API]
        dev-fe[Dev Frontend]
    end

    subgraph "Staging"
        stg-db[(Staging DB)]
        stg-redis[(Staging Redis)]
        stg-api[Staging API]
        stg-fe[Staging Frontend]
    end

    subgraph "Production"
        prod-db[(Prod DB)]
        prod-redis[(Prod Redis)]
        prod-api[Prod API]
        prod-fe[Prod Frontend]
    end

    subgraph "Shared Services"
        llm-api[LLM APIs]
        maps-api[Maps APIs]
        monitoring[Monitoring]
    end

    dev-api --> dev-db
    dev-api --> dev-redis
    stg-api --> stg-db
    stg-api --> stg-redis
    prod-api --> prod-db
    prod-api --> prod-redis

    dev-api --> llm-api
    stg-api --> llm-api
    prod-api --> llm-api

    monitoring -.-> dev-api
    monitoring -.-> stg-api
    monitoring -.-> prod-api
```

### Configuration Management
```mermaid
flowchart TD
    A[Environment Variables] --> B[Config Service]
    C[Kubernetes Secrets] --> B
    D[HashiCorp Vault] --> B
    E[AWS Secrets Manager] --> B

    B --> F[Application Config]
    F --> G[Database Connections]
    F --> H[API Keys]
    F --> I[Feature Flags]
    F --> J[Rate Limits]

    G --> K[PostgreSQL]
    H --> L[External APIs]
    I --> M[A/B Testing]
    J --> N[Throttling]
```

## Monitoring & Observability

### Metrics Dashboard
```mermaid
graph TB
    subgraph "Application Metrics"
        A[Request Rate]
        B[Response Time]
        C[Error Rate]
        D[Active Users]
    end

    subgraph "Infrastructure Metrics"
        E[CPU Usage]
        F[Memory Usage]
        G[Disk I/O]
        H[Network Traffic]
    end

    subgraph "Business Metrics"
        I[Trip Completions]
        J[User Satisfaction]
        K[AI Cost per Trip]
        L[Revenue per User]
    end

    subgraph "Alerting Rules"
        M[SLA Violations]
        N[Resource Exhaustion]
        O[Security Incidents]
        P[Business KPI Drops]
    end

    A --> M
    B --> M
    C --> M
    E --> N
    F --> N
    I --> P
    J --> P
```

### Log Aggregation
```mermaid
flowchart LR
    subgraph "Log Sources"
        A[Application Logs]
        B[Access Logs]
        C[Error Logs]
        D[Audit Logs]
    end

    subgraph "Collection"
        E[Fluentd/Fluent Bit]
        F[Log Rotation]
        G[Structured Logging]
    end

    subgraph "Storage & Analysis"
        H[Elasticsearch]
        I[Kibana Dashboard]
        J[Log Retention Policy]
    end

    subgraph "Alerting"
        K[Error Rate Spikes]
        L[Security Events]
        M[Performance Degradation]
    end

    A --> E
    B --> E
    C --> E
    D --> E

    E --> F --> G
    G --> H --> I
    I --> J

    H --> K
    H --> L
    H --> M
```

## Security Hardening

### Security Layers
```mermaid
graph TB
    subgraph "Network Security"
        A[WAF Protection]
        B[DDoS Mitigation]
        C[VPC/Private Subnets]
        D[Security Groups]
    end

    subgraph "Application Security"
        E[JWT Authentication]
        F[Rate Limiting]
        G[Input Validation]
        H[HTTPS Encryption]
    end

    subgraph "Data Security"
        I[Encryption at Rest]
        J[Encryption in Transit]
        K[PII Anonymization]
        L[Backup Encryption]
    end

    subgraph "Access Control"
        M[RBAC]
        N[API Key Management]
        O[Audit Logging]
        P[Secrets Management]
    end

    A --> E --> I --> M
    B --> F --> J --> N
    C --> G --> K --> O
    D --> H --> L --> P
```

## Disaster Recovery

### Backup Strategy
```mermaid
timeline
    title Backup & Recovery Timeline

    Daily : Point-in-time Recovery
          : Automated DB Snapshots
          : Log File Backups

    Weekly : Full System Backup
           : Cross-region Replication
           : Recovery Testing

    Monthly : Disaster Recovery Drill
            : Backup Verification
            : RTO/RPO Assessment

    Quarterly : Business Continuity Review
              : Update Recovery Procedures
              : Stakeholder Training
```

### Multi-region Deployment
- **Primary Region**: US-East (Virginia)
- **Secondary Region**: EU-West (Ireland)
- **Failover Time**: < 5 minutes (automated)
- **Data Sync**: Real-time replication with 15-second lag
- **Recovery Objectives**: RTO < 1 hour, RPO < 5 minutes
