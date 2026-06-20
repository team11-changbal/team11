# Agent_3: Suno API Architect (JSON Bridge / MCP Bridge)

**Version:** v1.0
**Status:** Draft (POC)
**Source spec:** `_orchestration_/orchestration_v2.md` (Section 3.3)

## Input
Agent_1 lyrics (EN) + Agent_2 music style result + history_match narration

## Output
1. Suno API JSON payload
2. Curator narration text
3. Regression log (minimum 3 iterations)

## MCP Integration
Puppeteer MCP or custom Suno MCP server

## Task (Optional)
Auto-generate the final request payload matching the unofficial Suno API spec (`POST /suno/v1/music`).

## Task Detail
- Construct a Suno-optimized `[Lyrics]` and `[Style of Music]` prompt
- Run ≥ 3 regression iterations to optimize style tags against lyrics/image/history
- Log each regression round to `POC/Agent_3/memory/` for future training
- Output curator narration (short, gallery-style description, like a music/art exhibition label)
- Optionally bridge to Suno via:
  - **Method A (MVP):** Copy-paste UI buttons for lyrics + style
  - **Method B (Advanced):** Unofficial Suno API via `POST /suno/v1/music`

## Suno API Payload Schema
```json
{
  "customMode": true,
  "instrumental": false,
  "model": "V5",
  "title": "[EN song title]",
  "prompt": "[Structured lyrics from Agent_1]",
  "tags": "[Style tags from Agent_2, max 1000 chars]"
}
```

## Regression Log Schema (per iteration)
```json
{
  "iteration": 1,
  "input_style_tags": "string",
  "adjustment_reason": "string",
  "updated_style_tags": "string",
  "confidence_score": 0.0
}
```

## Folder Structure
```text
POC/Agent_3/skill.md
POC/Agent_3/memory/regression_log_YYYY-MM-DD.json
POC/Agent_3/memory/trained_index.md
```

> Note: orchestration_v2.md refers to this index file as `training_index.md`. The file actually present in this repo is `trained_index.md` — kept as-is here to match what already exists rather than renaming it.

## Downstream
Output feeds `Agent_4` (full music concept + narration).
