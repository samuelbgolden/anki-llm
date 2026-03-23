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

**Answer:** Pre-fetch ~15 cards ahead in a background queue. If the user exhausts the prefetch buffer (reviews faster than the LLM generates), fall back to showing the original text for those cards and continue queuing further out. No loading states or visible delays to the user.

---

## Q3: Should paraphrases be cached?

Once a paraphrase is generated for a card, should it be saved and reused, or should a fresh one be generated every review session?

Options:
- **Always fresh** — new paraphrase every time the card appears (maximum variety, more LLM calls)
- **Cache N variants per card** — generate and store a fixed number of variants (e.g., 5), cycle through them across sessions (fewer LLM calls over time)
- **Cache per session only** — generate once per review session, fresh next time Anki opens

**Answer:** Always fresh to start. If latency becomes a real-world problem after testing, introduce caching as an improvement.

---

## Q4: How should cloze deletion cards be handled?

Cloze cards use blanks: e.g. "The {{c1::mitochondria}} is the powerhouse of the cell" — during review the blank is shown as "[...]" and the user must recall the hidden word.

Paraphrasing these is tricky because the blank position is semantically meaningful. Options:

- **Paraphrase around the blank** — reword the surrounding text but keep the blank in place (e.g. "In cellular biology, [...] serves as the cell's power source")
- **Skip cloze cards entirely** — only paraphrase Basic/standard cards; show cloze cards as-is
- **Treat as regular text** — send the rendered text (with "[...]") to the LLM and let it rephrase naturally
