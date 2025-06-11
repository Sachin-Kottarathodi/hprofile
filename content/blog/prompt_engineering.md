---
title: "Prompt Engineering Notes"
date: 2025-06-11
tags: ["genai", "prompt-engineering", "llm"]
Categories: ["tech"]
description: "Comprehensive notes on prompt engineering strategies, sampling controls, and advanced prompting methods inspired by Google's whitepaper."
draft: false
---

# 🧠 Prompt Engineering Notes

Prompt engineering is the process of designing high-quality prompts to guide LLMs to produce accurate outputs. This involves refining length, structure, tone, and clarity.

## 📌 Model-Specific Setup

### ✅ Choose Your LLM and Configure:
- Model-specific prompts and capabilities
- Sampling parameters:
  - **Output Length**  
    More tokens = more compute cost. Reducing length just truncates output.
  - **ReAct model warning**: Emits irrelevant tokens post-response.

### 🔧 Sampling Controls

| Parameter | Effect |
|----------|--------|
| `Temperature` | Low = deterministic; High = creative/random |
| `Top-K` | Limit prediction to top K likely tokens |
| `Top-P` | Nucleus sampling = choose from top cumulative probability P |
| `Num Tokens` | Max output length |

#### Practical Guidelines:
- **Temperature = 0.2, Top-P = 0.95, Top-K = 30** → Balanced
- **Temperature = 0.9, Top-P = 0.99, Top-K = 40** → Creative
- **Temperature = 0.1, Top-P = 0.9, Top-K = 20** → Factual
- **Temperature = 0** → Tasks with one correct answer (math, facts)

> ⚠️ Beware of infinite loops & repetition at low temperature + long max tokens.

---

## 💡 Prompting Techniques

- **Zero-Shot**
- **One-Shot**
- **Few-Shot** → Include edge cases!

---

## 🧭 Role, Context, and System Prompting

| Type | Use |
|------|-----|
| `System` | Model's base behavior, safety (e.g., “Be respectful...”) |
| `Role` | Output style: Formal, Humorous, Persuasive, etc. |
| `Context` | Immediate task input (e.g., "You're writing a blog...") |

---

## 🧠 Advanced Prompting Methods

### Step-Back Prompting
- Ask a general question before the main task to activate relevant knowledge.
- Helps mitigate bias and improve performance.

### Chain-of-Thought (CoT)
- Use: “Let’s think step-by-step.”
- Great for reasoning-heavy tasks.
- CoT + few-shot = even better.

### Self-Consistency Prompting
- Generate multiple CoTs using high temperature.
- Extract final answers → choose the majority.

### Tree-of-Thoughts (ToT)
- Explores multiple CoTs in parallel.
- Great for planning and complex reasoning.  
  → [Read paper](https://arxiv.org/pdf/2305.08291)

### ReAct (Reason + Act)
- Combine reasoning with action (e.g., API calls, tools).
- Resend state, prompts, and reasoning continually.

---

## 🤖 Auto-Prompting & Code Prompting

- Write prompts that write better prompts.
- Use BLEU / ROUGE to evaluate prompt performance.
- **Code prompting** = Specify interfaces, structure, error handling, etc.

---

## 🔥 Best Practices

- **Give Examples**
- **Be Clear & Concise**
- **Prefer Positive Instructions** over negative constraints.
- **Use strong action verbs**: Act, Analyze, Generate, Rank, etc.
- **Control Max Token Length**
- **Tune CoT setups**:
  - Use greedy decoding (temp = 0)
  - Extract answers separately from reasoning
- **Log & Document** all prompt versions and results

---

## 📎 Reference: Google Whitepaper

**Title:** Prompt Engineering: Best Practices and Reasoning Strategies  
**Author:** Lee Boonstra, Google Cloud  
**Link:** [Read PDF](https://www.gptaiflow.tech/assets/files/2025-01-18-pdf-1-TechAI-Goolge-whitepaper_Prompt%20Engineering_v4-af36dcc7a49bb7269a58b1c9b89a8ae1.pdf)