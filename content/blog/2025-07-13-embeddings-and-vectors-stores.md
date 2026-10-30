---
title: "Embedding & Vector Stores: Notes"
date: 2025-07-13
tags: ["GenAI", "LLMs", "Embeddings", "Vector Stores",]
Categories: ["tech"]
draft: false
---

# Embedding & Vector Stores

## What are Embeddings?

- **Low-dimensional numerical representation** of almost anything (text, audio, images, video).
- A **lossy compression** technique: mapping complex data into simpler space while retaining semantic meaning.
- Enables **pattern recognition** — embeddings capture **semantic similarity** and **underlying meaning**.
- Useful for **efficient large-scale processing**.
- Think of it as: _"encoding meaning in vector form."_

## Use Cases

- **Retrieval & Recommendation** systems.
- **Search engines**: precompute embeddings for billions of pages, convert search query to embedding, find nearest neighbours in vector space.
- **Joint Embeddings**:
  - Handle **multi-modal data** (e.g., text & images).
  - Map different modalities into the same embedding space for cross-modal matching.

## Evaluation Metrics

How effective is your embedding?

- **Precision**: Of retrieved results, how many were relevant?
- **Recall**: Of all relevant results, how many did we retrieve?
- **Precision@k** / **Recall@k**: Focused on top-k results.
- **NDCG (Normalized Discounted Cumulative Gain)**: Prioritizes relevance ordering — higher is better.
- Evaluate and compare different embedding models based on these.

## Embeddings in RAG (Retrieval-Augmented Generation)

- Embeddings help retrieve relevant knowledge base data.
- Used to **boost LLM prompts** with accurate context.
- Process:
  - **Create index**.
  - **Chunk documents** and store in a **vector DB**.
  - Retrieve via **semantic search** — not just keyword match.
- **Meaning-based search** wins over keyword-based.

## Creating Labeled Data

- LLMs can generate **synthetic questions** to scale training data.
- Useful when labeled examples are scarce.

---

## Types of Embeddings

### 🔤 Text Embeddings

- Convert **words/sentences** to vectors.

#### Tokenization

- Text → tokens → numerical IDs.
- Techniques:
  - One-hot encoding
  - Word embeddings

#### Word Embeddings

- **Word2Vec**: Meaning from context.
  - CBOW (Continuous Bag of Words): predict target from context.
  - Skip-gram: predict context from target.
- **GloVe**: Co-occurrence matrix + factorization.
- **Swivel**: Efficient for massive corpora.

#### Document Embeddings

- **BOW (Bag of Words)** + statistical models:
  - **LSA** (Latent Semantic Analysis)
  - **LDA** (Latent Dirichlet Allocation)
  - **TF-IDF**
- **Doc2Vec**: Incorporates word order.
- **BERT**: Contextualized embeddings.
- **Multi-vector Embeddings**: Multiple views into one document.

### 🧮 Structured Data Embedding

- Example: Table → **PCA** (Principal Component Analysis).

### 🌐 Graph Embedding

- Encodes **relational/social structures** into vector space.

---

## Training Embedding Models

- **Dual Encoder Architecture**
- **Pretraining** on large corpora
- **Fine-tuning** on domain-specific datasets

---

## Vector Search

- Match embeddings close to a **query embedding**.
- More powerful than plain text search.

### Techniques

- **LSH (Locality-Sensitive Hashing)** — like assigning zip codes to vectors.
- **HNSW (Hierarchical Navigable Small World)**
- **SCAN (Scalable Approximate Nearest Neighbour)**

---

> ⚠️ **Note**: Embeddings aren’t always perfect.  
> Sometimes, traditional text-based methods are better depending on the task.