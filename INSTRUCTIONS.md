# Complete Step-by-Step Guide to Create the Assignment

---

## Part 1: GitHub Repository Setup

### Step 1: Create New GitHub Repository

1. Go to https://github.com/new
2. **Repository name:** `mixoads-backend-assignment`
3. **Description:** "Backend Engineering Technical Assignment for Mixo Ads"
4. **Visibility:** Private (you'll share access per candidate)
5. **Initialize:**  Add a README file
6. Click **Create repository**

---

### Step 2: Clone Repository Locally

```bash
git clone https://github.com/YOUR_USERNAME/mixoads-backend-assignment.git
cd mixoads-backend-assignment
```

---

### Step 3: Create Directory Structure

```bash
# Create main source directory
mkdir -p src

# Create mock API directory
mkdir -p mock-api

# Verify structure
ls -la
# You should see: src/ mock-api/ README.md
```

---

## Part 2: Create All Files

### File 1: `.gitignore`

```bash
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
package-lock.json

# Build outputs
dist/
build/

# Environment variables
.env
.env.local

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
EOF
```

---

### File 2: `package.json` (Main App)

```bash
cat > package.json << 'EOF'
{
  "name": "mixoads-backend-assignment",
  "version": "1.0.0",
  "description": "Backend engineering assignment for Mixo Ads",
  "main": "src/index.ts",
  "scripts": {
    "start": "ts-node src/index.ts",
    "dev": "ts-node-dev --respawn src/index.ts"
  },
  "keywords": ["backend", "assignment", "mixoads"],
  "author": "Mixo Ads",
  "license": "ISC",
  "dependencies": {
    "node-fetch": "^2.7.0",
    "pg": "^8.11.0",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "@types/node-fetch": "^2.6.9",
    "@types/pg": "^8.10.9",
    "ts-node": "^10.9.2",
    "ts-node-dev": "^2.0.0",
    "typescript": "^5.3.3"
  }
}
EOF
```

---

### File 3: `tsconfig.json`

```bash
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "mock-api"]
}
EOF
```

---

### File 4: `.env.example`

```bash
cat > .env.example << 'EOF'
# Ad Platform API Configuration
AD_PLATFORM_API_URL=http://localhost:3001
AD_PLATFORM_EMAIL=admin@mixoads.com
AD_PLATFORM_PASSWORD=SuperSecret123!

# Database Configuration (for future use)
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mixoads
DB_USER=postgres
DB_PASSWORD=postgres

# Optional: Use mock DB for assignment (set to true)
USE_MOCK_DB=true
EOF
```

---

### File 5: `src/index.ts`

```bash
cat > src/index.ts << 'EOF'
import dotenv from 'dotenv';
import { syncAllCampaigns } from './syncCampaigns';

dotenv.config();

async function main() {
  console.log('🚀 Starting campaign sync...');
  console.log('='.repeat(60));
  
  try {
    await syncAllCampaigns();
    console.log('\n✅ Sync completed successfully!');
  } catch (error) {
    console.error('\n❌ Sync failed:', error);
    process.exit(1);
  }
}

main();
EOF
```

---

### File 6: `src/syncCampaigns.ts` (The Broken God Function)

```bash
cat > src/syncCampaigns.ts << 'EOF'
import fetch from 'node-fetch';
import { saveCampaignToDB } from './database';

/**
 * THIS CODE IS INTENTIONALLY BROKEN
 * 
 * Issues present:
 * - Hardcoded credentials
 * - No error handling
 * - No retry logic
 * - No rate limit handling
 * - No pagination handling
 * - No timeout handling
 * - Sequential processing (slow)
 * - God function (everything in one place)
 * - Credentials logged
 * - No separation of concerns
 */

export async function syncAllCampaigns() {
  console.log('Syncing campaigns from Ad Platform...\n');
  
  // BAD: Hardcoded credentials in code
  const email = "admin@mixoads.com";
  const password = "SuperSecret123!";
  
  // BAD: Encoding credentials inline every time
  const authString = Buffer.from(`${email}:${password}`).toString('base64');
  
  // BAD: Logging credentials (base64 is trivially decoded)
  console.log(`Using auth: Basic ${authString}`);
  
  // === STEP 1: Get Access Token ===
  console.log('\n📝 Step 1: Getting access token...');
  
  // BAD: No try-catch error handling
  // BAD: No timeout
  const authResponse = await fetch('http://localhost:3001/auth/token', {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${authString}`
    }
  });
  
  // BAD: Not checking if response is ok
  const authData: any = await authResponse.json();
  const accessToken = authData.access_token;
  
  // BAD: Logging the full access token
  console.log(`✓ Got access token: ${accessToken}`);
  
  // === STEP 2: Fetch Campaigns ===
  console.log('\n📊 Step 2: Fetching campaigns...');
  
  // BAD: Only fetches first page (ignores pagination)
  // BAD: No rate limit handling (will hit 10 req/min limit)
  // BAD: No retry logic for 503 errors
  // BAD: No timeout handling
  const campaignsResponse = await fetch('http://localhost:3001/api/campaigns?page=1&limit=10', {
    headers: {
      'Authorization': `Bearer ${accessToken}`
    }
  });
  
  // BAD: Doesn't check response.ok or status code
  const campaignsData: any = await campaignsResponse.json();
  
  console.log(`✓ Found ${campaignsData.data.length} campaigns`);
  console.log(`   Pagination: page ${campaignsData.pagination.page}, has_more: ${campaignsData.pagination.has_more}`);
  
  // BAD: Ignores pagination.has_more - only processes first 10 campaigns
  
  // === STEP 3: Sync Each Campaign ===
  console.log('\n🔄 Step 3: Syncing campaigns to database...');
  
  let successCount = 0;
  
  // BAD: Sequential processing - very slow
  // Should be parallel with concurrency control
  for (const campaign of campaignsData.data) {
    console.log(`\n   Syncing: ${campaign.name} (ID: ${campaign.id})`);
    
    try {
      // BAD: No rate limit handling
      // BAD: No retry logic for failures
      // BAD: No timeout (sync endpoint takes 2 seconds - could hang)
      const syncResponse = await fetch(`http://localhost:3001/api/campaigns/${campaign.id}/sync`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ campaign_id: campaign.id })
      });
      
      const syncData: any = await syncResponse.json();
      
      // BAD: Database operation mixed with API logic
      // BAD: No transaction handling
      // BAD: Will create duplicates if run multiple times
      await saveCampaignToDB(campaign);
      
      successCount++;
      console.log(`   ✓ Successfully synced ${campaign.name}`);
      
    } catch (error: any) {
      // BAD: Individual failures handled but continue processing
      // But no logging of what failed
      console.error(`   ✗ Failed to sync ${campaign.name}:`, error.message);
      // BAD: Exposes full error details
    }
  }
  
  console.log('\n' + '='.repeat(60));
  console.log(`✅ Sync complete: ${successCount}/${campaignsData.data.length} campaigns synced`);
  console.log('='.repeat(60));
}
EOF
```

---

### File 7: `src/database.ts`

```bash
cat > src/database.ts << 'EOF'
import { Pool } from 'pg';

/**
 * THIS CODE IS INTENTIONALLY BROKEN
 * 
 * Issues:
 * - Creates new pool connection on every call
 * - No upsert logic (creates duplicates)
 * - SQL injection vulnerability
 * - No transaction handling
 * - Pool not closed properly
 */

// BAD: Function creates new pool every time it's called
// Should reuse a single pool
async function getDB() {
  return new Pool({
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT || '5432'),
    database: process.env.DB_NAME || 'mixoads',
    user: process.env.DB_USER || 'postgres',
    password: process.env.DB_PASSWORD || 'postgres'
  });
}

export async function saveCampaignToDB(campaign: any) {
  // For assignment purposes, use mock DB
  if (process.env.USE_MOCK_DB === 'true') {
    console.log(`      [MOCK DB] Saved campaign: ${campaign.id}`);
    
    // Simulate duplicates issue
    // In real DB, this would fail on second run
    return;
  }
  
  // BAD: Creating new pool connection for each save
  const pool = await getDB();
  
  try {
    // BAD: Direct INSERT - will fail if row already exists
    // Should use UPSERT (INSERT ... ON CONFLICT)
    
    // BAD: String concatenation - SQL injection vulnerability
    // Should use parameterized queries
    const query = `
      INSERT INTO campaigns (id, name, status, budget, impressions, clicks, conversions, synced_at)
      VALUES ('${campaign.id}', '${campaign.name}', '${campaign.status}', 
              ${campaign.budget}, ${campaign.impressions}, ${campaign.clicks}, 
              ${campaign.conversions}, NOW())
    `;
    
    await pool.query(query);
    
    // BAD: Not closing the pool properly
    // Creates connection leak
    
  } catch (error: any) {
    // BAD: Just re-throw without handling duplicates
    // Should handle constraint violations gracefully
    throw new Error(`Database error: ${error.message}`);
  }
}
EOF
```

---

### File 8: `mock-api/package.json`

```bash
cat > mock-api/package.json << 'EOF'
{
  "name": "mock-ad-platform-api",
  "version": "1.0.0",
  "description": "Mock Ad Platform API for testing the assignment",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "keywords": ["mock", "api", "testing"],
  "author": "Mixo Ads",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF
```

---

### File 9: `mock-api/server.js` (The Mock API)

```bash
cat > mock-api/server.js << 'EOF'
const express = require('express');
const app = express();

// Track request counts for rate limiting
const requestCounts = new Map();
const RATE_LIMIT = 10; // 10 requests per minute
const RATE_WINDOW = 60000; // 1 minute in ms

// Track token issuance for expiry simulation
const TOKEN_EXPIRY_TIME = 3600; // 1 hour in seconds
let tokenIssuedAt = Date.now();

// Counter for simulating random failures
let requestCounter = 0;

app.use(express.json());

// Logging middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
  next();
});

// ============================================================================
// AUTH ENDPOINT
// ============================================================================

app.post('/auth/token', (req, res) => {
  const authHeader = req.headers.authorization;
  
  // Check for Basic auth header
  if (!authHeader || !authHeader.startsWith('Basic ')) {
    return res.status(401).json({ 
      error: 'Unauthorized',
      message: 'Missing or invalid Authorization header. Expected: Basic <base64>' 
    });
  }
  
  // Decode base64 credentials
  const base64Creds = authHeader.replace('Basic ', '');
  let decoded;
  
  try {
    decoded = Buffer.from(base64Creds, 'base64').toString('utf-8');
  } catch (error) {
    return res.status(401).json({ 
      error: 'Invalid base64 encoding' 
    });
  }
  
  const [email, password] = decoded.split(':');
  
  // Validate credentials
  if (email === 'admin@mixoads.com' && password === 'SuperSecret123!') {
    tokenIssuedAt = Date.now();
    
    console.log('✓ Token issued successfully');
    
    return res.json({
      access_token: 'mock_access_token_' + Date.now(),
      token_type: 'Bearer',
      expires_in: TOKEN_EXPIRY_TIME,
      issued_at: Math.floor(tokenIssuedAt / 1000)
    });
  }
  
  console.log('✗ Invalid credentials');
  res.status(401).json({ 
    error: 'Invalid credentials',
    message: 'Email or password is incorrect'
  });
});

// ============================================================================
// RATE LIMITING MIDDLEWARE
// ============================================================================

function rateLimitMiddleware(req, res, next) {
  const clientId = req.headers['x-client-id'] || 'default';
  const now = Date.now();
  
  if (!requestCounts.has(clientId)) {
    requestCounts.set(clientId, []);
  }
  
  const requests = requestCounts.get(clientId);
  
  // Remove requests outside the time window
  const recentRequests = requests.filter(time => now - time < RATE_WINDOW);
  
  if (recentRequests.length >= RATE_LIMIT) {
    console.log(`✗ Rate limit exceeded for client: ${clientId}`);
    return res.status(429).json({
      error: 'Rate limit exceeded',
      message: `Too many requests. Limit: ${RATE_LIMIT} per minute`,
      retry_after: 60
    });
  }
  
  recentRequests.push(now);
  requestCounts.set(clientId, recentRequests);
  
  next();
}

// ============================================================================
// AUTH CHECK MIDDLEWARE
// ============================================================================

function authMiddleware(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
      error: 'Unauthorized',
      message: 'Missing or invalid Bearer token'
    });
  }
  
  const token = authHeader.replace('Bearer ', '');
  
  // Check if token is expired (1 hour from issuance)
  const tokenAge = Date.now() - tokenIssuedAt;
  if (tokenAge > TOKEN_EXPIRY_TIME * 1000) {
    console.log('✗ Token expired');
    return res.status(401).json({ 
      error: 'Token expired',
      message: 'Access token has expired. Please obtain a new token.',
      expired_at: new Date(tokenIssuedAt + TOKEN_EXPIRY_TIME * 1000).toISOString()
    });
  }
  
  next();
}

// ============================================================================
// CAMPAIGNS ENDPOINT
// ============================================================================

app.get('/api/campaigns', authMiddleware, rateLimitMiddleware, (req, res) => {
  requestCounter++;
  
  // Simulate random 503 Service Unavailable (20% of requests)
  if (requestCounter % 5 === 0) {
    console.log('✗ Simulating 503 Service Unavailable');
    return res.status(503).json({
      error: 'Service temporarily unavailable',
      message: 'The service is experiencing issues. Please retry after a short delay.'
    });
  }
  
  // Simulate random timeout (10% of requests - just hang)
  if (requestCounter % 10 === 0) {
    console.log('✗ Simulating timeout (no response)');
    // Don't send response - simulate timeout
    return;
  }
  
  // Parse pagination params
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  
  // Generate mock campaign data
  const campaigns = [];
  for (let i = 0; i < limit; i++) {
    const id = (page - 1) * limit + i + 1;
    campaigns.push({
      id: `campaign_${id}`,
      name: `Campaign ${id}`,
      status: 'active',
      budget: 1000 + id * 100,
      impressions: Math.floor(Math.random() * 10000),
      clicks: Math.floor(Math.random() * 500),
      conversions: Math.floor(Math.random() * 50),
      created_at: new Date(Date.now() - id * 86400000).toISOString()
    });
  }
  
  console.log(`✓ Returning ${campaigns.length} campaigns (page ${page})`);
  
  res.json({
    data: campaigns,
    pagination: {
      page,
      limit,
      total: 100, // Total of 100 campaigns available
      has_more: page < 10 // 10 pages total
    }
  });
});

// ============================================================================
// CAMPAIGN SYNC ENDPOINT
// ============================================================================

app.post('/api/campaigns/:id/sync', authMiddleware, rateLimitMiddleware, (req, res) => {
  const { id } = req.params;
  
  console.log(`Syncing campaign: ${id}`);
  
  // Simulate slow API response (2 seconds)
  setTimeout(() => {
    res.json({
      success: true,
      campaign_id: id,
      synced_at: new Date().toISOString(),
      message: 'Campaign data synced successfully'
    });
  }, 2000);
});

// ============================================================================
// ERROR ENDPOINT (for testing error handling)
// ============================================================================

app.get('/api/error', (req, res) => {
  const error = new Error('Simulated server error');
  console.error(error);
  
  // BAD PRACTICE: Exposing full stack trace
  res.status(500).json({
    error: error.message,
    stack: error.stack // Never do this in production!
  });
});

// ============================================================================
// HEALTH CHECK
// ============================================================================

app.get('/health', (req, res) => {
  res.json({ 
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// ============================================================================
// START SERVER
// ============================================================================

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log('='.repeat(60));
  console.log('🚀 Mock Ad Platform API Server');
  console.log('='.repeat(60));
  console.log(`📍 Server running on: http://localhost:${PORT}`);
  console.log(`🔑 Valid credentials: admin@mixoads.com / SuperSecret123!`);
  console.log(`⏱️  Token expiry: ${TOKEN_EXPIRY_TIME} seconds (1 hour)`);
  console.log(`🚦 Rate limit: ${RATE_LIMIT} requests per minute`);
  console.log('='.repeat(60));
  console.log('\nEndpoints:');
  console.log('  POST /auth/token              - Get access token');
  console.log('  GET  /api/campaigns           - List campaigns (paginated)');
  console.log('  POST /api/campaigns/:id/sync  - Sync campaign data');
  console.log('  GET  /health                  - Health check');
  console.log('='.repeat(60));
});
EOF
```

---

### File 10: `mock-api/README.md`

```bash
cat > mock-api/README.md << 'EOF'
# Mock Ad Platform API

This is a mock API that simulates a real ad platform with realistic constraints and failure modes.

## Quick Start

```bash
cd mock-api
npm install
npm start
```

The API will run on `http://localhost:3001`

---

## Authentication Flow

### Step 1: Get Access Token

**Endpoint:** `POST /auth/token`  
**Auth Type:** Basic Auth (base64 encoded `email:password`)  
**Valid Credentials:** `admin@mixoads.com` / `SuperSecret123!`

**Example:**
```bash
# Encode credentials
echo -n "admin@mixoads.com:SuperSecret123!" | base64
# Output: YWRtaW5AbWl4b2Fkcy5jb206U3VwZXJTZWNyZXQxMjMh

# Get token
curl -X POST http://localhost:3001/auth/token \
  -H "Authorization: Basic YWRtaW5AbWl4b2Fkcy5jb206U3VwZXJTZWNyZXQxMjMh"
```

**Response:**
```json
{
  "access_token": "mock_access_token_1702412345678",
  "token_type": "Bearer",
  "expires_in": 3600,
  "issued_at": 1702412345
}
```

**Important:** Tokens expire after 1 hour (3600 seconds).

---

## API Endpoints

### GET /api/campaigns

Fetch paginated campaign data.

**Auth:** Bearer Token (required)  
**Query Parameters:**
- `page` (integer, default: 1) - Page number
- `limit` (integer, default: 10) - Results per page

**Example:**
```bash
curl http://localhost:3001/api/campaigns?page=1&limit=10 \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Response:**
```json
{
  "data": [
    {
      "id": "campaign_1",
      "name": "Campaign 1",
      "status": "active",
      "budget": 1100,
      "impressions": 8432,
      "clicks": 234,
      "conversions": 12,
      "created_at": "2024-12-11T10:30:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "has_more": true
  }
}
```

**Total Data:** 100 campaigns across 10 pages.

---

### POST /api/campaigns/:id/sync

Sync a campaign's data (simulates updating campaign info).

**Auth:** Bearer Token (required)  
**Path Parameter:** `id` - Campaign ID

**Example:**
```bash
curl -X POST http://localhost:3001/api/campaigns/campaign_1/sync \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response:**
```json
{
  "success": true,
  "campaign_id": "campaign_1",
  "synced_at": "2024-12-12T14:30:00.000Z",
  "message": "Campaign data synced successfully"
}
```

**Note:** This endpoint has an intentional 2-second delay to simulate slow API responses.

---

## API Constraints & Failure Modes

### Rate Limiting

- **Limit:** 10 requests per minute per client
- **Response Code:** 429 Too Many Requests
- **Response:**
  ```json
  {
    "error": "Rate limit exceeded",
    "message": "Too many requests. Limit: 10 per minute",
    "retry_after": 60
  }
  ```
- **Header:** `retry-after: 60` (seconds to wait)

**Tip:** Use the `X-Client-ID` header to track different clients separately.

---

### Token Expiry

- **Duration:** 1 hour (3600 seconds) from token issuance
- **Response Code:** 401 Unauthorized
- **Response:**
  ```json
  {
    "error": "Token expired",
    "message": "Access token has expired. Please obtain a new token.",
    "expired_at": "2024-12-12T15:30:00.000Z"
  }
  ```

**Tip:** Track `issued_at` and `expires_in` from the token response to refresh proactively.

---

### Simulated Failures

The API intentionally simulates real-world failure scenarios:

1. **503 Service Unavailable (20% of requests)**
   - Response Code: 503
   - Should be retried with exponential backoff

2. **Timeout / No Response (10% of requests)**
   - Server doesn't respond at all
   - Client should have timeout handling

3. **Slow Responses (all sync requests)**
   - `/api/campaigns/:id/sync` always takes 2 seconds
   - Tests handling of slow APIs

---

## Testing Scenarios

### Test 1: Rate Limiting
```bash
# Make 15 requests quickly
for i in {1..15}; do
  curl http://localhost:3001/api/campaigns \
    -H "Authorization: Bearer YOUR_TOKEN"
  echo ""
done

# Should see 429 errors after request #10
```

### Test 2: Pagination
```bash
# Fetch all 10 pages
for page in {1..10}; do
  curl "http://localhost:3001/api/campaigns?page=$page&limit=10" \
    -H "Authorization: Bearer YOUR_TOKEN"
done

# Should get 100 total campaigns
```

### Test 3: Token Expiry
```bash
# Get a token
TOKEN=$(curl -s -X POST http://localhost:3001/auth/token \
  -H "Authorization: Basic YWRtaW5AbWl4b2Fkcy5jb206U3VwZXJTZWNyZXQxMjMh" \
  | jq -r '.access_token')

# Wait 1 hour (or modify server.js to expire faster)
# Then try to use the token - should get 401
```

### Test 4: Handling Failures
```bash
# Run sync multiple times
# Some will succeed, some will get 503, some will timeout
for i in {1..20}; do
  curl -X POST http://localhost:3001/api/campaigns/campaign_1/sync \
    -H "Authorization: Bearer YOUR_TOKEN"
  echo ""
done
```

---

## Health Check

**Endpoint:** `GET /health`

```bash
curl http://localhost:3001/health
```

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2024-12-12T14:30:00.000Z",
  "uptime": 123.456
}
```

---

## Development Tips

### Modify Failure Rates

Edit `server.js`:

```javascript
// Line ~85: Change 503 frequency (currently 20%)
if (requestCounter % 5 === 0) { // Change % 5 to % 10 for 10%

// Line ~92: Change timeout frequency (currently 10%)
if (requestCounter % 10 === 0) { // Change % 10 to % 20 for 5%
```

### Speed Up Token Expiry for Testing

```javascript
// Line ~6: Change from 1 hour to 1 minute
const TOKEN_EXPIRY_TIME = 60; // 60 seconds instead of 3600
```

### Disable Rate Limiting

Comment out the `rateLimitMiddleware` in the endpoint handlers:

```javascript
// app.get('/api/campaigns', authMiddleware, rateLimitMiddleware, (req, res) => {
app.get('/api/campaigns', authMiddleware, (req, res) => {
```

---

## Troubleshooting

**Problem:** "Cannot GET /"  
**Solution:** The API doesn't have a root route. Use `/health` to check if it's running.

**Problem:** 401 Unauthorized  
**Solution:** Check your auth header format. For token endpoint use `Basic <base64>`, for other endpoints use `Bearer <token>`.

**Problem:** Connection refused  
**Solution:** Make sure the server is running on port 3001: `npm start`

---

## API Behavior Summary

| Feature | Behavior | Test Method |
|---------|----------|-------------|
| Rate Limit | 10 req/min | Make 15 quick requests |
| Token Expiry | 1 hour | Wait or modify server.js |
| 503 Errors | 20% random | Make multiple requests |
| Timeouts | 10% random | Make multiple requests |
| Slow Response | 2s for sync | Time the sync endpoint |
| Pagination | 10 pages, 10/page | Fetch all pages |

EOF
```

---

### File 11: `README.md` (Main Assignment Instructions)

```bash
cat > README.md << 'EOF'
# Mixo Ads - Backend Engineering Assignment

## 🎯 Overview

You've inherited a backend service that syncs campaign performance data from an Ad Platform API to our database. The previous developer got it "working" for a single test account but left before handling production scale.

**The problem:** The code works... sometimes. It breaks frequently, has no error handling, and won't scale past 5 clients.

**Your job:** Fix it, restructure it, and document it properly.

**Time estimate:** 4-5 hours  
**Deadline:** 72 hours from receiving this assignment

---

## 🚀 Setup Instructions

### Prerequisites
- Node.js 18+ and npm
- Git

### Step 1: Fork this repository

Click the "Fork" button in the top right of this repository.

### Step 2: Clone your fork

```bash
git clone https://github.com/YOUR_USERNAME/mixoads-backend-assignment.git
cd mixoads-backend-assignment
```

### Step 3: Install dependencies

```bash
# Install main app dependencies
npm install

# Install mock API dependencies
cd mock-api
npm install
cd ..
```

### Step 4: Set up environment variables

```bash
cp .env.example .env
```

The `.env` file contains the credentials needed for the mock API.

### Step 5: Start the Mock API

In one terminal window:

```bash
cd mock-api
npm start
```

The mock API will run on http://localhost:3001

You should see:
```
🚀 Mock Ad Platform API Server
📍 Server running on: http://localhost:3001
🔑 Valid credentials: admin@mixoads.com / SuperSecret123!
```

### Step 6: Run the broken sync script

In another terminal window:

```bash
npm start
```

**Watch it fail.** 🔥

You'll see various issues:
- Only 10 campaigns synced (should be 100)
- Credentials logged in plain text
- Random failures
- Slow sequential processing
- And more...

---

## 🐛 What's Wrong?

The code has **multiple critical issues**. Some are obvious, some are subtle. You need to find and fix them.

**Categories of issues:**
- ❌ Authentication problems (hardcoded, logged, insecure)
- ❌ Rate limiting not handled (API has 10 req/min limit)
- ❌ No error handling (crashes on failures)
- ❌ No retry logic (transient failures kill the job)
- ❌ No timeout handling (hangs forever)
- ❌ Pagination broken (only gets first 10 of 100 campaigns)
- ❌ Performance problems (sequential, slow)
- ❌ Code structure disasters (god function, mixed concerns)
- ❌ Database issues (duplicates, SQL injection, connection leaks)
- ❌ Security problems (exposed credentials, stack traces)

---

## ✅ What You Need To Do

### 1. Fix the Critical Bugs (Must Do)

- **Authentication:** Move credentials to environment variables, stop logging them
- **Rate Limiting:** Handle 429 responses with proper backoff
- **Error Handling:** Add try-catch blocks, don't crash on failures
- **Retry Logic:** Implement exponential backoff for transient failures
- **Timeout Handling:** Add timeouts so requests don't hang forever
- **Pagination:** Fetch all 100 campaigns, not just the first 10
- **Database:** Fix duplicates, use parameterized queries, proper connection management

### 2. Improve Code Structure (Should Do)

- **Break apart the god function** - It's 100+ lines doing everything
- **Separate concerns:**
  - Auth management (separate module)
  - API client (separate module)
  - Database operations (separate module)
  - Business logic (sync service)
- **Remove copy-pasted code**
- **Make it testable** - separate logic from I/O
- **Clean up naming** - use descriptive variable/function names
- **Add proper logging** - structured logs without sensitive data

### 3. Document Everything (Must Do)

Complete the `SUBMISSION.md` file (template provided) with:

**Part A: What Was Broken**
- List 5-7 major issues you found
- Explain why each was a problem
- Be specific about the impact

**Part B: How You Fixed It**
- Explain your approach to each fix
- Why did you structure it this way?
- What trade-offs did you make?
- Link to specific code changes

**Part C: How to Run It**
- Clear setup instructions
- What should happen when it works?
- How to test it?

**Part D: Production Considerations**
- What would you add before deploying to production?
- What monitoring/alerting would you set up?
- What could still go wrong?
- How would you scale to 100+ clients?

---

## 🎮 The Mock API

The mock API simulates a real ad platform with realistic constraints:

**Authentication:**
- Use Basic auth (base64 email:password) to get Bearer token
- Tokens expire after 1 hour
- Credentials: `admin@mixoads.com` / `SuperSecret123!`

**Rate Limits:**
- 10 requests per minute
- Returns 429 with `retry-after` header if exceeded

**Reliability Issues:**
- Random 503 errors (20% of requests) - should retry
- Random timeouts (10% of requests) - should timeout
- Slow sync endpoint (2 seconds) - should handle

**Data:**
- 100 total campaigns
- Paginated: 10 campaigns per page
- Must fetch all 10 pages to get complete data

See `mock-api/README.md` for full API documentation.

---

## 📤 Submission Instructions

### When you're done:

1. **Commit your changes** with clear, logical commits
   ```bash
   git add .
   git commit -m "Fix authentication and rate limiting"
   git commit -m "Refactor into separate modules"
   git commit -m "Add comprehensive error handling"
   ```

2. **Complete SUBMISSION.md** with all required sections

3. **Push to your fork**
   ```bash
   git push origin main
   ```

4. **Open a Pull Request** back to the original repository
   - Click "Contribute" → "Open pull request"
   - Base: `mixoads/mixoads-backend-assignment` (main)
   - Compare: `your-username/mixoads-backend-assignment` (main)

### Your PR should include:

- **PR Title:** `Backend Engineer Assignment - [Your Name]`
- **PR Description:**
  - Brief summary of changes
  - Time spent (be honest!)
  - Any blockers or questions
  - One thing you're proud of

---

## 📊 What We're Evaluating

### Must Have (70% of score):
- ✅ Fixed critical operational bugs
- ✅ System works reliably (we'll test it)
- ✅ Clear, comprehensive documentation
- ✅ Code is readable and organized

### Nice to Have (30% of score):
- 🌟 Significantly improved code structure
- 🌟 Added thoughtful logging/monitoring
- 🌟 Considered edge cases we didn't mention
- 🌟 Documentation shows deep production thinking

### Red Flags (Auto-Reject):
- ❌ Only fixed syntax errors, missed systemic issues
- ❌ Fixed one thing but broke another
- ❌ Can't explain decisions in documentation
- ❌ Over-engineered (10 classes for simple sync)
- ❌ Didn't test - code doesn't actually work

---

## 💡 Tips

- **Test your fixes** - Run it multiple times, make sure it works
- **Document as you go** - Easier than writing everything at the end
- **Don't over-engineer** - Practical fixes > perfect architecture
- **Use any resources** - Google, Stack Overflow, LLMs, docs - whatever helps
- **Ask if stuck** - Email hari@mixoads.com, we'll respond quickly

**Remember:** We want to see how you think, not just that you can write code. Your documentation is as important as your code.

---

## ❓ Questions?

Email: hari@mixoads.com

---

**Good luck! We're excited to see your work.** 🚀
EOF
```

---

### File 12: `SUBMISSION.md` (Template for candidates)

```bash
cat > SUBMISSION.md << 'EOF'
# Backend Engineer Assignment - Submission

**Name:** [Your Name]  
**Date:** [Submission Date]  
**Time Spent:** [Honest estimate]  
**GitHub:** [Your GitHub username]

---

## Part 1: What Was Broken

List the major issues you identified. For each issue, explain what was wrong and why it mattered.

### Issue 1: [Issue Name]
**What was wrong:**  
[Detailed explanation]

**Why it mattered:**  
[Impact on system - crashes? data loss? security? performance?]

**Where in the code:**  
[File and line numbers or function names]

---

### Issue 2: [Issue Name]
**What was wrong:**  
[Detailed explanation]

**Why it mattered:**  
[Impact]

**Where in the code:**  
[Location]

---

### Issue 3: [Issue Name]
**What was wrong:**  


**Why it mattered:**  


**Where in the code:**  


---

[Continue for 5-7 major issues total]

---

## Part 2: How I Fixed It

For each issue above, explain your fix in detail.

### Fix 1: [Issue Name]

**My approach:**  
[What did you do to fix it?]

**Why this approach:**  
[Why did you choose this solution over alternatives?]

**Trade-offs:**  
[What compromises did you make? What would you do differently with more time?]

**Code changes:**  
[Link to commits, files, or specific line numbers]

---

### Fix 2: [Issue Name]

**My approach:**  


**Why this approach:**  


**Trade-offs:**  


**Code changes:**  


---

[Continue for all fixes]

---

## Part 3: Code Structure Improvements

Explain how you reorganized/refactored the code.

**What I changed:**  
[Describe the new structure - what modules/files did you create?]

**Why it's better:**  
[Improved testability? Separation of concerns? Reusability?]

**Architecture decisions:**  
[Any patterns you used? Class-based? Functional? Why?]

---

## Part 4: Testing & Verification

How did you verify your fixes work?

**Test scenarios I ran:**
1. [Scenario 1 - e.g., "Ran sync 10 times to test reliability"]
2. [Scenario 2 - e.g., "Made 20 requests to test rate limiting"]
3. [etc.]

**Expected behavior:**  
[What should happen when it works correctly?]

**Actual results:**  
[What happened when you tested?]

**Edge cases tested:**  
[What unusual scenarios did you test?]

---

## Part 5: Production Considerations

What would you add/change before deploying this to production?

### Monitoring & Observability
[What metrics would you track? What alerts would you set up?]

### Error Handling & Recovery
[What additional error handling would you add?]

### Scaling Considerations
[How would this handle 100+ clients? What would break first?]

### Security Improvements
[What security enhancements would you add?]

### Performance Optimizations
[What could be made faster or more efficient?]

---

## Part 6: Limitations & Next Steps

Be honest about what's still not perfect.

**Current limitations:**  
[What's still not production-ready?]

**What I'd do with more time:**  
[If you had another 5 hours, what would you improve?]

**Questions I have:**  
[Anything you're unsure about or would want to discuss?]

---

## Part 7: How to Run My Solution

Clear step-by-step instructions.

### Setup
```bash
# Step-by-step commands
```

### Running
```bash
# How to start everything
```

### Expected Output
```
# What should you see when it works?
```

### Testing
```bash
# How to verify it's working correctly
```

---

## Part 8: Additional Notes

Any other context, thoughts, or reflections on the assignment.

[Your thoughts here]

---

## Commits Summary

List your main commits and what each one addressed:

1. `[commit hash]` - [Description of what this commit fixed]
2. `[commit hash]` - [Description]
3. etc.

---

**Thank you for reviewing my submission!**
EOF
```

---

## Part 3: Commit and Push Everything

```bash
# Check all files created
git status

# Add all files
git add .

# Commit
git commit -m "Initial broken assignment setup"

# Push to GitHub
git push origin main
```

---

## Part 4: Set Up Branch Protection

1. Go to your repository on GitHub
2. Click **Settings** → **Branches**
3. Click **Add rule**
4. Branch name pattern: `main`
5. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (set to 1)
   - ✅ Include administrators
6. Click **Create** or **Save changes**

---

## Part 5: Test the Assignment Yourself

Before sending to candidates, test it works:

```bash
# Terminal 1: Start mock API
cd mock-api
npm install
npm start

# Terminal 2: Run broken code
cd ..
npm install
npm start
```

**You should see:**
- Credentials logged
- Only 10 campaigns synced (not 100)
- Some random failures
- Slow sequential processing

**This confirms the assignment is working as intended - broken! 🔥**

---

## Part 6: Share with Candidates

### Option A: Individual Access (Private Repo)

1. Keep repo private
2. For each candidate, go to **Settings** → **Manage access**
3. Click **Add people**
4. Add their GitHub username with **Read** access
5. Email them:

```
Subject: Mixo Ads - Backend Engineering Assignment

Hi [Name],

Here's your technical assignment:

Repository: https://github.com/mixoads/mixoads-backend-assignment

Instructions:
1. Fork the repository
2. Follow the README.md for setup
3. Fix the broken code
4. Complete SUBMISSION.md
5. Open a Pull Request back to the original repo

Deadline: 72 hours from now

Questions? Reply to this email.

Good luck!
- Hari
```

### Option B: Public Repo (Easier)

1. Make repo public
2. Send all candidates the same link
3. They fork → fix → submit PR

---

## Part 7: Review Submissions

When candidates submit PRs:

1. Go to **Pull Requests** tab in your repo
2. Click on their PR
3. Review:
   - **Files changed** tab - see their code fixes
   - **SUBMISSION.md** - read their documentation
   - **Commits** tab - see their git hygiene
4. Test their code:
   ```bash
   # Add their remote
   git remote add candidate-name https://github.com/candidate/mixoads-backend-assignment.git
   
   # Fetch their branch
   git fetch candidate-name
   
   # Check out their code
   git checkout candidate-name/main
   
   # Test it
   cd mock-api && npm start &
   npm start
   ```
5. Leave feedback in PR comments
6. Approve or request changes

---

## Checklist Before Sending

- [ ] All files created and pushed to GitHub
- [ ] Branch protection enabled on `main`
- [ ] Tested the broken code yourself
- [ ] Mock API runs correctly
- [ ] Main app fails as expected
- [ ] README.md is clear and complete
- [ ] SUBMISSION.md template is comprehensive
- [ ] `.env.example` has correct credentials

---

## Troubleshooting

**Problem:** npm install fails  
**Solution:** Delete `package-lock.json` and `node_modules`, run `npm install` again

**Problem:** Mock API won't start  
**Solution:** Check if port 3001 is already in use: `lsof -i :3001`

**Problem:** TypeScript errors  
**Solution:** Make sure TypeScript is installed: `npm install -g typescript`

**Problem:** Candidates can't fork  
**Solution:** Make sure repo is either public or they have access

---

**You're all set! The assignment is ready to use.** 🎉