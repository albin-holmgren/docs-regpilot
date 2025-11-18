---
title: 'Error Handling'
description: 'Complete guide to API errors and how to handle them'
---

# Error Codes & Handling

Complete guide to RegPilot API errors and how to handle them.

## HTTP Status Codes

| Code | Status | Meaning |
|------|--------|---------|
| `200` | Success | Request completed successfully |
| `400` | Bad Request | Invalid request format or parameters |
| `401` | Unauthorized | Missing or invalid API key |
| `403` | Forbidden | Insufficient permissions |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server-side error |
| `503` | Service Unavailable | Temporary service disruption |

## Error Response Format

All errors return JSON with this structure:

```typescript
{
  error: string,           // Error message
  message?: string,        // Detailed explanation
  details?: object,        // Additional context
  code?: string           // Error code (e.g., 'RATE_LIMIT_EXCEEDED')
}
```

## Common Errors

### 400 Bad Request

#### Missing Required Field
```json
{
  "error": "Messages array is required",
  "code": "MISSING_MESSAGES"
}
```

**Solution:**
```typescript
// ❌ Bad
body: JSON.stringify({
  quality: 'balanced'
})

// ✅ Good
body: JSON.stringify({
  messages: [{ role: 'user', content: 'Hello' }],
  quality: 'balanced'
})
```

#### Invalid Quality Tier
```json
{
  "error": "Invalid quality tier. Must be: cheap, balanced, or frontier",
  "code": "INVALID_QUALITY"
}
```

### 401 Unauthorized

#### Missing API Key
```json
{
  "error": "Missing API key. Use one of: X-API-Key: YOUR_KEY, X-RegPilot-Key: YOUR_KEY, or Authorization: Bearer YOUR_KEY",
  "headers_checked": ["x-api-key", "x-regpilot-key", "api-key", "authorization"]
}
```

**Solution:**
```typescript
// ✅ Add API key header
headers: {
  'X-API-Key': process.env.REGPILOT_API_KEY!,
  'Content-Type': 'application/json'
}
```

#### Invalid API Key
```json
{
  "error": "Invalid or inactive API key",
  "code": "INVALID_API_KEY"
}
```

**Solution:** Verify your API key in the dashboard and regenerate if needed.

### 429 Too Many Requests

```json
{
  "error": "Rate limit exceeded",
  "retry_after": 60,
  "limit": 100,
  "window": "1 minute"
}
```

**Response Headers:**
- `Retry-After: 60` (seconds)
- `X-RateLimit-Limit: 100`
- `X-RateLimit-Remaining: 0`
- `X-RateLimit-Reset: 1699999999`

**Solution:**
```typescript
async function chatWithRetry(messages: any[]) {
  const response = await fetch('https://regpilot.dev/api/ai/chat', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ messages, quality: 'balanced' })
  });

  if (response.status === 429) {
    const retryAfter = parseInt(response.headers.get('retry-after') || '60');
    console.log(`Rate limited. Waiting ${retryAfter} seconds...`);
    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return chatWithRetry(messages); // Retry
  }

  return response;
}
```

### 500 Internal Server Error

```json
{
  "error": "Internal server error",
  "message": "Failed to process request",
  "request_id": "req_abc123"
}
```

**Solution:** Contact support@regpilot.dev with the `request_id`.

## Error Handling Patterns

### Basic Error Handling

```typescript
try {
  const response = await fetch('https://regpilot.dev/api/ai/chat', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      messages: [{ role: 'user', content: 'Hello' }]
    })
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`API Error ${response.status}: ${error.error}`);
  }

  const text = await response.text();
  return text;

} catch (error) {
  console.error('Request failed:', error);
  throw error;
}
```

### Advanced Error Handling

```typescript
async function robustChat(messages: any[]) {
  let retries = 3;
  let delay = 1000;

  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch('https://regpilot.dev/api/ai/chat', {
        method: 'POST',
        headers: {
          'X-API-Key': process.env.REGPILOT_API_KEY!,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ messages, quality: 'balanced' })
      });

      // Success
      if (response.ok) {
        return await response.text();
      }

      // Handle specific errors
      const error = await response.json();

      switch (response.status) {
        case 400:
          // Bad request - don't retry
          throw new Error(`Bad request: ${error.error}`);
        
        case 401:
          // Invalid API key - don't retry
          throw new Error('Invalid API key');
        
        case 429:
          // Rate limited - wait and retry
          const retryAfter = parseInt(response.headers.get('retry-after') || '60');
          console.log(`Rate limited. Waiting ${retryAfter}s...`);
          await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
          continue;
        
        case 500:
        case 503:
          // Server error - retry with backoff
          if (i < retries - 1) {
            console.log(`Server error. Retrying in ${delay}ms...`);
            await new Promise(resolve => setTimeout(resolve, delay));
            delay *= 2; // Exponential backoff
            continue;
          }
          throw new Error(`Server error: ${error.message}`);
        
        default:
          throw new Error(`Unexpected error: ${error.error}`);
      }

    } catch (error) {
      if (i === retries - 1) throw error;
    }
  }

  throw new Error('Max retries exceeded');
}
```

### Python Error Handling

```python
import requests
import time
import os

def chat_with_retry(messages, max_retries=3):
    delay = 1
    
    for attempt in range(max_retries):
        try:
            response = requests.post(
                'https://regpilot.dev/api/ai/chat',
                headers={
                    'X-API-Key': os.getenv('REGPILOT_API_KEY'),
                    'Content-Type': 'application/json'
                },
                json={'messages': messages, 'quality': 'balanced'}
            )
            
            # Success
            if response.status_code == 200:
                return response.text
            
            # Rate limited
            if response.status_code == 429:
                retry_after = int(response.headers.get('retry-after', 60))
                print(f'Rate limited. Waiting {retry_after}s...')
                time.sleep(retry_after)
                continue
            
            # Server error - retry with backoff
            if response.status_code >= 500:
                if attempt < max_retries - 1:
                    print(f'Server error. Retrying in {delay}s...')
                    time.sleep(delay)
                    delay *= 2
                    continue
            
            # Other errors - raise
            response.raise_for_status()
        
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(delay)
            delay *= 2
    
    raise Exception('Max retries exceeded')
```

## Validation Errors

### Invalid Message Format

```json
{
  "error": "Invalid message format. Each message must have 'role' and 'content'",
  "code": "INVALID_MESSAGE_FORMAT"
}
```

### Token Limit Exceeded

```json
{
  "error": "Token limit exceeded. Maximum 8000 tokens per request",
  "code": "TOKEN_LIMIT_EXCEEDED",
  "details": {
    "estimated_tokens": 12000,
    "limit": 8000
  }
}
```

## Best Practices

### 1. Always Check Status

```typescript
if (!response.ok) {
  const error = await response.json();
  // Handle error
}
```

### 2. Implement Exponential Backoff

```typescript
const delay = 1000 * Math.pow(2, attempt);
await new Promise(resolve => setTimeout(resolve, delay));
```

### 3. Log Request IDs

```typescript
const requestId = response.headers.get('x-request-id');
console.error(`Request failed: ${requestId}`);
```

### 4. Circuit Breaker Pattern

```typescript
class CircuitBreaker {
  private failures = 0;
  private lastFailure = 0;
  private threshold = 5;
  private timeout = 60000; // 1 minute

  async call(fn: () => Promise<any>) {
    if (this.isOpen()) {
      throw new Error('Circuit breaker is open');
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private isOpen() {
    return this.failures >= this.threshold &&
           Date.now() - this.lastFailure < this.timeout;
  }

  private onSuccess() {
    this.failures = 0;
  }

  private onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
  }
}
```

---

**Next:** [Rate Limits →](./rate-limits.md)
