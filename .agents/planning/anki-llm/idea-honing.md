# Idea Honing: anki-llm

This document tracks the requirements clarification Q&A process.

---

## Q1: What gets paraphrased?

When a card is shown during review, which parts should be paraphrased?

Options:
- **Answer/back only** — the question stays fixed (good anchor), only the explanation is reworded
- **Both sides** — question and answer are both paraphrased
- **Configurable per deck/note type** — user decides per deck

**Answer:** Paraphrase the question/front side only. The answer stays fixed so it remains the reliable ground truth. The question is reworded each review so the user can't pattern-match the prompt.

---

## Q2: How should latency be handled?

Calling an LLM takes time (typically 1–5 seconds for a small local model). When the user flips to the question side of a card, what should they see while waiting?

Options:
- **Show original text first, then swap** — the original question appears immediately, then gets replaced by the paraphrase once ready
- **Show a loading indicator** — blank or spinner until paraphrase is ready, then show it
- **Pre-fetch during the previous card** — start generating the paraphrase for the next card while the user is reviewing the current one, so it's ready instantly
