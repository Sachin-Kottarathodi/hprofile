---
title: "GenAI Agents"
date: 2025-07-14
tags: ["GenAI", "LLMs", "Agents"]
Categories: ["tech"]
draft: false
---


## What is a Generative AI Agent?

- A GenAI agent is an application that **achieves goals** by **observing the world** and **acting upon it using tools**.
- It is **autonomous**, capable of acting **independently** of human intervention.
- It reasons, makes decisions, and **proactively** chooses steps to achieve its goal — even without explicit instructions.
- Focus is on the **specific type of agents** powered by **Generative AI models**.

---

## Core Components (Cognitive Architecture)

- **Model** – The foundation (e.g., LLM).
- **Orchestration** – The reasoning loop and logic.
- **Tools** – How the agent interacts with external systems.

---

## Agent Architecture – Interaction with the Outside World

### Tools:  
- **Functions**  
- **Extensions**  
- **Data Stores**  
- **Plugins**  

> These bridge the foundational model with the real world.

---

## Extensions

- Allow **direct integration** between the agent and APIs.
- Teach the agent how to call APIs using **examples**.
- Executed **on the agent side**.

---

## Functions

- Model outputs a **function name + arguments**, but **doesn’t call APIs directly**.
- Executed **on the client-side**.

### When to Use Functions Over Extensions:

- API must be called from **outside the agent architecture** (e.g., frontend/middleware).
- **Security/auth** restrictions block direct agent access to APIs.
- Need **manual review** or **batch processing**.
- Need additional **client-side transformation** of API responses.
- For **stubbing** or rapid prototyping without deploying full infrastructure.

---

## Data Stores

- Feed agents **external knowledge** without retraining or fine-tuning.
- Accept raw data (docs, files, etc.).
- Converts documents to **embeddings** using a vector DB.
- Enables **semantic search** to find and use relevant info at inference time.
