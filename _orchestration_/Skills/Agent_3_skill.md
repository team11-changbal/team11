# Agent_3: Suno API Architect (JSON Bridge / MCP Bridge)

**Version:** v3.0
**Status:** Active (POC)
**Source spec:** `orchestration_v3.md` (Section 3.3)
**API Reference:** `gcui-art/suno-api` · OAS 3.0 · v1.1.0 · `https://suno.gcui.ai/docs`
**Updated:** 2026-06-20

---

## Changelog v3.0 (from v2.1)

| Area | Change |
|---|---|
| API version | Upgraded to reflect **OAS 3.0 / v1.1.0** spec from `suno.gcui.ai/docs` |
| Endpoint count | All **13 endpoints** from the live spec fully documented (v2.1 had 10) |
| New endpoints | `GET /api/persona` added (persona info + clips); `GET /api/clip` schema completed |
| `generate_stems` | Full request/response schema from verified mintlify docs |
| `get_aligned_lyrics` | Full response schema added (word-level timestamps) |
| `concat` | Request/response schema completed |
| `generate_lyrics` | Full request/response schema added |
| `extend_audio` | `continue_at` parameter documented |
| `GET /api/get` | `ids` query param behaviour documented (comma-separated, omit = all) |
| AudioInfo schema | `stem_from_id` field added; `model_name` confirmed; CDN URL pattern documented |
| Error codes | Standardised 400 / 402 / 500 / 503 table across all endpoints |
| Regression logic | Unchanged from v2.1 |
| Curator narration | Unchanged from v2.1 |

---

## Task

Receive lyrics and music style from Agent_1 and Agent_2, run a minimum 3-iteration regression loop to optimise the Suno prompt, then submit via `POST /api/custom_generate`. Manage the full async lifecycle (submit → poll → `streaming`). Produce three outputs: (1) submitted Suno request + result clips, (2) curator narration, (3) full regression log. Fall back to Method A (copy-paste) if credentials are unavailable.

## Core Rule

Never pass Agent_2's raw `suno_style_tags` directly into `tags` without at least one regression iteration. The `prompt` field must preserve Suno section markers (`[Verse]`, `[Chorus]`, `[Bridge]`, etc.) exactly as produced by Agent_1. The `tags` field must be coherent with lyric mood, instrumentation, and historical genre era before submission.

---

## Prerequisites & Environment

| Env Variable | Required | Description |
|---|---|---|
| `SUNO_COOKIE` | **Y** | Cookie from `suno.com` — browser DevTools → Network → any request with `?__clerk_api_version` → Headers → Cookie value. Refresh periodically. |
| `TWOCAPTCHA_KEY` | **Y** | API key from `2captcha.com`. Suno serves hCaptcha; 2Captcha workers solve them automatically. Keep balance topped up. |
| `BROWSER` | **Y** | `chromium` or `firefox`. macOS host recommended — fewest captchas. |
| `BROWSER_HEADLESS` | Y (prod) | Set `true` for server/production environments. |
| `BROWSER_LOCALE` | N | Default `en`. Use `en` or `ru` for best 2Captcha worker coverage. |
| `BROWSER_GHOST_CURSOR` | N | `false` recommended — no measurable captcha reduction. |
| `SUNO_API_BASE_URL` | **Y** | Base URL of deployed `gcui-art/suno-api` instance. Local: `http://localhost:3000`. Vercel: `https://<domain>.vercel.app`. |

**Deployment options:**
- Vercel (one-click): clone repo at `github.com/gcui-art/suno-api`, set env vars in Vercel dashboard
- Local: `git clone https://github.com/gcui-art/suno-api && npm install && npm run dev`
- Docker: `docker compose build && docker compose up` *(GPU acceleration disabled in Docker — slow CPU = use local/Vercel)*

**Health check before any generation call:**
```
GET /api/get_limit
```
Returns `{ "credits_left": N, "period": "day", "monthly_limit": N, "monthly_usage": N }`. If `credits_left: 0` → fall back to Method A.

---

## Input

### Input Field Specification

| Field | Type | Required | Source | Description | Example |
|---|---|---|---|---|---|
| `lyrics_en` | string | **Y** | Agent_1 | Full English lyrics with Suno section markers. Becomes the `prompt` field. Markers (`[Verse 1]`, `[Pre-Chorus]`, `[Chorus]`, `[Verse 2]`, `[Bridge]`) are mandatory — Suno uses them for song structure. | `"[Verse 1]\nI stood at the window like a debt unpaid..."` |
| `song_title` | string | **Y** | Agent_1 | English song title. Becomes `title` in the API request. | `"The Weight of New Coins"` |
| `history_match` | object | **Y** | Agent_1 | `{ date_event, lyric_hook }`. Anchor for curator narration and regression evaluation. | `{"date_event": "June 20, 1948 — Deutsche Mark reform", "lyric_hook": "A new currency in my pocket..."}` |
| `genre` | string | **Y** | Agent_2 | Canonical genre name for coherence checks. | `"Fusion Jazz"` |
| `genre_id` | string | **Y** | Agent_2 | Skills DB ID for era validation. | `"SKILLS_4"` |
| `suno_style_tags` | string | **Y** | Agent_2 | Starting point for regression iteration 1. NOT used directly as the final `tags`. Max 1000 chars. | `"Fusion jazz, jazz rock, fender rhodes electric piano, psychedelic atmosphere, mid-tempo"` |
| `tempo_bpm` | number | **Y** | Agent_2 | Target BPM. Used to check tempo tag coverage in regression. | `98` |
| `instrumentation` | array[string] | **Y** | Agent_2 | Ordered instrument list. All must appear (by name or alias) in final `tags`. | `["Fender Rhodes electric piano", "distortion electric guitar", "analog synthesizer"]` |
| `artist_episode` | string | **Y** | Agent_2 | Representative artist narrative. Primary input for curator narration. | `"Miles Davis entered the 1969 Bitches Brew sessions without sheet music..."` |
| `historical_context` | string | **Y** | Agent_2 | Genre's era background. Secondary input for curator narration. | `"Fusion Jazz emerged in the late 1960s..."` |
| `artwork_prompt` | string | **Y** | Agent_2 | Album art image generation prompt. Passed through unchanged to Agent_4. | `"1970s sci-fi book cover art, psychedelic space landscape..."` |
| `mood_analysis` | string | N | Agent_1 | Emotional tone summary. Used in regression lyric–tag alignment check. | `"Melancholic introspection with undercurrent of quiet hope; low arousal..."` |

### Input Schema (JSON)

```json
{
  "lyrics_en": "string",
  "song_title": "string",
  "history_match": { "date_event": "string", "lyric_hook": "string" },
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

## Task Logic

### Phase 1 — Regression Loop (Min 3, Max 5 Iterations)

Stop when `confidence_score ≥ 0.85` or after 5 iterations. Final `updated_style_tags` → `tags` in the API call.

#### Three Coherence Checks per Iteration

| Check | Pass Criterion | Failure Action |
|---|---|---|
| **Lyric–Tag Alignment** | Tags reflect the emotional register of the lyrics. | Add/swap emotional descriptor tags. |
| **Instrumentation Coverage** | Every instrument in the `instrumentation` array appears (by name or alias) in the tag string. | Insert missing; remove contradictory ones. |
| **Historical–Genre Coherence** | Tags do not contradict the genre's era (e.g. no "808 drums" in Cool Jazz tags). | Replace anachronistic tags with era-accurate alternatives. |

#### Confidence Scoring

| Score | Meaning |
|---|---|
| 0.00–0.49 | Major conflicts; full rewrite needed. |
| 0.50–0.74 | 1–2 checks failing; targeted fixes. |
| 0.75–0.84 | Good; minor tuning only. |
| 0.85–1.00 | Production-ready; stop iterating. |

#### Regression Log Entry Schema

```json
{
  "iteration": 1,
  "input_style_tags": "string",
  "adjustment_reason": "string (must be explicit — e.g. 'Added walking bass — missing from instrumentation list; removed 808 drums — anachronistic for Fusion Jazz era')",
  "updated_style_tags": "string",
  "confidence_score": 0.00
}
```

---

### Phase 2 — API Call: `POST /api/custom_generate`

**This is the primary Life Soundtrack generation endpoint.**

```
POST {SUNO_API_BASE_URL}/api/custom_generate
Content-Type: application/json
```

#### Request Body

| Field | Type | Required | Description | Life Soundtrack Value |
|---|---|---|---|---|
| `prompt` | string | **Y** | Full song lyrics with Suno section markers. Section markers (`[Verse]`, `[Chorus]`, `[Bridge]`, etc.) tell Suno how to structure the song. **Do not** put style descriptions here — they belong in `tags`. | `lyrics_en` from Agent_1 (markers preserved exactly) |
| `tags` | string | **Y** | Comma-separated genre + production descriptors. Primary lever for shaping sound: instrumentation, tempo, vocal style, mood. Max ~1000 chars. Case-insensitive. Specific > generic. | Final `updated_style_tags` from last regression iteration |
| `title` | string | **Y** | Song title. Appears in the `AudioInfo` response. | `song_title` from Agent_1 |
| `make_instrumental` | boolean | N | `true` = ignores `prompt`, generates instrumental only. **Default: `false`**. | Always `false` (lyrics present) |
| `model` | string | N | Suno model. **Default: `chirp-v3-5`**. Current stable model on gcui-art/suno-api. | `"chirp-v3-5"` |
| `wait_audio` | boolean | N | `true` = server waits ≤100s for generation before responding. `false` (default) = returns immediately with clip IDs + `"submitted"` status — use polling. | `false` (always use polling flow) |
| `negative_tags` | string | N | Styles/elements to exclude. E.g. `"no piano, no drums"`. Set only if regression identifies tag pollution. | `null` unless regression requires it |

#### Full Request Payload Example

```json
{
  "prompt": "[Verse 1]\nI stood at the window like a debt unpaid\nWatching the fog decide what it erased\nSomewhere a shop window filled overnight\nWith things we forgot we were allowed to want\n\n[Pre-Chorus]\nA new currency\nBut the same old hands\n\n[Chorus]\nThe weight of new coins\nIn a pocket full of old rain\nI keep counting what I've lost\nIn the language of what remains\n\n[Verse 2]\nThe city held its breath like a sealed vault\nThen opened — and no one could name the cost\nI carry a date pressed into copper\nEverything changed and nothing was crossed out\n\n[Bridge]\nMaybe economy is just a word\nFor the way we decide what has worth\nI am still here at the window\nStill learning the exchange rate of earth",
  "tags": "Fusion jazz, jazz rock, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm, psychedelic analog synthesizer, distortion electric guitar, introspective breathy vocals, walking bass, complex ride cymbal, cinematic slow build",
  "title": "The Weight of New Coins",
  "make_instrumental": false,
  "model": "chirp-v3-5",
  "wait_audio": false,
  "negative_tags": null
}
```

#### Tagging Best Practices

| Goal | Tag Pattern | Example |
|---|---|---|
| Genre + sub-genre | Lead with primary genre | `Fusion jazz, jazz rock` |
| Tempo precision | Explicit BPM > vague labels | `mid-tempo 98bpm` (not just `mid-tempo`) |
| Vocal character | Descriptor + type | `introspective breathy vocals` |
| Mood | Compound descriptors | `melancholic cosmic atmosphere` |
| Instrument specificity | Full instrument name | `fender rhodes electric piano` (not just `piano`) |
| Exclusions | Use `negative_tags`, NOT inside `tags` | `negative_tags: "no heavy metal guitar"` |

---

### Phase 3 — Async Polling: `GET /api/get`

`POST /api/custom_generate` with `wait_audio: false` returns **immediately** with two clip objects at `"submitted"` status. Poll every 5 seconds until both reach `"streaming"`.

```
GET {SUNO_API_BASE_URL}/api/get?ids={id1},{id2}
```

#### AudioInfo Object Schema

| Field | Type | Description |
|---|---|---|
| `id` | string | Clip UUID. Used with `/api/get`, `/api/clip`, `/api/extend_audio`, `/api/generate_stems`, `/api/get_aligned_lyrics`. |
| `status` | string | See lifecycle table below. |
| `audio_url` | string \| null | CDN MP3 URL. Format: `https://cdn1.suno.ai/{id}.mp3`. Populated at `"streaming"`. |
| `image_url` | string \| null | CDN JPEG album art URL. Populated at `"streaming"`. |
| `title` | string | Mirrors request `title`. |
| `tags` | string | Mirrors request `tags`. |
| `prompt` | string | Mirrors request `prompt` (lyrics). |
| `model_name` | string | Model used. E.g. `"chirp-v3-5"`. |
| `duration` | number \| null | Track length in seconds. Available when complete. |
| `created_at` | string | ISO 8601 creation timestamp. |
| `stem_from_id` | string \| null | Populated for stem clips only — ID of the source song. |

#### Status Lifecycle

| Status | Meaning | Agent_3 Action |
|---|---|---|
| `"submitted"` | Queued, not started. | Continue polling. |
| `"queued"` | Actively in queue. | Continue polling. |
| `"streaming"` | ✅ Ready. `audio_url` + `image_url` populated. | Capture. Stop polling. |
| `"complete"` | Fully processed (equivalent to streaming for most uses). | Capture. Stop polling. |
| `"error"` | Generation failed. | Log; retry once; if second failure → Method A. |

#### Polling Code (JavaScript)

```javascript
const BASE_URL = process.env.SUNO_API_BASE_URL;

async function pollForCompletion(id1, id2, maxAttempts = 60) {
  for (let i = 0; i < maxAttempts; i++) {
    const res = await fetch(`${BASE_URL}/api/get?ids=${id1},${id2}`);
    const data = await res.json();

    const done = s => s === 'streaming' || s === 'complete';
    if (done(data[0].status) && done(data[1].status)) {
      return {
        clip1: { id: data[0].id, audio_url: data[0].audio_url, image_url: data[0].image_url, duration: data[0].duration },
        clip2: { id: data[1].id, audio_url: data[1].audio_url, image_url: data[1].image_url, duration: data[1].duration }
      };
    }
    if (data[0].status === 'error' || data[1].status === 'error') {
      throw new Error(`Generation error: ${JSON.stringify(data)}`);
    }
    await new Promise(r => setTimeout(r, 5000));
  }
  throw new Error('Polling timeout after 5 minutes');
}
```

#### Polling Code (Python)

```python
import time, requests, os

BASE_URL = os.environ.get('SUNO_API_BASE_URL', 'http://localhost:3000')

def poll_for_completion(id1, id2, max_attempts=60):
    done = lambda s: s in ('streaming', 'complete')
    for _ in range(max_attempts):
        data = requests.get(f"{BASE_URL}/api/get?ids={id1},{id2}").json()
        if done(data[0]['status']) and done(data[1]['status']):
            return {
                'clip1': {'id': data[0]['id'], 'audio_url': data[0]['audio_url'],
                          'image_url': data[0]['image_url'], 'duration': data[0]['duration']},
                'clip2': {'id': data[1]['id'], 'audio_url': data[1]['audio_url'],
                          'image_url': data[1]['image_url'], 'duration': data[1]['duration']}
            }
        if data[0]['status'] == 'error' or data[1]['status'] == 'error':
            raise Exception(f"Generation error: {data}")
        time.sleep(5)
    raise Exception('Polling timeout after 5 minutes')
```

---

## All 13 Endpoints — OAS 3.0 v1.1.0 Reference

Base: `{SUNO_API_BASE_URL}`

---

### 1. `POST /api/generate`
Generate audio from a text prompt (Suno writes lyrics and style automatically).

| Param | Type | Req | Description |
|---|---|---|---|
| `prompt` | string | Y | Text description of desired music. Suno generates lyrics and style from this. |
| `make_instrumental` | boolean | N | `true` = no vocals. Default `false`. |
| `wait_audio` | boolean | N | `true` = wait ≤100s for result. Default `false`. |

**Response:** Array of 2 `AudioInfo` objects (same schema as `/api/custom_generate`).
**Life Soundtrack use:** Not used directly — Agent_3 always uses `/api/custom_generate`.

---

### 2. `POST /v1/chat/completions`
OpenAI-compatible wrapper around `/api/generate`. Accepts OpenAI chat message format; returns music generation result.

| Param | Type | Req | Description |
|---|---|---|---|
| `model` | string | Y | Model name (passed through). |
| `messages` | array | Y | OpenAI messages array. Last user message is used as the generation prompt. |
| `stream` | boolean | N | Streaming mode flag. |

**Response:** OpenAI-format chat completion response with music details embedded in assistant message.
**Life Soundtrack use:** Not used directly — enables GPT Action / LLM agent integration as a future extension.

---

### 3. `POST /api/custom_generate` ⭐ PRIMARY ENDPOINT
Generate audio with full control — custom lyrics, style tags, and title (Custom Mode).
*(See full spec above in Phase 2.)*

| Param | Type | Req | Description |
|---|---|---|---|
| `prompt` | string | Y | Song lyrics with structure markers. |
| `tags` | string | Y | Comma-separated style/genre descriptors. Max ~1000 chars. |
| `title` | string | Y | Song title. |
| `make_instrumental` | boolean | N | Default `false`. |
| `model` | string | N | Default `chirp-v3-5`. |
| `wait_audio` | boolean | N | Default `false`. |
| `negative_tags` | string | N | Elements to exclude. |

**Response:** Array of 2 `AudioInfo` objects.

---

### 4. `POST /api/extend_audio`
Extend an existing clip with new lyrics/content from a given timestamp.

| Param | Type | Req | Description |
|---|---|---|---|
| `audio_id` | string | Y | UUID of the clip to extend. |
| `prompt` | string | N | Additional lyrics for the extension. Use structure markers. |
| `continue_at` | number | N | Timestamp in seconds from which to branch (e.g. `30.0`). Default = end of clip. |
| `tags` | string | N | Style tags for the extension. Can differ from original to transition sound. |
| `negative_tags` | string | N | Elements to exclude from the extension. |
| `title` | string | N | Title for the extended clip. |
| `model` | string | N | Default `chirp-v3-5`. |
| `wait_audio` | boolean | N | Default `false`. Poll same as generation. |

**Response:** Array of new `AudioInfo` objects with fresh clip IDs. Poll with `/api/get`.
**Life Soundtrack use:** If generated song is too short (<2:30), extend with the Bridge section. After extension, use `/api/concat` to merge into one file.

```json
{
  "audio_id": "a3f9-c821-49bc-8d02",
  "prompt": "[Bridge]\nMaybe economy is just a word\nFor the way we decide what has worth",
  "tags": "Fusion jazz, fender rhodes electric piano, melancholic cosmic atmosphere, mid-tempo 98bpm",
  "title": "The Weight of New Coins (Extended)",
  "model": "chirp-v3-5",
  "wait_audio": false
}
```

---

### 5. `POST /api/generate_stems`
Separate an existing clip into vocal track + instrumental (backing) track.

| Param | Type | Req | Description |
|---|---|---|---|
| `audio_id` | string | Y | UUID of the Suno song to stem. Must exist in the Suno account. |

**Response:** Array of 2 stem `AudioInfo` objects.

| Field | Type | Description |
|---|---|---|
| `id` | string | Stem clip UUID. |
| `status` | string | `submitted` / `queued` / `streaming` / `complete` / `error`. |
| `created_at` | string | ISO 8601 timestamp. |
| `title` | string | E.g. `"The Weight of New Coins (Vocals)"` / `"...(Instrumental)"`. |
| `stem_from_id` | string | UUID of the original song (matches `audio_id` in request). |
| `duration` | string | Duration in seconds (note: string type in response). |

**Error codes:** 400 (missing `audio_id`), 402 (out of credits), 500, 503.
**Life Soundtrack use:** Agent_4 can request stems for karaoke mode or TikTok instrumental backing.

```json
{ "audio_id": "a3f9-c821-49bc-8d02" }
```

---

### 6. `POST /api/generate_lyrics`
Generate song lyrics only (no music) from a text prompt. Life Soundtrack does not use this — Agent_1 generates lyrics — but it is available for fallback or standalone lyric drafts.

| Param | Type | Req | Description |
|---|---|---|---|
| `prompt` | string | Y | Text description of desired lyrics (topic, mood, style). |

**Response:**

```json
{
  "text": "string (full lyrics with structure markers)",
  "title": "string (suggested song title)",
  "status": "string"
}
```

---

### 7. `GET /api/get`
Get audio information for one or more clips. Used for polling generation status and retrieving final `audio_url` + `image_url`.

| Query Param | Type | Req | Description |
|---|---|---|---|
| `ids` | string | N | Comma-separated clip UUIDs. E.g. `?ids=uuid1,uuid2`. **If omitted, returns all clips in the account** (use with care on shared accounts). |

**Response:** Array of `AudioInfo` objects (schema above). Poll every 5 seconds until `status = "streaming"` or `"complete"`.

```
GET /api/get?ids=a3f9-c821-49bc-8d02,b7e1-d435-92af-1c88
```

---

### 8. `GET /api/get_limit`
Get current quota information. Run this before any generation call.

**No parameters.**

**Response:**

```json
{
  "credits_left": 50,
  "period": "day",
  "monthly_limit": 50,
  "monthly_usage": 0
}
```

If `credits_left: 0` → fall back to Method A immediately.

---

### 9. `GET /api/get_aligned_lyrics`
Get word-level timestamp data for all lyrics in a clip. Useful for karaoke display, lyric video sync, or Agent_4 TikTok hook zone validation.

| Query Param | Type | Req | Description |
|---|---|---|---|
| `id` | string | Y | Single clip UUID. |

**Response:**

```json
{
  "aligned_lyrics": [
    {
      "word": "string",
      "start_time": 0.00,
      "end_time": 0.00
    }
  ],
  "paragraph_timestamps": [
    {
      "section": "string (e.g. '[Verse 1]')",
      "start_time": 0.00,
      "end_time": 0.00
    }
  ]
}
```

**Life Soundtrack use:** Pass `paragraph_timestamps` to Agent_4 to get precise timestamps for the TikTok hook zone (replaces estimated timestamps from song structure guesses).

---

### 10. `GET /api/clip`
Get detailed information for a single clip by ID.

| Query Param | Type | Req | Description |
|---|---|---|---|
| `id` | string | Y | Single clip UUID. |

**Response:** Single `AudioInfo` object (same schema as `/api/get` items).
**Life Soundtrack use:** Retrieve individual clip details when only one ID is needed (e.g., after clip selection by user).

---

### 11. `POST /api/concat`
Stitch a base clip and its extensions into a single complete audio file.

| Param | Type | Req | Description |
|---|---|---|---|
| `clip_id` | string | Y | UUID of the base clip (original generation). Extensions are automatically discovered from Suno's account history. |

**Response:** Single `AudioInfo` object for the merged clip.
**Life Soundtrack use:** After `/api/extend_audio`, call `/api/concat` to produce the final full-length track.

```json
{ "clip_id": "a3f9-c821-49bc-8d02" }
```

---

### 12. `GET /api/persona`
Get persona information and the clips associated with a persona. Personas allow consistent vocal style across multiple generations.

| Query Param | Type | Req | Description |
|---|---|---|---|
| `id` | string | N | Persona UUID. If omitted, returns all personas in the account. |

**Response:**

```json
[
  {
    "id": "string (persona UUID)",
    "name": "string",
    "description": "string",
    "image_url": "string | null",
    "clips": [
      {
        "id": "string (clip UUID)",
        "title": "string",
        "audio_url": "string",
        "image_url": "string"
      }
    ]
  }
]
```

**Persona Workflow (for consistent vocal style across sessions):**
1. Generate a song via `/api/custom_generate` — get `clip_id`.
2. Note the `clip_id` of the generation you want to use as the vocal reference.
3. Retrieve persona via `GET /api/persona` — use the `id` for subsequent generations.
4. In future `/api/custom_generate` calls, the persona can be referenced via MCP bridge or future API params.

**Note:** Persona-based generation (passing `persona_id` to `/api/custom_generate`) is a V5+ feature per the Suno API ecosystem; confirm availability in the deployed gcui-art/suno-api version.
**Life Soundtrack use:** Optional — persona enables consistent vocal identity across a user's recurring "Life Soundtrack" sessions (future feature).

---

### 13. `POST /api/generate` (OpenAI format alias)
*(See Endpoint 2 — `/v1/chat/completions`)*

---

## Delivery Method Decision

| Condition | Method |
|---|---|
| `SUNO_COOKIE` + `TWOCAPTCHA_KEY` set AND `GET /api/get_limit` returns `credits_left > 0` | **Method B** — Direct `POST /api/custom_generate` + polling |
| Missing credentials OR `credits_left: 0` OR health check fails | **Method A** — Copy-paste UI |

**Method A Output Format** (displayed in UX with individual copy buttons):

```
┌───────────────────────────────────────────────────────────────┐
│  TITLE                                          [Copy]        │
│  The Weight of New Coins                                      │
├───────────────────────────────────────────────────────────────┤
│  STYLE TAGS                                     [Copy]        │
│  Fusion jazz, jazz rock, fender rhodes electric piano,        │
│  melancholic cosmic atmosphere, mid-tempo 98bpm, psychedelic  │
│  analog synthesizer, distortion electric guitar,              │
│  introspective breathy vocals, walking bass, complex ride     │
│  cymbal, cinematic slow build                                 │
├───────────────────────────────────────────────────────────────┤
│  LYRICS                                         [Copy]        │
│  [Verse 1]                                                    │
│  I stood at the window like a debt unpaid                     │
│  ...                                                          │
└───────────────────────────────────────────────────────────────┘
  → Paste into suno.com/create in Custom Mode
```

---

## Output

### Output Field Specification

| Field | Type | Required | Description |
|---|---|---|---|
| `suno_request` | object | **Y** | Exact JSON body sent (or prepared for Method A) for `POST /api/custom_generate`. |
| `suno_result` | object \| null | **Y** | Populated after successful generation + polling. `null` if Method A. |
| `suno_result.clip1` | object | Y (Method B) | `{ id, audio_url, image_url, duration, status }` |
| `suno_result.clip2` | object | Y (Method B) | Same as `clip1`. Agent_4 selects preferred clip. |
| `curator_narration` | string | **Y** | 3–5 sentence gallery-style exhibition label. |
| `regression_log` | array | **Y** | All iteration entries (min 3). Written to memory folder. |
| `final_confidence_score` | number | **Y** | Score from the last regression iteration. |
| `delivery_method` | string | **Y** | `"method_a"` or `"method_b"`. |
| `artwork_prompt` | string | **Y** | Passed through from Agent_2 unchanged. |

### Output Schema — Orchestration Contract (JSON)

```json
{
  "suno_request": {
    "prompt": "string",
    "tags": "string",
    "title": "string",
    "make_instrumental": false,
    "model": "chirp-v3-5",
    "wait_audio": false,
    "negative_tags": "string | null"
  },
  "suno_result": {
    "clip1": {
      "id": "string",
      "audio_url": "string",
      "image_url": "string",
      "duration": "number",
      "status": "string"
    },
    "clip2": {
      "id": "string",
      "audio_url": "string",
      "image_url": "string",
      "duration": "number",
      "status": "string"
    }
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

---

## Curator Narration Rules

- **Length:** 3–5 sentences.
- **Sentence 1:** Open with the historical event context (not the song title or artist name).
- **Sentence 2:** Connect the historical event to the essay's emotional register.
- **Sentence 3:** Reference the genre's artist episode as a production parallel.
- **Final sentence:** Describe what the listener will experience or feel.
- **Voice:** Museum or music festival exhibition label — evocative, precise, culturally informed. Not promotional.

---

## Error Handling

| Error | Detection | Response |
|---|---|---|
| Missing `SUNO_COOKIE` or `TWOCAPTCHA_KEY` | Env check on init | Fall back to Method A |
| `credits_left: 0` | Pre-flight `/api/get_limit` | Fall back to Method A; notify user |
| HTTP 4xx/5xx on POST | HTTP status check | Log; retry once after 10s; second failure → Method A |
| `status: "error"` on clip | Poll loop check | Log clip ID; try second clip; both failed → Method A |
| Polling timeout (60 attempts / ~5 min) | Attempt counter | Log timeout; fall back to Method A |
| Method A fallback | Any of the above | Set `delivery_method: "method_a"`, `suno_result: null` |

---

## Folder Structure & Logging

```text
POC/Agent_3/skill.md
POC/Agent_3/memory/regression_log_YYYY-MM-DD.json
POC/Agent_3/memory/trained_index.md         ← do NOT rename to training_index.md
```

`trained_index.md` row format:

| Date | Song Title | Genre | Model | Score | Iterations | Method | Clip 1 ID | Clip 2 ID |
|---|---|---|---|---|---|---|---|---|
| 2026-06-20 | The Weight of New Coins | Fusion Jazz | chirp-v3-5 | 0.91 | 3 | method_b | a3f9-c821 | b7e1-d435 |

---

## Downstream

| Receiver | Fields |
|---|---|
| **Agent_4** | `suno_request`, `suno_result`, `curator_narration`, `regression_log`, `final_confidence_score`, `artwork_prompt`, `delivery_method` |
| **`trained_index.md`** | Date, title, genre, model, score, iterations, method, clip IDs |
| **`regression_log_YYYY-MM-DD.json`** | Full `regression_log` array |
