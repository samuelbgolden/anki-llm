# Research: Anki File Formats & Deck Structure

## Summary

Understanding Anki's file format is important for understanding how cards are stored, but for the **add-on approach**, we interact with the collection via the `anki` Python module — not the raw file. Still useful for understanding the data model.

---

## The `.apkg` Format

An `.apkg` file is a **ZIP archive** containing:
- `collection.anki2` (or `.anki21`, `.anki21b`) — SQLite database
- Numbered files (`1`, `2`, `3`, ...) — media files (images, audio)
- `media` — JSON mapping from numeric filenames back to original names

### Format Versions

| Version | Database File | Era | Notes |
|---|---|---|---|
| Legacy (Anki 2.0) | `collection.anki2` | 2012 | JSON in TEXT columns |
| Legacy 2 (Anki 2.1) | `collection.anki21` | 2018 | JSON in TEXT columns |
| Current | `collection.anki21b` | 2020+ | Protobuf in BLOB columns, zstd compression |

---

## SQLite Database Schema

### `notes` Table — Where Card Content Lives

The most important table for us. Contains the actual text content of flashcards.

| Column | Description |
|---|---|
| `id` | Epoch milliseconds timestamp |
| `guid` | Globally unique note ID |
| `mid` | Note type/model ID |
| `flds` | **All fields concatenated, separated by `\x1f` (unit separator char)** |
| `sfld` | Sort field |
| `tags` | Space-separated tags |

**Key insight:** `flds` stores all fields (e.g., Front, Back) as a single string delimited by `\x1f`. For a standard 2-field card: `Front text\x1fBack text`.

### `cards` Table — Review State

| Column | Description |
|---|---|
| `id` | Card ID |
| `nid` | Note ID (FK to notes) |
| `did` | Deck ID |
| `ord` | Which card template (0 = first template) |
| `type` | 0=new, 1=learning, 2=review, 3=relearning |
| `due` | Due date/position |
| `ivl` | Interval in days |

### `col` Table — Collection Metadata

- Contains deck definitions, note type (model) definitions
- Models define which fields exist and what templates look like
- Templates define the HTML for question/answer sides

---

## Note Types (Models)

Each note has a **model** that defines:
- `flds` — field definitions (names, order)
- `tmpls` — card templates (HTML for front/back)
- Common models: Basic (Front/Back), Basic (Reversed), Cloze

**Standard Basic model fields:**
1. `Front` — question text
2. `Back` — answer text (this is what we paraphrase)

**Cloze model:** Uses `{{c1::text}}` syntax. Needs special handling to preserve cloze markers.

---

## How the Add-on Accesses Data

Via the `anki` Python module (no raw SQLite needed):

```python
from aqt import mw

# Access current card during review
card = mw.reviewer.card
note = card.note()

# Get fields by name
back_text = note["Back"]  # for Basic notes
front_text = note["Front"]

# Get all fields
for field_name, field_value in note.items():
    print(field_name, field_value)
```

The `card_will_show` hook provides the rendered **HTML** (after template substitution), not raw field values. We can modify this HTML or access the underlying note to get raw text.

---

## Implications for Our Design

1. **We target the rendered HTML** via `card_will_show` — this is the safest approach as it doesn't modify the underlying deck data
2. **We do NOT write back to the database** during review — paraphrasing is display-only, preserving the original card content
3. **Cloze cards** need special handling — must preserve `{{c1::...}}` markers in the source, but the rendered HTML shows filled-in cloze text
4. **Rich formatting** — card content may include HTML, images, audio. We need to strip HTML before sending to LLM and re-inject after.

---

## Compatibility

- The add-on approach via `card_will_show` is **format-agnostic** — it works regardless of which `.apkg` version created the deck
- Existing decks imported into Anki work automatically
- No migration or conversion needed

---

## References
- [Understanding the Anki APKG Format (Eiko Wagenknecht)](https://eikowagenknecht.com/posts/understanding-the-anki-apkg-format/)
- [AnkiDroid Database Structure Wiki](https://github.com/ankidroid/Anki-Android/wiki/Database-Structure)
- [Packaged Decks — Anki Manual](https://docs.ankiweb.net/importing/packaged-decks.html)
- [ankisync2 — Python library for .apkg](https://github.com/patarapolw/ankisync2)
