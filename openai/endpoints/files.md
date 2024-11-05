# Files

Files are used to upload documents that can be used with features like Assistants, Fine-tuning, and Batch API.

## Upload File

### POST https://api.openai.com/v1/files

Upload a file that can be used across various endpoints. 

### Limits
- Individual files: up to 512 MB
- Organization total: up to 100 GB
- Assistants API: up to 2 million tokens
- Batch API: JSONL files up to 100 MB
- Contact support for increased limits

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file | file | Yes | The File object (not file name) to be uploaded |
| purpose | string | Yes | Intended purpose: "assistants" (Assistants/Message files), "vision" (Assistants image inputs), "batch" (Batch API), or "fine-tune" (Fine-tuning) |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.files.create(
  file=open("mydata.jsonl", "rb"),
  purpose="fine-tune"
)
```

### Response

```json
{
  "id": "file-abc123",
  "object": "file",
  "bytes": 120000,
  "created_at": 1677610602,
  "filename": "mydata.jsonl",
  "purpose": "fine-tune"
}
```

## List Files

### GET https://api.openai.com/v1/files

Returns a list of files.

### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| purpose | string | No | Filter by purpose |
| limit | integer | No | Number of objects (1-10,000, default: 10,000) |
| order | string | No | Sort by created_at: "asc" or "desc" (default) |
| after | string | No | Cursor for pagination |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.files.list()
```

### Response

```json
{
  "data": [
    {
      "id": "file-abc123",
      "object": "file",
      "bytes": 175,
      "created_at": 1613677385,
      "filename": "salesOverview.pdf",
      "purpose": "assistants"
    },
    {
      "id": "file-abc123",
      "object": "file",
      "bytes": 140,
      "created_at": 1613779121,
      "filename": "puppy.jsonl",
      "purpose": "fine-tune"
    }
  ],
  "object": "list"
}
```

## Retrieve File

### GET https://api.openai.com/v1/files/{file_id}

Returns information about a specific file.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file_id | string | Yes | ID of the file to retrieve |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.files.retrieve("file-abc123")
```

## Delete File

### DELETE https://api.openai.com/v1/files/{file_id}

Delete a file.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file_id | string | Yes | ID of the file to delete |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.files.delete("file-abc123")
```

### Response

```json
{
  "id": "file-abc123",
  "object": "file",
  "deleted": true
}
```

## Retrieve File Content

### GET https://api.openai.com/v1/files/{file_id}/content

Returns the contents of the specified file.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file_id | string | Yes | ID of the file to retrieve content from |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

content = client.files.content("file-abc123")
```

## The File Object

Represents a document uploaded to OpenAI.

### Fields
| Field | Type | Description |
|-------|------|-------------|
| id | string | File identifier |
| bytes | integer | File size in bytes |
| created_at | integer | Creation timestamp (Unix) |
| filename | string | Name of the file |
| object | string | Always "file" |
| purpose | string | Intended purpose ("assistants", "assistants_output", "batch", "batch_output", "fine-tune", "fine-tune-results", "vision") |

### Example

```json
{
  "id": "file-abc123",
  "object": "file",
  "bytes": 120000,
  "created_at": 1677610602,
  "filename": "salesOverview.pdf",
  "purpose": "assistants"
}
```

### Deprecated Fields
- `status`: File status (uploaded/processed/error)
- `status_details`: Fine-tuning validation failure details
