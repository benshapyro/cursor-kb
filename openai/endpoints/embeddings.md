# Embeddings

Get a vector representation of a given input that can be easily consumed by machine learning models and algorithms.

Related guide: Embeddings

## Create Embeddings

### POST https://api.openai.com/v1/embeddings

Creates an embedding vector representing the input text.

### Request Body

#### Required Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| input | string or array | Yes | Input text to embed. Can be a string or array of tokens. For multiple inputs, pass an array of strings or token arrays. Limits: max 8192 tokens for text-embedding-ada-002, non-empty string, arrays must be ≤2048 dimensions. |
| model | string | Yes | ID of the model to use. See List models API or Model overview for available models. |

#### Optional Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| encoding_format | string | No | Format for embeddings: "float" (default) or "base64" |
| dimensions | integer | No | Number of dimensions for output embeddings. Only for text-embedding-3 and later. |
| user | string | No | Unique end-user identifier for abuse monitoring. |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.embeddings.create(
  model="text-embedding-ada-002",
  input="The food was delicious and the waiter...",
  encoding_format="float"
)
```

### Response

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [
        0.0023064255,
        -0.009327292,
        .... (1536 floats total for ada-002)
        -0.0028842222,
      ],
      "index": 0
    }
  ],
  "model": "text-embedding-ada-002",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

## Response Objects

### The Embedding Object

Represents an embedding vector returned by the embedding endpoint.

#### Fields
| Field | Type | Description |
|-------|------|-------------|
| index | integer | Position of the embedding in the list of embeddings |
| embedding | array | The embedding vector (list of floats). Length varies by model |
| object | string | Always "embedding" |

Example:
```json
{
  "object": "embedding",
  "embedding": [
    0.0023064255,
    -0.009327292,
    .... (1536 floats total for ada-002)
    -0.0028842222,
  ],
  "index": 0
}
```

### Usage Information

The response includes usage statistics in the `usage` object:
- `prompt_tokens`: Number of tokens in the input
- `total_tokens`: Total tokens processed

### Notes

1. The length of the embedding vector depends on the model used. For example:
   - text-embedding-ada-002: 1536 dimensions
   - Newer models may have configurable dimensions

2. When processing multiple inputs in a single request:
   - Pass an array of strings or token arrays
   - Each embedding will have a unique index in the response
   - Usage information covers all inputs combined

3. Best practices:
   - Monitor token usage through the usage field
   - Consider batching multiple inputs in a single request
   - Be mindful of the model's token limits
   - Use appropriate encoding_format based on your application needs
