# 07 — Open questions

1. **Transcription**: cloud speech-to-text (cost, privacy) or local whisper.cpp (setup, speed)? Both behind config?
2. **Podcast input**: is a direct episode/audio URL enough, or do we need RSS feed subscriptions?
3. **Transcript access beyond Claude Code**: should other clients send transcripts too, or only summaries via `report_session`?
4. **Continuous mode cost**: what monthly LLM budget is acceptable? This sets relevance-gate thresholds and default models.
5. **Prompt injection from sources**: how strict should we be? (e.g. strip instructions from extracted text, or show advice as quotes only.)
6. **Harness inspection**: should the AI send current harness files (CLAUDE.md, skills list) to `suggest_harness_refinements`, or should the user register them in the UI once?
7. **Language**: should notes be in the source language or always in one user-chosen language?
8. **Product name**: working name "info-shock" (from the repo).
9. **Later**: hosted / multi-user version? This would change the DB (Postgres), auth, and MCP transport.
