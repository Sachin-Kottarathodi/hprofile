---
title: "Dynamic Programming"
date: 2026-02-05
draft: false
tags:
  - algorithms
  - dynamic-programming
categories:
  - tech
---

> **AI polished. Not AI fabricated.**
>
> These notes are based primarily on the MIT 15.053 Dynamic Programming notes by Dimitris Bertsimas and John Tsitsiklis. The ideas and examples reflect my own study and understanding. AI was used to improve organization and presentation.

# A Framework for Dynamic Programming

## Reference

This note is based primarily on:

* Dimitris Bertsimas and John Tsitsiklis, *Introduction to Linear Optimization*, Chapter 11: Dynamic Programming.
* MIT 15.053 Tutorial: Dynamic Programming.

Reference:

https://web.mit.edu/15.053/www/AMP-Chapter-11.pdf

---

## Overview

Dynamic programming is an optimization strategy that transforms a complex problem into a sequence of simpler problems through a multistage procedure.

Rather than being a single algorithm, it provides a general framework applicable to many problem types. Solving a problem with dynamic programming often requires reformulating it so that it can be expressed in terms of stages, states, and transitions.

The fundamental idea is to solve a problem in stages and build toward an overall optimal solution.

---

## Bellman's Principle of Optimality

The central principle of dynamic programming is:

> An optimal policy has the property that, whatever the initial state and decision are, the remaining decisions must constitute an optimal policy for the state resulting from the first decision.

An optimal solution therefore consists of optimal solutions to its remaining subproblems.

---

## Stages

A problem is decomposed into a sequence of stages.

Stages may represent:

* Time periods
* Array indices
* Physical locations
* Items under consideration

Examples:

| Problem      | Stage         |
| ------------ | ------------- |
| House Robber | House index   |
| Knapsack     | Item index    |
| Unique Paths | Grid cell     |
| Coin Change  | Amount        |
| LIS          | Current index |

---

## State

The state is the minimum amount of information required to evaluate future decisions without needing to know how the process arrived there.

A state should satisfy the following properties.

### Path Independence

Future decisions should not depend on the path used to reach the current state.

The state should summarize all relevant history.

### Completeness

The state must contain enough information to evaluate future consequences.

### Transition Linkage

The state must naturally evolve from one stage to the next through some transition function.

---

## State and Value Function

In a dynamic programming table, the indices represent the state.

The stored value represents the value function.

Examples:

### House Robber

State:

```
i
```

Value:

```
Maximum money obtainable
```

### Knapsack

State:

```
(item, capacity)
```

Value:

```
Maximum profit
```

### Coin Change

State:

```
amount
```

Value:

```
Minimum number of coins
```

---

## Patterns in States

### Physical Location

Common in graph and path problems.

Examples:

* Current node
* Grid position

### Resource Availability

Common in allocation problems.

Examples:

* Remaining capacity
* Remaining budget

### Inventory Levels

Common in scheduling and logistics.

Examples:

* Current stock
* Remaining inventory

### Cumulative Progress

Common in expansion problems.

Examples:

* Prefix length
* Amount completed
* Number of items processed

---

## Transition Patterns

Transitions describe how a decision moves the process from one state to another.

### Consumption Pattern

```text
nextState = currentState - decision
```

Examples:

* Knapsack
* Coin Change

### Balance Pattern

```text
nextState = currentState + supply - demand
```

Examples:

* Inventory problems
* Production planning

### Discrete Movement

```text
nextState = currentState ± 1
```

Examples:

* Grid problems
* Path problems

---

## Recursive Optimization

Dynamic programming connects the value of the current state to the value of future states.

General form:

```text
Vn(sn) = max [ fn(dn, sn) + Vn-1(sn-1) ]
```

where:

* `sn` is the current state.
* `dn` is the decision.
* `fn` is the immediate reward.
* `Vn-1` is the value of the next stage.

---

## Redundant State Variables

A state should contain only information that affects:

* Future decisions.
* Future rewards.
* State transitions.

Information that does not influence these factors is redundant.

### House Robber

Poor state:

```text
(index, robbedPrevious)
```

Simplified state:

```text
index
```

### Coin Change

Poor state:

```text
(amount, coinsUsedSoFar)
```

Simplified state:

```text
amount
```

### LIS

Poor state:

```text
(index, path)
```

Simplified state:

```text
index
```

---

## Forward and Backward Induction

### Forward Induction

Forward induction starts from the initial state and proceeds toward the destination.

It is particularly useful when there is a fixed starting point and multiple possible destinations.

In deterministic problems, forward induction is often straightforward.

### Backward Induction

Backward induction starts from the destination and works backward toward the origin.

It is especially important when uncertainty is present because future outcomes must be evaluated before decisions can be made optimally.

Backward induction forms the basis of:

* Markov Decision Processes
* Stochastic Dynamic Programming
* Reinforcement Learning
* Optimal Control

---

## Deterministic versus Stochastic Problems

In deterministic problems, a decision leads to a known next state.

Both forward and backward induction are generally possible.

In stochastic problems, a decision leads to uncertain outcomes.

Backward induction becomes necessary because future values must be evaluated before making current decisions.

---

## Recursion Before Tabulation

Many dynamic programming problems are easier to understand recursively.

The recursive formulation naturally exposes:

* States
* Choices
* Transitions
* Base cases

Repeated subproblems motivate memoization.

Bottom-up dynamic programming is often a tabular form of the same recurrence.

---

## General Procedure

1. Identify the stages.
2. Define the state.
3. Determine the transitions.
4. Write the recursive relationship.
5. Specify the base case.
6. Compute the value function.
7. Recover the optimal policy if necessary.

---

## Summary

Dynamic programming can be viewed as a method for compressing history.

The state retains all information necessary for future decisions and discards information that no longer affects the problem.

The main challenge in dynamic programming is usually not writing the recurrence. The more difficult task is identifying the appropriate state representation.

---

> **AI polished. Not AI fabricated.**
>
> Reference:
>
> *Dimitris Bertsimas and John Tsitsiklis, MIT 15.053 — Dynamic Programming*
>
> https://web.mit.edu/15.053/www/AMP-Chapter-11.pdf
