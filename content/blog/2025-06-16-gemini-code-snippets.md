---
title: "Gemini Code Snippets"
date: 2025-06-16
tags: ["llm", "evaluation", "prompt-engineering"]
Categories: ["tech"]
description: "Ref: 5-Day Gen AI Intensive Course with Google"
draft: false
---


## Prompt Engineering Using Gemini API

### Setup

```python
from google import genai
from google.genai import types
client = genai.Client(api_key=GOOGLE_API_KEY)
```

### Basic Usage

```python
response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="Explain AI to me like I'm a kid."
)
print(response.text)
```

### Interactive Chat

```python
chat = client.chats.create(model='gemini-2.0-flash', history=[])
response = chat.send_message('Hello! My name is Zlork.')
print(response.text)
```

### Listing Available Models

```python
for model in client.models.list():
    print(model.name)
```

### Model Output Settings

```python
# Max output tokens
short_config = types.GenerateContentConfig(max_output_tokens=200)

# High temperature for creative responses
high_temp_config = types.GenerateContentConfig(temperature=2.0)

response = client.models.generate_content(
    model='gemini-2.0-flash',
    config=short_config,
    contents='What could be done with a 1000 dollars and no idea...'
)
print(response.text)
```

---

## Function Calling with Gemini

- Docs: [Function Calling Guide](https://ai.google.dev/gemini-api/docs/function-calling?example=meeting)

```python
tools = types.Tool(function_declarations=[set_light_values_declaration])
config = types.GenerateContentConfig(tools=[tools])
```

---

## MIME Type Example for Structured Output

```python
config = types.GenerateContentConfig(
    temperature=0.1,
    response_mime_type="application/json",
    response_schema=PizzaOrder,
)
```

---

## Chain-of-Thought Prompting

Chain-of-thought prompting helps improve output quality by asking the model to **show reasoning steps**.

> This technique improves factual grounding but may **increase token costs**.

- [Gemini Cookbook: ReAct](https://github.com/google-gemini/cookbook/blob/main/examples/Search_Wikipedia_using_ReAct.ipynb)
- [ReAct GitHub](https://github.com/ysymyth/ReAct)

---

## Code Execution Tool (Gemini)

```python
config = types.GenerateContentConfig(
    tools=[types.Tool(code_execution=types.ToolCodeExecution())],
)
```

---