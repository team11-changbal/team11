# Agent_2: Cultural & Musical Skill Matcher

**Version:** v0.1 (draft — not fully confirmed)
**Status:** Pending — awaiting updates from a separate example DB (optional)
**Source spec:** `_orchestration_/orchestration_v2.md` (Section 3.2)

> ⚠️ This skill is not finalized. The matching logic below reflects the orchestration_v2 spec only. A separate example DB (mood/date → genre sample mappings) is planned to refine matching, but that integration is optional and still TBD.

## Task
Match the mood derived by Agent_1 to the most artistically synergistic genre from the embedded SKILLS DATABASE.

## Core Rule
Don't just output a genre name — draw on historical background and a representative artist episode (e.g. Charlie Parker's improvisation technique, Giorgio Moroder's synthesizer experiments) to produce concrete instrumentation, tempo, and mixing texture that Suno can act on.

## Input
Agent_1 output (`mood_analysis` + `history_match`)

## Output
Matched genre, tempo, instrumentation, representative artist episode, artwork prompt

## Skills DB
13 genres (Cool Jazz → Ballad Rock) — see `Examples/Agent_2_DB_skill.md`

## Input Schema
```json
{
  "mood_analysis": "string",
  "history_match": { "date_event": "string", "lyric_hook": "string" }
}
```

## Output Schema
```json
{
  "genre": "string",
  "tempo_bpm": "number",
  "instrumentation": ["string"],
  "suno_style_tags": "string (max 1000 chars)",
  "artist_episode": "string",
  "artwork_prompt": "string"
}
```

## Open Items (Not Yet Confirmed)
- Genre-matching logic against the 13-genre Skills DB is not finalized.
- A separate example DB of mood/date → genre samples is planned to improve matching accuracy. Optional, scheduling TBD.

## Downstream
Output feeds `Agent_3` (style tags + artwork prompt) alongside Agent_1's lyrics.
