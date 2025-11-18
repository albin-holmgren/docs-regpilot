---
title: 'Chat Streaming API'
description: 'Real-time streaming chat endpoint for interactive AI applications'
---

# POST /api/ai/chat (Streaming)

Real-time streaming chat endpoint for interactive AI applications.

## Endpoint

```
POST https://regpilot.dev/api/ai/chat
```

## Authentication

Required header: `X-API-Key: sk_your_api_key`

See [Authentication](/03-api-reference/authentication) for details.

## Request

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `X-API-Key` | Yes | Your project API key |
| `Content-Type` | Yes | Must be `application/json` |

### Body Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `messages` | Array | Yes | - | Array of message objects |
| `quality` | String | No | `balanced` | `cheap`, `balanced`, or `frontier` |
| `temperature` | Number | No | `0.7` | Randomness (0-2) |
| `model` | String | No | Auto | Specific model override |
| `governorMetadata` | Object | No | - | Compliance validation metadata |

### Message Object

```typescript
{
  role: 'user' | 'assistant' | 'system',
  content: string
}
```

### Request Example

```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "Explain quantum computing in simple terms."
    }
  ],
  "quality": "balanced",
  "temperature": 0.7
}
```

## Response

### Response Headers

| Header | Description |
|--------|-------------|
| `Content-Type` | `text/event-stream` |
| `x-regpilot-model` | Model used (e.g., `openai/gpt-4o-mini`) |
| `x-credits-charged` | Credits charged for request |
| `x-credits-remaining` | Remaining credit balance |
| `x-regpilot-prompt-tokens` | Input tokens used |
| `x-regpilot-completion-tokens` | Output tokens generated |
| `x-cache-status` | `HIT` or `MISS` |

### Governor Headers (if enabled)

| Header | Description |
|--------|-------------|
| `x-governor-validated` | `true` if validated |
| `x-governor-approved` | `true` if approved |
| `x-governor-risk-level` | `low`, `medium`, `high`, `critical` |
| `x-governor-risk-score` | Risk score (0-100) |
| `x-governor-violations` | Number of violations found |
| `x-governor-audit-id` | Audit trail ID |

### Response Body

Streaming text response via Server-Sent Events.

## Code Examples

<CodeGroup>

```typescript TypeScript/JavaScript
const response = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [
      { role: 'user', content: 'Write a haiku about coding' }
    ],
    quality: 'balanced'
  })
});

// Stream the response
const reader = response.body?.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  
  const text = decoder.decode(value);
  process.stdout.write(text);
}

// Check response headers
console.log('Model:', response.headers.get('x-regpilot-model'));
console.log('Credits:', response.headers.get('x-credits-charged'));
```

```python Python
import os
import requests

response = requests.post(
    'https://regpilot.dev/api/ai/chat',
    headers={
        'X-API-Key': os.getenv('REGPILOT_API_KEY'),
        'Content-Type': 'application/json'
    },
    json={
        'messages': [
            {'role': 'user', 'content': 'Write a haiku about coding'}
        ],
        'quality': 'balanced'
    },
    stream=True
)

# Stream the response
for chunk in response.iter_content(chunk_size=None):
    if chunk:
        print(chunk.decode('utf-8'), end='', flush=True)

# Check headers
print('\nModel:', response.headers.get('x-regpilot-model'))
print('Credits:', response.headers.get('x-credits-charged'))
```

```bash cURL
curl -X POST https://regpilot.dev/api/ai/chat \
  -H "X-API-Key: $REGPILOT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "Write a haiku about coding"}
    ],
    "quality": "balanced"
  }' \
  --no-buffer
```

</CodeGroup>

## Quality Tiers

RegPilot automatically routes your request to the optimal model based on quality:

| Quality | Model | Speed | Cost | Best For |
|---------|-------|-------|------|----------|
| `cheap` | Claude Haiku | ⚡ Fast | $ | High-volume, simple queries |
| `balanced` | GPT-4o Mini | ⚡ Fast | $$ | General-purpose applications |
| `frontier` | GPT-4o | 🔄 Moderate | $$$ | Complex reasoning, code generation |

## Governor Integration

Add compliance validation to your requests:

```typescript
const response = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [
      { role: 'user', content: 'Can I sue my employer for discrimination?' }
    ],
    quality: 'balanced',
    governorMetadata: {
      actionType: 'legal_advice',
      recipientCountry: 'US',
      senderId: 'user_12345',
      senderRole: 'customer_support',
      department: 'legal'
    }
  })
});

// Check Governor validation results
const validated = response.headers.get('x-governor-validated');
const approved = response.headers.get('x-governor-approved');
const riskLevel = response.headers.get('x-governor-risk-level');
const riskScore = response.headers.get('x-governor-risk-score');

console.log(`Validated: ${validated}, Approved: ${approved}`);
console.log(`Risk: ${riskLevel} (${riskScore}/100)`);
```

### Governor Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `actionType` | String | Yes | Type of action (see below) |
| `recipientCountry` | String | No | Recipient's country code |
| `recipientUserId` | String | No | Recipient user ID |
| `recipientEmail` | String | No | Recipient email |
| `senderId` | String | Yes | Sender identifier |
| `senderRole` | String | No | Sender's role |
| `department` | String | No | Department name |

### Action Types

- `customer_support` - General customer service (Low risk)
- `legal_advice` - Legal queries (Medium risk)
- `medical_advice` - Health/medical queries (Medium risk)
- `hr_message` - Human resources communications (Medium risk)
- `suspension` - Account actions (High risk)
- `refund_denial` - Payment decisions (High risk)
- `policy_warning` - Policy enforcement (Medium risk)
- `other` - General content (Low risk)

## Multi-turn Conversations

Maintain conversation context by including message history:

```typescript
const conversationHistory = [
  { role: 'user', content: 'What is machine learning?' },
  { role: 'assistant', content: 'Machine learning is a subset of AI...' },
  { role: 'user', content: 'Can you give me an example?' }
];

const response = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: conversationHistory,
    quality: 'balanced'
  })
});
```

<Tip>
  Keep conversation history under 4000 tokens for optimal performance. RegPilot automatically handles context window management.
</Tip>

## Model Override

Specify a particular model instead of using quality tiers:

```typescript
const response = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Hello!' }],
    model: 'gpt-4o'  // Override quality tier
  })
});
```

### Supported Models

- **OpenAI:** `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`
- **Anthropic:** `claude-3-opus-20240229`, `claude-3-sonnet-20240229`, `claude-3-haiku-20240307`
- **Mistral:** `mistral-large-latest`, `mistral-medium-latest`

## Caching

RegPilot automatically caches responses for identical requests:

```typescript
// First request - Cache MISS
const response1 = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'What is 2+2?' }],
    quality: 'cheap'
  })
});
console.log('Cache:', response1.headers.get('x-cache-status')); // MISS

// Second identical request - Cache HIT
const response2 = await fetch('https://regpilot.dev/api/ai/chat', {
  method: 'POST',
  headers: {
    'X-API-Key': process.env.REGPILOT_API_KEY!,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'What is 2+2?' }],
    quality: 'cheap'
  })
});
console.log('Cache:', response2.headers.get('x-cache-status')); // HIT
console.log('Credits:', response2.headers.get('x-credits-charged')); // 0 (free!)
```

<Note>
  Cache hits are **free**! You're only charged for cache misses.
</Note>

## Error Handling

```typescript
try {
  const response = await fetch('https://regpilot.dev/api/ai/chat', {
    method: 'POST',
    headers: {
      'X-API-Key': process.env.REGPILOT_API_KEY!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      messages: [{ role: 'user', content: 'Hello!' }],
      quality: 'balanced'
    })
  });

  if (!response.ok) {
    const error = await response.json();
    
    switch (response.status) {
      case 400:
        console.error('Bad request:', error.error);
        break;
      case 401:
        console.error('Invalid API key');
        break;
      case 429:
        console.error('Rate limit exceeded');
        const retryAfter = response.headers.get('retry-after');
        console.log(`Retry after ${retryAfter} seconds`);
        break;
      case 500:
        console.error('Server error:', error.message);
        break;
      default:
        console.error('Unexpected error:', error);
    }
    return;
  }

  // Process stream
  const reader = response.body?.getReader();
  const decoder = new TextDecoder();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    process.stdout.write(decoder.decode(value));
  }

} catch (error) {
  console.error('Network error:', error);
}
```

## Rate Limits

Rate limits vary by plan:

| Plan | Requests/Minute | Burst |
|------|-----------------|-------|
| Free | 10 | 20 |
| Startup | 100 | 200 |
| Growth | 500 | 1000 |
| Enterprise | Unlimited | Unlimited |

See [Rate Limits](/03-api-reference/rate-limits) for details.

## Best Practices

### 1. Stream Processing
Always process streams efficiently:

```typescript
const reader = response.body?.getReader();
const decoder = new TextDecoder();
let buffer = '';

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  
  buffer += decoder.decode(value, { stream: true });
  
  // Process complete chunks
  const lines = buffer.split('\n');
  buffer = lines.pop() || '';
  
  for (const line of lines) {
    if (line.trim()) {
      console.log(line);
    }
  }
}
```

### 2. Error Recovery
Implement retry logic for transient errors:

```typescript
async function chatWithRetry(messages, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch('https://regpilot.dev/api/ai/chat', {
        method: 'POST',
        headers: {
          'X-API-Key': process.env.REGPILOT_API_KEY!,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ messages, quality: 'balanced' })
      });
      
      if (response.ok) return response;
      
      if (response.status === 429) {
        const retryAfter = parseInt(response.headers.get('retry-after') || '60');
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
        continue;
      }
      
      throw new Error(`HTTP ${response.status}`);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

### 3. Context Management
Trim old messages to stay within token limits:

```typescript
function trimConversation(messages, maxTokens = 4000) {
  // Rough estimate: 1 token ≈ 4 characters
  let totalChars = messages.reduce((sum, m) => sum + m.content.length, 0);
  
  while (totalChars > maxTokens * 4 && messages.length > 2) {
    messages.splice(1, 1); // Remove oldest non-system message
    totalChars = messages.reduce((sum, m) => sum + m.content.length, 0);
  }
  
  return messages;
}
```

---

**Next:** [Complete (Non-Streaming) →](./chat-complete.md)
