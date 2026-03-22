# Research: Distribution & Zero-Setup Strategy

## Summary

The core challenge is making an LLM-powered tool that requires zero technical setup. This is solved by combining: (1) native Anki add-on distribution via AnkiWeb, and (2) Ollama as the LLM backend with in-app guided setup.

---

## The Problem

LLMs require either:
- **Cloud API** (needs key, account, sometimes payment)
- **Local model** (needs installation, download, technical comfort)

Our constraint: "Easy for anyone regardless of technical ability."

---

## Solution: Guided In-Add-on Setup

Rather than eliminating setup entirely (which isn't fully achievable while staying local), we **minimize and guide** the setup:

### Step 1: Add-on Installation (Zero friction)
- User opens Anki → Tools → Add-ons → Get Add-ons
- Enters our numeric code (e.g., `1234567890`)
- Anki downloads and installs automatically
- **No file downloads, no command line, no Python knowledge needed**

### Step 2: First Launch (Guided Ollama Setup)
On first run, the add-on checks if Ollama is running:

```
┌────────────────────────────────────────────────┐
│  anki-llm Setup                                │
│                                                │
│  To paraphrase cards, we need a local AI.      │
│  Ollama is free, private, and takes 2 minutes. │
│                                                │
│  1. Download Ollama: [Open Download Page]      │
│  2. Install it (double-click the installer)    │
│  3. Click [Check Again] when done              │
│                                                │
│  [Check Again]  [Use Online Fallback Instead]  │
└────────────────────────────────────────────────┘
```

### Step 3: Model Download (Automated)
Once Ollama is detected, the add-on automatically pulls the recommended model:
```python
requests.post("http://localhost:11434/api/pull", json={"name": "llama3.2:3b"})
```
This can show a progress dialog. One-time download (~2 GB).

### Step 4: Ready
After setup, everything is automatic. No further configuration needed.

---

## Fallback: Online Keyless API

If the user clicks "Use Online Fallback Instead":
- Use mlvoca API or similar truly-keyless endpoint
- Show clear disclosure: "Responses go through an external server"
- Quality may be lower (TinyLlama vs Llama 3.2)
- No model download required
- Works immediately, even offline checks are skipped

---

## Ollama Installation: Platform Details

| Platform | Method | User Steps |
|---|---|---|
| **macOS** | DMG from ollama.com | Download → Open DMG → Drag to Applications → Open |
| **Windows** | .exe installer | Download → Double-click → Follow prompts (2-3 min) |
| **Linux** | Shell script | `curl -fsSL https://ollama.com/install.sh \| sh` |

Ollama starts automatically as a desktop app / background service. No terminal needed after install on Mac/Windows.

---

## Detecting Ollama

```python
import requests

def is_ollama_running() -> bool:
    try:
        r = requests.get("http://localhost:11434/", timeout=2)
        return r.status_code == 200
    except Exception:
        return False

def is_model_available(model_name: str) -> bool:
    try:
        r = requests.get("http://localhost:11434/api/tags", timeout=2)
        models = [m["name"] for m in r.json().get("models", [])]
        return any(model_name in m for m in models)
    except Exception:
        return False
```

---

## Anki Add-on Distribution (AnkiWeb)

- **Upload:** Zip the add-on folder, upload to AnkiWeb developer portal
- **Install:** User enters numeric ID in Anki's add-on dialog
- **Update:** Anki auto-notifies users of updates
- **User config:** Stored in `user_files/` so it survives upgrades
- **Size limit:** AnkiWeb has a file size limit — we cannot bundle Ollama (~3.25 GB)

### What we CAN bundle:
- `requests` library (pure Python, ~100 KB)
- Our add-on code
- Config UI assets
- Caching layer

---

## Configuration Options (Exposed to User)

Via Anki's Tools → Add-ons → anki-llm → Config:

```json
{
  "backend": "ollama",
  "ollama_url": "http://localhost:11434",
  "ollama_model": "llama3.2:3b",
  "fallback_backend": "mlvoca",
  "cache_variants_per_card": 5,
  "paraphrase_answer_only": true,
  "enabled_note_types": [],
  "show_original_toggle": true
}
```

---

## Privacy Considerations

- With Ollama: all data stays on device — **fully private**
- With online fallback: card text is sent to external server — must disclose clearly
- Medical students may have sensitive study content (patient info, etc.) — Ollama is the right default

---

## References
- [Ollama Download](https://ollama.com/download)
- [Ollama Windows Guide](https://skywork.ai/blog/llm/ollama-windows-guide-install-run-local-ai-on-pc/)
- [mlvoca Free LLM API](https://mlvoca.github.io/free-llm-api/)
- [AnkiWeb Add-on Sharing](https://addon-docs.ankiweb.net/)
- [Integrating Into Desktop App (Ollama GitHub Issue)](https://github.com/ollama/ollama/issues/7419)
