# ============================================================================
# CFI PLATFORM - SYSTEM ARCHITECTURE
# Complete technical design, infrastructure, and deployment strategy
# ============================================================================

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLIENT LAYER                                 │
├──────────────┬──────────────────┬──────────────────────────────┤
│ Merchant     │ Admin Panel      │ Mobile App                   │
│ Dashboard    │ (React)          │ (React Native)               │
│ (Next.js)    │                  │                              │
└──────────────┴──────────────────┴──────────────────────────────┘
               │                      │                  │
        ┌──────▼──────────────────────▼──────────────────▼────┐
        │           API GATEWAY (Kong/Nginx)                  │
        │  - Rate Limiting                                    │
        │  - Authentication                                   │
        │  - Request Routing                                  │
        │  - SSL/TLS Termination                              │
        └──────┬─────────────────────────────────────────────┘
               │
    ┌──────────┼──────────────┐
    │          │              │
┌───▼──┐  ┌───▼────┐  ┌──────▼────┐
│REST  │  │GraphQL │  │ Websocket │
│API   │  │API     │  │ (Real-time)
└───┬──┘  └───┬────┘  └──────┬────┘
    │         │              │
    └─────────┴──────────────┘
              │
    ┌─────────▼──────────────┐
    │  Core Services Layer   │
    ├────────────────────────┤
    │ • Auth Service         │
    │ • Merchant Service     │
    │ • Transaction Service  │
    │ • Settlement Service   │
    │ • Wallet Service       │
    │ • Treasury Service     │
    │ • Reporting Service    │
    └─────────┬──────────────┘
              │
    ┌─────────┴──────────────────┐
    │                            │
┌───▼──────┐  ┌────────────┐  ┌──▼─────────┐
│ Risk &   │  │ Blockchain │  │ Treasury   │
│ Fraud    │  │ Listener   │  │ Engine     │
│ Engine   │  │ Service    │  │            │
└───┬──────┘  └────────────┘  └──┬─────────┘
    │                            │
    └─────────┬──────────────────┘
              │
    ┌─────────▼──────────────┐
    │ Data & Persistence     │
    ├────────────────────────┤
    │ • PostgreSQL (Primary) │
    │ • Redis (Cache/Queue)  │
    │ • Elasticsearch (Logs) │
    │ • S3 (Storage)         │
    └─────────┬──────────────┘
              │
    ┌─────────┴──────────────────┐
    │                            │
┌───▼────┐  ┌─────────┐  ┌──────▼───┐
│Bitcoin │  │Ethereum │  │Polygon   │
│Network │  │Network  │  │Network   │
└────────┘  └─────────┘  └──────────┘
```

---

## 1. Frontend Architecture

### Web Application (Next.js)

**Structure:**
```
apps/web/
├── public/
│   ├── images/
│   ├── icons/
│   └── fonts/
├── src/
│   ├── pages/
│   │   ├── dashboard/
│   │   ├── transactions/
│   │   ├── settlements/
│   │   └── settings/
│   ├── components/
│   │   ├── Layout/
│   │   ├── Dashboard/
│   │   ├── Forms/
│   │   └── Charts/
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useTransaction.ts
│   │   └── useSettlement.ts
│   ├── styles/
│   ├── utils/
│   │   ├── api.ts
│   │   ├── validation.ts
│   │   └── formatting.ts
│   └── store/
│       ├── authSlice.ts
│       ├── merchantSlice.ts
│       └── transactionSlice.ts
├── package.json
└── tsconfig.json
```

**Tech Stack:**
- Framework: Next.js 14+ (React 18)
- Styling: TailwindCSS
- State: Redux Toolkit
- HTTP Client: Axios
- Forms: React Hook Form
- Charts: Chart.js / D3.js
- Auth: NextAuth.js
- TypeScript: Full type coverage

### Mobile Application (React Native)

**Structure:**
```
apps/mobile/
├── app/
│   ├── (auth)/
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (app)/
│   │   ├── dashboard.tsx
│   │   ├── transactions.tsx
│   │   └── settings.tsx
│   └── _layout.tsx
├── components/
├── hooks/
├── services/
├── store/
└── navigation/
```

**Tech Stack:**
- Framework: React Native with Expo
- Navigation: React Navigation
- State: Redux
- HTTP: Axios
- Biometric: React Native Biometrics
- Storage: AsyncStorage

### Admin Dashboard (Next.js)

**Structure:**
```
apps/admin/
├── src/
│   ├── pages/
│   │   ├── risk/
│   │   ├── compliance/
│   │   ├── merchants/
│   │   ├── treasury/
│   │   └── reports/
│   ├── components/
│   ├── modules/
│   │   ├── risk/
│   │   ├── treasury/
│   │   ├── payouts/
│   │   ├── fraud/
│   │   └── logs/
│   └── utils/
```

---

## 2. Backend Architecture

### Core API Service

**Entry Point: `services/api/src/app.js`**

```javascript
Express.js Server
├── Middleware
│   ├── Authentication (JWT)
│   ├── Authorization (RBAC)
│   ├── Rate Limiting
│   ├── Request Logging
│   └── Error Handling
├── Routes
│   ├── /api/auth/*
│   ├── /api/merchants/*
│   ├── /api/transactions/*
│   ├── /api/settlements/*
│   ├── /api/risk/*
│   ├── /api/treasury/*
│   ├── /api/admin/*
│   └── /api/webhooks/*
├── Controllers
│   └── Business Logic
├── Services
│   └── Domain Logic
├── Models (Prisma)
│   └── Database Access
└── Utilities
    └── Helpers & Common Functions
```

### Microservices

#### 1. Blockchain Listener Service
```
Purpose: Real-time blockchain monitoring

Components:
- Bitcoin Node Interface (RPC)
- Ethereum Node Interface (Web3.js)
- Polygon Listener (Alchemy API)
- Transaction Detector
- Confirmation Tracker
- Event Broadcaster (Redis Pub/Sub)

Triggers:
- Wallet address monitoring
- Block confirmation updates
- Mempool transaction tracking

Output:
- Confirmed transaction events
- Settlement eligibility updates
- Error/failure notifications
```

#### 2. Risk & Fraud Engine
```
Purpose: Real-time threat detection & scoring

Components:
- Transaction Analyzer
- Velocity Checker
- Amount Anomaly Detector
- AML/OFAC Screener
- Pattern Memory
- ML Model (optional)
- Risk Event Escalator

Inputs:
- Transaction details
- Merchant profile
- Historical patterns

Outputs:
- Risk score (0-100)
- Risk events
- Escalation triggers
- Action recommendations
```

#### 3. Treasury Engine
```
Purpose: Automated settlement & liquidity management

Components:
- Settlement Orchestrator
- Wallet Monitor
- Rebalance Controller
- Multi-sig Handler
- Bank Interface
- Liquidity Calculator

Scheduled Tasks:
- Daily settlement (2 AM UTC)
- Hourly rebalancing check
- Liquidity monitoring
- Cold storage verification

Interfaces:
- Bank APIs (ACH, Wire)
- Blockchain Networks
- Internal APIs
```

---

## 3. Data Layer

### PostgreSQL Database

**Primary Datastore:**
- Relational data model
- ACID compliance
- Full-text search (pg_trgm)
- JSON support
- Multi-version concurrency control

**Schema:**
```
Users
├── Admin Users
├── Merchants
│   ├── Wallets
│   │   ├── Transactions
│   │   │   ├── Risk Events
│   │   │   ├── Disputes
│   │   │   └── Settlements
│   │   └── Treasury Accounts
│   └── Liquidity Pools
├── AML Watchlist
├── Audit Logs
├── Compliance Reports
└── System Configs
```

**Replication:**
- Primary-Replica setup
- WAL-based streaming replication
- Point-in-time recovery capability
- Automated backups (daily full, hourly incremental)

### Redis Cache & Queue

**Session Store:**
- JWT tokens (TTL: 1 hour)
- User sessions
- Rate limiting counters
- Real-time notifications

**Message Queue (BullMQ):**
- Transaction processing
- Email sending
- Report generation
- Background jobs

**Pattern Memory:**
- Merchant transaction profiles
- Velocity calculations
- Risk scoring data
- Watchlist cache

### Elasticsearch

**Logging & Search:**
- Application logs
- Audit trail indexing
- Transaction search
- Full-text analytics
- Dashboard metrics

**Index Strategy:**
- Daily indices: `logs-YYYY-MM-DD`
- Retention: 90 days
- Shards: 3, Replicas: 2

---

## 4. Security Architecture

### Authentication

**JWT Flow:**
```
User Login
  ↓
Validate Credentials (bcrypt)
  ↓
Generate JWT (HS256 signed)
  ↓
Generate Refresh Token
  ↓
Store Refresh Token in Redis
  ↓
Return tokens to client
  ↓
Client stores JWT (memory), Refresh Token (secure cookie)
```

**Token Specifications:**
- Access Token: 1 hour expiry
- Refresh Token: 7 days expiry
- Issued by: auth-service
- Algorithm: HS256

### Authorization

**Role-Based Access Control (RBAC):**
```
Roles:
- Super Admin: All permissions
- Compliance Officer: Risk, KYC, reports
- Risk Manager: Risk events, risk engine
- Operations: Settlements, merchant support
- Finance: Treasury, approvals
- Support: View-only access
```

### Encryption

**Data at Rest:**
- AES-256 for sensitive fields
- SSN/Bank account/API keys encrypted
- Encrypted backups
- Key management via AWS KMS

**Data in Transit:**
- TLS 1.3 for all communications
- Certificate pinning (mobile apps)
- HSTS headers
- Encrypted JSON Web Tokens

---

## 5. Deployment Architecture

### Development Environment
```
Local Docker Compose:
- PostgreSQL (dev)
- Redis (cache)
- API service
- All in one network
- Hot reload enabled
- Mock blockchain RPCs
```

### Staging Environment
```
Kubernetes Cluster:
- 3 worker nodes
- Managed PostgreSQL (RDS)
- Managed Redis (ElastiCache)
- API: 3 replicas
- Listener: 1 + 1 standby
- Staging blockchain RPCs
```

### Production Environment
```
Kubernetes Cluster:
- 10+ worker nodes
- Multi-AZ PostgreSQL
- Redis Cluster (6 nodes)
- API: 5-20 replicas (auto-scaling)
- Listener: 2 + 1 standby
- Private blockchain nodes
- CDN (CloudFront)
- WAF protection
```

---

## 6. Performance & Scalability

### Horizontal Scaling
- **API Service**: CPU-based auto-scaling (50-80%)
- **Blockchain Listener**: Scale with address count
- **Risk Engine**: Worker pool for calculations
- **Database**: Read replicas for queries

### Caching Strategy
- Application cache: Redis (5-minute TTL)
- Query cache: Database-level
- Asset cache: CDN (1-day TTL)
- Watchlist cache: 24-hour TTL

### Performance Targets

| Metric | Target |
|--------|--------|
| API Response (p95) | < 500ms |
| Risk Calculation | < 1 second |
| Settlement Processing | < 24 hours |
| Data Availability | 99.9% |
| Blockchain Lag | < 5 minutes |
| Database Query (p95) | < 100ms |

---

## 7. Monitoring & Observability

### Metrics Collection
```
Prometheus scrapes:
- API latency (histogram)
- Request volume (counter)
- Transaction throughput (gauge)
- Settlement success rate (gauge)
- Risk event volume (counter)
- Database connections (gauge)
- Cache hit rate (gauge)
```

### Logging
```
ELK Stack:
- Elasticsearch: Log storage
- Logstash: Log parsing
- Kibana: Visualization
- Retention: 30 days
- Structured JSON logging
```

### Alerting
```
PagerDuty Integration:
- High error rate (> 1%)
- API latency p95 > 1000ms
- Blockchain listener lag
- Settlement failures
- Risk scoring delays
- Database connectivity loss
```

---

## 8. Disaster Recovery

### RTO/RPO
- **RTO (Recovery Time)**: 1 hour
- **RPO (Recovery Point)**: 15 minutes

### Backup Strategy
```
Daily:
- Full database backup (encrypted)
- S3 storage (multi-region)
- Test restoration (weekly)

Hourly:
- Incremental backups
- Point-in-time recovery
- Transaction log archiving
```

### Failover Procedure
```
1. Detect primary failure
2. Promote standby replica
3. Update DNS records
4. Verify data consistency
5. Notify stakeholders
6. Begin investigation
7. Plan restoration
```

---

## 9. Technology Stack Summary

| Layer | Technology |
|-------|-----------|
| **Frontend Web** | Next.js, React, TailwindCSS |
| **Frontend Mobile** | React Native, Expo |
| **Backend API** | Node.js, Express, TypeScript |
| **ORM** | Prisma |
| **Primary DB** | PostgreSQL 15+ |
| **Cache** | Redis Cluster |
| **Search** | Elasticsearch |
| **Message Queue** | BullMQ (Redis-backed) |
| **Blockchain RPC** | Infura, Alchemy, self-hosted |
| **Container** | Docker |
| **Orchestration** | Kubernetes |
| **Infrastructure** | AWS/GCP/DigitalOcean |
| **IaC** | Terraform |
| **Monitoring** | Prometheus, Grafana |
| **Logging** | ELK Stack |
| **Auth** | JWT, 2FA (TOTP/SMS) |
| **Testing** | Jest, Supertest |
| **CI/CD** | GitHub Actions |

---

## 10. Development Workflow

### Local Development
```bash
# Setup
git clone repo
npm install
./scripts/start-dev.sh

# Development
npm run dev          # Start all services
npm run db:studio   # Prisma studio
npm run test        # Run tests

# Deployment
git push origin feature-branch
# GitHub Actions auto-tests
# Create pull request
# Code review
# Merge to main
# Auto-deploy to staging
```

### Release Process
```
1. Version bump (semantic versioning)
2. Changelog update
3. Tag release
4. Build Docker images
5. Push to registry
6. Deploy to staging (2 days testing)
7. Manual approval
8. Deploy to production
9. Monitor metrics
10. Issue postmortem if needed
```

---

## 11. Cost Optimization

### Estimated Monthly Costs

| Component | Usage | Cost |
|-----------|-------|------|
| **Kubernetes** | 10 nodes | $800 |
| **PostgreSQL** | 500GB, HA | $600 |
| **Redis** | 6 nodes | $300 |
| **Data Transfer** | 50TB | $400 |
| **Blockchain RPC** | 50M requests | $500 |
| **CDN** | 100GB | $100 |
| **Monitoring** | All services | $200 |
| **Backup/Storage** | 500GB | $50 |
| **SSL/Domain** | 10 domains | $50 |
| ****TOTAL** | | **~$3,050** |

---

## 12. Future Enhancements

1. **GraphQL API** - Alternative to REST
2. **Websocket Support** - Real-time updates
3. **Mobile Push** - Notifications for app
4. **Layer 2** - Optimism/Arbitrum support
5. **DeFi Integration** - Yield farming for reserves
6. **AI/ML** - Advanced risk modeling
7. **Staking** - Earn rewards on holdings
8. **Multi-signature** - Enhanced security

---

**Architecture Version:** 1.0
**Last Updated:** January 2024
**Maintainer:** Platform Engineering
