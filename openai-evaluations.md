# Evaluating Model Performance Beta

When developing with AI models, it's essential to continuously test their outputs to ensure they are accurate and useful. Regularly running evaluations (often called evals) on your model's outputs using test data helps you build and maintain high-quality and reliable AI applications.

OpenAI provides built-in tools in the OpenAI dashboard to create and run evals on test datasets. Here's how the process works:

1. **Generate a test dataset**
2. **Define and run evals against your dataset**
3. **Tweak your prompt and/or fine-tune your model to improve performance**
4. **Repeat until satisfied 🚀**

Let's see how this is done!

## Generate a Test Data Set

In software development, you often have to create test data (sometimes called fixtures) that your program needs in order to validate your software is working properly. Your unit tests would execute your code with fixture data and ensure the output is what you expect.

Similarly, your evals will require a set of test inputs that your model should be able to reply to properly. Having good test data is very important in optimizing LLM accuracy, because if your model is tested with data that's not representative of the types of requests it's going to get, you can't be confident in how it will perform on new, unknown inputs.

### Generate Datasets from Real Traffic

One of the best ways to generate representative test data sets is using real production requests from users. This is possible using Stored Completions. In your code that generates LLM responses, use the `store: true` parameter, and include metadata tags that you can later use to filter your completions, as in the examples below for an IT support chatbot:

#### Store Completions in the API with Metadata

```javascript
import OpenAI from "openai";
const openai = new OpenAI();

const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are a corporate IT support expert." },
    { role: "user", content: "How can I hide the dock on my Mac?" },
  ],
  store: true,
  metadata: {
    role: "manager",
    department: "accounting",
    source: "homepage"
  }
});

console.log(response.choices[0]);
```

This will make the completion show up in the dashboard here.

**Stored Completion**

Please note that Stored Completions contain unfiltered content from API prompts and completions. When you use content from Stored Completions for fine-tuning, you are responsible for making sure you have the appropriate permissions to use this content and that it does not include any personal information or other sensitive data.

From here, you can define an eval to judge the output of the model.

## Define and Run an Eval Against Your Test Data

Once you have created a test data set, either manually or by using the flow from the completions UI, you can define the parameters of your eval run. If you followed the step above and generated test data from production traffic, you won't need to run completions again. You can go right into defining the criteria for your eval.

### Create Criteria

There are a number of evaluation criteria to choose from (sometimes called graders) - these tests will help assess the quality of your model responses. One flexible option is a model grader, which you can prompt to grade model outputs as you see fit.

### Add Criteria

Once you have defined the criteria for your model, you can run your eval!

### Iterate and Improve

After your eval runs, you will see resulting scores in the dashboard. By iterating on your prompts and criteria, you will be able to improve your model outputs over time. Having good evals and good test data in place can help you iterate on prompts and try new models with more confidence that your generation results are in good standing.

## Fine-tuning

Improve a model's ability to generate responses tailored to your use case.

## Model Distillation

Learn how to distill large model results to smaller, cheaper, and faster models.
```