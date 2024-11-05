# OpenAI API Reference

## Introduction

You can interact with the API through HTTP requests from any language, via our official Python bindings, our official Node.js library, or a community-maintained library.

### Installation

**Python:**
```bash
pip install openai
```

**Node.js:**
```bash
npm install openai
```

## Authentication

### API Keys

The OpenAI API uses API keys for authentication. You can create API keys at a user or service account level. Service accounts are tied to a "bot" individual and should be used to provision access for production systems. Each API key can be scoped to one of the following:

- **Project keys** - Provides access to a single project (preferred option); access Project API keys by selecting the specific project you wish to generate keys against.
- **User keys** - Our legacy keys. Provides access to all organizations and all projects that user has been added to; access API Keys to view your available keys. We highly advise transitioning to project keys for best security practices, although access via this method is currently still supported.

> **Important:** Your API key is a secret! Do not share it with others or expose it in any client-side code (browsers, apps). Production requests must be routed through your own backend server where your API key can be securely loaded from an environment variable or key management service.

All API requests should include your API key in an Authorization HTTP header:
```http
Authorization: Bearer OPENAI_API_KEY
```

### Organizations and Projects (Optional)

For users who belong to multiple organizations or are accessing their projects through their legacy user API key, you can pass a header to specify which organization and project is used for an API request. Usage from these API requests will count as usage for the specified organization and project.

To access the Default project in an organization, leave out the OpenAI-Project header.

**Example curl command:**
```bash
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "OpenAI-Organization: org-G7PRfJ7TxSlP83IHl8Qkijvw" \
  -H "OpenAI-Project: $PROJECT_ID"
```

**Python example:**
```python
from openai import OpenAI

client = OpenAI(
  organization='org-G7PRfJ7TxSlP83IHl8Qkijvw',
  project='$PROJECT_ID',
)
```

**Node.js example:**
```javascript
import OpenAI from "openai";

const openai = new OpenAI({
    organization: "org-G7PRfJ7TxSlP83IHl8Qkijvw",
    project: "$PROJECT_ID",
});
```

Organization IDs can be found on your Organization settings page. Project IDs can be found on your General settings page by selecting the specific project.

## Making Requests

Here's an example of your first API request:

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
     "model": "gpt-4o-mini",
     "messages": [{"role": "user", "content": "Say this is a test!"}],
     "temperature": 0.7
   }'
```

Example response:
```json
{
    "id": "chatcmpl-abc123",
    "object": "chat.completion",
    "created": 1677858242,
    "model": "gpt-4o-mini",
    "usage": {
        "prompt_tokens": 13,
        "completion_tokens": 7,
        "total_tokens": 20,
        "completion_tokens_details": {
            "reasoning_tokens": 0
        }
    },
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "\n\nThis is a test!"
            },
            "logprobs": null,
            "finish_reason": "stop",
            "index": 0
        }
    ]
}
```

## Streaming

The OpenAI API provides the ability to stream responses back to a client for partial results. We follow the Server-sent events standard. Our official Node and Python libraries include helpers to make parsing these events simpler.

**Python streaming example:**
```python
from openai import OpenAI

client = OpenAI()

stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Say this is a test"}],
    stream=True,
)
for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="")
```

**Node.js streaming example:**
```javascript
import OpenAI from "openai";

const openai = new OpenAI();

async function main() {
    const stream = await openai.chat.completions.create({
        model: "gpt-4o-mini",
        messages: [{ role: "user", content: "Say this is a test" }],
        stream: True,
    });
    for await (const chunk of stream) {
        process.stdout.write(chunk.choices[0]?.delta?.content || "");
    }
}

main();
```

> **Note:** Parsing Server-sent events is non-trivial and should be done with caution. Simple strategies like splitting by a new line may result in parsing errors. We recommend using existing client libraries when possible.

## Debugging Requests

### Response Headers

Important HTTP headers returned with API responses include:

**API meta information:**
- `openai-organization`: The organization associated with the request
- `openai-processing-ms`: Time taken processing your API request
- `openai-version`: REST API version used for this request (currently 2020-10-01)
- `x-request-id`: Unique identifier for this API request (used in troubleshooting)

**Rate limiting information:**
- `x-ratelimit-limit-requests`
- `x-ratelimit-limit-tokens`
- `x-ratelimit-remaining-requests`
- `x-ratelimit-remaining-tokens`
- `x-ratelimit-reset-requests`
- `x-ratelimit-reset-tokens`

### Request ID Access

OpenAI recommends logging request IDs in production deployments for efficient troubleshooting.

**Python example:**
```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    messages=[{
        "role": "user",
        "content": "Say this is a test",
    }],
    model="gpt-4o-mini",
)

print(response._request_id)
```

**JavaScript example:**
```javascript
import OpenAI from 'openai';
const client = new OpenAI();

const response = await client.chat.completions.create({
    messages: [{ role: 'user', content: 'Say this is a test' }],
    model: 'gpt-4o-mini'
});

console.log(response._request_id);
```

### Accessing Raw Response Objects

**Python example:**
```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.with_raw_response.create(
    messages=[{
        "role": "user",
        "content": "Say this is a test",
    }],
    model="gpt-4o-mini",
)
print(response.headers.get('x-ratelimit-limit-tokens'))

# get the object that `chat.completions.create()` would have returned
completion = response.parse()
print(completion)
```

**JavaScript example:**
```javascript
import OpenAI from 'openai';
const client = new OpenAI();

const response = await client.chat.completions.create({
    messages: [{ role: 'user', content: 'Say this is a test' }],
    model: 'gpt-4o-mini'
}).asResponse();

// access the underlying Response object
console.log(response.headers.get('x-ratelimit-limit-tokens'));
```
