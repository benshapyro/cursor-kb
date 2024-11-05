# OpenAI API Endpoints

- [Audio](#audio)
- [Chat](#chat) 
- [Embeddings](#embeddings)
- [Fine-tuning](#fine-tuning)
- [Batch](#batch)
- [Files](#files)
- [Uploads](#uploads)
- [Images](#images)
- [Models](#models)
- [Moderations](#moderations)

# Audio

Learn how to turn audio into text or text into audio.

Related guide: Speech to text

## Create Speech

### POST https://api.openai.com/v1/audio/speech

Generates audio from the input text.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| model | string | Yes | One of the available TTS models: `tts-1` or `tts-1-hd` |
| input | string | Yes | The text to generate audio for. Maximum length is 4096 characters. |
| voice | string | Yes | The voice to use. Supported voices: `alloy`, `echo`, `fable`, `onyx`, `nova`, and `shimmer`. |
| response_format | string | No | The audio format. Supported formats: `mp3` (default), `opus`, `aac`, `flac`, `wav`, and `pcm`. |
| speed | number | No | The speed of generated audio. Range: 0.25 to 4.0. Default: 1.0 |

#### Example Request

```python
from pathlib import Path
import openai

speech_file_path = Path(__file__).parent / "speech.mp3"
response = openai.audio.speech.create(
  model="tts-1",
  voice="alloy",
  input="The quick brown fox jumped over the lazy dog."
)
response.stream_to_file(speech_file_path)
```

## Create Transcription

### POST https://api.openai.com/v1/audio/transcriptions

Transcribes audio into the input language.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file | file | Yes | Audio file in supported formats: flac, mp3, mp4, mpeg, mpga, m4a, ogg, wav, or webm |
| model | string | Yes | Model ID. Currently only `whisper-1` is available |
| language | string | No | Input audio language in ISO-639-1 format |
| prompt | string | No | Optional text to guide model's style or continue previous segment |
| response_format | string | No | Output format: `json` (default), `text`, `srt`, `verbose_json`, or `vtt` |
| temperature | number | No | Sampling temperature between 0 and 1. Default: 0 |
| timestamp_granularities[] | array | No | Timestamp detail level: `word` or `segment`. Requires `verbose_json` format |

#### Example Request

```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.transcriptions.create(
  model="whisper-1",
  file=audio_file
)
```

#### Response

```json
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger. This is a place where you can get to do that."
}
```

## Create Translation

### POST https://api.openai.com/v1/audio/translations

Translates audio into English.

#### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| file | file | Yes | Audio file in supported formats: flac, mp3, mp4, mpeg, mpga, m4a, ogg, wav, or webm |
| model | string | Yes | Model ID. Currently only `whisper-1` is available |
| prompt | string | No | Optional text to guide model's style (in English) |
| response_format | string | No | Output format: `json` (default), `text`, `srt`, `verbose_json`, or `vtt` |
| temperature | number | No | Sampling temperature between 0 and 1. Default: 0 |

#### Example Request

```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.translations.create(
  model="whisper-1",
  file=audio_file
)
```

#### Response

```json
{
  "text": "Hello, my name is Wolfgang and I come from Germany. Where are you heading today?"
}
```

## Response Objects

### The Transcription Object (JSON)

Basic JSON response format:

```json
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger. This is a place where you can get to do that."
}
```

### The Transcription Object (Verbose JSON)

Detailed response format with additional metadata:

```json
{
  "task": "transcribe",
  "language": "english",
  "duration": 8.470000267028809,
  "text": "The beach was a popular spot on a hot summer day. People were swimming in the ocean, building sandcastles, and playing beach volleyball.",
  "segments": [
    {
      "id": 0,
      "seek": 0,
      "start": 0.0,
      "end": 3.319999933242798,
      "text": " The beach was a popular spot on a hot summer day.",
      "tokens": [
        50364, 440, 7534, 390, 257, 3743, 4008, 322, 257, 2368, 4266, 786, 13, 50530
      ],
      "temperature": 0.0,
      "avg_logprob": -0.2860786020755768,
      "compression_ratio": 1.2363636493682861,
      "no_speech_prob": 0.00985979475080967
    }
  ]
}
```

The verbose JSON format includes:
- `language`: Input audio language
- `duration`: Audio duration in seconds
- `text`: Transcribed text
- `words`: (When requested) Word-level timestamps
- `segments`: Detailed segment information including timing and confidence metrics


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
```

# Images

Given a prompt and/or an input image, the model will generate a new image.

Related guide: Image generation

## Create Image

### POST https://api.openai.com/v1/images/generations

Creates an image given a prompt.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| prompt | string | Yes | A text description of the desired image(s). Max length: 1000 characters for dall-e-2, 4000 characters for dall-e-3. |
| model | string | No | Model to use for image generation. Default: dall-e-2 |
| n | integer | No | Number of images to generate (1-10). For dall-e-3, only n=1 is supported. Default: 1 |
| quality | string | No | Image quality. "hd" for finer details (dall-e-3 only). Default: standard |
| response_format | string | No | Format of generated images: "url" (default) or "b64_json". URLs valid for 60 minutes. |
| size | string | No | Size of generated images. Options: 256x256, 512x512, 1024x1024 for dall-e-2; 1024x1024, 1792x1024, 1024x1792 for dall-e-3. Default: 1024x1024 |
| style | string | No | Image style: "vivid" or "natural" (dall-e-3 only). Default: vivid |
| user | string | No | Unique identifier for end-user, for abuse monitoring. |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.images.generate(
  model="dall-e-3",
  prompt="A cute baby sea otter",
  n=1,
  size="1024x1024"
)
```

### Response

```json
{
  "created": 1589478378,
  "data": [
    {
      "url": "https://..."
    }
  ]
}
```

## Create Image Edit

### POST https://api.openai.com/v1/images/edits

Creates an edited or extended image given an original image and a prompt.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| image | file | Yes | Image to edit. Must be a valid PNG, <4MB, and square. If no mask, image must have transparency. |
| prompt | string | Yes | Text description of desired image(s). Max length: 1000 characters. |
| mask | file | No | Additional image indicating edit areas. Must be a valid PNG, <4MB, and same dimensions as image. |
| model | string | No | Model to use for image generation. Default: dall-e-2 |
| n | integer | No | Number of images to generate (1-10). Default: 1 |
| size | string | No | Size of generated images. Options: 256x256, 512x512, 1024x1024. Default: 1024x1024 |
| response_format | string | No | Format of generated images: "url" (default) or "b64_json". URLs valid for 60 minutes. |
| user | string | No | Unique identifier for end-user, for abuse monitoring. |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.images.edit(
  image=open("otter.png", "rb"),
  mask=open("mask.png", "rb"),
  prompt="A cute baby sea otter wearing a beret",
  n=2,
  size="1024x1024"
)
```

### Response

```json
{
  "created": 1589478378,
  "data": [
    {
      "url": "https://..."
    },
    {
      "url": "https://..."
    }
  ]
}
```

## Create Image Variation

### POST https://api.openai.com/v1/images/variations

Creates a variation of a given image.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| image | file | Yes | Image for variation. Must be a valid PNG, <4MB, and square. |
| model | string | No | Model to use for image generation. Default: dall-e-2 |
| n | integer | No | Number of images to generate (1-10). For dall-e-3, only n=1 is supported. Default: 1 |
| response_format | string | No | Format of generated images: "url" (default) or "b64_json". URLs valid for 60 minutes. |
| size | string | No | Size of generated images. Options: 256x256, 512x512, 1024x1024. Default: 1024x1024 |
| user | string | No | Unique identifier for end-user, for abuse monitoring. |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

response = client.images.create_variation(
  image=open("image_edit_original.png", "rb"),
  n=2,
  size="1024x1024"
)
```

### Response

```json
{
  "created": 1589478378,
  "data": [
    {
      "url": "https://..."
    },
    {
      "url": "https://..."
    }
  ]
}
```

## The Image Object

Represents the URL or the content of an image generated by the OpenAI API.

### Fields
| Field | Type | Description |
|-------|------|-------------|
| b64_json | string | Base64-encoded JSON of the generated image, if response_format is b64_json. |
| url | string | URL of the generated image, if response_format is url (default). |
| revised_prompt | string | The prompt used to generate the image, if revised. |

### Example

```json
{
  "url": "...",
  "revised_prompt": "..."
}
```


# Models

List and describe the various models available in the API. You can refer to the Models documentation to understand what models are available and the differences between them.

## List Models

### GET https://api.openai.com/v1/models

Lists the currently available models, and provides basic information about each one such as the owner and availability.

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.models.list()
```

### Response

```json
{
  "object": "list",
  "data": [
    {
      "id": "model-id-0",
      "object": "model",
      "created": 1686935002,
      "owned_by": "organization-owner"
    },
    {
      "id": "model-id-1",
      "object": "model",
      "created": 1686935002,
      "owned_by": "organization-owner"
    },
    {
      "id": "model-id-2",
      "object": "model",
      "created": 1686935002,
      "owned_by": "openai"
    }
  ],
  "object": "list"
}
```

## Retrieve Model

### GET https://api.openai.com/v1/models/{model}

Retrieves a model instance, providing basic information about the model such as the owner and permissioning.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| model | string | Yes | The ID of the model to retrieve |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.models.retrieve("gpt-3.5-turbo-instruct")
```

### Response

```json
{
  "id": "gpt-3.5-turbo-instruct",
  "object": "model",
  "created": 1686935002,
  "owned_by": "openai"
}
```

## Delete a Fine-tuned Model

### DELETE https://api.openai.com/v1/models/{model}

Delete a fine-tuned model. You must have the Owner role in your organization to delete a model.

### Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| model | string | Yes | The model to delete |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

client.models.delete("ft:gpt-4o-mini:acemeco:suffix:abc123")
```

### Response

```json
{
  "id": "ft:gpt-4o-mini:acemeco:suffix:abc123",
  "object": "model",
  "deleted": true
}
```

## The Model Object

Describes an OpenAI model offering that can be used with the API.

### Fields
| Field | Type | Description |
|-------|------|-------------|
| id | string | The model identifier for API endpoints |
| created | integer | Unix timestamp (seconds) of model creation |
| object | string | Always "model" |
| owned_by | string | Organization that owns the model |

### Example

```json
{
  "id": "davinci",
  "object": "model",
  "created": 1686935002,
  "owned_by": "openai"
}
```

### Notes

1. Model availability may vary based on:
   - Your organization's access level
   - Model deprecation status
   - Fine-tuning status (for custom models)

2. Best practices:
   - Cache model lists when possible
   - Handle model unavailability gracefully
   - Check model compatibility with intended endpoints
   - Consider model capabilities and limitations for your use case


# Moderations

Given text and/or image inputs, classifies if those inputs are potentially harmful across several categories.

Related guide: Moderations

## Create Moderation

### POST https://api.openai.com/v1/moderations

Classifies if text and/or image inputs are potentially harmful. Learn more in the moderation guide.

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| input | string or array | Yes | Input to classify. Can be a single string, array of strings, or array of multi-modal input objects |
| model | string | No | Content moderation model to use. Default: omni-moderation-latest |

### Example Request

```python
from openai import OpenAI
client = OpenAI()

moderation = client.moderations.create(input="I want to kill them.")
print(moderation)
```

### Response

```json
{
  "id": "modr-AB8CjOTu2jiq12hp1AQPfeqFWaORR",
  "model": "text-moderation-007",
  "results": [
    {
      "flagged": true,
      "categories": {
        "sexual": false,
        "hate": false,
        "harassment": true,
        "self-harm": false,
        "sexual/minors": false,
        "hate/threatening": false,
        "violence/graphic": false,
        "self-harm/intent": false,
        "self-harm/instructions": false,
        "harassment/threatening": true,
        "violence": true
      },
      "category_scores": {
        "sexual": 0.000011726012417057063,
        "hate": 0.22706663608551025,
        "harassment": 0.5215635299682617,
        "self-harm": 2.227119921371923e-6,
        "sexual/minors": 7.107352217872176e-8,
        "hate/threatening": 0.023547329008579254,
        "violence/graphic": 0.00003391829886822961,
        "self-harm/intent": 1.646940972932498e-6,
        "self-harm/instructions": 1.1198755256458526e-9,
        "harassment/threatening": 0.5694745779037476,
        "violence": 0.9971134662628174
      }
    }
  ]
}
```

## The Moderation Object

Represents if a given text input is potentially harmful.

### Fields
| Field | Type | Description |
|-------|------|-------------|
| id | string | Unique identifier for the moderation request |
| model | string | Model used to generate moderation results |
| results | array | List of moderation objects |

### Results Object Fields
| Field | Type | Description |
|-------|------|-------------|
| flagged | boolean | Whether the content violates OpenAI's usage policies |
| categories | object | Dict of category -> boolean indicating policy violations |
| category_scores | object | Dict of category -> float indicating model confidence |
| category_applied_input_types | object | Dict of category -> array of input types analyzed |

### Example

```json
{
  "id": "modr-0d9740456c391e43c445bf0f010940c7",
  "model": "omni-moderation-latest",
  "results": [
    {
      "flagged": true,
      "categories": {
        "harassment": true,
        "harassment/threatening": true,
        "sexual": false,
        "hate": false,
        "hate/threatening": false,
        "illicit": false,
        "illicit/violent": false,
        "self-harm/intent": false,
        "self-harm/instructions": false,
        "self-harm": false,
        "sexual/minors": false,
        "violence": true,
        "violence/graphic": true
      },
      "category_scores": {
        "harassment": 0.8189693396524255,
        "harassment/threatening": 0.804985420696006,
        "sexual": 1.573112165348997e-6,
        "hate": 0.007562942636942845,
        "hate/threatening": 0.004208854591835476,
        "illicit": 0.030535955153511665,
        "illicit/violent": 0.008925306722380033,
        "self-harm/intent": 0.00023023930975076432,
        "self-harm/instructions": 0.0002293869201073356,
        "self-harm": 0.012598046106750154,
        "sexual/minors": 2.212566909570261e-8,
        "violence": 0.9999992735124786,
        "violence/graphic": 0.843064871157054
      },
      "category_applied_input_types": {
        "harassment": ["text"],
        "harassment/threatening": ["text"],
        "sexual": ["text", "image"],
        "hate": ["text"],
        "hate/threatening": ["text"],
        "illicit": ["text"],
        "illicit/violent": ["text"],
        "self-harm/intent": ["text", "image"],
        "self-harm/instructions": ["text", "image"],
        "self-harm": ["text", "image"],
        "sexual/minors": ["text"],
        "violence": ["text", "image"],
        "violence/graphic": ["text", "image"]
      }
    }
  ]
}
```

### Notes

1. Category Scores:
   - Range from 0 to 1
   - Higher scores indicate higher confidence
   - Typically flag content when score exceeds model-specific thresholds

2. Input Types:
   - Text inputs are always analyzed
   - Image inputs are analyzed for applicable categories
   - Multi-modal inputs combine both text and image analysis

3. Best Practices:
   - Use appropriate thresholds for your use case
   - Consider both category flags and scores
   - Handle flagged content according to your policies
   - Monitor moderation results for quality assurance
