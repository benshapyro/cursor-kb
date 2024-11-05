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
