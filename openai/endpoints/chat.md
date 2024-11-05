# Chat

Given a list of messages comprising a conversation, the model will return a response.

Related guide: Chat Completions

## Create Chat Completion

### POST https://api.openai.com/v1/chat/completions

Creates a model response for the given chat conversation. Learn more in the text generation, vision, and audio guides.

### Request Body

### Required Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| messages | array | Yes | A list of messages comprising the conversation. Supports different message types (text, images, audio) depending on the model. |
| model | string | Yes | ID of the model to use. See model endpoint compatibility table for details. |

### Optional Parameters

#### Basic Configuration
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| temperature | number | No | Sampling temperature between 0 and 2. Higher = more random. Default: 1 |
| top_p | number | No | Nucleus sampling probability mass (0 to 1). Default: 1 |
| n | integer | No | Number of chat completion choices to generate. Default: 1 |
| stream | boolean | No | Enable server-sent events for partial message deltas. Default: false |
| stop | string/array | No | Up to 4 sequences where API will stop generating tokens. Default: null |
| max_completion_tokens | integer | No | Upper bound for generated tokens, including visible and reasoning tokens. |

#### Output Control
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| response_format | object | No | Specify output format. Supports JSON schema and JSON object formats. |
| modalities | array | No | Requested output types (e.g., ["text"], ["text", "audio"]). Default: ["text"] |
| logprobs | boolean | No | Whether to return log probabilities of output tokens. Default: false |
| top_logprobs | integer | No | Number of most likely tokens (0-20) to return per position. Requires logprobs=true. |

#### Behavior Tuning
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| frequency_penalty | number | No | Number between -2.0 and 2.0. Positive values decrease repetition. Default: 0 |
| presence_penalty | number | No | Number between -2.0 and 2.0. Positive values encourage new topics. Default: 0 |
| logit_bias | map | No | Modify likelihood of specified tokens. Maps token IDs to bias values (-100 to 100). |

#### Tool Integration
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| tools | array | No | List of tools (currently only functions) the model may call. Max 128 functions. |
| tool_choice | string/object | No | Controls tool usage. Options: "none", "auto", "required", or specific tool. |
| parallel_tool_calls | boolean | No | Enable parallel function calling during tool use. Default: true |

#### System Configuration
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| seed | integer | No | For deterministic sampling (beta). Check system_fingerprint for backend changes. |
| service_tier | string | No | Latency tier for processing ("auto" or "default"). Default: "auto" |
| store | boolean | No | Whether to store output for model distillation/evals. Default: false |
| metadata | object | No | Developer-defined tags for filtering completions in dashboard. |
| user | string | No | Unique end-user identifier for abuse monitoring. |

### Example Request

```Request (python)
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
  model="gpt-4o",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
  ]
)

print(completion.choices[0].message)
```

### Response

```Response (json)
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-4o-mini",
  "system_fingerprint": "fp_44709d6fcb",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "\n\nHello there, how may I assist you today?",
    },
    "logprobs": null,
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21,
    "completion_tokens_details": {
      "reasoning_tokens": 0
    }
  }
}
```

## Response Objects

### The Chat Completion Object

The chat completion object contains the following fields:

#### Core Fields
- `id` (string) - Unique identifier for the chat completion
- `object` (string) - Always "chat.completion"
- `created` (integer) - Unix timestamp of creation
- `model` (string) - Model used for completion

#### Content and Choices
- `choices` (array) - List of chat completion choices

#### Configuration
- `system_fingerprint` (string) - Backend configuration fingerprint
- `service_tier` (string) - Service tier used (if specified in request)

#### Usage Statistics  
- `usage` (object) - Usage statistics for the request

Example:
```Chat Completion Object (json)
{
  "id": "chatcmpl-123456",
  "object": "chat.completion",
  "created": 1728933352,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hi there! How can I assist you today?",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 19,
    "completion_tokens": 10,
    "total_tokens": 29,
    "prompt_tokens_details": {
      "cached_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0
    }
  },
  "system_fingerprint": "fp_6b68a8204b"
}
```

### The Chat Completion Chunk Object

Represents a streamed chunk of a chat completion response.

#### Core Fields
- `id` (string) - Unique identifier (same for all chunks)
- `object` (string) - Always "chat.completion.chunk"
- `created` (integer) - Unix timestamp (same for all chunks)
- `model` (string) - Model used for generation

#### Content
- `choices` (array) - List of completion choices

#### Configuration 
- `system_fingerprint` (string) - Backend configuration fingerprint
- `service_tier` (string) - Service tier used (if specified)

#### Usage Statistics
- `usage` (object) - Token usage statistics (only in final chunk with stream_options)

```Chat Completion Chunk (json)
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "system_fingerprint": "fp_44709d6fcb", "choices":[{"index":0,"delta":{"role":"assistant","content":""},"logprobs":null,"finish_reason":null}]}

{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "system_fingerprint": "fp_44709d6fcb", "choices":[{"index":0,"delta":{"content":"Hello"},"logprobs":null,"finish_reason":null}]}

....

{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "system_fingerprint": "fp_44709d6fcb", "choices":[{"index":0,"delta":{},"logprobs":null,"finish_reason":"stop"}]}
```