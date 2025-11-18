---
title: 'Chat Complete API'
description: 'Complete AI response in a single request for batch processing'
---

# POST /api/ai/complete (Non-Streaming)

Get complete AI response in a single request for batch processing and non-interactive applications.

## Endpoint

```
POST https://regpilot.dev/api/ai/complete
```

## Authentication

Required header: `X-API-Key: sk_your_api_key`

## Request

### Body Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `messages` | Array | Yes | - | Array of message objects |
| `quality` | String | No | `balanced` | `cheap`, `balanced`, or `frontier` |
| `temperature` | Number | No | `0.7` | Randomness (0-2) |
| `maxTokens` | Number | No | `1000` | Maximum tokens to generate |
| `mode` | String | No | `text` | `text` or `json` |

## Response

### Response Body

```typescript
{
  text: string,              // Generated response
  provider: string,          // AI provider used
  model: string,            // Model name
  cacheHit: boolean,        // Was cached?
  usage: {
    promptTokens: number,
    completionTokens: number,
    totalTokens: number,
    costCents: number       // Cost in cents
  }
}
```

## Code Examples

<CodeGroup>

```typescript TypeScript
const response = await fetch('https://regpilot.dev/api/ai/complete', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [
      { role: 'user', content: 'Summarize quantum computing' }
    ],
    quality: 'balanced',
    maxTokens: 500
  })
});

const data = await response.json();
console.log(data.text);
console.log(`Used ${data.usage.totalTokens} tokens, cost: $${data.usage.costCents / 100}`);
```

```python Python
import os
import requests

response = requests.post(
    'https://regpilot.dev/api/ai/complete',
    headers={
        'X-API-Key': os.getenv('REGPILOT_API_KEY'),
        'Content-Type': 'application/json'
    },
    json={
        'messages': [
            {'role': 'user', 'content': 'Summarize quantum computing'}
        ],
        'quality': 'balanced',
        'maxTokens': 500
    }
)

data = response.json()
print(data['text'])
print(f"Used {data['usage']['totalTokens']} tokens, cost: ${data['usage']['costCents'] / 100}")
```

</CodeGroup>

## JSON Mode

Request structured JSON responses:

```typescript
const response = await fetch('https://regpilot.dev/api/ai/complete', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [
      { 
        role: 'user', 
        content: 'List 3 programming languages. Return as JSON with name and description fields.' 
      }
    ],
    mode: 'json',
    quality: 'balanced'
  })
});

const data = await response.json();
const languages = JSON.parse(data.text);
console.log(languages);
// Output: [{"name": "Python", "description": "..."}, ...]
```

## Use Cases

### Batch Processing

```typescript
const prompts = [
  'Translate "hello" to Spanish',
  'Translate "hello" to French',
  'Translate "hello" to German'
];

const results = await Promise.all(
  prompts.map(async (prompt) => {
    const response = await fetch('https://regpilot.dev/api/ai/complete', {
      method: 'POST',
      headers: {
        'X-API-Key': process.env.REGPILOT_API_KEY!,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        messages: [{ role: 'user', content: prompt }],
        quality: 'cheap'
      })
    });
    return response.json();
  })
);

results.forEach((r, i) => {
  console.log(`${prompts[i]}: ${r.text}`);
});
```

### Background Jobs

```typescript
// Queue job
async function processDocument(documentId: string) {
  const document = await getDocument(documentId);
  
  const response = await fetch('https://regpilot.dev/api/ai/complete', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      messages: [
        { role: 'user', content: `Summarize this document: ${document.content}` }
      ],
      quality: 'balanced',
      maxTokens: 1000
    })
  });
  
  const result = await response.json();
  await saveSummary(documentId, result.text);
  
  return result;
}
```

---

**Next:** [Usage Stats →](./usage-stats.md)
