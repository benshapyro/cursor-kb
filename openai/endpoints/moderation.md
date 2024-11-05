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
