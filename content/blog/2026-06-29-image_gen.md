---
title: "How Image Generation Actually Works"
date: 2026-06-29
draft: false
tags:
  - tech
  - genai
categories:
  - tech
---
# Fom Pixels to Diffusion: How Image Generation Actually Works

> This post began as scattered notes and questions while trying to understand each of these topics in more detail. The deeper the questions went, the more the concepts started connecting, but the notes themselves remained fragmented. AI was used to piece those fragments together into a coherent sequence. The result is a connected mental model of the topics discussed. It is intentionally simplified, as the goal at this stage is to understand the core ideas without getting lost in the deeper mathematical and implementation details.


---

I started learning language models from the text-generation side.

Tokens. Embeddings. Transformers. Attention. Q, K, V. Autoregressive generation. Decoder-only models.

Then I moved into image generation and initially tried to map everything directly onto what I already knew.

That immediately created a problem.

If text generation is essentially:

```text
token → token → token → token → ...
```

why can't image generation do the same thing?

An image is just data.

A pixel has values.

So perhaps:

```text
pixel → pixel → pixel → pixel → ...
```

and a sufficiently large Transformer could generate an image.

Technically, it could.

Practically, this becomes a terrible way to represent the problem.

And understanding why eventually leads to diffusion models.

---

# 1. Start with the image itself

A normal RGB image can be represented as:

```text
Height × Width × 3
```

The `3` represents:

```text
R = Red
G = Green
B = Blue
```

For example:

```text
512 × 512 × 3
```

contains:

```text
512 × 512 × 3 = 786,432
```

individual numerical values.

A naive idea would be to treat each pixel, or some representation of each pixel, as a token in a sequence.

Then image generation becomes something like:

```text
pixel 1
   ↓
pixel 2
   ↓
pixel 3
   ↓
...
   ↓
pixel 786,432
```

This is theoretically possible.

But there are two immediate problems.

First, the sequence becomes enormous.

Second, images don't naturally have the same sequential structure as text.

---

# 2. Why text is naturally suited to autoregression

Consider:

> The cat sat on the

The next token is constrained by what came before:

```text
The → cat → sat → on → the → ???
```

An autoregressive model predicts:

```text
P(next token | previous tokens)
```

Then it generates the next token and feeds it back into the model.

```text
Prompt
  ↓
predict token 1
  ↓
append token 1
  ↓
predict token 2
  ↓
append token 2
  ↓
...
```

This is sequential, but language itself has a natural sequential structure.

An image doesn't.

Consider:

```text
┌───────────────────────┐
│                       │
│       🐱              │
│          🏠           │
│                       │
└───────────────────────┘
```

There is no obvious reason why the model should first generate the top-left pixel, then the next pixel, then the next.

The entire image has spatial relationships.

The pixels all influence one another.

So forcing an image into a gigantic left-to-right sequence is possible, but inefficient.

---

# 3. Could we just use a Transformer anyway?

Yes.

This is an important distinction.

The problem isn't that a Transformer *cannot* generate images.

The problem is that the representation would be extremely expensive.

Imagine treating every pixel as a token:

```text
Image
 ↓
786,432 positions
 ↓
Transformer
 ↓
generate one position at a time
```

Now the model has an enormous sequence.

And if generation is autoregressive:

```text
1 → 2 → 3 → 4 → ... → 786,432
```

you have an enormous number of sequential generation steps.

So the problem is not fundamentally:

> "Transformers can't generate images."

It is:

> **"This is a very inefficient representation and generation strategy for images."**

That distinction matters.

---

# 4. The idea of learning a better representation

This leads to a much more important idea.

Instead of operating directly on the raw image, can we find a **compact representation** of the image?

Something like:

```text
Image
  ↓
Encoder
  ↓
Compact representation
  ↓
Do expensive computation here
  ↓
Decoder
  ↓
Image
```

This is the basic intuition behind latent representations.

The model learns a smaller space that preserves the information necessary to reconstruct the image.

For example, conceptually, an image might go from:

```text
512 × 512 × 3
```

to something more like:

```text
64 × 64 × 4
```

The exact dimensions depend on the model, but the important point is the enormous reduction in the number of values.

Notice something interesting here:

```text
512 × 512 × 3 = 786,432
64 × 64 × 4  = 16,384
```

So even though the latent representation has **4 channels rather than 3**, it has dramatically fewer total values because the spatial dimensions have been compressed.

It is not simply:

> 3 values per pixel become 4 values per pixel.

It is:

> **A large spatial representation is compressed into a much smaller learned representation.**

---

# 5. The encoder becomes important

Now there is an obvious consequence.

If the encoder produces a bad representation:

```text
Good image
    ↓
Bad encoder
    ↓
Bad latent representation
    ↓
Diffusion
    ↓
Decoder
    ↓
Bad image
```

The diffusion model cannot magically recover information that the encoder threw away.

So the encoder has an important job:

> **Compress the image while preserving the information that matters for reconstruction and generation.**

The decoder has the reverse job:

> **Turn the compact latent representation back into an image.**

This gives us an important separation of responsibilities:

```text
Encoder / Decoder
    ↓
How do I represent an image compactly?

Diffusion model
    ↓
How do I generate good representations in that space?
```

That is the basic architectural intuition behind **latent diffusion**.

---

# 6. Diffusion starts with a strange idea

Now comes the part that initially seemed almost absurd.

Instead of trying to directly learn:

```text
random noise → beautiful image
```

we can construct a process that goes in the opposite direction first.

Take a real image.

Add a little noise.

Then more.

Then more.

Eventually:

```text
Real image
   ↓
slightly noisy
   ↓
more noisy
   ↓
very noisy
   ↓
almost pure noise
   ↓
pure noise
```

This is the **forward diffusion process**.

The remarkable idea is:

> If we know how an image was progressively destroyed by noise, can a neural network learn how to reverse that process?

That gives us:

```text
PURE NOISE
    ↓
remove noise
    ↓
remove noise
    ↓
remove noise
    ↓
...
    ↓
IMAGE
```

This is the fundamental intuition behind diffusion-based generation.

---

# 7. Why call it diffusion?

The name comes from the connection to physical diffusion processes.

In physics, a system can gradually move from a structured state toward a more random/disordered state.

Diffusion models borrow this idea mathematically.

We deliberately take structured data and gradually corrupt it with noise.

Then we learn the reverse process.

The remarkable part is not merely that noise can be removed.

It is that by learning the reverse process across a huge amount of data, the model learns something about the **distribution of the data itself**.

---

# 8. From density models to score-based thinking

This connects to an earlier question about classical density models.

A density model tries to describe:

> **Where is data likely to occur?**

Imagine all possible images as an enormous space.

Most arbitrary points in that space are nonsense.

The space of realistic images occupies some complicated region:

```text
All possible representations

      .       .
  .       ███████
      █████████████
 .   ███████████████
     █████████████
       ███████
  .                 .
       .
```

The model wants to understand the structure of this distribution.

Classical probabilistic models often try to explicitly model the probability density:

[
p(x)
]

But for complicated high-dimensional distributions, directly calculating and normalizing these probabilities can become extremely difficult.

This is where the **score function** becomes useful.

The score is essentially related to:

[
\nabla_x \log p(x)
]

Informally:

> **Which direction should I move this point to get toward a region of higher probability?**

So instead of explicitly asking:

> "What is the exact probability of this image?"

we can ask something more local:

> **"In which direction should I move this noisy representation to make it more like something from the data distribution?"**

This idea is deeply connected to modern score-based generative modeling and diffusion.

---

# 9. DDPM — putting the idea into a trainable model

One of the foundational formulations is:

**DDPM — Denoising Diffusion Probabilistic Model.**

The training idea is surprisingly simple.

Take a real image:

```text
x₀
```

Add a known amount of noise:

```text
xₜ
```

The model receives the noisy image and the timestep.

It is trained to predict the noise that was added.

Conceptually:

```text
Original image
      +
Known random noise
      ↓
Noisy image
      ↓
Neural network
      ↓
"What noise is present?"
      ↓
Predicted noise
      ↓
Compare with actual noise
      ↓
Loss
      ↓
Backpropagation
```

A simplified PyTorch-style training loop might look conceptually like:

```python
noise = torch.randn_like(x)

noisy_x = add_noise(x, noise, t)

predicted_noise = model(noisy_x, t)

loss = mse_loss(predicted_noise, noise)

loss.backward()
optimizer.step()
```

The important thing isn't memorizing the PyTorch code.

The important thing is understanding what it means:

> **The network learns to recognize and predict noise at different levels of corruption.**

---

# 10. What are beta, alpha and alpha-bar?

The implementation introduces variables such as:

```text
β (beta)
α (alpha)
ᾱ (alpha-bar)
```

They describe how the noise schedule evolves.

The useful mental model is:

```text
βₜ
 ↓
How much new noise is introduced at this timestep?

αₜ
 ↓
How much signal is retained at this timestep?

ᾱₜ
 ↓
How much of the original signal remains
after accumulating the process up to this timestep?
```

A common relationship is:

[
\alpha_t = 1-\beta_t
]

And α-bar is the cumulative product of the α values up to that point.

The exact equations matter when implementing DDPM, but at this stage the conceptual meaning is more important:

> **They control the balance between original signal and accumulated noise.**

So:

```text
t = early
→ mostly image

t = middle
→ image + substantial noise

t = late
→ mostly noise
```

The model learns to operate across this entire range.

---

# 11. But what neural network actually removes the noise?

Enter the **U-Net**.

The name is literal: the architecture roughly looks like a U.

```text
                 ┌───────────────┐
                 │               │
Input ──→ ↓ ──→ ↓ ──→ bottleneck ──→ ↑ ──→ ↑ ──→ Output
          │       │                │       │
          └───────┼────────────────┘       │
                  │
             skip connections
```

It has two broad sides.

### Downsampling path

The network progressively compresses the spatial representation.

It can learn increasingly abstract features:

```text
edges
 ↓
shapes
 ↓
parts
 ↓
objects / structure
```

### Upsampling path

It reconstructs a representation at higher spatial resolution.

### Skip connections

The network connects corresponding levels of the downsampling and upsampling paths.

This allows fine-grained spatial information to be carried forward.

That matters because image generation requires both:

```text
Global information:
"This is a face."

Local information:
"An edge exists exactly here."
```

The U-Net gives the network a way to work with both.

---

# 12. Diffusion generation

Once the network has learned the denoising process, generation starts from random noise.

```text
Random noise
     ↓
U-Net predicts noise
     ↓
Remove some noise
     ↓
U-Net predicts noise
     ↓
Remove some noise
     ↓
...
     ↓
Image
```

It is not:

> "The model knows the final image and simply reveals it."

Rather, each step makes the representation more consistent with the learned data distribution.

A rough image gradually emerges.

---

# 13. Why not do this directly on pixels?

We could.

That is called **pixel-space diffusion**.

But high-resolution images contain enormous numbers of values.

So instead of:

```text
Image
 ↓
diffusion over millions of pixel values
```

we can do:

```text
Image
 ↓
Encoder
 ↓
Latent representation
 ↓
Diffusion
 ↓
Latent representation
 ↓
Decoder
 ↓
Image
```

This is **latent diffusion**.

It is much more computationally practical.

---

# 14. Stable Diffusion and Latent Diffusion

This distinction is important.

**Diffusion model** is the broad family.

**Latent diffusion** is the technique of performing diffusion in a learned compressed representation rather than directly in pixel space.

**Stable Diffusion** is a well-known family of models built around latent diffusion.

So:

```text
Diffusion
    │
    ├── Pixel-space diffusion
    │
    └── Latent diffusion
             │
             └── Stable Diffusion family
```

The basic Stable Diffusion mental model is therefore:

```text
Text prompt
     ↓
Text encoder
     ↓
Text representation
     
Image
  ↓
Image encoder
  ↓
Latent representation
  ↓
Diffusion / U-Net
  ↓
Latent representation
  ↓
Image decoder
  ↓
Generated image
```

The exact architecture of modern image-generation systems varies, but this is the useful conceptual foundation.

---

# 15. Where does the text prompt enter?

Now the next question becomes interesting.

Suppose the prompt is:

> **"A red car driving through snow."**

The diffusion model is operating on an image representation.

But it needs to know what image it should generate.

So there are two representations:

```text
TEXT

"A red car driving through snow"
              ↓
       text representation


IMAGE

noisy latent representation
              ↓
         image features
```

We need a mechanism to connect them.

That mechanism is **cross-attention**.

---

# 16. First understand attention in text

In a text Transformer, each token produces:

```text
Q = Query
K = Key
V = Value
```

A useful mental model:

```text
Q = "What information am I looking for?"

K = "What kind of information do I contain?"

V = "Here is the actual information."
```

The model compares Queries against Keys:

```text
Q × K
   ↓
relevance
   ↓
attention weights
   ↓
weighted Values
```

So the core operation is approximately:

[
Attention(Q,K,V)
================

softmax
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)V
]

In self-attention:

```text
Q ← text
K ← text
V ← text
```

The text looks at itself.

---

# 17. Self-attention in images

Images can also use self-attention.

Now the representations are image features:

```text
Image feature 1
Image feature 2
Image feature 3
...
```

Each feature can ask:

> "Which other image features are relevant to me?"

So:

```text
Q ← image
K ← image
V ← image
```

This allows distant parts of the image to interact.

For example, information about one part of an object can influence another part.

---

# 18. Cross-attention connects image and text

Now we have something different.

The image representation has Queries.

The text representation provides Keys and Values.

Conceptually:

```text
                 TEXT
                  │
             K + V
                  │
                  ↓
             Cross-attention
                  ↑
                  │
                  Q
                IMAGE
```

Or:

```text
Q ← image
K ← text
V ← text
```

The image representation is effectively asking:

> **"Which parts of the text are relevant to what I'm currently generating?"**

For example:

```text
Prompt:

"A red car driving through snow"
```

An image feature corresponding to the car might strongly attend to:

```text
red
car
```

Another feature might attend more strongly to:

```text
driving
snow
```

This provides the bridge between language and image generation.

---

# 19. Why Q, K and V are useful here

Think of a library.

```text
Query:
"What information am I looking for?"

Key:
"What topic does this information belong to?"

Value:
"Here is the actual information."
```

For cross-attention:

```text
Image feature
    ↓
Query
    ↓
compare against
    ↓
Text Keys
    ↓
determine relevance
    ↓
retrieve weighted Text Values
    ↓
update image representation
```

So cross-attention is not merely "mixing text and image."

It is giving the image-generation process a mechanism to **select relevant information from the text representation**.

---

# 20. Why not simply concatenate text and image?

Because the two modalities are different.

You have:

```text
Text:
[token representations...]

Image:
[visual representations...]
```

Cross-attention gives the model a structured way to say:

> "For this visual feature, retrieve the relevant information from the text."

That is much more useful than simply putting the two sequences next to one another.

---

# 21. Encoder vs decoder — a useful detour from text generation

At this point, image generation brings us back to a distinction from language models.

There are three broad Transformer architectures:

```text
Encoder-only
    ↓
BERT

Decoder-only
    ↓
GPT / Llama / Mistral / StarCoder

Encoder-decoder
    ↓
T5 / FLAN-T5
```

### Encoder-only

BERT primarily exists to **understand** input.

It can use bidirectional attention:

```text
token ←→ token ←→ token ←→ token
```

A token can see context on both sides.

### Decoder-only

GPT-style models are designed for autoregressive generation:

```text
token 1
  ↓
token 2
  ↓
token 3
  ↓
token 4
```

Causal masking prevents a position from seeing future tokens.

### Encoder-decoder

T5 separates the two jobs:

```text
Input
  ↓
Encoder
  ↓
rich representation
  ↓
Decoder
  ↓
Output
```

The encoder understands the input.

The decoder generates the output.

---

# 22. But doesn't a decoder-only model still encode the prompt?

Absolutely.

"Decoder-only" does **not** mean:

> "There is no encoding."

There is still:

```text
Token ID
   ↓
Embedding
   ↓
Transformer layers
   ↓
Contextual representation
```

The distinction is that there is **no separate encoder network**.

The decoder layers themselves progressively construct contextual representations while maintaining causal masking.

So:

```text
Decoder-only:

Prompt
 ↓
Token embeddings
 ↓
Transformer layer
 ↓
Transformer layer
 ↓
...
 ↓
Contextual representation
 ↓
Next-token prediction
```

Whereas T5 is:

```text
Prompt
 ↓
Separate encoder
 ↓
Encoded representation
 ↓
Separate decoder
 ↓
Output
```

This distinction became important when comparing language generation with image generation.

---

# 23. Why not make the decoder bidirectional?

This led to another question.

Suppose we're generating:

```python
def add(a, b):
    ???
```

An autoregressive model must generate:

```text
return
    ↓
a
    ↓
+
    ↓
b
```

One token at a time.

But what if the model could see both sides?

If the complete sequence were already:

```python
def add(a, b):
    return a + b
```

then a bidirectional model could allow:

```text
def ←→ add ←→ (...) ←→ return ←→ a ←→ + ←→ b
```

Every token could see every other token.

This provides richer contextual information.

So why not always generate this way?

Because when generating from scratch, the future does not exist yet.

If we have:

```python
def add(a, b):
    [MASK]
```

there is no future content to look at.

---

# 24. But bidirectional generation can be useful

Suppose we aren't generating an entire sequence from nothing.

Suppose we have:

```python
def calculate_total(items):
    total = 0

    for item in items:
        [MASK]

    return total
```

Now the model has context **before and after the missing piece**.

It can use both:

```text
BEFORE
    ↓
for item in items:
    [MASK]
return total
    ↑
AFTER
```

This is a much more constrained problem.

The model doesn't have to invent the entire program.

It needs to fill a hole.

This is why bidirectional/masked approaches are useful for things like:

* fill-in-the-blank
* code infilling
* text editing
* rewriting
* some translation approaches

The more structure already exists, the more useful bidirectional context can become.

---

# 25. The fundamental trade-off

This creates a useful comparison.

### Autoregressive generation

```text
Prompt
  ↓
token 1
  ↓
token 2
  ↓
token 3
  ↓
token 4
  ↓
...
```

Each decision becomes context for the next.

Advantages:

* naturally coherent sequential generation
* excellent for open-ended generation
* simple training objective
* each decision conditions the next

Disadvantage:

* inherently sequential generation
* latency grows with output length

### Bidirectional / non-autoregressive-style generation

```text
Existing context
       ↓
[MASK] [MASK] [MASK] [MASK]
       ↓
generate/refine in parallel
```

Potential advantage:

* many positions can be processed together
* potentially much faster

But:

* simultaneously generated tokens have weaker mutual conditioning
* maintaining global semantic consistency becomes harder
* generation requires a different procedure

This is one reason autoregressive language generation remains so powerful despite its sequential nature.

---

# 26. And this explains something about images

Images don't have the same obvious left-to-right sequential structure as text.

Imagine:

```text
[ ][ ][ ][ ]
[ ][ ][ ][ ]
[ ][ ][ ][ ]
[ ][ ][ ][ ]
```

It is perfectly reasonable for all regions to influence one another.

A diffusion model doesn't have to say:

```text
region 1
  ↓
region 2
  ↓
region 3
  ↓
...
```

Instead, it can iteratively refine the **whole representation**.

```text
Noise
 ↓
rough global structure
 ↓
better structure
 ↓
better details
 ↓
final image
```

This is one of the deep reasons diffusion is such a natural approach for image generation.

---

# 27. Diffusion is not simply "bidirectional text generation"

This distinction is important.

Diffusion does **not** mean:

> "We made a Transformer bidirectional and now it generates images."

The underlying generation mechanism is different.

Diffusion starts with noise and performs a sequence of denoising/refinement steps.

The model learns a transformation related to the direction in which the data distribution becomes more likely.

So:

```text
Autoregressive language:

previous sequence
      ↓
next token
      ↓
next token
      ↓
next token
```

versus:

```text
Diffusion:

random noise
      ↓
denoise/refine
      ↓
denoise/refine
      ↓
denoise/refine
      ↓
image
```

Both are generative, but they exploit very different structures.

---

# 28. Why latent diffusion makes the whole thing practical

Now combine everything.

Raw pixel-space diffusion:

```text
Image
 ↓
millions of pixel values
 ↓
diffusion
 ↓
millions of pixel values
```

Latent diffusion:

```text
Image
 ↓
Encoder
 ↓
small latent representation
 ↓
Diffusion / U-Net
 ↓
small latent representation
 ↓
Decoder
 ↓
Image
```

The expensive part happens in the smaller space.

And the text prompt can condition the process through cross-attention:

```text
                         TEXT
                          ↓
                    Text Encoder
                          ↓
                       K + V
                          │
                          │
                          ↓
                    Cross-Attention
                          ↑
                          │
                       Q
                          ↑
Image → Image Encoder → Latent → U-Net
                                      ↓
                                  denoising
                                      ↓
                                   Latent
                                      ↓
                                  Decoder
                                      ↓
                                    Image
```

This is the mental model I was looking for.

---

# 29. Putting the entire pipeline together

A simplified text-to-image generation pipeline now looks like this:

```text
                TEXT PROMPT
                     │
                     ▼
              Tokenization
                     │
                     ▼
               Text Encoder
                     │
              Text representation
                     │
                K + V
                     │
                     │
                     ▼
Random noise → Latent representation
                     │
                     ▼
                   U-Net
                     │
              Cross-attention
                     ↑
                     │
             Text representation
                     │
                     ▼
             Predict / remove noise
                     │
                     ▼
          Repeat denoising steps
                     │
                     ▼
             Final latent
                     │
                     ▼
                Image Decoder
                     │
                     ▼
               Generated Image
```

There are additional components and implementation details in real systems, but this is enough to connect the major pieces.

---

# 30. The connection back to language models

The comparison is useful because the two systems are solving related problems differently.

### Language generation

```text
Text
 ↓
Tokens
 ↓
Embeddings
 ↓
Transformer
 ↓
Causal attention
 ↓
Next-token probabilities
 ↓
Choose token
 ↓
Repeat
```

The fundamental unit is the **token**.

The model generates a sequence.

---

### Image generation

```text
Text prompt
 ↓
Text representation
 ↓
Condition diffusion process

Random latent
 ↓
U-Net / denoising network
 ↓
Cross-attention with text
 ↓
Refined latent
 ↓
Repeat
 ↓
Image decoder
 ↓
Image
```

The fundamental working representation is a **continuous latent space**, not a sequence of millions of individual RGB values.

The model doesn't have to commit to pixel 1 before pixel 2.

It can progressively refine a whole representation.

---

# 31. The surprising part

The thing that initially seemed crazy was the physical intuition behind diffusion:

```text
Image
 ↓
destroy it with noise
 ↓
pure randomness
```

Then learn:

```text
pure randomness
 ↓
recover structure
 ↓
recover more structure
 ↓
recover details
 ↓
image
```

The model isn't memorizing one image and playing it backward.

It is learning the statistical structure of the data distribution well enough that starting from random noise, repeated denoising steps can move a representation toward something that looks like a valid sample from that distribution.

That is the part that makes diffusion so fascinating.

A physical process of **adding randomness** becomes the foundation for a computational process of **learning how to reverse that randomness**.

---

# 32. The mental map to keep

I don't need to remember every DDPM coefficient, every PyTorch tensor operation, or every U-Net implementation detail.

The useful map is:

```text
IMAGE GENERATION
│
├── Raw image
│     └── pixels / RGB
│
├── Representation
│     └── Encoder → latent space
│
├── Diffusion
│     ├── add noise during training
│     └── learn to reverse / denoise
│
├── DDPM
│     └── foundational denoising formulation
│
├── U-Net
│     └── predicts the noise / denoising direction
│
├── Attention
│     ├── self-attention
│     │     └── image ↔ image
│     │
│     └── cross-attention
│           └── image ↔ text
│
├── Latent Diffusion
│     └── perform diffusion in compressed space
│
└── Stable Diffusion
      └── practical family built around latent diffusion
```

And beside it:

```text
TEXT GENERATION
│
├── Tokenizer
│     └── text → token IDs
│
├── Embedding
│     └── token IDs → vectors
│
├── Transformer
│     ├── self-attention
│     └── contextual representations
│
├── Decoder-only
│     └── no separate encoder
│
├── Causal masking
│     └── don't see future tokens
│
└── Autoregressive generation
      └── token → token → token → ...
```

The two worlds are different, but they share a lot of machinery:

```text
                TRANSFORMERS / NEURAL NETWORKS
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
           TEXT                  IMAGE
              │                     │
       tokens / embeddings     visual / latent features
              │                     │
       attention mechanisms    attention mechanisms
              │                     │
      autoregressive           diffusion
      generation               generation
              │                     │
         next token              image
```

The key realization is that **there isn't one universal way to make a generative model**.

Text happens to have a strong sequential structure, making autoregressive generation extraordinarily effective.

Images have enormous spatial structure and don't naturally want to be generated one pixel after another, making latent representations and iterative diffusion a much more practical approach.

And the deeper common idea underneath both is the same:

> **Learn a representation of the data, learn the structure of that representation, and then learn a way to generate new points that fit that learned structure.**
