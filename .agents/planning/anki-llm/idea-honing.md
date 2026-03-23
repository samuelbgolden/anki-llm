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

**Answer:** Paraphrase around the blank. The system prompt sent to the LLM will explain cloze deletions and instruct it to reword the surrounding context while keeping the blank (`[...]`) in a semantically equivalent position.

---

## Q5: Should users be able to see the original card text?

During review, should there be a way to reveal the original (un-paraphrased) question?

Options:
- **No toggle** — paraphrase is shown, original is never visible during review
- **Toggle button** — a small button (e.g. "Show original") lets users reveal the original text if confused by the paraphrase
- **Keyboard shortcut only** — a hotkey reveals the original without cluttering the UI

**Answer:** Small toggle button. Unobtrusive, visible during review, lets users reveal the original question if the paraphrase is confusing.

---

## Q6: Where does the LLM run?

Options:
- **Local model only** — run a model on the user's machine (e.g. via Ollama); no data leaves the device
- **Remote API only** — call an external service (e.g. OpenAI, Anthropic); simpler to set up, costs money
- **User's choice** — configurable; support both local and remote backends

**Answer:** Local only via Ollama. A one-time setup script (cross-platform Python) handles installing Ollama and pulling the default model. The plugin checks if Ollama is reachable on startup and shows a clear error with setup instructions if not.

---

## Q7: Which model should be the default?

Ollama supports many models. The setup script needs to pull something concrete. What should the default be?

Options:
- **llama3.2:3b** — small (2 GB), fast on CPU, good quality for simple paraphrasing
- **mistral:7b** — larger (4 GB), better quality but needs more RAM/GPU
- **gemma3:4b** — Google's model, similar size to llama3.2, strong instruction following

**Answer:** `gemma3:4b`. The setup script will pull this by default. Users can swap models in the plugin config by changing the model name to any Ollama-compatible model they have installed.

---

## Q8: How should the plugin be configured?

Users may want to tweak settings like the Ollama URL, model name, or prefetch queue size. Where should config live?

Options:
- **Anki's built-in Tools → Add-ons config** — JSON editor Anki provides for all add-ons; no extra UI to build
- **Custom settings dialog** — a proper GUI dialog in Anki with labeled fields
- **Config file only** — a plain JSON/TOML file in the add-on folder; no in-Anki UI

**Answer:** Anki's built-in Tools → Add-ons config. JSON editor, no extra UI to build. Config will expose: `ollama_url`, `model`, and `prefetch_queue_size`.

---

## Q9: Should the paraphrasing feature be toggleable?

Should users be able to turn off paraphrasing (and see original cards) without uninstalling the add-on?

Options:
- **No toggle** — if the add-on is enabled, paraphrasing is always on
- **Global on/off** — a menu item or toolbar button to disable paraphrasing session-wide
- **Per-deck toggle** — enable/disable per deck in deck options
