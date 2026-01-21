# Demonstrating DSPy Predict vs Recursive Language Models (RLM) on Long-Context Reasoning

## One Sentence Summary

**`dspy.Predict` does not fail because it reasons poorly — it fails because it hits rate limits when forced to process long context in a single prompt.**

Recursive Language Models (RLMs) exist to solve this *operational* failure.

---

## What This Demo Is Proving

This demo is **not** about:
- Accuracy
- Chain-of-thought quality
- Whether the model “knows” the answer

The demo proves that:

> **Single-shot prompting becomes unusable at scale because it concentrates token usage into one call and repeatedly hits TPM limits.**

---

## The Failure Mode

### Predict’s Operating Assumption

`Predict` assumes:
- The full context fits into one prompt
- One large LLM call is acceptable
- Retries are cheap

This breaks down when:
- Context grows large
- Users re-run cells
- The system retries after failures

Result:
- Token-per-minute (TPM) exhaustion
- Rate limiting
- Cascading failures

---

## Why This Is Not a Toy Problem

In real systems:
- Long documents are common
- Users rerun notebooks
- Pipelines retry automatically
- Multiple users share the same quota

Even if **Predict gets the right answer**, it often:
- Cannot run reliably
- Cannot be repeated
- Cannot scale to production

Accuracy becomes irrelevant if the call never completes.

---

## Why RLM Does Not Get Rate Limited (in the Same Way)

RLM changes the unit of work:

| Predict | RLM |
|------|----|
| One huge prompt | Many small prompts |
| Large TPM spike | Steady TPM usage |
| Retry = full cost | Retry = local cost |
| All-or-nothing | Incremental |

RLM:
- Spreads token usage over time
- Keeps each call under TPM thresholds
- Fails gracefully instead of catastrophically

---

## Key Insight

> **Rate limits are a systems problem, not a reasoning problem.**

RLM is a systems-level solution.

---

## Why Predict “Looks Fine” in Demos

Predict appears to work because:
- Context is still small
- The notebook is run once
- No retries occur

The moment you:
- Increase document size
- Rerun cells
- Share quotas
- Add retries

Predict becomes unstable.

---

## The Real Differentiator

| Question | Wrong Focus | Correct Focus |
|--------|------------|---------------|
| Which answers correctly? | Accuracy | Reliability |
| Which reasons better? | CoT | Token flow |
| Which scales? | Prompt cleverness | Call structure |

---
