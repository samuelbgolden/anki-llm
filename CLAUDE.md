# anki-llm

Anki add-on that paraphrases flashcard questions using a local LLM (Ollama), breaking pattern memorization and forcing genuine semantic understanding.

## Project structure

```
anki-llm/
├── __init__.py          # Add-on entry point, hooks registration
├── prefetch.py          # Background prefetch queue (15 cards ahead)
├── llm.py               # Ollama API client
├── config.py            # Config access + defaults
├── vendor/              # Bundled dependencies (requests, etc.)
├── user_files/          # User config, persists across upgrades
└── setup.py             # One-time cross-platform setup script (not part of add-on)
```

## Decisions locked in

- **What gets paraphrased:** Front/question side only. Answer stays fixed as ground truth.
- **Cloze cards:** Paraphrase around the blank — reword surrounding text, keep `[...]` in semantically equivalent position. System prompt explains cloze to the LLM.
- **Latency:** Pre-fetch 15 cards ahead in a background queue. If buffer exhausted, fall back to original text — no loading states shown to user.
- **Caching:** Always fresh (no caching). Revisit if latency is a real problem post-testing.
- **Original text toggle:** Small unobtrusive button during review to reveal the original question.
- **LLM backend:** Local Ollama only (`http://localhost:11434`). No remote API.
- **Default model:** `gemma3:4b`. Configurable.
- **Setup:** One-time cross-platform Python setup script installs Ollama and pulls `gemma3:4b`. Plugin checks if Ollama is reachable on startup and shows a clear error if not.
- **Config:** Anki's built-in Tools → Add-ons JSON editor. Exposes: `ollama_url`, `model`, `prefetch_queue_size`.
- **Global toggle:** Tools menu item "Enable paraphrasing" — persists to config. When off, no LLM calls made.
- **Exclusions:** Cards tagged `no-paraphrase` are skipped and shown as-is.

## Key Anki APIs

```python
# Primary hook — fires before card HTML is rendered
from aqt import gui_hooks
gui_hooks.card_will_show.append(on_card_will_show)
# context == "reviewQuestion" for front side (what we modify)

# Pre-fetch trigger — fires after question is shown
gui_hooks.reviewer_did_show_question.append(on_question_shown)

# Background threading — never call Ollama on main thread
from aqt.operations import QueryOp
QueryOp(parent=mw, op=do_llm_call, success=on_success).run_in_background()

# Access current card
from aqt import mw
card = mw.reviewer.card
note = card.note()
tags = note.tags  # list of strings — check for "no-paraphrase"

# Config
from aqt import mw
config = mw.addonManager.getConfig(__name__)
```

## Ollama API

```python
# Chat completion (OpenAI-compatible)
POST http://localhost:11434/v1/chat/completions

# Check if running
GET http://localhost:11434/

# List installed models
GET http://localhost:11434/api/tags

# Pull a model
POST http://localhost:11434/api/pull  {"name": "gemma3:4b"}
```

## Default config schema

```json
{
  "ollama_url": "http://localhost:11434",
  "model": "gemma3:4b",
  "prefetch_queue_size": 15,
  "enabled": true
}
```

## Critical constraints

- **Never modify the underlying card/note data** — paraphrasing is display-only via `card_will_show`
- **Never call Qt/UI code from inside a `QueryOp` background op** — gather UI data before starting the op
- **Pure Python dependencies only** in `vendor/` — no C-extensions (they're platform-specific)
- Card content may include HTML, images, audio — strip HTML before sending to LLM, preserve structure after
- `user_files/` survives AnkiWeb upgrades; everything else is wiped on update — store nothing important outside it
