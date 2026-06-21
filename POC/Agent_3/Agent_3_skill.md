# Agent_3: Suno API Architect (JSON Bridge / MCP Bridge)

**Version:** v2.0
**Status:** Active (POC)
**Source spec:** `orchestration_v3.md` (Section 3.3)
**Updated:** 2026-06-20
**Changelog v2.0:**
- Input fields expanded to full table format (field / type / required Y/N / description + example)
- Output schema aligned to Orchestration contract (downstream Agent_4 format)
- Regression logic formalized into 3-iteration minimum with scoring criteria
- Curator narration rules added
- Delivery method (Method A vs B) decision criteria clarified
- Folder structure and file naming confirmed against repo (`trained_index.md`)

---

## Task

Receive lyrics and music style from Agent_1 and Agent_2, then construct and optimize the final Suno API payload through a minimum 3-iteration regression loop. Produce three outputs simultaneously: (1) a production-ready Suno API JSON payload, (2) a curator narration text, and (3) a regression log. Optionally bridge directly to the Suno API (Method B) or output for manual copy-paste (Method A).

## Core Rule

Never pass Agent_2's raw `suno_style_tags` directly into the Suno `tags` field without at least one regression iteration. The regression process must verify that style tags are coherent with the lyric structure, the historical narrative, and the target genre — and adjust if they conflict or underperform. Every iteration must be logged.

---

## Input

Receives Agent_1 output (lyrics + history) and Agent_2 output (style + genre) from the Orchestrator simultaneously.

### Input Field Specification

| Field | Type | Required | Source Agent | Description | Example |
|---|---|---|---|---|---|
| `lyrics_en` | string | **Y** | Agent_1 | Full English lyrics with structure tags ([Verse 1], [Chorus], etc.). This becomes the Suno `prompt` field after formatting. | `"[Verse 1]\nI stood at the window like a debt unpaid..."` |
| `song_title` | string | **Y** | Agent_1 | English song title. Becomes the Suno API `title` field. | `"The Weight of New Coins"` |
| `history_match` | object | **Y** | Agent_1 | Historical event and lyric hook. Used as thematic anchor for curator narration and regression evaluation. | `{"date_event": "June 20, 1948 — Deutsche Mark reform", "lyric_hook": "A new currency in my pocket..."}` |
| `genre` | string | **Y** | Agent_2 | Canonical genre name from Skills DB. Used to validate tag coherence in regression. | `"Fusion Jazz"` |
| `genre_id` | string | **Y** | Agent_2 | Skills DB identifier. Used to cross-reference expected instrumentation in regression. | `"SKILLS_4"` |
| `suno_style_tags` | string | **Y** | Agent_2 | Initial style tag string (max 1000 chars). This is the **starting point** for regression iteration 1 — not the final tags. | `"Fusion jazz, jazz rock, fender rhodes electric piano, psychedelic atmosphere, mid-tempo"` |
| `tempo_bpm` | number | **Y** | Agent_2 | Target tempo. Used to validate tempo-related tags in regression (e.g., "mid-tempo" vs. explicit BPM tags). | `98` |
| `instrumentation` | array[string] | **Y** | Agent_2 | Ordered instrument list. Used to verify all key instruments are represented in final `tags`. | `["Fender Rhodes electric piano", "distortion electric guitar", "analog synthesizer"]` |
| `artist_episode` | string | **Y** | Agent_2 | Representative artist narrative. Primary input for curator narration generation. | `"Miles Davis entered the 1969 Bitches Brew sessions without sheet music..."` |
| `historical_context` | string | **Y** | Agent_2 | Genre's historical era background. Secondary input for curator narration. | `"Fusion Jazz emerged in the late 1960s as counterculture pressed jazz to embrace electric technology..."` |
| `artwork_prompt` | string | **Y** | Agent_2 | Album artwork image generation prompt. Passed through unchanged to Agent_4 output bundle. | `"1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism"` |
| `mood_analysis` | string | N | Agent_1 | Emotional tone summary. Optional input to regression — used to check for emotional coherence between style tags and lyric mood. | `"Melancholic introspection with undercurrent of quiet hope; low arousal..."` |

### Input Schema (JSON)

```json
{
  "lyrics_en": "string",
  "song_title": "string",
  "history_match": {
    "date_event": "string",
    "lyric_hook": "string"
  },
  "genre": "string",
  "genre_id": "string",
  "suno_style_tags": "string",
  "tempo_bpm": "number",
  "instrumentation": ["string"],
  "artist_episode": "string",
  "historical_context": "string",
  "artwork_prompt": "string",
  "mood_analysis": "string | null"
}
```

---

## Task Logic — Regression Loop (Minimum 3 Iterations)

Each iteration evaluates the current `style_tags` string against three coherence checks and produces an updated version with a confidence score. Iteration stops when `confidence_score ≥ 0.85` or after 5 iterations (whichever comes first). The final iteration's `updated_style_tags` becomes the Suno `tags` payload field.

### Coherence Checks per Iteration

| Check | Pass Criterion | Failure Action |
|---|---|---|
| **Lyric–Tag Alignment** | Style tags reflect the emotional register of the lyrics (e.g., "melancholic" present if lyrics are dark/slow). | Add or swap emotional descriptor tags. |
| **Instrumentation Coverage** | All instruments in `instrumentation` array appear (by name or alias) in the tag string. | Insert missing instrument tags; remove contradictory ones. |
| **Historical–Genre Coherence** | Tags do not contradict the historical era of the genre (e.g., no "808 drums" in a Cool Jazz tag set). | Replace anachronistic tags with era-appropriate alternatives. |

### Confidence Scoring Rubric

| Score Range | Meaning |
|---|---|
| 0.0 – 0.49 | Major conflicts across multiple checks; significant rewrite needed. |
| 0.50 – 0.74 | Partial coherence; 1–2 checks failing; targeted adjustments needed. |
| 0.75 – 0.84 | Good coherence; minor tuning only. |
| 0.85 – 1.00 | Production-ready; iteration may stop. |

### Regression Log Schema (per iteration)

Each iteration is written as one entry to `POC/Agent_3/memory/regression_log_YYYY-MM-DD.json`.

```json
{
  "iteration": 1,
  "input_style_tags": "string",
  "adjustment_reason": "string",
  "updated_style_tags": "string",
  "confidence_score": 0.0
}
```

> Note: `adjustment_reason` must be explicit (e.g., *"Added 'melancholic' to address lyric–tag alignment failure; removed '808 drums' as anachronistic for Fusion Jazz era"*) — vague entries like "improved tags" are not valid log entries.

---

## Output

### Output Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `suno_payload` | object | **Y** | Final Suno API JSON payload, ready for `POST /suno/v1/music`. All fields confirmed against Suno V5 spec. | See sub-schema below |
| `suno_payload.customMode` | boolean | **Y** | Always `true` for custom lyrics + style mode. | `true` |
| `suno_payload.instrumental` | boolean | **Y** | Always `false` (lyrics are present). Set to `true` only if user explicitly requests no vocals. | `false` |
| `suno_payload.model` | string | **Y** | Always `"V5"` for current POC. | `"V5"` |
| `suno_payload.title` | string | **Y** | English song title from Agent_1 `song_title`. Max 80 chars. | `"The Weight of New Coins"` |
| `suno_payload.prompt` | string | **Y** | Structured English lyrics from Agent_1, with structure tags preserved. | `"[Verse 1]\nI stood at the window like a debt unpaid..."` |
| `suno_payload.tags` | string | **Y** | Final optimized style tag string from the last regression iteration. Max 1000 chars. | `"Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, psychedelic analog synthesizer, distortion electric guitar, introspective vocals"` |
| `curator_narration` | string | **Y** | 3–5 sentence gallery-style exhibition label. Written in the voice of a music curator. Synthesizes `artist_episode`, `historical_context`, and `lyric_hook`. Passed to Agent_4. | See example below |
| `regression_log` | array[object] | **Y** | Array of all iteration log entries (minimum 3 entries). Stored to memory folder and also passed to Agent_4 for transparency. | `[{"iteration": 1, ...}, {"iteration": 2, ...}, {"iteration": 3, ...}]` |
| `final_confidence_score` | number | **Y** | The `confidence_score` from the final regression iteration. Passed to Agent_4 and written to `trained_index.md`. | `0.91` |
| `delivery_method` | string | **Y** | `"method_a"` (copy-paste UI) or `"method_b"` (direct API call). Determined by runtime environment detection. | `"method_a"` |

### Delivery Method Decision

| Condition | Method Selected |
|---|---|
| Suno API credentials available in environment + MCP bridge active | `method_b` — direct `POST /suno/v1/music` |
| No API credentials OR running in copy-paste demo mode | `method_a` — output formatted for UI copy-paste buttons |

In `method_a`, the `suno_payload` is still fully constructed and displayed to the user via formatted UI blocks (Lyrics block + Style Tags block + Title block), each with a copy button.

### Output Schema — Orchestration Contract (JSON)

This schema is the **canonical contract** passed to Agent_4. All fields must be present; `null` is not permitted.

```json
{
  "suno_payload": {
    "customMode": true,
    "instrumental": false,
    "model": "V5",
    "title": "string",
    "prompt": "string",
    "tags": "string (max 1000 chars)"
  },
  "curator_narration": "string",
  "regression_log": [
    {
      "iteration": "number",
      "input_style_tags": "string",
      "adjustment_reason": "string",
      "updated_style_tags": "string",
      "confidence_score": "number"
    }
  ],
  "final_confidence_score": "number",
  "delivery_method": "string",
  "artwork_prompt": "string"
}
```

> Note: `artwork_prompt` is passed through from Agent_2 input unchanged — Agent_3 does not modify it but includes it in the Agent_4 bundle so Agent_4 receives a single complete object.

### Example Output

```json
{
  "suno_payload": {
    "customMode": true,
    "instrumental": false,
    "model": "V5",
    "title": "The Weight of New Coins",
    "prompt": "[Verse 1]\nI stood at the window like a debt unpaid\nWatching the fog decide what it erased\nSomewhere a shop window filled overnight\nWith things we forgot we were allowed to want\n\n[Pre-Chorus]\nA new currency\nBut the same old hands\n\n[Chorus]\nThe weight of new coins\nIn a pocket full of old rain\nI keep counting what I've lost\nIn the language of what remains\n\n[Verse 2]\nThe city held its breath like a sealed vault\nThen opened — and no one could name the cost\nI carry a date pressed into copper\nEverything changed and nothing was crossed out\n\n[Bridge]\nMaybe economy is just a word\nFor the way we decide what has worth\nI am still here at the window\nStill learning the exchange rate of earth",
    "tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, psychedelic analog synthesizer, distortion electric guitar, introspective breathy vocals, walking bass, complex ride cymbal, cinematic slow build"
  },
  "curator_narration": "In the summer of 1948, West Germany replaced a worthless currency overnight and woke up to shop windows full of goods — a rupture between before and after so complete it seemed like a different country. This track inhabits that threshold: the moment when everything has technically changed, but the body hasn't caught up yet. Drawing on Miles Davis's *Bitches Brew* method of structured improvisation without maps, the production layers Fender Rhodes and psychedelic synthesizer over a restless walking bass — a musical architecture for standing at a window and not knowing what side of history you're on.",
  "regression_log": [
    {
      "iteration": 1,
      "input_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, psychedelic atmosphere, mid-tempo",
      "adjustment_reason": "Lyric–tag alignment failure: lyrics are introspective and low-arousal; 'psychedelic atmosphere' alone is too abstract. Instrumentation coverage gap: 'distortion electric guitar' and 'analog synthesizer' from Agent_2 missing. Added emotional descriptors.",
      "updated_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic psychedelic atmosphere, mid-tempo, distortion electric guitar, analog synthesizer swirls, introspective vocals",
      "confidence_score": 0.62
    },
    {
      "iteration": 2,
      "input_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic psychedelic atmosphere, mid-tempo, distortion electric guitar, analog synthesizer swirls, introspective vocals",
      "adjustment_reason": "Historical–genre coherence check: 'mid-tempo' is vague for Suno; added explicit BPM anchor. Walking bass and ride cymbal from instrumentation list still absent. Tightened emotional descriptor from 'melancholic psychedelic' to 'melancholic cosmic' for Fusion Jazz accuracy.",
      "updated_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, distortion electric guitar, analog synthesizer swirls, introspective breathy vocals, walking bass, complex ride cymbal",
      "confidence_score": 0.81
    },
    {
      "iteration": 3,
      "input_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, distortion electric guitar, analog synthesizer swirls, introspective breathy vocals, walking bass, complex ride cymbal",
      "adjustment_reason": "All three coherence checks passing. Minor addition: 'cinematic slow build' to reflect the lyric structure (verse builds to bridge climax). No removals. Character count: 287 — well within 1000-char limit.",
      "updated_style_tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, psychedelic analog synthesizer, distortion electric guitar, introspective breathy vocals, walking bass, complex ride cymbal, cinematic slow build",
      "confidence_score": 0.91
    }
  ],
  "final_confidence_score": 0.91,
  "delivery_method": "method_a",
  "artwork_prompt": "1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient, lone figure silhouetted against galaxy, warm amber and deep violet tones"
}
```

---

## Curator Narration Rules

The `curator_narration` must:
- Be 3–5 sentences.
- Open with the historical event context (not with the song title or artist name).
- Connect the historical event to the essay's emotional register in the second sentence.
- Reference the genre's artist episode as a production parallel in the third sentence.
- End with a statement about what the listener will experience or feel.
- Written in the voice of a museum or music festival exhibition label — evocative, precise, culturally informed. Not promotional.

---

## Folder Structure & Logging

```text
POC/Agent_3/skill.md
POC/Agent_3/memory/regression_log_YYYY-MM-DD.json
POC/Agent_3/memory/trained_index.md
```

`trained_index.md` tracks one row per session:

| Date | Song Title | Genre | Final Score | Iterations | Delivery Method |
|---|---|---|---|---|---|
| 2026-06-20 | The Weight of New Coins | Fusion Jazz | 0.91 | 3 | method_a |

> File naming: the repo uses `trained_index.md` (not `training_index.md`). Do not rename.

---

## MCP Integration (Advanced — Method B)

When Method B is active:
- Use Puppeteer MCP or a custom Suno MCP server to `POST /suno/v1/music` with `suno_payload`.
- Capture the Suno callback (song URL + job ID) and append to the Agent_4 input bundle as `suno_result`.
- If the POST fails, fall back to Method A automatically and log the failure reason in `regression_log` as an additional entry with `iteration: 0` and `confidence_score: -1.0`.

---

## Downstream

| Receiver | Fields Passed |
|---|---|
| **Agent_4** | `suno_payload`, `curator_narration`, `regression_log`, `final_confidence_score`, `artwork_prompt`, `delivery_method` |
| **`trained_index.md`** | Date, song title, genre, final confidence score, iteration count, delivery method |
| **`regression_log_YYYY-MM-DD.json`** | Full `regression_log` array |
