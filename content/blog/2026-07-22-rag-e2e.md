---
title: "RAG Retrieval: From Keyword Search to Vector Search"
date: 2026-07-22
tags: ["GenAI", "LLMs", "RAGs"]
Categories: ["tech"]
draft: false
---

> Fully AI generated notes.

---

## 1. Why Does RAG Exist?

An LLM primarily carries knowledge in its **parameters**.

During training:

```text
Training Data
     ↓
Model Training
     ↓
Weights / Parameters
     ↓
Parametric Knowledge
```

Once trained, asking the model a question does not normally cause it to search through its original training documents.

The answer is generated from what has been learned into the model's parameters.

That creates obvious limitations:

* knowledge can become stale
* private organizational data was never part of training
* the model may not know niche information
* updating knowledge by retraining is expensive
* generated answers aren't inherently grounded in a specific source

RAG — **Retrieval-Augmented Generation** — adds external knowledge.

```text
                  USER QUERY
                       │
                       ▼
                    RETRIEVER
                       │
                Search knowledge
                       │
                       ▼
                Relevant documents
                       │
                       ▼
             Query + retrieved context
                       │
                       ▼
                      LLM
                       │
                       ▼
                    ANSWER
```

This creates a useful separation:

```text
LLM Parameters
     ↓
Parametric Memory

External Documents
     ↓
Non-Parametric / External Memory
```

The interesting problem then becomes:

> **Given a query and potentially millions of documents, how do we find the right information?**

That takes us into **information retrieval**.

---

# 2. The Simplest Retrieval: Word Matching

Imagine a corpus:

```text
D1: Java supports concurrency
D2: Python is a programming language
D3: Java virtual threads improve concurrency
```

And the query:

```text
"Java concurrency"
```

The simplest idea is:

> Find documents containing the query words.

This leads to **Bag of Words** representations.

Conceptually:

```text
Vocabulary:

Java
Python
concurrency
virtual
threads
programming
```

A document can be represented by which vocabulary terms occur within it.

Because a document contains only a tiny fraction of the entire vocabulary, most dimensions are zero.

Hence:

> **Sparse representation**

And retrieval based on these representations becomes:

> **Sparse retrieval**

---

# 3. Term Frequency — TF

Not every word appearing in a document is equally important.

One simple signal is:

> **How often does this term occur inside this document?**

That's **Term Frequency (TF)**.

Suppose:

```text
D1:
"Java Java Java concurrency programming"
```

Then:

```text
TF(Java)        = high
TF(concurrency) = lower
```

TF answers:

> **How important/frequent is this term within this particular document?**

But TF alone has a problem.

Words appearing everywhere aren't necessarily useful for identifying relevant documents.

---

# 4. Inverse Document Frequency — IDF

Suppose our corpus contains 1 million documents.

If:

```text
"programming"
```

appears in 800,000 documents, seeing it tells us relatively little.

But:

```text
"virtual-thread-pinning"
```

might occur in only 500 documents.

That term is much more discriminative.

This gives us **Inverse Document Frequency (IDF)**.

Conceptually:

```text
Appears in many documents
        ↓
Common term
        ↓
Low IDF


Appears in few documents
        ↓
Rare term
        ↓
High IDF
```

A common formulation is:

[
IDF(t)=\log\frac{N}{df(t)}
]

where:

```text
N     = number of documents
df(t) = number of documents containing term t
```

A critical distinction:

```text
TF
↓
How many times does the term occur
inside THIS document?


DF
↓
How many DOCUMENTS contain the term?
```

If:

```text
D1 = "Java Java Java Java"
```

then:

```text
TF(Java, D1) = 4

but

DF contribution = 1 document
```

We don't count the same document four times when calculating document frequency.

---

# 5. TF-IDF

Now combine the two ideas:

[
TF\text{-}IDF = TF \times IDF
]

A term becomes important when it is:

```text
Frequent in THIS document
           +
Rare across ALL documents
```

This provides a much better relevance signal than raw word counts.

The evolution so far:

```text
Bag of Words
     ↓
Term Frequency
     ↓
Inverse Document Frequency
     ↓
TF-IDF
```

---

# 6. BM25

BM25 builds upon the same general lexical-retrieval intuition.

Suppose:

```text
Query:
"Java virtual threads"
```

For a document, BM25 effectively evaluates the contribution of the query terms:

```text
score(Java)
    +
score(virtual)
    +
score(threads)
    ↓
Document relevance score
```

But BM25 improves upon simple TF-IDF in important ways.

### Term-frequency saturation

Suppose:

```text
D1:
"Java Java Java Java Java Java Java..."
```

Seven occurrences should not necessarily make the document seven times more relevant than one occurrence.

BM25 makes additional occurrences progressively less valuable.

Conceptually:

```text
Relevance
   │
   │            _________
   │         __/
   │      __/
   │   __/
   │__/
   └──────────────────────
        Term Frequency
```

### Document-length normalization

A 10,000-word document naturally has more opportunities to contain a term than a 100-word document.

BM25 accounts for document length.

### Rare terms remain important

BM25 also incorporates an IDF-like notion.

So a rare query term can contribute more relevance than an extremely common one.

---

# 7. Sparse Retrieval

TF-IDF and BM25 belong to the **sparse retrieval** family.

The mental model:

```text
Query:
"Java concurrency"

Document:
"Java concurrency with virtual threads"

       ↓

Strong lexical overlap

       ↓

High relevance
```

Sparse retrieval is particularly good when exact terminology matters:

```text
Product IDs
Error codes
Names
Technical terms
Acronyms
Exact phrases
```

But there is an obvious weakness.

Consider:

```text
Query:
"How do I run multiple tasks simultaneously in Java?"

Document:
"Java concurrency using virtual threads"
```

The meanings are highly related.

But the words don't overlap very much.

This takes us to **dense retrieval**.

---

# 8. Dense Retrieval

Instead of representing text using vocabulary dimensions, an embedding model converts text into a dense numerical vector.

```text
"Java concurrency"

        ↓
Embedding Model
        ↓

[0.21, -0.81, 0.17, 0.42, ...]
```

Likewise:

```text
"Executing multiple tasks simultaneously"

        ↓
Embedding Model
        ↓

[0.19, -0.79, 0.21, 0.39, ...]
```

These vectors can be close even though the original text uses different words.

Now retrieval asks:

> **Which document vectors are closest to the query vector?**

rather than:

> Which documents contain the same words?

So:

```text
SPARSE RETRIEVAL

Query
 ↓
Words
 ↓
Word overlap
 ↓
Relevant documents


DENSE RETRIEVAL

Query
 ↓
Embedding
 ↓
Vector similarity
 ↓
Semantically similar documents
```

---

# 9. Cross-Encoder Retrieval

One way to calculate relevance is to give the model both the query and document together.

```text
Query ───────┐
             │
             ├── Transformer
             │        ↓
Document ────┘   Relevance Score
```

Because the model sees both simultaneously, it can model very detailed interactions between them.

This can provide excellent relevance scoring.

But imagine having:

```text
10,000,000 documents
```

For every query:

```text
Query + D1 → Model
Query + D2 → Model
Query + D3 → Model
...
Query + D10,000,000 → Model
```

That is far too expensive for first-stage retrieval.

---

# 10. Bi-Encoder / Dual-Encoder Retrieval

Instead, encode the query and documents independently.

Documents can be embedded ahead of time:

```text
D1 → Encoder → Vector 1
D2 → Encoder → Vector 2
D3 → Encoder → Vector 3
...
```

Store those vectors.

Then at query time:

```text
Query
  ↓
Encoder
  ↓
Query Vector
  ↓
Find nearby document vectors
```

This is dramatically more scalable.

The trade-off:

> Query and document don't interact deeply while their representations are being produced.

Their interaction occurs through vector similarity afterward.

This can lose some fine-grained relevance information compared with a cross-encoder.

---

# 11. ColBERT — A Middle Ground

A normal bi-encoder might reduce an entire query to:

```text
Query → ONE vector
```

and an entire document to:

```text
Document → ONE vector
```

ColBERT retains token-level representations.

For:

```text
Query:
"Java concurrency"
```

we might conceptually retain:

```text
Java        → vector Q1
concurrency → vector Q2
```

And for a document:

```text
"Java supports concurrent programming"
```

retain:

```text
Java        → D1
supports    → D2
concurrent  → D3
programming → D4
```

For each query-token representation, ColBERT finds the best matching document-token representation.

```text
Java ─────────────→ Java

concurrency ──────→ concurrent
```

This **MaxSim** interaction preserves more fine-grained matching than collapsing everything into a single vector.

The trade-off is increased storage and retrieval complexity.

---

# 12. The Scaling Problem

Dense retrieval now gives us another problem.

Suppose we have:

```text
10,000,000 document vectors
```

A query arrives.

Naively:

```text
Query Vector
      ↓
Compare with D1
Compare with D2
Compare with D3
...
Compare with D10,000,000
```

That's an exhaustive nearest-neighbor search.

Conceptually:

[
O(N)
]

As the vector collection becomes huge, comparing against everything becomes expensive.

This leads to:

# Approximate Nearest Neighbor Search — ANN

Instead of demanding the mathematically exact nearest neighbors:

> **Find very likely nearest neighbors much faster.**

This trades some retrieval recall for enormous improvements in latency and scalability.

Three important names appear here:

```text
ANN
 │
 ├── IVF
 ├── HNSW
 └── PQ
```

But they solve somewhat different problems.

---

# 13. IVF — Reduce the Search Space

**IVF — Inverted File Index**

The core idea:

> Don't search the entire vector space. First determine which regions are promising.

During indexing, run something like K-means over the document vectors.

```text
                   VECTOR SPACE

        Cluster A                Cluster B

       • • • •                    • •
     • • • • •                  • • •
        • •                      • •


                    Cluster C

                   • • •
                 • • • • •
                   • •
```

Each cluster has a centroid.

Documents are assigned to nearby clusters.

At query time:

```text
Query
  ↓
Find nearby cluster centroids
  ↓
Search only those clusters
  ↓
Find nearest document vectors
```

Instead of:

```text
Search 10,000,000 vectors
```

we might search only a fraction of them.

The key mental model:

> **IVF reduces WHERE we search.**

---

# 14. HNSW — Navigate Instead of Scan

**HNSW — Hierarchical Navigable Small World**

HNSW takes a completely different approach.

Vectors become nodes in a graph.

Nearby vectors have graph connections.

```text
A ───── B
│       │
│       │
C ───── D ───── E
         \
          F
```

Instead of scanning every vector, search navigates through this graph toward increasingly promising neighbors.

HNSW adds hierarchical layers.

```text
Sparse upper layer
        ↓
Fast large jumps
        ↓
More detailed layer
        ↓
Smaller jumps
        ↓
Dense bottom layer
        ↓
Nearest neighbors
```

Conceptually:

```text
Start somewhere
      ↓
Move toward query
      ↓
Get closer
      ↓
Descend layer
      ↓
Refine search
      ↓
Nearest candidates
```

The key mental model:

> **HNSW reduces search work by navigating a graph rather than scanning the whole vector collection.**

---

# 15. Product Quantization — Reduce Storage

Now consider another problem.

Suppose there are:

```text
10,000,000 vectors
```

Each vector has:

```text
1024 dimensions
```

And each dimension is:

```text
float32 = 4 bytes
```

One vector therefore needs:

[
1024 \times 4 = 4096 \text{ bytes}
]

For 10 million vectors:

[
10,000,000 \times 4096
======================

40.96\text{ GB}
]

That's a substantial amount of memory just for the vectors.

This is where **Product Quantization (PQ)** enters.

The important idea—not the implementation details—is:

> **Approximate large vectors using compact codes rather than storing every floating-point value.**

Suppose:

```text
1024-D vector
```

is split into:

```text
16 sub-vectors
```

Each sub-vector contains:

```text
1024 / 16 = 64 dimensions
```

For each sub-vector position, K-means learns representative vectors called **centroids**.

Then instead of storing:

```text
[0.23, 0.91, -0.72, 0.14, ...]
```

we store:

```text
"this sub-vector looks most like centroid #17"
```

Across 16 sub-vectors, the original vector becomes something conceptually like:

```text
[17, 203, 4, 91, 7, ..., 28]
```

These centroid IDs are the **PQ codes**.

If each subspace has:

```text
256 possible centroids
```

then a centroid ID requires:

```text
8 bits = 1 byte
```

Therefore:

```text
16 sub-vectors
×
1 byte
=
16 bytes/vector
```

instead of:

```text
4096 bytes/vector
```

For the example above:

```text
Original vectors:
≈ 40.96 GB

PQ codes:
≈ 160 MB
```

That's an enormous reduction.

But information has been discarded.

So distance calculations become approximate.

The key mental model:

> **PQ reduces HOW MUCH vector data we store and process, at the cost of precision.**

The details of codebooks, lookup tables, asymmetric distance computation and exact scoring can be revisited when vector-database internals become important.

---

# 16. IVF vs HNSW vs PQ

This distinction is worth remembering.

```text
IVF
 ↓
Partition the vector space
 ↓
Search fewer regions


HNSW
 ↓
Build a navigable graph
 ↓
Reach nearby vectors quickly


PQ
 ↓
Compress vectors
 ↓
Store and compare cheaper approximations
```

So they aren't simply three competing versions of the same algorithm.

They attack different scaling problems.

And techniques can be combined.

For example:

```text
IVF
 ↓
Find promising regions

+

PQ
 ↓
Store vectors compactly

=

IVF-PQ
```

---

# 17. Retrieval Accuracy vs Efficiency

Approximate retrieval introduces an important system-design trade-off.

Exact search:

```text
High accuracy
     ↓
High compute / memory cost
```

Approximate search:

```text
Much faster / smaller
     ↓
Potentially miss some good candidates
```

This is why retrieval systems often care about metrics such as **Recall@K**.

Suppose exact nearest-neighbor search identifies the true best 100 documents.

Approximate retrieval finds 94 of them.

```text
Recall@100 = 94%
```

Whether that's acceptable depends on the application.

For many RAG systems, first-stage retrieval doesn't need perfect ordering.

It needs to ensure:

> **The genuinely useful documents survive into the candidate set.**

---

# 18. Reranking

This brings us back to the cross-encoder.

Earlier, cross-encoders were too expensive:

```text
Query × 10,000,000 documents
```

But what if dense retrieval first reduces the candidates?

```text
10,000,000 documents
        ↓
ANN retrieval
        ↓
Top 100 candidates
        ↓
Cross-encoder
        ↓
Top 10
```

Now expensive query-document interaction becomes practical.

This creates a common retrieval architecture:

```text
                 QUERY
                   │
                   ▼
            Fast Retriever
                   │
          millions of docs
                   │
                   ▼
            Top Candidates
                   │
                   ▼
               Reranker
                   │
          deeper Q-D interaction
                   │
                   ▼
            Best Documents
```

The first stage optimizes:

> **Recall + speed**

The second stage optimizes:

> **Precision**

---

# 19. Sparse + Dense = Hybrid Retrieval

Sparse and dense retrieval have complementary strengths.

Consider:

```text
"ERR_JAVA_4821"
```

An embedding model may not understand this identifier particularly well.

BM25 can match it exactly.

Conversely:

```text
"How do I execute many lightweight tasks simultaneously?"
```

may semantically match:

```text
"Java virtual thread concurrency"
```

even with weak lexical overlap.

Dense retrieval is useful here.

Therefore modern RAG systems frequently combine both.

```text
                         QUERY
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        Sparse Search              Dense Search
            BM25                   Embeddings
              │                         │
              ▼                         ▼
        Exact lexical              Semantic
          matches                   matches
              │                         │
              └────────────┬────────────┘
                           │
                           ▼
                    Merge Candidates
                           │
                           ▼
                       Reranker
                           │
                           ▼
                    Best Documents
                           │
                           ▼
                          LLM
```

This is **hybrid retrieval**.

---

# 20. Retrieval and Generation Are Separate Problems

One of the most useful RAG distinctions is:

```text
                  RAG
                   │
         ┌─────────┴─────────┐
         │                   │
     RETRIEVAL           GENERATION
         │                   │
"What information       "What should I
 should the model        say using that
 receive?"               information?"
```

A bad RAG answer does **not** automatically mean the LLM failed.

Suppose:

```text
User Query
    ↓
Retriever chooses wrong chunks
    ↓
Perfectly capable LLM
    ↓
Wrong / incomplete answer
```

The generation model never received the information it needed.

Conversely:

```text
User Query
    ↓
Excellent retrieval
    ↓
Correct documents
    ↓
Poor generation/reasoning
    ↓
Bad answer
```

So RAG quality must be understood as at least two separate systems:

```text
Retrieval Quality

+

Generation Quality

=

Overall RAG Quality
```

---

# 21. The Complete Retrieval Progression

The entire story now fits together.

```text
                        INFORMATION RETRIEVAL
                                  │
                                  ▼
                           Bag of Words
                                  │
                                  ▼
                               TF-IDF
                                  │
                                  ▼
                                BM25
                                  │
                                  ▼
                        SPARSE RETRIEVAL
                                  │
                    Exact lexical matching
                                  │
                                  │
                                  ▼
                         DENSE RETRIEVAL
                                  │
                              Embeddings
                                  │
                  Semantic similarity search
                                  │
                     ┌────────────┴────────────┐
                     │                         │
                     ▼                         ▼
                 Bi-Encoder                ColBERT
                 one vector             token vectors
                     │                         │
                     ▼                         │
                Large-scale                    │
               vector search                   │
                     │                         │
                     ▼                         │
                    ANN ◄───────────────────────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
         IVF        HNSW        PQ
          │          │          │
       reduce      graph      compress
       search    navigation    vectors
       space
          │
          └──────────┬──────────┘
                     │
                     ▼
              Candidate Documents
                     │
                     ▼
                  Reranker
                     │
                     ▼
               Best Documents
                     │
                     ▼
              Add to LLM Context
                     │
                     ▼
                    LLM
                     │
                     ▼
                  ANSWER
```

---

# 22. What Is Actually Worth Remembering?

The implementation details can disappear from memory.

The structure shouldn't.

### Sparse vs Dense

```text
Sparse
→ words matching words
→ TF-IDF / BM25

Dense
→ meaning matching meaning
→ embeddings
```

### Bi-encoder vs Cross-encoder

```text
Bi-encoder
→ encode query and documents independently
→ scalable

Cross-encoder
→ query + document together
→ richer interaction
→ expensive
→ excellent for reranking
```

### ANN

```text
Millions of vectors
       ↓
Exact comparison becomes expensive
       ↓
Approximate Nearest Neighbor search
```

### IVF

> **Reduce where I search.**

### HNSW

> **Navigate efficiently toward nearby vectors.**

### PQ

> **Compress what I store.**

### Reranking

> **Retrieve broadly and cheaply first; evaluate a small candidate set more carefully afterward.**

### Hybrid retrieval

> **Use lexical and semantic retrieval together because each catches things the other misses.**

---

# 23. The RAG Mental Model

When everything else is forgotten, reconstruct it from this:

```text
                         USER QUERY
                              │
                              ▼
                    Understand the query
                              │
                              ▼
                         RETRIEVAL
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
              BM25                      Embeddings
             Sparse                       Dense
                │                           │
                └─────────────┬─────────────┘
                              ▼
                       ANN at scale
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
                IVF          HNSW          PQ
                              │
                              ▼
                      Candidate chunks
                              │
                              ▼
                           Rerank
                              │
                              ▼
                     Best relevant chunks
                              │
                              ▼
                    Query + RAG Context
                              │
                              ▼
                             LLM
                              │
                              ▼
                           ANSWER
```

RAG itself is simple:

> **Retrieve useful information before asking the model to generate.**

The complexity comes from making **"retrieve useful information"** work accurately, quickly and cheaply at scale.

And that is why concepts that existed long before LLMs—TF-IDF, BM25, inverted indexes, nearest-neighbor search, clustering, quantization and information retrieval—suddenly become part of understanding modern generative AI systems.
