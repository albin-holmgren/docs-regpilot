---
title: 'Authentication'
description: 'Complete guide to API key authentication'
---

# Authentication

RegPilot uses API key authentication for all requests.

## API Key Format

API keys follow the format:
```
sk_[32_character_hash]
```

Example: `sk_7f8a9b0c1d2e3f4g5h6i7j8k9l0m1n2`

## Getting Your API Key

<Steps>
  <Step title="Navigate to Dashboard">
    Log into [https://regpilot.dev](https://regpilot.dev)
  </Step>
  
  <Step title="Select Project">
    Choose the project you want to create a key for
  </Step>
  
  <Step title="Manage API Keys">
    Click **"Manage API Keys"** in the Overview page
  </Step>
  
  <Step title="Create New Key">
    1. Click **"Create New Key"**
    2. Give it a descriptive name
    3. Copy the key immediately
    
    <Warning>
      You'll only see the key once! Store it securely.
    </Warning>
  </Step>
</Steps>

## Authentication Methods

RegPilot supports multiple header formats for maximum compatibility:

### Recommended: X-API-Key Header (Production-Safe)

```bash
X-API-Key: sk_your_api_key
```

<Tip>
  **Best for production:** This header is never stripped by proxies or CDNs, unlike `Authorization`.
</Tip>

**Example:**
```bash
curl -X POST https://regpilot.dev/api/ai/chat \
  -H "X-API-Key: sk_7f8a9b0c1d2e3f4g5h6i7j8k9l0m1n2" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

### Alternative: Authorization Bearer

```bash
Authorization: Bearer sk_your_api_key
```

<Warning>
  **Production Warning:** Some hosting providers (including Vercel) strip the `Authorization` header. Use `X-API-Key` instead.
</Warning>

**Example:**
```bash
curl -X POST https://regpilot.dev/api/ai/chat \
  -H "Authorization: Bearer sk_7f8a9b0c1d2e3f4g5h6i7j8k9l0m1n2" \
  -H "Content-Type: application/json" \
  -d '{"messages": [...]}'
```

### Other Supported Headers

RegPilot checks these headers in priority order:

1. `X-API-Key` (recommended)
2. `X-RegPilot-Key`
3. `Api-Key`
4. `Authorization`
5. `X-Authorization`
6. `Proxy-Authorization`

## Code Examples

<CodeGroup>

```typescript TypeScript/Node.js
const REGPILOT_API_KEY = process.env.REGPILOT_API_KEY;

const response = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': REGPILOT_API_KEY,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello!' }]
  })
});
```

```python Python
import os
import requests

REGPILOT_API_KEY = os.getenv('REGPILOT_API_KEY')

response = requests.post(
    'https://regpilot.dev/api/ai/chat',
    headers={
        'X-API-Key': REGPILOT_API_KEY,
        'Content-Type': 'application/json'
    },
    json={
        'messages': [{'role': 'user', 'content': 'Hello!'}]
    }
)
```

```go Go
package main

import (
    "bytes"
    "encoding/json"
    "net/http"
    "os"
)

func main() {
    apiKey := os.Getenv("REGPILOT_API_KEY")
    
    body, _ := json.Marshal(map[string]interface{}{
        "messages": []map[string]string{
            {"role": "user", "content": "Hello!"},
        },
    })
    
    req, _ := http.NewRequest("POST", 
        "https://regpilot.dev/api/ai/chat", 
        bytes.NewBuffer(body))
    
    req.Header.Set("X-API-Key", apiKey)
    req.Header.Set("Content-Type", "application/json")
    
    client := &http.Client{}
    resp, _ := client.Do(req)
    defer resp.Body.Close()
}
```

```ruby Ruby
require 'net/http'
require 'json'

api_key = ENV['REGPILOT_API_KEY']
uri = URI('https://regpilot.dev/api/ai/chat')

http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Post.new(uri)
request['X-API-Key'] = api_key
request['Content-Type'] = 'application/json'
request.body = {
  messages: [{ role: 'user', content: 'Hello!' }]
}.to_json

response = http.request(request)
```

</CodeGroup>

## Environment Variables

Store API keys securely in environment variables:

### Local Development

```bash
# .env
REGPILOT_API_KEY=sk_your_api_key_here
```

### Vercel

```bash
vercel env add REGPILOT_API_KEY
```

### Netlify

```bash
netlify env:set REGPILOT_API_KEY sk_your_api_key_here
```

### Docker

```yaml
# docker-compose.yml
services:
  app:
    environment:
      - REGPILOT_API_KEY=sk_your_api_key_here
```

### GitHub Actions

```yaml
# .github/workflows/test.yml
env:
  REGPILOT_API_KEY: ${{ secrets.REGPILOT_API_KEY }}
```

## API Key Management

### Creating Multiple Keys

You can create multiple API keys for different purposes:

- **Development:** `sk_dev_...`
- **Staging:** `sk_staging_...`
- **Production:** `sk_prod_...`
- **CI/CD:** `sk_ci_...`

<Tip>
  Use descriptive names to track key usage in the dashboard.
</Tip>

### Key Permissions

Each API key is scoped to:
- ✅ **Project-level** access
- ✅ **Read/Write** permissions for AI Gateway
- ✅ **Read** permissions for logs and analytics
- ❌ **No admin** permissions (use dashboard for settings)

### Rotating Keys

Best practice: Rotate API keys every 90 days

<Steps>
  <Step title="Create New Key">
    Generate a new API key in the dashboard
  </Step>
  
  <Step title="Update Applications">
    Deploy the new key to all environments
  </Step>
  
  <Step title="Monitor Usage">
    Verify new key is working
  </Step>
  
  <Step title="Revoke Old Key">
    Delete the old key from dashboard
  </Step>
</Steps>

### Revoking Keys

To revoke a key:
1. Go to **Manage API Keys**
2. Click **"Revoke"** next to the key
3. Confirm revocation

<Warning>
  Revoked keys stop working immediately. Update your applications first!
</Warning>

## Security Best Practices

### ✅ Do's

- Store keys in environment variables
- Use different keys for dev/staging/prod
- Rotate keys regularly (every 90 days)
- Monitor key usage in dashboard
- Revoke compromised keys immediately
- Use `X-API-Key` header in production

### ❌ Don'ts

- Never commit keys to Git
- Never share keys in Slack/email
- Never log keys to console
- Never expose keys in client-side code
- Never use the same key across environments
- Never use `Authorization: Bearer` in production (gets stripped)

## Error Responses

### 401 Unauthorized

**Missing API Key:**
```json
{
  "error": "Missing API key. Use one of: X-API-Key: YOUR_KEY, X-RegPilot-Key: YOUR_KEY, or Authorization: Bearer YOUR_KEY",
  "headers_checked": [
    "x-api-key",
    "x-regpilot-key",
    "api-key",
    "authorization",
    "x-authorization",
    "proxy-authorization"
  ]
}
```

**Invalid API Key:**
```json
{
  "error": "Invalid or inactive API key"
}
```

**Revoked API Key:**
```json
{
  "error": "API key has been revoked"
}
```

### 403 Forbidden

```json
{
  "error": "Insufficient permissions for this operation"
}
```

## Testing Authentication

### Test with cURL

```bash
# Should return 401
curl -X POST https://regpilot.dev/api/ai/chat \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "test"}]}'

# Should return 200
curl -X POST https://regpilot.dev/api/ai/chat \
  -H "X-API-Key: $REGPILOT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "test"}]}'
```

### Test Script

```typescript
// test-auth.ts
async function testAuth() {
  const apiKey = process.env.REGPILOT_API_KEY;
  
  if (!apiKey) {
    console.error('❌ REGPILOT_API_KEY not set');
    return;
  }
  
  try {
    const response = await fetch('https://regpilot.dev/api/ai/chat', {
      method: 'POST',
      headers: {
        'X-API-Key': apiKey,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        messages: [{ role: 'user', content: 'test' }],
        quality: 'cheap'
      })
    });
    
    if (response.ok) {
      console.log('✅ Authentication successful');
      console.log('Model:', response.headers.get('x-regpilot-model'));
    } else {
      console.error('❌ Authentication failed:', response.status);
      const error = await response.json();
      console.error(error);
    }
  } catch (error) {
    console.error('❌ Network error:', error);
  }
}

testAuth();
```

## Rate Limiting

API keys are subject to rate limits based on your plan:

| Plan | Rate Limit |
|------|------------|
| Free | 10 req/min |
| Startup | 100 req/min |
| Growth | 500 req/min |
| Enterprise | Unlimited |

See [Rate Limits](/03-api-reference/rate-limits) for details.

## Monitoring Key Usage

Track API key usage in the dashboard:

- **Last Used:** Timestamp of last request
- **Total Requests:** Lifetime request count
- **Active Environments:** Where key is being used
- **Error Rate:** Failed request percentage

<Note>
  Keys not used for 90 days are automatically flagged for review.
</Note>

---

**Next:** [Chat API (Streaming) →](./chat-streaming.md)
