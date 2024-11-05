# Text Generation

Learn how to generate text from a prompt using OpenAI's APIs.

## Overview

OpenAI provides simple APIs to use large language models for text generation, similar to ChatGPT. These models have been trained on vast quantities of data to understand multimedia inputs and natural language instructions. From these prompts, models can generate various types of text responses, including:
- Code
- Mathematical equations
- Structured JSON data
- Human-like prose

## Quickstart

To generate text, use the chat completions endpoint in the REST API or OpenAI's official SDKs:

```python
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {
            "role": "user",
            "content": "Write a haiku about recursion in programming."
        }
    ]
)

print(completion.choices[0].message)
```

## Choosing a Model

When making a text generation request, select models based on your needs:

- **gpt-4o**: High intelligence and strong performance, higher cost per token
- **gpt-4o-mini**: Faster and less expensive, with slightly lower capabilities
- **o1 family**: Advanced reasoning and coding capabilities, slower but more thorough for complex tasks

Experiment with different models in the Playground to find the best fit for your prompts. [More information on choosing a model](#).

## Building Prompts

The process of crafting prompts, known as prompt engineering, involves giving the model precise instructions, examples, and context information to improve output quality and accuracy.

### Message Types

#### User Messages
User messages contain instructions for desired output, similar to ChatGPT inputs:

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Write a haiku about programming."
        }
      ]
    }
  ]
});
```

#### System Messages
System messages provide top-level instructions for model behavior:

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": `
            You are a helpful assistant that answers programming questions 
            in the style of a southern belle from the southeast United States.
          `
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Are semicolons optional in JavaScript?"
        }
      ]
    }
  ]
});
```

Example response:
```
Well, sugar, that's a fine question you've got there! Now, in the world of 
JavaScript, semicolons are indeed a bit like the pearls on a necklace – you 
might slip by without 'em, but you sure do look more polished with 'em in place. 

Technically, JavaScript has this little thing called "automatic semicolon 
insertion" where it kindly adds semicolons for you where it thinks they 
oughta go. However, it's not always perfect, bless its heart. Sometimes, it 
might get a tad confused and cause all sorts of unexpected behavior.
```

#### Assistant Messages
Assistant messages represent previous model-generated responses and can be used for few-shot learning:

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "user",
      "content": [{ "type": "text", "text": "knock knock." }]
    },
    {
      "role": "assistant",
      "content": [{ "type": "text", "text": "Who's there?" }]
    },
    {
      "role": "user",
      "content": [{ "type": "text", "text": "Orange." }]
    }
  ]
});
```

### Context Management

- Consider token limits and context windows
- Monitor both input and output tokens
- Use the tokenizer tool (tiktoken) to count tokens

## Additional Data and RAG

The message types can be used to provide additional information to the model outside its training data. This might include:
- Database query results
- Text documents
- Other resources

This technique, known as retrieval augmented generation (RAG), helps the model generate more relevant responses. [Learn more about RAG techniques here](#).

## Conversations and Context

While text generation requests are independent and stateless (except when using assistants), you can implement multi-turn conversations by providing additional messages as parameters. Here's an example:

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "user",
      "content": [{ "type": "text", "text": "knock knock." }]
    },
    {
      "role": "assistant",
      "content": [{ "type": "text", "text": "Who's there?" }]
    },
    {
      "role": "user",
      "content": [{ "type": "text", "text": "Orange." }]
    }
  ]
});
```

## Managing Context

As conversations grow more complex, consider:

### Token Limits
- **Output Tokens**: Model-specific limits for generated responses
  - Example: gpt-4o-2024-08-06 can generate up to 16,384 output tokens

### Context Windows
- Total tokens for input + output (and reasoning tokens for some models)
  - Example: gpt-4o-2024-08-06 has a 128k token context window
- Large prompts may exceed context windows, resulting in truncated outputs

### Token Counting
- Use the tokenizer tool (tiktoken library) to count tokens in text

## Optimization Strategies

### Accuracy
- Ensure complete information in prompts
- Implement proper prompt engineering
- Consider using:
  - RAG techniques
  - Model fine-tuning
  - Input/output formatting guidelines

### Cost Management
- Reduce token usage
- Select appropriate model sizes
- Balance capabilities vs. cost
- [Learn more about cost optimization](#)

### Latency Optimization
- Implement efficient prompt engineering
- Use parallel processing where possible
- Select faster models when appropriate
- Consider multi-threading and async operations
- [Learn more about latency optimization](#)


