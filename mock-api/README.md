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
