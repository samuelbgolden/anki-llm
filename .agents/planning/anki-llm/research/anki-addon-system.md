# Research: Anki Add-on System

## Summary

Anki has a mature, well-documented add-on system that is **the right integration path** for this project. A native add-on lets us hook directly into the card review pipeline with no external tools required beyond Anki itself.

---

## Add-on Architecture

- Add-ons are **Python modules** loaded by Anki at startup
- The Anki UI is **Python/PyQt** (desktop), with some screens using TypeScript/Svelte
- Add-ons can register **hooks** (event listeners) and modify UI
- Distributed via **AnkiWeb** — users install with a numeric code from within Anki (Tools → Add-ons → Get Add-ons)
- This is the easiest possible distribution: no download, no zip, no file management

## The Critical Hook: `card_will_show`

This is the key hook for our use case. It fires **before** a card's HTML is rendered to the user and allows us to modify the HTML.

```python
from aqt import gui_hooks

def on_card_will_show(html: str, card, context: str) -> str:
    if context == "reviewAnswer":
        # Modify html here — return the modified version
        return modified_html
    return html

gui_hooks.card_will_show.append(on_card_will_show)
```

### Context Values
- `"reviewQuestion"` — front of card during review
- `"reviewAnswer"` — back of card during review
- `"clayoutQuestion"` / `"clayoutAnswer"` — card layout editor
- `"previewQuestion"` / `"previewAnswer"` — card previewer

**For our use case: intercept `"reviewAnswer"` only.**

## Background Threading (Critical for LLM Calls)

LLM API calls are slow. Running them on the main thread would freeze Anki's UI. Anki provides `QueryOp` for background operations:

```python
from aqt.operations import QueryOp

def do_llm_call(col):
    # This runs in a background thread — safe for network/slow ops
    return call_ollama_api(card_text)

def on_success(paraphrased_text):
    # This runs on the main thread — safe to update UI
    update_card_display(paraphrased_text)

QueryOp(parent=mw, op=do_llm_call, success=on_success).run_in_background()
```

**Important:** Never call Qt/UI code from inside the background op. Gather any UI data before starting the op.

## Bundling Python Dependencies

Add-ons can bundle third-party Python packages:
- Place packages in a subfolder (e.g., `vendor/`)
- Adjust `sys.path` to include it
- Works for **pure Python packages** (e.g., `requests`, `httpx`)
- C-extension packages (NumPy, etc.) are more complex — must bundle per-platform builds
- For Ollama integration, `requests` (pure Python) is sufficient — no complex deps

## Distribution via AnkiWeb

- Upload a zip of the add-on folder to AnkiWeb
- Users get a numeric code (e.g., `1234567890`) and install within Anki
- **On upgrade, all files are deleted except the `user_files/` folder** — store user config there
- This is the gold standard for Anki add-on distribution — zero friction for users

## Additional Hooks of Interest

- `reviewer_did_show_question` — fires after question is shown (could trigger pre-fetching paraphrase)
- `reviewer_did_show_answer` — fires after answer is shown
- `webview_did_receive_js_message` — intercept JS→Python messages (allows interactive UI in cards)

## AnkiConnect (Alternative Approach)

[AnkiConnect](https://github.com/FooSoft/anki-connect) is an existing add-on that exposes a REST API on port 8765. External scripts can talk to Anki via HTTP. This is how tools like `anki-llm` CLI work. **Not ideal for our real-time use case** — adds a second moving part. Native add-on is better.

---

## References
- [Official Anki Add-on Docs](https://addon-docs.ankiweb.net/)
- [Hooks and Filters](https://addon-docs.ankiweb.net/hooks-and-filters.html)
- [Background Operations](https://addon-docs.ankiweb.net/background-ops.html)
- [Reviewer JavaScript](https://addon-docs.ankiweb.net/reviewer-javascript.html)
- [Python Modules / Bundling](https://addon-docs.ankiweb.net/python-modules.html)
- [AnkiConnect GitHub](https://github.com/FooSoft/anki-connect)
