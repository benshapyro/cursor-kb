# Fine-tuning

Manage fine-tuning jobs to tailor a model to your specific training data.

Related guide: Fine-tune models

## Create Fine-tuning Job

### POST https://api.openai.com/v1/fine_tuning/jobs

Creates a fine-tuning job to create a new model from a given dataset.

### Request Body

The request body accepts the following parameters:

#### Required Parameters

- **model** (string)
  - Name of the model to fine-tune
  - Must be a supported model
  
- **training_file** (string) 
  - ID of uploaded training data file
  - Must be in JSONL format with purpose=fine-tune

#### Optional Parameters

- **hyperparameters** (object)
  - Fine-tuning hyperparameters configuration

- **suffix** (string)
  - Custom suffix for model name
  - Maximum 64 characters

- **validation_file** (string)
  - ID of validation data file

- **integrations** (array)
  - List of enabled integrations

- **seed** (integer)
  - Seed for reproducibility

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.fine_tuning.jobs.create(
  training_file="file-abc123",
  model="gpt-4o-mini"
)
```

### Response

```json
{
  "object": "fine_tuning.job",
  "id": "ftjob-abc123",
  "model": "gpt-4o-mini-2024-07-18",
  "created_at": 1721764800,
  "fine_tuned_model": null,
  "organization_id": "org-123",
  "result_files": [],
  "status": "queued",
  "validation_file": null,
  "training_file": "file-abc123"
}
```

## List Fine-tuning Jobs

### GET https://api.openai.com/v1/fine_tuning/jobs

List your organization's fine-tuning jobs.

### Query Parameters
#### Required Parameters
None

#### Optional Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| after | string | No | ID of last job from previous page |
| limit | integer | No | Number of jobs to retrieve (default: 20) |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.fine_tuning.jobs.list()
```

## List Fine-tuning Events

### GET https://api.openai.com/v1/fine_tuning/jobs/{fine_tuning_job_id}/events

Get status updates for a fine-tuning job.

### Parameters
#### Required Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| fine_tuning_job_id | string | Yes | ID of the fine-tuning job |

#### Optional Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| after | string | No | ID of last event from previous page |
| limit | integer | No | Number of events to retrieve (default: 20) |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.fine_tuning.jobs.list_events(
  fine_tuning_job_id="ftjob-abc123",
  limit=2
)
```

## List Fine-tuning Checkpoints

### GET https://api.openai.com/v1/fine_tuning/jobs/{fine_tuning_job_id}/checkpoints

List checkpoints for a fine-tuning job.

### Parameters
#### Required Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| fine_tuning_job_id | string | Yes | ID of the fine-tuning job |

#### Optional Parameters  
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| after | string | No | ID of last checkpoint from previous page |
| limit | integer | No | Number of checkpoints to retrieve (default: 10) |

## Training Data Formats

### Chat Model Format

```json
{
  "messages": [
    { "role": "user", "content": "What is the weather in San Francisco?" },
    {
      "role": "assistant",
      "tool_calls": [
        {
          "id": "call_id",
          "type": "function",
          "function": {
            "name": "get_current_weather",
            "arguments": "{\"location\": \"San Francisco, USA\", \"format\": \"celsius\"}"
          }
        }
      ]
    }
  ],
  "parallel_tool_calls": false,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
                "type": "string",
                "description": "The city and country, eg. San Francisco, USA"
            },
            "format": { "type": "string", "enum": ["celsius", "fahrenheit"] }
          },
          "required": ["location", "format"]
        }
      }
    }
  ]
}
```

### Completions Model Format

```json
{
  "prompt": "What is the answer to 2+2",
  "completion": "4"
}
```

## Response Objects

### The Fine-tuning Job Object

Represents the status and details of a fine-tuning job.

#### Core Fields
- `id` (string) - Object identifier
- `created_at` (integer) - Creation timestamp
- `model` (string) - Base model being fine-tuned
- `organization_id` (string) - Organization owner

#### Status Fields
- `status` (string) - Job status (validating_files/queued/running/succeeded/failed/cancelled)
- `error` (object) - Failure information (if failed)
- `finished_at` (integer) - Completion timestamp (null if running)
- `estimated_finish` (integer) - Estimated completion timestamp

#### Training Configuration
- `hyperparameters` (object) - Training hyperparameters
- `training_file` (string) - Training file ID
- `validation_file` (string) - Validation file ID
- `seed` (integer) - Random seed used

#### Results
- `fine_tuned_model` (string) - Name of created model (null if running)
- `result_files` (array) - Result file IDs
- `trained_tokens` (integer) - Total billable tokens processed
- `integrations` (array) - Enabled integrations

### The Fine-tuning Job Event Object

Represents an event that occurred during fine-tuning.

#### Fields
- `id` (string) - Event identifier
- `created_at` (integer) - Event timestamp
- `level` (string) - Event level
- `message` (string) - Event message
- `object` (string) - Always "fine_tuning.job.event"

### The Fine-tuning Job Checkpoint Object

Represents a checkpoint saved during model training.

#### Core Fields
- `id` (string) - Checkpoint identifier
- `created_at` (integer) - Creation timestamp
- `fine_tuning_job_id` (string) - Parent job ID
- `object` (string) - Always "fine_tuning.job.checkpoint"

#### Training Progress
- `step_number` (integer) - Training step number
- `fine_tuned_model_checkpoint` (string) - Checkpoint model name
- `metrics` (object) - Training metrics at checkpoint
