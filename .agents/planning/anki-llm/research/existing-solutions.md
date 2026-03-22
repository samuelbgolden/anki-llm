# Research: Existing Solutions

## Summary

Several tools exist at the intersection of Anki and LLMs, but **none solve the exact problem** of real-time per-review paraphrasing. The gap our project fills is clear.

---

## Existing Projects

### 1. AnkiAIUtils — Reformulator
**Repo:** [github.com/thiswillbeyourgithub/AnkiAIUtils](https://github.com/thiswillbeyourgithub/AnkiAIUtils)
**Forum:** [Anki Forums post](https://forums.ankiweb.net/t/i-made-ankiaiutils-reformulator-automatically-improve-your-anki-cards/53684)

**What it does:** Batch-rephrases cards in your collection, modifying the stored card content permanently (with rollback capability). Preserves cloze deletions, media, and formatting. Uses LiteLLM so supports virtually all LLM providers.

**Key difference from our project:**
- **Batch, offline, one-time** operation — processes cards before reviewing
- **Modifies the stored deck** — permanently changes the card text
- **Requires technical setup** — Python scripts, LiteLLM config, API keys
- **Not real-time** — the same rephrased version is shown every time after processing

**What we can learn:**
- Their approach to preserving cloze deletions is valuable
- HTML stripping before LLM call is important
- LiteLLM as a multi-provider abstraction is worth considering

---

### 2. anki-llm CLI
**Repo:** [github.com/raine/anki-llm](https://github.com/raine/anki-llm)
**Blog:** [brightcoding.dev](https://www.blog.brightcoding.dev/2025/12/12/anki-llm-revolutionize-your-flashcard-creation-with-ai-powered-cli-tools/)

**What it does:** CLI toolkit for bulk-processing Anki cards. Requires AnkiConnect add-on + Anki running. Uses GPT-4o, Gemini Flash. Can process 1000 cards for <$0.03 with Gemini Flash.

**Key difference:**
- CLI tool — requires command-line comfort
- Requires AnkiConnect (another add-on)
- Modifies cards permanently (batch)
- Paid API required (OpenAI, Gemini)

---

### 3. AnkiBrain / AnkiChatGPT
**AnkiWeb:** [ankiweb.net/shared/info/1915225457](https://ankiweb.net/shared/info/1915225457)

**What it does:** Native Anki add-on integrating GPT-4/3.5. Primarily for generating new cards and explaining concepts during review.

**Key difference:**
- Requires OpenAI API key
- Focused on card generation and explanation, not paraphrasing
- Premium model dependency

---

### 4. Anki GPT-4 Flashcard Generator
**AnkiWeb:** [ankiweb.net/shared/info/2041588669](https://ankiweb.net/shared/info/2041588669)

**What it does:** Generates new flashcards from selected text using GPT-4.

**Key difference:**
- Card generation, not review-time paraphrasing
- Requires API key

---

### 5. flashcard-inator
**Repo:** [github.com/francescopeluso/flashcard-inator](https://github.com/francescopeluso/flashcard-inator)

**What it does:** Generates Anki flashcards from Obsidian notes using Ollama or LM Studio. Supports local LLMs.

**What we can learn:**
- Proof that Ollama + Anki workflow is viable
- Pattern for detecting and connecting to local Ollama

---

## Gap Analysis

```
                    | Real-time | Native Add-on | No API Key | Local/Offline |
--------------------|-----------|---------------|------------|---------------|
AnkiAIUtils         |    No     |      No       |     No     |    Maybe      |
anki-llm CLI        |    No     |      No       |     No     |      No       |
AnkiBrain           |   No*     |     Yes       |     No     |      No       |
flashcard-inator    |    No     |      No       |    Yes     |     Yes       |
**anki-llm (ours)** |  **Yes**  |    **Yes**    |  **Yes**   |   **Yes**     |
```

*AnkiBrain can explain cards during review, but doesn't paraphrase the answer text itself.

---

## Key Differentiator

**anki-llm (our project)** is the only solution that:
1. Operates **in real-time during review** — different wording every session
2. Is a **native Anki add-on** — installable with a code, no CLI needed
3. **Requires no API key** — works with Ollama out of the box
4. **Does not modify stored cards** — paraphrasing is display-only, preserving the original deck

This is a genuine gap in the ecosystem.

---

## References
- [AnkiAIUtils GitHub](https://github.com/thiswillbeyourgithub/AnkiAIUtils)
- [anki-llm CLI GitHub](https://github.com/raine/anki-llm)
- [AnkiBrain on AnkiWeb](https://ankiweb.net/shared/info/1915225457)
- [Anki GPT-4 Generator on AnkiWeb](https://ankiweb.net/shared/info/2041588669)
- [flashcard-inator GitHub](https://github.com/francescopeluso/flashcard-inator)
