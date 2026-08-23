---
title: "NLP & Explainable AI (XAI) — Quick Reference Notes"
date: 2026-08-23
tags: ["GenAI", "LLMs"]
Categories: ["tech"]
draft: false
---

> These are quick reference notes meant to help reconstruct the concepts later rather than serve as a deep dive. AI summarizeed. 

## NLP Techniques — Quick Reference

| Technique | Concept | One-liner |
|---|---|---|
| **Bag of Words (BoW)** | Sparse text representation | Represents text using word occurrence/counts while largely ignoring word order and context. |
| **Naive Bayes** | Probabilistic classifier | Uses Bayes' theorem with a strong feature-independence assumption; historically common for text classification. |
| **VADER** | Lexicon-based sentiment analysis | Uses a predefined sentiment lexicon and rules to estimate positive, negative, neutral, and compound sentiment. |
| **BERT Embeddings** | Contextual representation | Represents words using surrounding context, so the same word can have different representations depending on usage. |
| **ABSA** | Aspect-Based Sentiment Analysis | Determines sentiment toward individual aspects rather than assigning one sentiment to the entire text. |
| **Zero-shot Classification** | No task-specific examples | Uses a pretrained/foundation model to perform classification without providing labeled examples. |
| **Few-shot Classification** | In-context examples | Provides a small number of examples in the prompt to demonstrate the desired classification behavior. |

### Evolution at a Glance

```text
Lexicon / Rules
    ↓
VADER
    ↓
Classical ML
    ↓
BoW → Naive Bayes
    ↓
Contextual Language Models
    ↓
BERT / Embeddings
    ↓
Fine-grained NLP
    ↓
ABSA
    ↓
Foundation Models
    ↓
Zero-shot / Few-shot
```

---

## XAI — Post-Hoc Explainability Quick Reference

**Post-hoc explainability** means explaining or interpreting a model's behavior **after the model has already produced its prediction**.

| Technique | Core Question | One-liner |
|---|---|---|
| **Attention Weights** | What did the model attend to? | Inspects how strongly different tokens or regions attend to one another; attention should not automatically be treated as a causal explanation. |
| **Saliency / Gradients** | Which inputs most influenced the output? | Measures how sensitive the model's output is to particular input features, often using gradients. |
| **LIME** | What explains this particular prediction locally? | Perturbs the input, observes prediction changes, and fits a simple interpretable model around that specific example. |
| **SHAP** | How much did each feature contribute? | Uses Shapley-value ideas from game theory to attribute portions of a prediction to individual input features. |

### Mental Model

```text
                    MODEL
                      │
                      ▼
                  Prediction
                      │
                      ▼
              POST-HOC EXPLANATION
                      │
       ┌──────────────┼──────────────┐
       │              │              │
   Attention       Saliency       LIME / SHAP
       │              │              │
What was the      What strongly    Why this
model attending   influenced the   particular
to?               output?          prediction?
```

### The Four One-Liners

- **Attention:** What was the model looking at?
- **Saliency:** What parts of the input strongly influenced the output?
- **LIME:** (Local Interpretable Model-agnostic Explanations) - Approximate this particular prediction locally with a simple model.
- **SHAP:** (SHapley Additive exPlanations) - Attribute the prediction among its input features.

> **Note:** These techniques provide interpretations or attributions of model behavior. They do not necessarily reveal the model's true internal reasoning.
