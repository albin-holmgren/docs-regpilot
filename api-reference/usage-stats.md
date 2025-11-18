---
title: 'Usage Statistics'
description: 'Track your AI API usage, costs, and performance metrics'
---

# GET /api/usage - Usage Statistics

Track your AI API usage, costs, and performance metrics.

## Endpoint

```
GET https://regpilot.dev/api/usage?projectId={project_id}
```

## Authentication

Required: `X-API-Key` header

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `projectId` | String | Yes | Your project ID |
| `period` | String | No | `day`, `week`, `month` (default: `month`) |

## Response

```typescript
{
  budget: {
    limit: number,        // Credit limit ($)
    used: number,         // Credits used ($)
    remaining: number     // Credits remaining ($)
  },
  cache: {
    hits: number,         // Cache hits
    misses: number,       // Cache misses
    hitRate: number       // Hit rate (0-100%)
  },
  models: {
    [model: string]: {
      requests: number,
      tokens: number,
      cost: number
    }
  },
  period: {
    start: string,        // ISO date
    end: string          // ISO date
  }
}
```

## Example

<CodeGroup>

```typescript TypeScript
const response = await fetch(
  `https://regpilot.dev/api/usage?projectId=${projectId}`,
  {
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!
    }
  }
);

const stats = await response.json();

console.log(`Budget: $${stats.budget.used}/$${stats.budget.limit}`);
console.log(`Cache Hit Rate: ${stats.cache.hitRate}%`);
console.log(`Savings: $${stats.cache.hits * 0.01}`); // Estimated
```

```python Python
import os
import requests

response = requests.get(
    f'https://regpilot.dev/api/usage?projectId={project_id}',
    headers={'X-API-Key': os.getenv('REGPILOT_API_KEY')}
)

stats = response.json()

print(f"Budget: ${stats['budget']['used']}/${stats['budget']['limit']}")
print(f"Cache Hit Rate: {stats['cache']['hitRate']}%")
```

</CodeGroup>

---

**Back:** [API Reference →](../README.md)
