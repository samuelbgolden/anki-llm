# Rough Idea: anki-llm

**Provided:** 2026-03-22

## Summary

There's an application called Anki that med students use to memorize and do flashcards. The concern is that memorization can become pattern-based — users pass cards because they've memorized what the card looks like and reads like, rather than having memorized the actual definition or semantic understanding.

## Core Concept

Use an LLM to augment Anki decks so that each time you see a card, the definition is identical in meaning but worded slightly differently. This breaks pattern memorization and forces genuine semantic understanding.

## Key Constraints / Preferences

- **Cost**: Should be cheap and easy — no need for a large/premium language model
- **Accessibility**: Easy for anyone to use regardless of technical ability — no API key setup required
- **Compatibility**: Should work seamlessly with existing Anki decks
- **Integration approach**: Unclear — could be a fork of Anki, or could leverage Anki's plugin system (research needed)
- **Name**: anki-llm

## Open Questions (at time of capture)

- Does Anki have a plugin system that could be used instead of forking?
- How do we enable LLM use without requiring users to set up API keys?
- What small/cheap LLM would work well for paraphrasing card definitions?
