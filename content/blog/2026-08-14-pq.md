---
title: "Product Quantization"
date: 2026-08-14
tags: ["GenAI", "LLMs", "RAGs"]
Categories: ["tech"]
draft: false
---

> **AI-polished:** This post originated from notes and questions while working through the topic in detail. AI was used to organize the notes, connect the ideas, and polish them into a coherent blog post.


# Product Quantization — From 40 GB of Vectors to 160 MB

Dense retrieval introduces a scaling problem beyond search latency: **memory**.

Suppose a vector database contains:

```text
10,000,000 vectors
```

Each vector contains:

```text
1024 dimensions
```

and each dimension is represented using:

```text
float32 = 4 bytes
```

The raw storage required for the vectors is:

[
10,000,000 \times 1024 \times 4
]

[
= 40,960,000,000\text{ bytes}
]

or approximately:

```text
40.96 GB
```

And this is only the vector data.

At hundreds of millions or billions of vectors, keeping every vector in full precision becomes increasingly expensive.

This is one of the problems **Product Quantization (PQ)** tries to solve.

The central idea is:

> **Instead of storing every floating-point value of every vector, split the vectors into smaller pieces, approximate each piece using a learned representative vector, and store only the ID of that representative.**

---

# 1. Split Each Vector into Subvectors

Suppose PQ is configured with:

```text
m = 16
```

Our original vector contains 1024 dimensions.

Therefore each vector is divided into 16 subvectors:

[
1024 / 16 = 64
]

Each subvector contains 64 dimensions.

Conceptually:

```text
Original 1024-D vector

┌────────┬────────┬────────┬────── ... ──────┬────────┐
│   S1   │   S2   │   S3   │                 │  S16   │
│  64-D  │  64-D  │  64-D  │                 │  64-D  │
└────────┴────────┴────────┴────── ... ──────┴────────┘
```

Do this for all database vectors:

```text
Vector 1 → [S1][S2][S3] ... [S16]
Vector 2 → [S1][S2][S3] ... [S16]
Vector 3 → [S1][S2][S3] ... [S16]
...
```

Nothing has been compressed yet.

We have simply divided the original vector space into smaller subspaces.

---

# 2. Learn Representative Centroids

Now take only the **first subvector position** across many training vectors:

```text
Vector 1 → S1 ─┐
Vector 2 → S1  │
Vector 3 → S1  │
Vector 4 → S1  ├──→ K-means
...            │
Vector N → S1 ─┘
```

Every `S1` is a 64-dimensional vector.

Suppose PQ is configured with:

```text
k = 256
```

Run K-means with:

```text
K = 256
```

K-means groups these 64-dimensional subvectors into 256 clusters.

Each cluster gets a representative center:

> **Centroid**

Conceptually:

```text
                   S1 subvectors

                • • •
              • • • •
                • •
                 ↓
            Centroid #0


                           • •
                         • • • •
                           •
                           ↓
                      Centroid #1


               ...

                      Centroid #255
```

For subspace `S1`, we therefore learn:

```text
Centroid 0
Centroid 1
Centroid 2
...
Centroid 255
```

These 256 centroid vectors collectively form a:

> **Codebook**

Therefore:

```text
All S1 subvectors
       ↓
     K-means
       ↓
256 representative centroids
       ↓
    Codebook 1
```

---

# 3. Build One Codebook per Subspace

Repeat this independently for every subvector position.

```text
All S1 subvectors
       ↓
K-means
       ↓
256 centroids
       ↓
Codebook 1


All S2 subvectors
       ↓
K-means
       ↓
256 centroids
       ↓
Codebook 2


All S3 subvectors
       ↓
K-means
       ↓
256 centroids
       ↓
Codebook 3


...


All S16 subvectors
       ↓
K-means
       ↓
256 centroids
       ↓
Codebook 16
```

Our PQ model therefore contains:

```text
16 codebooks
×
256 centroids per codebook
```

Each centroid is itself a 64-dimensional representative vector.

These centroids are learned during **PQ training/index construction**.

They are not rediscovered when a query arrives.

---

# 4. Compress a Database Vector

Now that the codebooks exist, the database vectors can be compressed.

Take one database vector:

```text
1024-D vector

[S1][S2][S3] ... [S16]
```

Consider `S1`.

It contains 64 float32 values.

Compare it against the 256 centroids in **Codebook 1**:

```text
S1
 │
 ├── distance → centroid 0
 ├── distance → centroid 1
 ├── distance → centroid 2
 │
 │
 └── distance → centroid 255
```

Suppose its nearest centroid is:

```text
Centroid #73
```

We now approximate:

```text
S1 ≈ Centroid #73
```

Instead of storing the original 64-dimensional subvector, store:

```text
73
```

This is the **PQ code** for `S1`.

---

# 5. Repeat for All 16 Subvectors

Suppose the nearest centroids are:

```text
S1  → centroid 73
S2  → centroid 191
S3  → centroid 22
S4  → centroid 5
...
S16 → centroid 8
```

The original vector containing:

```text
1024 float32 values
```

is now represented as:

```text
[73, 191, 22, 5, ..., 8]
```

These 16 numbers are its **PQ codes**.

They mean:

```text
S1  ≈ centroid 73  from Codebook 1
S2  ≈ centroid 191 from Codebook 2
S3  ≈ centroid 22  from Codebook 3
...
S16 ≈ centroid 8   from Codebook 16
```

The original values are no longer stored in the codes-only representation.

Instead, we store instructions for approximating the original vector using the learned codebooks.

---

# 6. Why Does This Compress So Well?

Recall:

```text
k = 256
```

Therefore each codebook contains 256 possible centroid IDs:

```text
0 ... 255
```

Since:

[
256 = 2^8
]

a centroid ID requires:

```text
8 bits = 1 byte
```

Each vector contains 16 subvectors.

Therefore:

```text
16 subvectors
×
1 byte per centroid ID
=
16 bytes/vector
```

Compare that with the original vector.

## Before PQ

```text
1024 dimensions
×
4 bytes per float32
=
4096 bytes/vector
```

## After PQ

```text
16 centroid IDs
×
1 byte
=
16 bytes/vector
```

Therefore:

[
4096 / 16 = 256
]

The PQ codes are:

```text
256× smaller
```

than the original vectors in this example.

---

# 7. Back to the 10-Million-Vector Example

Originally:

[
10,000,000 \times 1024 \times 4
]

gives:

```text
40.96 GB
```

After PQ:

[
10,000,000 \times 16 \times 1
]

gives:

```text
160,000,000 bytes
≈ 160 MB
```

Conceptually:

```text
10 million
1024-D
float32 vectors

        ↓

     40.96 GB

        ↓

       PQ
 m=16, k=256

        ↓

10 million
× 16 centroid IDs
× 1 byte

        ↓

      ~160 MB
```

That is approximately:

```text
256× compression
```

for the stored vector codes.

The codebooks themselves also consume memory, but they are small compared with storing millions of full vectors.

---

# 8. But How Do We Search Compressed Vectors?

This is the other half of Product Quantization.

Suppose a query arrives.

The embedding model produces the normal full-precision query vector:

```text
Query

1024-D float32 vector
```

An important point:

> **The query does not normally need to be converted into PQ codes.**

Instead, we keep the query at full precision.

This is called:

> **Asymmetric Distance Computation — ADC**

Why asymmetric?

Because the two sides have different representations:

```text
Query                    Database

Full precision           PQ compressed
1024-D vector             16 centroid IDs
```

---

# 9. Split the Query into the Same Subspaces

The query is divided exactly like the database vectors:

```text
Query 1024-D vector

┌────────┬────────┬────────┬────── ... ──────┬────────┐
│   Q1   │   Q2   │   Q3   │                 │  Q16   │
│  64-D  │  64-D  │  64-D  │                 │  64-D  │
└────────┴────────┴────────┴────── ... ──────┴────────┘
```

So:

```text
Query
 ↓
16 × 64-D subvectors
```

But unlike database vectors, these query subvectors are **not replaced by centroid IDs**.

---

# 10. Build a Distance Lookup Table for the Query

Take `Q1`.

Codebook 1 already contains:

```text
256 centroids
```

Calculate the distance between `Q1` and every centroid in Codebook 1:

```text
Q1 → centroid 0   = distance
Q1 → centroid 1   = distance
Q1 → centroid 2   = distance
...
Q1 → centroid 255 = distance
```

This gives:

```text
256 distances
```

Do the same for `Q2` using Codebook 2:

```text
Q2 → centroid 0
Q2 → centroid 1
...
Q2 → centroid 255
```

Again:

```text
256 distances
```

Repeat for all 16 query subvectors.

The result is:

```text
16 subspaces
×
256 centroid distances
=
4096 distances
```

This is the **per-query distance lookup table**.

Conceptually:

```text
                 Centroid IDs

             0    1    2    ...   255

Q1          d10  d11  d12   ...   d1,255
Q2          d20  d21  d22   ...   d2,255
Q3          d30  d31  d32   ...   d3,255
...
Q16         ...                   ...
```

If each distance is stored as float32:

[
4096 \times 4 = 16,384\text{ bytes}
]

or:

```text
16 KB
```

per query.

This table is computed **once when the query arrives**.

It is then reused while scoring database vectors.

---

# 11. Score a Compressed Database Vector

Recall one of our compressed database vectors:

```text
[73, 191, 22, 5, ..., 8]
```

This tells us:

```text
Subspace 1  → centroid 73
Subspace 2  → centroid 191
Subspace 3  → centroid 22
Subspace 4  → centroid 5
...
Subspace 16 → centroid 8
```

Now use those IDs to access the query's distance table.

```text
Lookup:

Q1  → centroid 73
Q2  → centroid 191
Q3  → centroid 22
Q4  → centroid 5
...
Q16 → centroid 8
```

Suppose those distances are:

```text
0.12
0.31
0.08
0.21
...
0.14
```

Add them:

```text
0.12
+ 0.31
+ 0.08
+ 0.21
...
+ 0.14

    ↓

Approximate distance between
the query and database vector
```

The database vector never needs to be reconstructed into its original 1024 floating-point values just to perform this scoring.

---

# 12. Why 16 Lookups and 15 Additions?

The database vector contains:

```text
16 PQ codes
```

Each code selects one distance from the lookup table.

Therefore scoring one database vector requires:

```text
16 table lookups
```

Those 16 distance values must then be summed.

Adding 16 numbers requires:

```text
15 additions
```

Therefore:

> **16 lookups, 15 additions per database vector**

after the per-query lookup table has been constructed.

This is one reason PQ can make large-scale vector search extremely efficient.

---

# 13. Why Keep the Query at Full Precision?

We compressed the database because there might be:

```text
10,000,000
```

database vectors.

But at search time there may be only:

```text
1 query vector
```

Compressing 10 million vectors saves enormous amounts of memory.

Compressing the single query saves almost nothing.

Worse, quantizing the query would throw away additional information.

So we keep:

```text
Query
 ↓
Full precision
```

while using:

```text
Database
 ↓
Compressed PQ representation
```

Hence:

```text
              ASYMMETRIC

Query                       Database

Full precision              Compressed
1024-D                      PQ codes
   │                           │
   │                           │
   └───────────┬───────────────┘
               │
               ▼
       Approximate distance
```

This is **Asymmetric Distance Computation**.

---

# 14. What Was Precomputed and What Happens at Query Time?

This distinction makes the whole process easier to remember.

## Index / Training Time

Do this ahead of time:

```text
Database vectors
       ↓
Split into 16 subvectors
       ↓
Run K-means independently
for each subspace
       ↓
Learn 16 codebooks
       ↓
256 centroids/codebook
       ↓
Map every database subvector
to nearest centroid
       ↓
Store PQ codes
```

So by search time:

```text
Database vector
=
[73, 191, 22, ..., 8]
```

is already available.

---

## Query Time

Now:

```text
Query
  ↓
Embedding
  ↓
1024-D full-precision vector
  ↓
Split into 16 subvectors
  ↓
For each query subvector,
calculate distance to
256 corresponding centroids
  ↓
16 × 256 distance table
  ↓
4096 distances
```

Then for each candidate database vector:

```text
Read its 16 PQ codes
        ↓
16 distance-table lookups
        ↓
15 additions
        ↓
Approximate distance
```

---

# 15. The Complete PQ Lifecycle

Putting everything together:

```text
                    INDEXING / TRAINING
                           │
                           ▼
              10 million 1024-D vectors
                           │
                           ▼
                  Split each into
                  16 × 64-D subvectors
                           │
                           ▼
             K-means per subvector position
                           │
                           ▼
                16 × 256 centroids
                           │
                           ▼
                    16 codebooks
                           │
                           ▼
              Map database subvectors
               to nearest centroids
                           │
                           ▼
                    Store IDs only
                           │
                           ▼
                   16 bytes/vector
                           │
                           ▼
                      ~160 MB


                       QUERY TIME
                           │
                           ▼
                    Query embedding
                           │
                           ▼
                 1024-D float32 vector
                           │
                           ▼
                    Split into 16
                           │
                           ▼
             Distance from each query
              subvector to its 256
                   codebook centroids
                           │
                           ▼
               16 × 256 lookup table
                           │
                           ▼
                   4096 distances
                           │
                           ▼
               Database PQ code
              [73,191,22,...,8]
                           │
                           ▼
                    16 lookups
                           │
                           ▼
                    15 additions
                           │
                           ▼
              Approximate Q ↔ D distance
```

---

# 16. What Did We Sacrifice?

PQ isn't lossless compression.

Originally:

```text
S1 = exact 64 floating-point values
```

After PQ:

```text
S1 ≈ centroid #73
```

Many different original subvectors can map to the same centroid.

That means some information has been discarded.

This is **quantization error**.

Therefore:

```text
Full-precision vectors
        ↓
More memory
        ↓
More precise distances


PQ-compressed vectors
        ↓
Much less memory
        ↓
Approximate distances
        ↓
Possible retrieval recall loss
```

The more aggressively vectors are compressed, the greater the potential distortion.

So PQ represents a classic systems trade-off:

> **Memory + search efficiency vs vector precision + retrieval recall**

---

# 17. PQ vs IVF — Don't Mix Them Up

These two ideas are easy to confuse because they are often used together.

### IVF asks:

> **Where should I search?**

It partitions the vector space into regions.

```text
Query
 ↓
Find promising IVF clusters
 ↓
Search only vectors in those clusters
```

It reduces the **search space**.

### PQ asks:

> **How can I store and score those vectors cheaply?**

```text
Full vector
 ↓
Subvectors
 ↓
Centroid IDs
 ↓
Compact representation
```

It reduces the **storage and distance-computation cost**.

Therefore:

```text
IVF
 ↓
Reduce WHERE we search


PQ
 ↓
Reduce WHAT we store
and make distance scoring cheap
```

And they can be combined:

```text
IVF
 ↓
Find promising region
 ↓
PQ
 ↓
Cheaply score compressed vectors
 ↓
Top candidates
```

This gives architectures such as **IVF-PQ**.

---

# 18. The Four Numbers Worth Remembering

For:

```text
10 million vectors
1024 dimensions
float32
m = 16
k = 256
```

### Original storage

[
10M \times 1024 \times 4
]

```text
≈ 40.96 GB
```

### PQ storage — codes only

[
10M \times 16 \times 1
]

```text
≈ 160 MB
```

### Per-query lookup table

[
16 \times 256
]

```text
4096 distance entries
```

With float32 distances:

```text
≈ 16 KB
```

### Scoring one database vector

```text
16 lookups
15 additions
```

---

# 19. The Mental Model

If all the implementation details are forgotten later, remember this:

```text
                    DATABASE

1024-D float32 vector
         ↓
Split into 16 pieces
         ↓
Each piece is 64-D
         ↓
K-means learned 256
representatives for each position
         ↓
Find nearest representative
for each piece
         ↓
Store its ID
         ↓
[73,191,22,...,8]
         ↓
16 bytes instead of 4096 bytes


                     QUERY

1024-D full-precision vector
         ↓
Split into same 16 pieces
         ↓
Compare each piece against
its 256 learned centroids
         ↓
Build distance lookup table
         ↓
Use DB's centroid IDs
to select distances
         ↓
Add 16 distances
         ↓
Approximate query ↔ document distance
```

Or in two sentences:

> **At indexing time, PQ splits database vectors into subvectors, learns representative centroids for each subspace using K-means, and replaces each database subvector with the ID of its nearest centroid.**

> **At query time, the query remains full precision; its distances to the learned centroids are calculated once, and the stored PQ codes are then used as cheap lookup-table indices to approximate the distance to each database vector.**

That is Product Quantization end to end.
