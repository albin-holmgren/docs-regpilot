---
title: 'Rate Limits'
description: 'Rate limiting policies and best practices'
---

# Rate Limits

Understand RegPilot's rate limiting policies and how to work within them.

## Rate Limits by Plan

| Plan | Requests/Minute | Burst Limit | Daily Limit |
|------|-----------------|-------------|-------------|
| **Free** | 10 | 20 | 1,000 |
| **Startup** | 100 | 200 | 100,000 |
| **Growth** | 500 | 1,000 | 1,000,000 |
| **Enterprise** | Unlimited | Unlimited | Unlimited |

## Rate Limit Headers

Every response includes rate limit information:

```typescript
const response = await fetch(/* ... */);

// Rate limit info
const limit = response.headers.get('x-ratelimit-limit');        // e.g., "100"
const remaining = response.headers.get('x-ratelimit-remaining'); // e.g., "95"
const reset = response.headers.get('x-ratelimit-reset');         // Unix timestamp

console.log(`${remaining}/${limit} requests remaining`);
console.log(`Resets at: ${new Date(parseInt(reset) * 1000)}`);
```

## Handling Rate Limits

### Check Before Making Request

```typescript
function canMakeRequest(headers: Headers): boolean {
  const remaining = parseInt(headers.get('x-ratelimit-remaining') || '0');
  return remaining > 0;
}

let lastResponse: Response;

if (lastResponse && !canMakeRequest(lastResponse.headers)) {
  const reset = parseInt(lastResponse.headers.get('x-ratelimit-reset') || '0');
  const wait = reset - Math.floor(Date.now() / 1000);
  console.log(`Waiting ${wait} seconds for rate limit reset...`);
  await new Promise(resolve => setTimeout(resolve, wait * 1000));
}
```

### Handle 429 Responses

```typescript
async function chatWithRateLimitHandling(messages: any[]) {
  const response = await fetch('https://regpilot.dev/api/ai/chat', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ messages })
  });

  if (response.status === 429) {
    const retryAfter = parseInt(response.headers.get('retry-after') || '60');
    console.log(`Rate limited. Retrying after ${retryAfter}s`);
    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
    return chatWithRateLimitHandling(messages); // Retry
  }

  return response;
}
```

## Best Practices

### 1. Request Queuing

```typescript
class RequestQueue {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private requestsPerMinute: number;
  private interval: number;

  constructor(requestsPerMinute: number) {
    this.requestsPerMinute = requestsPerMinute;
    this.interval = 60000 / requestsPerMinute; // ms between requests
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, this.interval));
    }
    
    this.processing = false;
  }
}

// Usage
const queue = new RequestQueue(100); // 100 req/min

const result = await queue.add(() =>
  fetch('https://regpilot.dev/api/ai/chat', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ messages: [...] })
  })
);
```

### 2. Batch Processing

```typescript
async function batchProcess(items: string[], batchSize: number = 10) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(item =>
        fetch('https://regpilot.dev/api/ai/chat', {
          method: 'POST',
          headers: {
            'X-API-Key': process.env.REGPILOT_API_KEY!,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            messages: [{ role: 'user', content: item }]
          })
        }).then(r => r.text())
      )
    );
    
    results.push(...batchResults);
    
    // Wait before next batch
    if (i + batchSize < items.length) {
      await new Promise(resolve => setTimeout(resolve, 6000)); // 6 seconds
    }
  }
  
  return results;
}
```

### 3. Monitor Usage

```typescript
class RateLimitMonitor {
  private requests: number[] = [];
  private window = 60000; // 1 minute

  recordRequest() {
    const now = Date.now();
    this.requests.push(now);
    this.requests = this.requests.filter(time => time > now - this.window);
  }

  getUsage(): { current: number, percentage: number } {
    const now = Date.now();
    this.requests = this.requests.filter(time => time > now - this.window);
    
    return {
      current: this.requests.length,
      percentage: (this.requests.length / 100) * 100 // Assuming 100 req/min limit
    };
  }

  shouldWait(limit: number): boolean {
    return this.requests.length >= limit;
  }
}
```

## Upgrade for Higher Limits

Need more capacity? Upgrade your plan:

<CardGroup cols={3}>
  <Card title="Startup" icon="rocket">
    **100 req/min**
    
    $399-$499/month
  </Card>
  
  <Card title="Growth" icon="chart-line">
    **500 req/min**
    
    $799-$999/month
  </Card>
  
  <Card title="Enterprise" icon="building">
    **Unlimited**
    
    $1,999-$2,499/month
  </Card>
</CardGroup>

---

**Next:** [API Reference Overview →](../README.md)
