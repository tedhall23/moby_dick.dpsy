# Demonstrating DSPy Predict vs Recursive Language Models (RLM) on Long-Context Reasoning

## Overview

This repository demonstrates **why Recursive Language Models (RLMs)** are necessary for reasoning over **long documents that cannot fit into a single prompt**, and why naïve single-shot prompting (`dspy.Predict`) breaks down under real-world constraints.

The goal is **not** to show that Predict is “worse” at reasoning, but to show that it **fails operationally** when faced with:

- Context windows that are too small
- Token-per-minute (TPM) rate limits
- Large volumes of irrelevant content
- Repeated retries that amplify cost and latency

This mirrors real enterprise use cases such as:
- Regulatory filings
- News or intelligence feeds
- Large policy or legal documents
- Customer support knowledge bases

---

## What This Demo Is (and Is Not)

### This **is**:
- A realistic simulation of long-document reasoning
- A comparison between **single-shot prompting** and **recursive decomposition**
- A demonstration of **why RLM exists**, not just how to call it
- A setup that mirrors customer complaints like:
  > “The answer is in there, but the model keeps failing or rate limiting”

### This **is not**:
- A benchmark of model intelligence
- A claim that Predict is “bad”
- A token-stuffing trick

---

## Architectural Comparison

### 1. `dspy.Predict` (Single-Shot Prompting)

**Mental model**:
> “Put everything in one prompt and ask the question.”

**Characteristics**:
- One LLM call
- Entire document must fit in the context window
- Sensitive to:
  - Prompt length
  - Rate limits
  - Retry amplification
- Works well **only when the document is small**

**Failure modes**:
- Prompt exceeds context window
- TPM rate limiting when repeatedly retried
- Latency spikes
- Cost grows superlinearly with document size

---

### 2. Recursive Language Models (RLM)

**Mental model**:
> “Read the document in pieces, reason locally, then compose the answer.”

**Key idea**:
RLM decomposes a long document into manageable chunks and performs **bounded reasoning at each step**, recursively aggregating results.

**Characteristics**:
- Multiple smaller LLM calls
- Each call fits comfortably in context
- Natural backpressure against rate limits
- Deterministic control over:
  - Chunk size
  - Reasoning depth
  - Cost per step

---

## Why QA Is a Trap (and Still Useful)

In toy examples, **Predict often “wins”** because:
- The answer exists verbatim
- Models are excellent at pattern matching
- The document is still small enough to fit

This demo intentionally shows that:
> **QA accuracy is not the differentiator — operational robustness is.**

The moment you scale document length:
- Predict fails *before* reasoning even begins
- RLM continues to function predictably

---

## Synthetic Long-Document Setup

We construct a document as a **list of sections**:

- Thousands of irrelevant “filler” sections
- A small number of semantically important facts
- No guarantee that important facts are adjacent

Example (simplified):

```python
sections = (
    ["Filler text " + str(i) for i in range(1000)] +
    ["Ahab had a peg leg made of whale bone."] +
    ["More filler " + str(i) for i in range(1000)] +
    ["Ahab had a furrowed brow."] +
    ["Even more filler " + str(i) for i in range(1000)] +
    ["Ahab had a long scar down his face."]
)
