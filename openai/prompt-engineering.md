# Prompt Engineering

Enhance results with prompt engineering strategies.

This guide shares strategies and tactics for getting better results from large language models (sometimes referred to as GPT models) like GPT-4. The methods described here can sometimes be deployed in combination for greater effect. We encourage experimentation to find the methods that work best for you.

You can also explore example prompts which showcase what our models are capable of:

- [Prompt examples](link-to-examples)
  Explore prompt examples to learn what GPT models can do

## Six Strategies for Getting Better Results

### 1. Write Clear Instructions

These models can't read your mind. If outputs are too long, ask for brief replies. If outputs are too simple, ask for expert-level writing. If you dislike the format, demonstrate the format you'd like to see. The less the model has to guess at what you want, the more likely you'll get it.

**Tactics:**
- Include details in your query to get more relevant answers
- Ask the model to adopt a persona
- Use delimiters to clearly indicate distinct parts of the input
- Specify the steps required to complete a task
- Provide examples
- Specify the desired length of the output

### 2. Provide Reference Text

Language models can confidently invent fake answers, especially when asked about esoteric topics or for citations and URLs. In the same way that a sheet of notes can help a student do better on a test, providing reference text to these models can help in answering with fewer fabrications.

**Tactics:**
- Instruct the model to answer using a reference text
- Instruct the model to answer with citations from a reference text

### 3. Split Complex Tasks into Simpler Subtasks

Just as it is good practice in software engineering to decompose a complex system into a set of modular components, the same is true of tasks submitted to a language model. Complex tasks tend to have higher error rates than simpler tasks. Furthermore, complex tasks can often be re-defined as a workflow of simpler tasks in which the outputs of earlier tasks are used to construct the inputs to later tasks.

**Tactics:**
- Use intent classification to identify the most relevant instructions for a user query
- For dialogue applications that require very long conversations, summarize or filter previous dialogue
- Summarize long documents piecewise and construct a full summary recursively

### 4. Give the Model Time to "Think"

If asked to multiply 17 by 28, you might not know it instantly, but can still work it out with time. Similarly, models make more reasoning errors when trying to answer right away, rather than taking time to work out an answer. Asking for a "chain of thought" before an answer can help the model reason its way toward correct answers more reliably.

**Tactics:**
- Instruct the model to work out its own solution before rushing to a conclusion
- Use inner monologue or a sequence of queries to hide the model's reasoning process
- Ask the model if it missed anything on previous passes

### 5. Use External Tools

Compensate for the weaknesses of the model by feeding it the outputs of other tools. For example, a text retrieval system (sometimes called RAG or retrieval augmented generation) can tell the model about relevant documents. A code execution engine like OpenAI's Code Interpreter can help the model do math and run code. If a task can be done more reliably or efficiently by a tool rather than by a language model, offload it to get the best of both.

**Tactics:**
- Use embeddings-based search to implement efficient knowledge retrieval
- Use code execution to perform more accurate calculations or call external APIs
- Give the model access to specific functions

### 6. Test Changes Systematically

Improving performance is easier if you can measure it. In some cases a modification to a prompt will achieve better performance on a few isolated examples but lead to worse overall performance on a more representative set of examples. Therefore to be sure that a change is net positive to performance it may be necessary to define a comprehensive test suite (also known as an "eval").

**Tactic:**
- Evaluate model outputs with reference to gold-standard answers

## Detailed Tactics

### Strategy: Write Clear Instructions

#### Include Details in Your Query

To get highly relevant responses, make sure requests provide important details or context. Otherwise, you're leaving it up to the model to guess what you mean.

Examples of worse vs. better queries:

| Worse | Better |
|-------|---------|
| How do I add numbers in Excel? | How do I add up a row of dollar amounts in Excel? I want to do this automatically for a whole sheet of rows with all the totals ending up on the right in a column called "Total". |
| Who's president? | Who was the president of Mexico in 2021, and how frequently are elections held? |
| Write code to calculate the Fibonacci sequence. | Write a TypeScript function to efficiently calculate the Fibonacci sequence. Comment the code liberally to explain what each piece does and why it's written that way. |
| Summarize the meeting notes. | Summarize the meeting notes in a single paragraph. Then write a markdown list of the speakers and each of their key points. Finally, list the next steps or action items suggested by the speakers, if any. |

[Continue with remaining detailed tactics...]

## Best Practices for Evaluation

When testing changes systematically, consider these guidelines for sample sizes needed for 95% confidence:

| Difference to detect | Sample size needed |
|---------------------|-------------------|
| 30% | ~10 |
| 10% | ~100 |
| 3% | ~1,000 |
| 1% | ~10,000 |

Good evals should be:
- Representative of real-world usage (or at least diverse)
- Contain many test cases for greater statistical power
- Easy to automate or repeat

Model-based evals can be useful when there exists a range of possible outputs that would be considered equally high in quality (e.g. for questions with long answers). The boundary between what can be realistically evaluated with a model-based eval and what requires a human to evaluate is fuzzy and is constantly shifting as models become more capable. We encourage experimentation to figure out how well model-based evals can work for your use case.

### Tactic: Evaluate model outputs with reference to gold-standard answers

Suppose it is known that the correct answer to a question should make reference to a specific set of known facts. Then we can use a model query to count how many of the required facts are included in the answer.

For example, using the following system message:

```SYSTEM
You will be provided with text delimited by triple quotes that is supposed to be the answer to a question. Check if the following pieces of information are directly contained in the answer:
Neil Armstrong was the first person to walk on the moon.
The date Neil Armstrong first walked on the moon was July 21, 1969.
For each of these points perform the following steps:
1 - Restate the point.
2 - Provide a citation from the answer which is closest to this point.
3 - Consider if someone reading the citation who doesn't know the topic could directly infer the point. Explain why or why not before making up your mind.
4 - Write "yes" if the answer to 3 was yes, otherwise write "no".
Finally, provide a count of how many "yes" answers there are. Provide this count as {"count": <insert count here>}.
```

Here's an example input where only one point is satisfied:

```SYSTEM
<insert system message above>
```
```USER
"""Neil Armstrong is famous for being the first human to set foot on the Moon. This historic event took place on July 21, 1969, during the Apollo 11 mission."""
```

Here's an example input where only one point is satisfied:

```SYSTEM
<insert system message above>
```
```USER
"""Neil Armstrong made history when he stepped off the lunar module, becoming the first person to walk on the moon."""
```

Here's an example input where none are satisfied:

```SYSTEM
<insert system message above>
```
```USER
"""In the summer of '69, a voyage grand,
Apollo 11, bold as legend's hand.
Armstrong took a step, history unfurled,
"One small step," he said, for a new world."""
```
There are many possible variants on this type of model-based eval. Consider the following variation which tracks the kind of overlap between the candidate answer and the gold-standard answer, and also tracks whether the candidate answer contradicts any part of the gold-standard answer:

```SYSTEM
Use the following steps to respond to user inputs. Fully restate each step before proceeding. i.e. "Step 1: Reason...".
Step 1: Reason step-by-step about whether the information in the submitted answer compared to the expert answer is either: disjoint, equal, a subset, a superset, or overlapping (i.e. some intersection but not subset/superset).
Step 2: Reason step-by-step about whether the submitted answer contradicts any aspect of the expert answer.
Step 3: Output a JSON object structured like: {"type_of_overlap": "disjoint" or "equal" or "subset" or "superset" or "overlapping", "contradiction": true or false}
```
Here's an example input with a substandard answer which nonetheless does not contradict the expert answer:

```SYSTEM
<insert system message above>
```
```USER
Question: """What event is Neil Armstrong most famous for and on what date did it occur? Assume UTC time."""
Submitted Answer: """Didn't he walk on the moon or something?"""
Expert Answer: """Neil Armstrong is most famous for being the first person to walk on the moon. This historic event occurred on July 21, 1969."""
```
Here's an example input with answer that directly contradicts the expert answer:

```SYSTEM
<insert system message above>
```
```USER
Question: """What event is Neil Armstrong most famous for and on what date did it occur? Assume UTC time."""
Submitted Answer: """On the 21st of July 1969, Neil Armstrong became the second person to walk on the moon, following after Buzz Aldrin."""
Expert Answer: """Neil Armstrong is most famous for being the first person to walk on the moon. This historic event occurred on July 21, 1969."""
```
Here's an example input with a correct answer that also provides a bit more detail than is necessary:

```SYSTEM
<insert system message above>
```
```USER
Question: """What event is Neil Armstrong most famous for and on what date did it occur? Assume UTC time."""
Submitted Answer: """At approximately 02:56 UTC on July 21st 1969, Neil Armstrong became the first human to set foot on the lunar surface, marking a monumental achievement in human history."""
Expert Answer: """Neil Armstrong is most famous for being the first person to walk on the moon. This historic event occurred on July 21, 1969."""
```