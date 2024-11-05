# Batch

Create large batches of API requests for asynchronous processing. The Batch API returns completions within 24 hours for a 50% discount.

Related guide: Batch

## Create Batch

### POST https://api.openai.com/v1/batches

Creates and executes a batch from an uploaded file of requests.

### Request Body

#### Required Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| input_file_id | string | Yes | ID of uploaded JSONL file containing requests (max 50,000 requests, 100 MB) |
| endpoint | string | Yes | Endpoint for batch requests (/v1/chat/completions, /v1/embeddings, or /v1/completions) |
| completion_window | string | Yes | Processing timeframe (currently only "24h" supported) |

#### Optional Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| metadata | object | No | Custom metadata for the batch |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.batches.create(
  input_file_id="file-abc123",
  endpoint="/v1/chat/completions",
  completion_window="24h"
)
```

### Response

```json
{
  "id": "batch_abc123",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "errors": null,
  "input_file_id": "file-abc123",
  "completion_window": "24h",
  "status": "validating",
  "output_file_id": null,
  "error_file_id": null,
  "created_at": 1711471533,
  "request_counts": {
    "total": 0,
    "completed": 0,
    "failed": 0
  },
  "metadata": {
    "customer_id": "user_123456789",
    "batch_description": "Nightly eval job"
  }
}
```

## Retrieve Batch

### GET https://api.openai.com/v1/batches/{batch_id}

Retrieves a batch by ID.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| batch_id | string | Yes | ID of the batch to retrieve |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.batches.retrieve("batch_abc123")
```

## Cancel Batch

### POST https://api.openai.com/v1/batches/{batch_id}/cancel

Cancels an in-progress batch. Status changes to "cancelling" (up to 10 minutes), then "cancelled" with partial results available.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| batch_id | string | Yes | ID of the batch to cancel |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.batches.cancel("batch_abc123")
```

## List Batches

### GET https://api.openai.com/v1/batches

List your organization's batches.

### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| after | string | No | Cursor for pagination (object ID) |
| limit | integer | No | Number of objects to return (1-100, default: 20) |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.batches.list()
```

## Data Formats

### The Request Input Object

Each line in the batch input file must be a JSON object with:

```json
{
  "custom_id": "request-1",
  "method": "POST",
  "url": "/v1/chat/completions",
  "body": {
    "model": "gpt-4o-mini",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is 2+2?"}
    ]
  }
}
```

### The Request Output Object

Each line in the output and error files contains:

```json
{
  "id": "batch_req_wnaDys",
  "custom_id": "request-2",
  "response": {
    "status_code": 200,
    "request_id": "req_c187b3",
    "body": {
      "id": "chatcmpl-9758Iw",
      "object": "chat.completion",
      "created": 1711475054,
      "model": "gpt-4o-mini",
      "choices": [
        {
          "index": 0,
          "message": {
            "role": "assistant",
            "content": "2 + 2 equals 4."
          },
          "finish_reason": "stop"
        }
      ],
      "usage": {
        "prompt_tokens": 24,
        "completion_tokens": 15,
        "total_tokens": 39
      },
      "system_fingerprint": null
    }
  },
  "error": null
}
```

## The Batch Object

### Fields
| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique batch identifier |
| object | string | Always "batch" |
| endpoint | string | API endpoint used |
| errors | object | Error information |
| input_file_id | string | Input file ID |
| completion_window | string | Processing timeframe |
| status | string | Current batch status |
| output_file_id | string | Successful outputs file ID |
| error_file_id | string | Failed requests file ID |

### Timestamps
| Field | Type | Description |
|-------|------|-------------|
| created_at | integer | Creation timestamp |
| in_progress_at | integer | Processing start timestamp |
| expires_at | integer | Expiration timestamp |
| finalizing_at | integer | Finalization start timestamp |
| completed_at | integer | Completion timestamp |
| failed_at | integer | Failure timestamp |
| expired_at | integer | Expiration timestamp |
| cancelling_at | integer | Cancellation start timestamp |
| cancelled_at | integer | Cancellation completion timestamp |

### Statistics
- `request_counts`: Object containing:
  - `total`: Total number of requests
  - `completed`: Number of completed requests
  - `failed`: Number of failed requests

### Metadata
- Custom key-value pairs (max 16 pairs)
- Keys: max 64 characters
- Values: max 512 characters
