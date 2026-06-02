# ============================================================================
# CFI PLATFORM - API SPECIFICATION
# REST API endpoints for merchant dashboard, admin, and integrations
# ============================================================================

## Base URL
- **Development**: `http://localhost:3001/api`
- **Staging**: `https://staging-api.cfiplatform.com/api`
- **Production**: `https://api.cfiplatform.com/api`

---

## Authentication

### POST /auth/register
Register a new merchant or user.

**Request:**
```json
{
  "email": "merchant@example.com",
  "password": "SecurePassword123!",
  "businessName": "TechStore Inc",
  "businessType": "ecommerce"
}
```

**Response (201):**
```json
{
  "id": "uuid-merchant-id",
  "email": "merchant@example.com",
  "businessName": "TechStore Inc",
  "kycStatus": "pending",
  "createdAt": "2024-01-10T12:00:00Z"
}
```

---

### POST /auth/login
Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "merchant@example.com",
  "password": "SecurePassword123!"
}
```

**Response (200):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "user": {
    "id": "uuid-merchant-id",
    "email": "merchant@example.com",
    "role": "merchant"
  }
}
```

---

## Merchants

### GET /merchants/me
Get current merchant profile (requires authentication).

**Response (200):**
```json
{
  "id": "uuid-merchant-id",
  "businessName": "TechStore Inc",
  "businessType": "ecommerce",
  "taxId": "12-3456789",
  "kycStatus": "approved",
  "kycLevel": "tier2",
  "settlementFrequency": "daily",
  "feeTier": "standard",
  "isActive": true,
  "totalVolume30d": 250000.50,
  "totalTransactions30d": 1250,
  "createdAt": "2024-01-01T00:00:00Z"
}
```

### PUT /merchants/me
Update merchant profile.

**Request:**
```json
{
  "businessName": "TechStore Inc Updated",
  "settlementFrequency": "weekly",
  "settlementBankAccount": "encrypted-account-number"
}
```

---

## Wallets

### GET /merchants/:merchantId/wallets
List all wallets for a merchant.

**Response (200):**
```json
{
  "data": [
    {
      "id": "wallet-uuid-1",
      "chain": "bitcoin",
      "address": "1A1z7agoat5JZ5sqAGCQqeLMgh5n8j5ja",
      "addressType": "deposit",
      "balance": {
        "satoshis": 2500000000,
        "btc": 25,
        "usd": 975000
      },
      "confirmationsRequired": 6,
      "isActive": true,
      "createdAt": "2024-01-10T12:00:00Z"
    }
  ],
  "total": 1
}
```

### POST /merchants/:merchantId/wallets
Create a new deposit wallet.

**Request:**
```json
{
  "chain": "ethereum",
  "addressType": "deposit"
}
```

---

## Transactions

### GET /transactions
List all transactions for current merchant.

**Query Parameters:**
- `status`: pending, confirming, confirmed, settled, failed
- `chain`: bitcoin, ethereum
- `from`: ISO date string
- `to`: ISO date string
- `limit`: 10-100 (default: 25)
- `offset`: pagination offset

**Response (200):**
```json
{
  "data": [
    {
      "id": "tx-uuid-1",
      "txHash": "0x1234567890abcdef...",
      "chain": "bitcoin",
      "direction": "inbound",
      "amountCrypto": 1.5,
      "amountUsd": 58500,
      "usdRate": 39000,
      "status": "confirmed",
      "confirmations": 6,
      "requiredConfirmations": 6,
      "settlementStatus": "settled",
      "riskCategory": "low",
      "riskScore": 15.5,
      "flaggedForReview": false,
      "createdAt": "2024-01-10T10:00:00Z",
      "confirmedAt": "2024-01-10T10:45:00Z",
      "settledAt": "2024-01-10T11:30:00Z"
    }
  ],
  "total": 150,
  "page": 1,
  "pages": 6
}
```

### GET /transactions/:txId
Get transaction details.

**Response (200):**
```json
{
  "id": "tx-uuid-1",
  "txHash": "0x1234567890abcdef...",
  "chain": "bitcoin",
  "direction": "inbound",
  "amountCrypto": 1.5,
  "amountUsd": 58500,
  "wallet": {
    "address": "1A1z7agoat5JZ5sqAGCQqeLMgh5n8j5ja",
    "chain": "bitcoin"
  },
  "status": "confirmed",
  "confirmations": 6,
  "settlementStatus": "settled",
  "riskAnalysis": {
    "score": 15.5,
    "category": "low",
    "flags": []
  },
  "timeline": [
    {
      "status": "pending",
      "timestamp": "2024-01-10T10:00:00Z"
    },
    {
      "status": "confirmed",
      "timestamp": "2024-01-10T10:45:00Z"
    },
    {
      "status": "settled",
      "timestamp": "2024-01-10T11:30:00Z"
    }
  ]
}
```

---

## Settlements

### GET /settlements
List all settlements for current merchant.

**Query Parameters:**
- `status`: pending, processing, completed, failed, reversed
- `from`: ISO date string
- `to`: ISO date string
- `limit`: 10-100 (default: 25)

**Response (200):**
```json
{
  "data": [
    {
      "id": "settlement-uuid-1",
      "totalAmount": 50000.00,
      "fee": 250.00,
      "netAmount": 49750.00,
      "settlementMethod": "ach",
      "status": "completed",
      "bankReference": "REF-20240110-001",
      "transactionCount": 5,
      "createdAt": "2024-01-10T08:00:00Z",
      "processedAt": "2024-01-10T09:30:00Z"
    }
  ],
  "total": 10,
  "totalVolume": 500000.00
}
```

---

## Risk & Compliance

### GET /risk/events
List risk events (admin only).

**Query Parameters:**
- `severity`: low, medium, high, critical
- `resolved`: true/false
- `limit`: 10-100 (default: 25)

**Response (200):**
```json
{
  "data": [
    {
      "id": "risk-event-uuid",
      "eventType": "high_velocity",
      "severity": "high",
      "score": 85.5,
      "reason": "Transaction volume 3x daily average",
      "patternMatched": "velocity_pattern_001",
      "actionTaken": "flag",
      "resolved": false,
      "createdAt": "2024-01-10T12:00:00Z"
    }
  ],
  "total": 5
}
```

### POST /risk/review/:riskEventId
Resolve a risk event (admin only).

**Request:**
```json
{
  "actionTaken": "approved",
  "resolutionNotes": "Customer verified - legitimate business growth"
}
```

---

## Treasury (Admin Only)

### GET /treasury/accounts
List all treasury accounts.

**Response (200):**
```json
{
  "data": [
    {
      "id": "treasury-uuid-1",
      "accountName": "Bitcoin Hot Wallet",
      "accountType": "hot_wallet",
      "chain": "bitcoin",
      "balance": {
        "crypto": 2.5,
        "usd": 97500
      },
      "allocationTarget": 25,
      "allocationCurrent": 24.5,
      "isActive": true
    }
  ],
  "totalBalance": 400000.00,
  "lastRebalanced": "2024-01-09T15:00:00Z"
}
```

### POST /treasury/rebalance
Trigger treasury rebalancing (admin only).

**Request:**
```json
{
  "sourceAccount": "treasury-uuid-1",
  "destinationAccount": "treasury-uuid-2",
  "amount": 10000.00
}
```

---

## Admin Dashboard

### GET /admin/dashboard
System overview (admin only).

**Response (200):**
```json
{
  "summary": {
    "totalVolume24h": 1250000.00,
    "totalTransactions24h": 450,
    "activeVaults": 4,
    "riskEventsOpen": 2
  },
  "merchants": {
    "total": 250,
    "activeToday": 180,
    "kycPending": 25,
    "flagged": 3
  },
  "treasury": {
    "totalBalance": 400000.00,
    "hotWalletBalance": 100000.00,
    "coldStorageBalance": 300000.00,
    "rebalanceNeeded": false
  }
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": true,
  "code": "VALIDATION_ERROR",
  "message": "Invalid request parameters",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

### 401 Unauthorized
```json
{
  "error": true,
  "code": "UNAUTHORIZED",
  "message": "Authentication required"
}
```

### 403 Forbidden
```json
{
  "error": true,
  "code": "FORBIDDEN",
  "message": "Insufficient permissions"
}
```

### 404 Not Found
```json
{
  "error": true,
  "code": "NOT_FOUND",
  "message": "Resource not found"
}
```

### 429 Too Many Requests
```json
{
  "error": true,
  "code": "RATE_LIMITED",
  "message": "Too many requests, please retry after 60 seconds",
  "retryAfter": 60
}
```

### 500 Internal Server Error
```json
{
  "error": true,
  "code": "INTERNAL_ERROR",
  "message": "An unexpected error occurred"
}
```

---

## Authentication Headers

All protected endpoints require:
```
Authorization: Bearer <JWT_TOKEN>
```

## Rate Limiting

- **Public endpoints**: 60 requests per minute
- **Authenticated endpoints**: 300 requests per minute
- **Admin endpoints**: 1000 requests per minute

---

## Pagination

Endpoints that return lists use cursor-based pagination:
```
?limit=25&offset=0
```

---

## Webhooks

CFI Platform supports webhooks for real-time events:

### Supported Events
- `transaction.confirmed`
- `transaction.settled`
- `settlement.completed`
- `risk.flagged`
- `merchant.kyc_approved`

**Example Webhook Payload:**
```json
{
  "eventId": "evt-uuid-123",
  "eventType": "transaction.confirmed",
  "timestamp": "2024-01-10T12:00:00Z",
  "data": {
    "transactionId": "tx-uuid-1",
    "merchantId": "merchant-uuid-1",
    "txHash": "0x1234567890abcdef...",
    "status": "confirmed",
    "confirmations": 6
  }
}
```
