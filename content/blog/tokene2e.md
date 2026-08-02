---
title: "Token E2E"
description: "Life of a Token"
date: 2026-05-21
tags: ["GenAI", "LLMs"]
Categories: ["tech"]
draft: false
---

> **Disclaimer**
>
> This post began as my scattered notes and questions while learning
> LLMs. I couldn't stitch them together so used AI to do that for me.
> The fragments are connected into a coherent model. It is intentionally
> simplified as I am not there yet for this to be too complex or
> detailed at this point.

------------------------------------------------------------------------

## Why this exists

I found myself collecting terms like BPE, RoPE, KV Cache, Teacher
Forcing, PagedAttention, Quantization and Data Parallelism. Individually
they made sense, but I had no mental map connecting them.

The easiest way to understand an LLM is to follow **the life of a
token**.

------------------------------------------------------------------------

## Stage 1 --- Raw Text

Everything starts with raw text from the user.

    "Explain TCP."

The model cannot consume characters directly.

------------------------------------------------------------------------

## Stage 2 --- Pre-tokenization

A deterministic preprocessing step separates punctuation, whitespace and
other lexical boundaries so the tokenizer does not create unnecessary
vocabulary variants.

------------------------------------------------------------------------

## Stage 3 --- Subword Tokenization (BPE)

The tokenizer converts text into reusable subword pieces and finally
into integer Token IDs.

Purpose:

-   Balance vocabulary size
-   Reduce out-of-vocabulary words
-   Keep context windows reasonably compact

------------------------------------------------------------------------

## Stage 4 --- Embedding Lookup

Each Token ID indexes into the embedding matrix.

    Token ID
        ↓
    Embedding Matrix
        ↓
    Dense Vector

The model now works entirely with vectors.

------------------------------------------------------------------------

## Stage 5 --- Positional Encoding (RoPE)

Transformers process tokens simultaneously, so position must be encoded.

RoPE injects relative positional information directly into attention
instead of using a fixed lookup table.

------------------------------------------------------------------------

## Stage 6 --- Self Attention

Every token computes:

-   Query
-   Key
-   Value

Attention determines which previous tokens are important for
understanding the current one.

During generation, **Causal Masking** prevents tokens from attending to
future tokens.

------------------------------------------------------------------------

## Stage 7 --- Pre-training

The model is trained using **Next Token Prediction**.

Teacher Forcing supplies the correct sequence during training, allowing
the entire context window to be processed in parallel.

Large models train using combinations of:

-   Data Parallelism
-   Tensor Parallelism
-   Pipeline Parallelism

------------------------------------------------------------------------

## Stage 8 --- Inference

Deployment is different from training.

Generation becomes autoregressive:

    Prompt
     ↓
    Predict one token
     ↓
    Append token
     ↓
    Predict next token

------------------------------------------------------------------------

## Stage 9 --- KV Cache

Previously computed Keys and Values are stored in GPU memory.

Instead of recomputing attention over the entire prompt every generation
step, only the newest token performs fresh computation.

------------------------------------------------------------------------

## Stage 10 --- PagedAttention

Serving engines like vLLM divide the KV Cache into pages instead of
requiring large contiguous allocations.

Benefits:

-   Better GPU memory utilization
-   Less fragmentation
-   Higher serving throughput

------------------------------------------------------------------------

## Stage 11 --- Quantization

Weights are stored using lower precision formats such as FP8 or INT4.

Benefits:

-   Smaller models
-   Less VRAM
-   Higher throughput
-   Lower serving cost

Trade-off:

-   Slight reduction in numerical precision.

------------------------------------------------------------------------

## The Complete Pipeline

``` text
Raw Text
        ↓
Pre-tokenization
        ↓
Subword Tokenization (BPE)
        ↓
Token IDs
        ↓
Embedding Matrix
        ↓
RoPE Positional Encoding
        ↓
Transformer Layers
        ↓
Self Attention + Causal Mask
        ↓
Next Token Prediction
        ↓
Autoregressive Generation
        ↓
KV Cache
        ↓
PagedAttention
        ↓
Quantization (Serving Optimization)
        ↓
Repeat until completion
```

------------------------------------------------------------------------