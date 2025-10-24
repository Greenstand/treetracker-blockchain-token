# TreeTracker Impact Token Microservice

A production-ready microservice for managing Impact Tokens in the TreeTracker ecosystem with Hyperledger Fabric blockchain integration.

<img width="681" height="678" alt="image" src="https://github.com/user-attachments/assets/26dc5356-a518-4cd9-a2f6-d7299fed1554" />

## 🌟 Features

- **Token Issuance**: Automatically issue Impact Tokens when tree captures are verified
- **Token Lifecycle Management**: Handle token states (basic → mature → traded → redeemed)
- **Blockchain Integration**: Full Hyperledger Fabric integration for immutable token records
- **Wallet Management**: Track user token balances and ownership
- **Analytics & Reporting**: Comprehensive metrics and impact statistics
- **RESTful API**: Clean, well-documented API endpoints
- **Authentication**: JWT-based authentication integrated with auth service
- **Database**: PostgreSQL with optimized schema and triggers
- **Caching**: Redis for performance optimization
- **Production Ready**: Docker, Kubernetes, health checks, logging, monitoring

## 📋 Prerequisites

- Node.js 20.x or higher
- PostgreSQL 15+
- Redis 7+
- Docker & Docker Compose (for containerized deployment)
- Kubernetes cluster (for production deployment)
- Access to existing TreeTracker services:
  - Auth Service (Port 30001)
  - Capture Service (Port 30002)
  - Hyperledger Fabric Network

## 🚀 Quick Start

### 1. Clone and Install

```bash
cd /root/greenstand-token
npm install
```

### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env with your configuration
```

**Key environment variables:**
- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` - PostgreSQL connection
- `REDIS_HOST`, `REDIS_PORT` - Redis connection
- `JWT_SECRET` - Must match auth service JWT secret
- `AUTH_SERVICE_URL` - Auth service endpoint
- `CAPTURE_SERVICE_URL` - Capture service endpoint
- `FABRIC_*` - Hyperledger Fabric configuration

### 3. Setup Database

```bash
# Run migrations
npm run migrate

# Or manually with psql
psql -U postgres -d treetracker_tokens -f src/database/schema.sql
```

### 4. Development Mode

```bash
npm run dev
```

The service will start on `http://localhost:3004`

### 5. Production Build

```bash
npm run build
npm start
```

## 🐳 Docker Deployment

### Using Docker Compose

```bash
# Start all services (PostgreSQL, Redis, Token Service)
docker-compose up -d

# View logs
docker-compose logs -f token-service

# Stop services
docker-compose down
```

### Build Docker Image

```bash
docker build -t treetracker-token-service:latest .
```

## ☸️ Kubernetes Deployment

### 1. Apply Configurations

```bash
# Create namespace
kubectl create namespace treetracker

# Apply deployment
kubectl apply -f k8s/deployment.yaml
```

### 2. Update Secrets

```bash
# Edit secrets in k8s/deployment.yaml before applying
# Or use kubectl to create secret
kubectl create secret generic token-service-secret \
  --from-literal=DB_PASSWORD=your-password \
  --from-literal=JWT_SECRET=your-jwt-secret \
  -n treetracker
```

### 3. Verify Deployment

```bash
# Check pods
kubectl get pods -n treetracker -l app=treetracker-token-service

# Check logs
kubectl logs -f deployment/treetracker-token-service -n treetracker

# Check service
kubectl get svc -n treetracker treetracker-token-service
```

## 📚 API Documentation

### Base URL

```
Production: https://tokens.treetracker.example.com/api
Development: http://localhost:3004/api
```

### Authentication

All API endpoints require JWT authentication:

```bash
Authorization: Bearer <access_token>
```

### Endpoints

#### Token Management

**Get Tokens** (Paginated)
```http
GET /api/tokens?page=1&limit=10&state=basic
```

Response:
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "captureId": "capture-001",
      "planterId": "user-001",
      "treeSpecies": "Mango",
      "state": "basic",
      "status": "active",
      "value": 1.0,
      "ecologicalValue": {
        "carbonOffset": 15.0,
        "waterRetention": 100,
        "biodiversityScore": 5.0
      },
      "issuedDate": "2025-10-24T02:00:00Z",
      "blockchainTxId": "tx-12345"
    }
  ],
  "pagination": {
    "total": 45,
    "page": 1,
    "pageSize": 10,
    "hasMore": true
  }
}
```

**Get Token by ID**
```http
GET /api/tokens/:id
```

**Issue New Token**
```http
POST /api/tokens
Content-Type: application/json

{
  "captureId": "capture-001",
  "planterId": "user-001",
  "treeSpecies": "Mango",
  "carbonOffset": 15.0,
  "waterRetention": 100,
  "biodiversityScore": 5.0
}
```

**Mature Token**
```http
POST /api/tokens/:id/mature
```

**Transfer Token**
```http
POST /api/tokens/transfer
Content-Type: application/json

{
  "tokenId": "token-uuid",
  "toUserId": "recipient-user-id",
  "metadata": {
    "reason": "carbon offset purchase"
  }
}
```

**Get Wallet Balance**
```http
GET /api/tokens/wallets/:userId/balance
```

#### Analytics

**Get Token Metrics**
```http
GET /api/analytics/metrics
```

Response:
```json
{
  "success": true,
  "data": {
    "totalTokens": 1250,
    "totalValue": 1875.50,
    "totalCarbonOffset": 18750.0,
    "totalWaterRetention": 125000,
    "tokensByState": {
      "basic": 800,
      "mature": 350,
      "traded": 80,
      "redeemed": 20
    },
    "averageTokenValue": 1.5,
    "issuanceRate": {
      "daily": 25,
      "weekly": 150,
      "monthly": 600
    }
  }
}
```

**Get Impact Statistics**
```http
GET /api/analytics/impact
```

**Get Dashboard Data**
```http
GET /api/analytics/dashboard
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  TreeTracker Web App                     │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│              Token Microservice (Port 3004)              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │   Token    │  │ Analytics  │  │ Validation │       │
│  │  Service   │  │  Service   │  │  Service   │       │
│  └────────────┘  └────────────┘  └────────────┘       │
└─────────────────────────────────────────────────────────┘
           │                 │                │
           ▼                 ▼                ▼
    ┌──────────┐     ┌──────────┐    ┌──────────┐
    │PostgreSQL│     │  Redis   │    │  Fabric  │
    │          │     │  Cache   │    │ Network  │
    └──────────┘     └──────────┘    └──────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│         External Services Integration                    │
│  ┌────────────────┐        ┌────────────────┐          │
│  │  Auth Service  │        │Capture Service │          │
│  │  (Port 30001)  │        │  (Port 30002)  │          │
│  └────────────────┘        └────────────────┘          │
└─────────────────────────────────────────────────────────┘
```

## 📊 Database Schema

The service uses PostgreSQL with the following main tables:

- **tokens** - Core token data
- **transactions** - Token transaction history
- **wallets** - User wallet aggregates
- **token_ownership** - Current ownership tracking
- **token_history** - Audit trail
- **analytics_cache** - Performance optimization

See `src/database/schema.sql` for complete schema with triggers and views.

## 🔐 Security

- **Authentication**: JWT tokens validated against auth service
- **Authorization**: Role-based access control
- **Input Validation**: All inputs validated with Joi
- **Rate Limiting**: Configurable request rate limits
- **CORS**: Configurable allowed origins
- **Helmet**: Security headers
- **SQL Injection**: Parameterized queries
- **Secrets**: Environment variables, never committed

## 📝 Logging

Structured logging with Winston:

- **Console**: Development mode
- **Files**: `logs/combined.log`, `logs/error.log`
- **Format**: JSON with timestamps
- **Levels**: error, warn, info, debug

## 🧪 Testing

```bash
# Run tests
npm test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage
```

## 🔧 Development

### Project Structure

```
treetracker-token-service/
├── src/
│   ├── config/          # Configuration files
│   │   └── database.ts  # Database connection
│   ├── database/        # Database schema and migrations
│   │   ├── schema.sql
│   │   └── migrate.ts
│   ├── middleware/      # Express middleware
│   │   └── auth.ts     # Authentication
│   ├── routes/          # API routes
│   │   ├── tokenRoutes.ts
│   │   └── analyticsRoutes.ts
│   ├── services/        # Business logic
│   │   ├── TokenService.ts
│   │   ├── FabricService.ts
│   │   └── AnalyticsService.ts
│   ├── types/           # TypeScript types
│   │   └── index.ts
│   ├── utils/           # Utilities
│   │   └── logger.ts
│   └── server.ts        # Express server
├── k8s/                 # Kubernetes manifests
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── package.json
└── tsconfig.json
```

### Code Quality

```bash
# Lint
npm run lint

# Format
npm run format
```

## 🔄 Integration with Treetracker Ecosystem

### Capture Service Integration

When a tree capture is verified in the capture service:
1. Capture service calls `POST /api/tokens` with capture details
2. Token service issues new Impact Token
3. Token recorded in database and blockchain
4. Wallet balance updated automatically

### Auth Service Integration

- JWT tokens from auth service are validated
- User context extracted from JWT payload
- Roles checked for authorization

### Frontend Integration

Update `treetracker-web/lib/api.ts`:

```typescript
// Replace mock endpoint
const TOKEN_SERVICE_URL = process.env.NEXT_PUBLIC_TOKEN_API_URL || 'http://localhost:3004/api';

async getImpactTokens(userId: string, page = 1, limit = 10) {
  const response = await this.axiosInstance.get(
    `${TOKEN_SERVICE_URL}/tokens?page=${page}&limit=${limit}&planterId=${userId}`
  );
  return response.data;
}
```

## 🚨 Troubleshooting

### Database Connection Issues

```bash
# Test connection
psql -h localhost -U postgres -d treetracker_tokens

# Check logs
tail -f logs/error.log
```

### Fabric Connection Issues

```bash
# Verify wallet exists
ls -la wallet/

# Check connection profile
cat connection-profile.json

# Service will continue without Fabric if not configured
```

### Token Issuance Failures

Check logs for:
- Database constraint violations
- Duplicate capture IDs
- Missing required fields
- Fabric transaction errors (non-blocking)

## 📈 Monitoring

### Health Check

```bash
curl http://localhost:3004/health
```

### Metrics (if enabled)

```bash
curl http://localhost:9090/metrics
```

### Kubernetes Monitoring

```bash
# Resource usage
kubectl top pods -n treetracker -l app=treetracker-token-service

# Logs
kubectl logs -f deployment/treetracker-token-service -n treetracker --tail=100
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

MIT License - See LICENSE file for details

## 🆘 Support

For issues and questions:
- Create an issue in the repository
- Contact the TreeTracker team
- Check existing documentation

## 🗺️ Roadmap

- [x] Core token management
- [x] Blockchain integration
- [x] Analytics and reporting
- [x] Docker and Kubernetes deployment
- [ ] Token marketplace features
- [ ] Advanced fraud detection
- [ ] Machine learning for token valuation
- [ ] Mobile SDK for direct integration
- [ ] GraphQL API
- [ ] Real-time notifications via WebSocket

---

**Built with 🌳 for sustainable reforestation**

*TreeTracker Impact Token Service - Tokenizing environmental impact for a greener future*
