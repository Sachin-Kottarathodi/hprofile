---
title: "NLP & Explainable AI (XAI) — Quick Reference Notes"
date: 2026-08-23
tags: ["GenAI", "LLMs"]
Categories: ["tech"]
draft: false
---
> AI generated - These are quick reference notes meant to help reconstruct the concepts later rather than serve as a deep dive.

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

## XAI — Post-Hoc Explainability

**Post-hoc explainability** means explaining or interpreting a model's behavior **after the model has already produced its prediction**.

| Technique | Core Question | One-liner |
|---|---|---|
| **Attention Weights** | What did the model attend to? | Inspects how strongly different tokens or regions attend to one another; attention should not automatically be treated as a causal explanation. |
| **Saliency / Gradients** | Which inputs most influenced the output? | Measures how sensitive the model's output is to particular input features, often using gradients. |
| **LIME** | What explains this particular prediction locally? | Perturbs the input, observes prediction changes, and fits a simple interpretable model around that specific example. |
| **SHAP** | How much did each feature contribute? | Uses Shapley-value ideas from game theory to attribute portions of a prediction to individual input features. |

## LIME vs SHAP — Example

Consider a sentiment model receiving:

```text
"The camera is excellent but the battery is terrible."
```

Suppose the model predicts:

```text
Negative: 0.82
```

Both LIME and SHAP can help explain this prediction, but they approach the problem differently.

### LIME — Local Interpretable Model-agnostic Explanations

LIME tries to understand the model's behavior **around this particular input**.

It creates many slightly modified versions of the input and observes how the model's prediction changes.

For example:

```text
Original:
"The camera is excellent but the battery is terrible."
→ Negative: 0.82

Remove "terrible":
"The camera is excellent but the battery is."
→ Negative: 0.18

Remove "excellent":
"The camera is but the battery is terrible."
→ Negative: 0.94

Remove "battery":
"The camera is excellent but the is terrible."
→ Negative: 0.61
```

From many such perturbations, LIME might observe:

```text
terrible   → strongly pushes toward NEGATIVE
battery    → somewhat important
excellent  → pushes toward POSITIVE
camera     → relatively small effect
```

LIME then fits a **simple interpretable model**, often something like a linear model, around these observations.

The simple model is not trying to reproduce the behavior of the original model everywhere. It only tries to approximate the model **near this particular input**.

Hence the name:

- **Local** — explains the model around one particular prediction.
- **Interpretable** — uses a simpler model that humans can understand.
- **Model-agnostic** — does not need to understand the internals of the original model; it only needs inputs and outputs.

Mental model:

```text
Original input
      ↓
Perturb the input many ways
      ↓
Run each variation through the model
      ↓
Observe how predictions change
      ↓
Fit a simple local model
      ↓
Identify locally important features
```

> **LIME = perturb the input and approximate the model's behavior locally.**

---

### SHAP — SHapley Additive exPlanations

SHAP approaches explanation as a **credit-allocation problem**.

It is based on **Shapley values** from cooperative game theory. Imagine several players cooperating to produce an outcome. Shapley values try to answer:

> How much credit should each player receive for the final result?

In machine learning:

```text
Game Theory                 Machine Learning

Players                     Input features
   ↓                              ↓
Work together               Combine together
   ↓                              ↓
Produce outcome             Produce prediction
   ↓                              ↓
Shapley values              SHAP values
```

Return to the sentiment example.

Suppose the model's baseline negative score is:

```text
Baseline = 0.40
```

For our sentence it predicts:

```text
Negative = 0.82
```

The features collectively moved the prediction from `0.40 → 0.82`.

SHAP tries to determine how much each feature contributed to that movement.

Conceptually:

```text
Baseline                         0.40

"terrible"                      +0.38
"battery"                       +0.12
"excellent"                     -0.14
"camera"                        +0.01
other features                  +0.05
                               ─────
Prediction                      0.82
```

The important question is: **how do we decide that `"terrible"` deserves +0.38?**

SHAP considers the contribution of the feature across different combinations of the other features:

```text
How much does "terrible" contribute alone?

How much does "terrible" contribute
when "battery" is already present?

How much does "terrible" contribute
when "excellent" is present?

How much does "terrible" contribute
when "battery" and "camera" are present?

...
```

The Shapley-value calculation aggregates these **marginal contributions** across possible feature combinations to assign credit to each feature.

This also explains the word **Additive** in SHAP:

```text
Baseline
    +
Feature contribution 1
    +
Feature contribution 2
    +
Feature contribution 3
    +
...
    ↓
Explained prediction
```

Hence:

**SH**apley **A**dditive ex**P**lanations.

Mental model:

```text
Prediction
     ↓
Treat features as cooperating players
     ↓
Consider different feature combinations
     ↓
Measure each feature's marginal contribution
     ↓
Calculate Shapley values
     ↓
Assign contribution to each feature
```

> **SHAP = distribute the prediction's credit among features using Shapley values.**

---

## LIME vs SHAP — The Distinction to Remember


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


```text
LIME
  ↓
Perturb the input
  ↓
Observe model predictions
  ↓
Fit a simple LOCAL approximation
  ↓
"Which features seem important
around this particular example?"


SHAP
  ↓
Consider features in different combinations
  ↓
Measure marginal contributions
  ↓
Calculate Shapley values
  ↓
"How much did each feature contribute
to this prediction relative to a baseline?"
```

The simplest distinction:

> **LIME = perturb and approximate locally.**

> **SHAP = distribute prediction credit among features using Shapley values.**

## Four XAI One-Liners

- **Attention:** What was the model looking at?
- **Saliency:** What parts of the input strongly influenced the output?
- **LIME:** Perturb the input and approximate this prediction locally with a simple model.
- **SHAP:** Attribute the prediction among its input features using Shapley values.

> **Important:** These techniques provide interpretations or attributions of model behavior. They do not necessarily reveal the model's true internal reasoning.
