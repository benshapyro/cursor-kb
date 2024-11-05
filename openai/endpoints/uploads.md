# Uploads

Allows you to upload large files in multiple parts.

## Create Upload

### POST https://api.openai.com/v1/uploads

Creates an intermediate Upload object that you can add Parts to. 

### Limits
- Maximum total size: 8 GB
- Expires: 1 hour after creation
- Part size: Maximum 64 MB per part

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| filename | string | Yes | Name of the file to upload |
| purpose | string | Yes | Intended purpose of the file (see File purposes documentation) |
| bytes | integer | Yes | Total number of bytes in the file |
| mime_type | string | Yes | MIME type of the file (must be supported for the purpose) |

### Example Request

```bash
curl https://api.openai.com/v1/uploads \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "purpose": "fine-tune",
    "filename": "training_examples.jsonl",
    "bytes": 2147483648,
    "mime_type": "text/jsonl"
  }'
```

### Response

```json
{
  "id": "upload_abc123",
  "object": "upload",
  "bytes": 2147483648,
  "created_at": 1719184911,
  "filename": "training_examples.jsonl",
  "purpose": "fine-tune",
  "status": "pending",
  "expires_at": 1719127296
}
```

## Add Upload Part

### POST https://api.openai.com/v1/uploads/{upload_id}/parts

Adds a Part to an Upload object. Parts can be added in parallel.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| upload_id | string | Yes | ID of the Upload |
| data | file | Yes | Chunk of bytes for this Part |

### Example Request

```bash
curl https://api.openai.com/v1/uploads/upload_abc123/parts \
  -F data="aHR0cHM6Ly9hcGkub3BlbmFpLmNvbS92MS91cGxvYWRz..."
```

### Response

```json
{
  "id": "part_def456",
  "object": "upload.part",
  "created_at": 1719185911,
  "upload_id": "upload_abc123"
}
```

## Complete Upload

### POST https://api.openai.com/v1/uploads/{upload_id}/complete

Completes the Upload and creates a usable File object.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| upload_id | string | Yes | ID of the Upload |
| part_ids | array | Yes | Ordered list of Part IDs |
| md5 | string | No | MD5 checksum for verification |

### Example Request

```bash
curl https://api.openai.com/v1/uploads/upload_abc123/complete \
  -d '{
    "part_ids": ["part_def456", "part_ghi789"]
  }'
```

### Response

```json
{
  "id": "upload_abc123",
  "object": "upload",
  "bytes": 2147483648,
  "created_at": 1719184911,
  "filename": "training_examples.jsonl",
  "purpose": "fine-tune",
  "status": "completed",
  "expires_at": 1719127296,
  "file": {
    "id": "file-xyz321",
    "object": "file",
    "bytes": 2147483648,
    "created_at": 1719186911,
    "filename": "training_examples.jsonl",
    "purpose": "fine-tune"
  }
}
```

## Cancel Upload

### POST https://api.openai.com/v1/uploads/{upload_id}/cancel

Cancels the Upload. No Parts may be added after cancellation.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| upload_id | string | Yes | ID of the Upload to cancel |

### Example Request

```bash
curl https://api.openai.com/v1/uploads/upload_abc123/cancel
```

## Response Objects

### The Upload Object

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique identifier |
| created_at | integer | Creation timestamp |
| filename | string | Name of the file |
| bytes | integer | Intended total bytes |
| purpose | string | Intended purpose |
| status | string | Upload status |
| expires_at | integer | Expiration timestamp |
| object | string | Always "upload" |
| file | object | Created File object (when completed) |

### The Upload Part Object

| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique identifier |
| created_at | integer | Creation timestamp |
| upload_id | string | Parent Upload ID |
| object | string | Always "upload.part" |

### Example Objects

```json
// Upload Object
{
  "id": "upload_abc123",
  "object": "upload",
  "bytes": 2147483648,
  "created_at": 1719184911,
  "filename": "training_examples.jsonl",
  "purpose": "fine-tune",
  "status": "completed",
  "expires_at": 1719127296,
  "file": {
    "id": "file-xyz321",
    "object": "file",
    "bytes": 2147483648,
    "created_at": 1719186911,
    "filename": "training_examples.jsonl",
    "purpose": "fine-tune"
  }
}

// Upload Part Object
{
    "id": "part_def456",
    "object": "upload.part",
    "created_at": 1719186911,
    "upload_id": "upload_abc123"
}
