# Agent_4: Global Music Director + UX (US Chart Candidate)

**Version:** v2.0
**Status:** Active (POC)
**Source spec:** `orchestration_v3.md` (Section 3.4)
**Updated:** 2026-06-20
**Changelog v2.0:**
- Input fields expanded to full table format (field / type / required Y/N / description + example)
- Output schema aligned to Orchestration contract (terminal agent — feeds UX render + publish endpoints)
- Commercial strategy logic formalized into 4 task areas
- TikTok hook zone identification rules added
- UX state machine fully specified with subscription gate and dual publish path
- Mixing/mastering direction table added

---

## Task

Receive the complete music concept from Agent_3 and produce the final commercial production package targeting US TikTok virality, Spotify playlist placement, and Billboard chart entry. This is the terminal agent: its output drives both the UX render layer and the publish endpoint integrations.

## Core Rule

Every recommendation must be platform-specific and actionable. Do not output generic music industry advice. The TikTok hook zone must reference a specific timestamp range within the song structure. The mixing direction must name specific techniques. The artwork prompt must be copy-paste ready for Midjourney or DALL-E without modification.

---

## Input

Receives Agent_3 output bundle from the Orchestrator.

### Input Field Specification

| Field | Type | Required | Source Agent | Description | Example |
|---|---|---|---|---|---|
| `suno_payload` | object | **Y** | Agent_3 | Complete Suno API payload including final optimized lyrics and style tags. Used to analyze song structure for TikTok hook zone and mixing direction. | `{"customMode": true, "model": "V5", "title": "The Weight of New Coins", "prompt": "[Verse 1]...", "tags": "Fusion jazz, fender rhodes..."}` |
| `curator_narration` | string | **Y** | Agent_3 | Gallery-style exhibition description. Used verbatim or lightly edited in the UX preview card and Spotify artist bio seed. | `"In the summer of 1948, West Germany replaced a worthless currency overnight..."` |
| `regression_log` | array[object] | **Y** | Agent_3 | Full regression iteration history. Surfaced in the UX transparency panel (optional display). | `[{"iteration": 1, ...}, ...]` |
| `final_confidence_score` | number | **Y** | Agent_3 | Style optimization confidence score (0.0–1.0). Displayed in quality indicator UI. If < 0.75, flag for user review before publish. | `0.91` |
| `artwork_prompt` | string | **Y** | Agent_3 (from Agent_2) | Image generation prompt for album art. Passed directly to the artwork generation call (Midjourney/DALL-E). No modification by Agent_4 except optional style suffix addition. | `"1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient"` |
| `delivery_method` | string | **Y** | Agent_3 | `"method_a"` or `"method_b"`. Determines whether UX shows a "Copy & Paste to Suno" flow or a "Generating…" spinner. | `"method_a"` |
| `genre` | string | N | Agent_2 (via Agent_3 bundle) | Canonical genre name. Used to calibrate platform strategy (e.g., Fusion Jazz targets different Spotify playlists than Disco). | `"Fusion Jazz"` |
| `mood_analysis` | string | N | Agent_1 (via Agent_3 bundle) | Emotional tone summary. Optional input for TikTok caption copy generation. | `"Melancholic introspection with undercurrent of quiet hope..."` |

### Input Schema (JSON)

```json
{
  "suno_payload": {
    "customMode": true,
    "instrumental": false,
    "model": "string",
    "title": "string",
    "prompt": "string",
    "tags": "string"
  },
  "curator_narration": "string",
  "regression_log": ["object"],
  "final_confidence_score": "number",
  "artwork_prompt": "string",
  "delivery_method": "string",
  "genre": "string | null",
  "mood_analysis": "string | null"
}
```

---

## Task Logic — 4 Commercial Production Areas

### Area 1 — Mixing & Mastering Direction

Produce a `mix_master_direction` object with platform-specific technical recommendations:

| Platform | Priority Target | Key Direction |
|---|---|---|
| **TikTok** | Hook clarity in 0–15 sec | Vocal +2dB above mix; low-end roll-off below 80Hz for mobile speaker; transient punch on beat 1 of chorus; no long intro — drop or hook within 5 seconds |
| **Spotify Playlist** | Full dynamic range + streaming loudness | LUFS target −14 LUFS integrated; avoid over-compression; retain dynamic contrast between verse and chorus |
| **Billboard Radio** | Loud, competitive master | LUFS target −9 to −11 LUFS; parallel compression on drums; stereo width enhancement on chorus; clear frequency separation between lead vocal and instrumentation |

Select the primary platform target based on `genre`:
- Jazz/Soul/Classical → Spotify-first
- Pop/Disco/Funk/Acid Jazz → TikTok-first then Spotify
- Ballad Rock/Country → Radio-first (Billboard)
- All genres: always generate all three directions regardless of primary.

### Area 2 — TikTok Hook Zone Identification

Parse the `suno_payload.prompt` lyric structure and identify the 15-second segment most likely to drive TikTok engagement:

**Hook Zone Selection Criteria (in priority order):**
1. **Chorus hook** — If the chorus contains the song title or a repeating line with strong rhythmic emphasis, the chorus opening is the default hook zone.
2. **Pre-Chorus peak** — If the pre-chorus builds to a release moment (pitch climb, dynamic swell), that 15-second window from pre-chorus peak into chorus opening is preferred.
3. **Bridge surprise** — If the bridge contains the song's most unexpected or emotionally intense moment, flag it as an alternate hook zone with rationale.

Output: `tiktok_hook_zone` as a timestamp range (e.g., `"0:45–1:00"`) based on estimated song structure, and `hook_lyric` — the exact lyric line(s) from that zone.

Estimated timing assumptions (for a 3:30 song):
- Verse 1: 0:00–0:40
- Pre-Chorus: 0:40–0:55
- Chorus: 0:55–1:25
- Verse 2: 1:25–2:00
- Pre-Chorus 2: 2:00–2:15
- Chorus 2: 2:15–2:45
- Bridge: 2:45–3:10
- Outro: 3:10–3:30

### Area 3 — Album Artwork Generation

Take `artwork_prompt` from input and:
1. Append a **style suffix** appropriate to the target platform aesthetic: `", square format, 3000x3000px, album cover composition, no text overlays"`.
2. Add a **mood qualifier** derived from `mood_analysis` if provided (e.g., `", melancholic warm tone"` or `", euphoric high-contrast"`).
3. Output the final `final_artwork_prompt` as a fully formed string ready for direct Midjourney `/imagine` or DALL-E API call.

### Area 4 — US Chart & Platform Strategy

Produce a `chart_strategy` object with:
- **Spotify playlist targets:** 3 specific playlist categories most likely to accept the track based on genre + mood (e.g., *"Evening Jazz," "Late Night Mood," "Cinematic Journeys"*).
- **TikTok challenge concept:** One-sentence description of a creator challenge or use-case that fits the hook zone lyric.
- **Release timing note:** General guidance on optimal release day/window (e.g., Fridays for Spotify New Music Friday eligibility).
- **Comparable artists:** 2–3 reference artists whose Spotify audience overlaps with the predicted listener profile, for pitch targeting.

---

## Output

### Output Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `global_title` | string | **Y** | Final confirmed English song title for global release. Identical to `suno_payload.title` unless Agent_4 recommends a revision (must flag reason). | `"The Weight of New Coins"` |
| `mix_master_direction` | object | **Y** | Platform-specific mixing and mastering technical recommendations. Three sub-objects: `tiktok`, `spotify`, `billboard`. | See sub-schema below |
| `tiktok_hook_zone` | object | **Y** | Identified 15-second TikTok hook window. Contains `timestamp_range`, `hook_lyric`, and `rationale`. | `{"timestamp_range": "0:50–1:05", "hook_lyric": "The weight of new coins / in a pocket full of old rain", "rationale": "Pre-chorus to chorus transition; highest pitch contrast and title lyric lands here"}` |
| `final_artwork_prompt` | string | **Y** | Complete image generation prompt including style suffix and mood qualifier. Ready for Midjourney or DALL-E. | `"1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient, lone figure silhouetted against galaxy, warm amber and deep violet tones, melancholic warm tone, square format, 3000x3000px, album cover composition, no text overlays"` |
| `chart_strategy` | object | **Y** | US platform targeting strategy. Contains `spotify_playlists`, `tiktok_challenge`, `release_timing`, `comparable_artists`. | See sub-schema below |
| `curator_narration_final` | string | **Y** | Final curator narration — Agent_3's version reviewed and lightly polished if needed. Used verbatim in the UX preview card. | `"In the summer of 1948, West Germany replaced a worthless currency overnight..."` |
| `ux_publish_spec` | object | **Y** | UX state definitions for the Lovable prototype. Contains the full user journey from essay input to publish, including subscription gate states. | See UX States section below |
| `quality_flag` | string | **Y** | `"approved"` if `final_confidence_score ≥ 0.75`; `"review_required"` if < 0.75. Drives UX gate before publish button is enabled. | `"approved"` |

### Output Schema — Orchestration Contract (JSON)

This is the **terminal contract** — Agent_4 output feeds the UX render layer and publish endpoints directly.

```json
{
  "global_title": "string",
  "mix_master_direction": {
    "tiktok": {
      "lufs_target": "string",
      "vocal_treatment": "string",
      "low_end": "string",
      "intro_rule": "string"
    },
    "spotify": {
      "lufs_target": "string",
      "dynamic_range": "string",
      "compression_note": "string"
    },
    "billboard": {
      "lufs_target": "string",
      "drum_treatment": "string",
      "stereo_width": "string",
      "vocal_separation": "string"
    }
  },
  "tiktok_hook_zone": {
    "timestamp_range": "string",
    "hook_lyric": "string",
    "rationale": "string"
  },
  "final_artwork_prompt": "string",
  "chart_strategy": {
    "spotify_playlists": ["string"],
    "tiktok_challenge": "string",
    "release_timing": "string",
    "comparable_artists": ["string"]
  },
  "curator_narration_final": "string",
  "ux_publish_spec": "object",
  "quality_flag": "string"
}
```

### Example Output

```json
{
  "global_title": "The Weight of New Coins",
  "mix_master_direction": {
    "tiktok": {
      "lufs_target": "−14 LUFS (mobile normalized)",
      "vocal_treatment": "Lead vocal +2dB above mix; slight saturation for warmth on mobile speakers; ensure vocal clarity without reverb muddiness",
      "low_end": "High-pass filter below 80Hz; punchy kick on beat 1 of chorus for hook impact on phone speaker",
      "intro_rule": "No intro longer than 5 seconds before first vocal or melodic hook; consider starting at pre-chorus for TikTok cut"
    },
    "spotify": {
      "lufs_target": "−14 LUFS integrated; −1 dBTP true peak",
      "dynamic_range": "Preserve at least 8 LU dynamic contrast between verse (quieter) and chorus (full); avoid brick-wall limiting",
      "compression_note": "Gentle bus compression (2:1 ratio, slow attack) to glue mix without killing Fender Rhodes dynamics"
    },
    "billboard": {
      "lufs_target": "−10 LUFS integrated for radio competitive loudness",
      "drum_treatment": "Parallel compression on drum bus (NY-style): 8:1 ratio, fast attack, blend 30% wet for punch without loss of transients",
      "stereo_width": "Mid-side processing on chorus: widen synthesizer to 110% stereo; keep bass and kick mono below 200Hz",
      "vocal_separation": "Notch competing instruments at 2–4kHz to create vocal presence pocket; de-esser at 7kHz"
    }
  },
  "tiktok_hook_zone": {
    "timestamp_range": "0:50–1:05",
    "hook_lyric": "The weight of new coins / In a pocket full of old rain",
    "rationale": "Pre-chorus to chorus transition: pitch climbs on 'weight of new coins' then lands on the downbeat of the chorus. The song title appears here, maximizing recall. High contrast between sparse pre-chorus texture and full-band chorus entry creates the dopamine-triggering release moment most likely to trigger TikTok replay behavior."
  },
  "final_artwork_prompt": "1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient, lone figure silhouetted against galaxy, warm amber and deep violet tones, melancholic warm tone, square format, 3000x3000px, album cover composition, no text overlays",
  "chart_strategy": {
    "spotify_playlists": [
      "Late Night Jazz (editorial — introspective jazz for late-night listening; 1.2M followers)",
      "Cinematic Journeys (editorial — instrumental and vocal tracks with filmic quality; 800K followers)",
      "Evening Mood (algorithmic — low-arousal evening listening; targets Fusion Jazz and Soul adjacency)"
    ],
    "tiktok_challenge": "The #NewCoinsChallenge: creators share a 15-second video of something they thought they'd lost but found again — overlaid with the chorus hook — tapping nostalgia and personal reinvention narratives that perform strongly with 25–35 demographic.",
    "release_timing": "Release on a Friday for Spotify New Music Friday eligibility. Pitch to editorial 7 days in advance. For TikTok, seed with 3–5 creator partnerships 48 hours before release to build momentum before the Friday algorithmic push.",
    "comparable_artists": [
      "Norah Jones (mood adjacency: introspective, adult contemporary crossover with jazz roots)",
      "Bonobo (Spotify audience overlap: late-night electronic-acoustic listeners who index on Fusion Jazz)",
      "GoGo Penguin (genre adjacency: contemporary jazz with cinematic emotional palette)"
    ]
  },
  "curator_narration_final": "In the summer of 1948, West Germany replaced a worthless currency overnight and woke up to shop windows full of goods — a rupture between before and after so complete it seemed like a different country. This track inhabits that threshold: the moment when everything has technically changed, but the body hasn't caught up yet. Drawing on Miles Davis's Bitches Brew method of structured improvisation without maps, the production layers Fender Rhodes and psychedelic synthesizer over a restless walking bass — a musical architecture for standing at a window and not knowing what side of history you're on.",
  "quality_flag": "approved"
}
```

---

## UX States — Full Specification

The following state machine defines all screens in the Lovable prototype. Each state has an ID, display name, entry condition, available actions, and exit transitions.

```
STATE MACHINE: Life Soundtrack UX

[S1: Essay Input]
  Entry: App launch / new session
  Display: Text area (10–15 sentence prompt visible), image/video upload button, date auto-filled (today)
  Actions: Submit essay → triggers Orchestrator pipeline
  Exit → [S2: Generating]

[S2: Generating]
  Entry: Essay submitted
  Display: Spinner + progress indicator showing agent pipeline steps
    - "Analyzing your mood..." (Agent_1 running)
    - "Matching your story to history..." (Agent_2 running)
    - "Composing your Suno prompt..." (Agent_3 running)
    - "Preparing your release strategy..." (Agent_4 running)
  Exit (success) → [S3: Preview Music]
  Exit (error) → [S1: Essay Input] with error toast

[S3: Preview Music]
  Entry: Full pipeline complete; quality_flag = "approved"
  Display:
    - Curator narration card (curator_narration_final)
    - Lyrics preview (EN, with language switcher for KO/JA/ZH/ES)
    - Style tags display (final suno_payload.tags)
    - TikTok hook zone highlight (timestamp + hook lyric)
    - Album artwork (generated from final_artwork_prompt)
    - Confidence score badge (final_confidence_score)
  Actions:
    - [Approve] → exit to [S4: Publish Gate]
    - [Regenerate] → return to [S2: Generating] with same essay, new seed
  Note: If quality_flag = "review_required", show warning banner; Approve button still enabled but labeled "Approve Anyway"
  Exit → [S4: Publish Gate]

[S4: Publish Gate]  ← SUBSCRIPTION GATE
  Entry: User clicks Approve in S3
  Condition check:
    - If user is in free trial (≤ 30 days since signup) → pass through to [S5: Share Options]
    - If free trial expired (> 30 days) → show subscription paywall modal
  Paywall Modal:
    - Message: "Your 30-day full-access trial has ended. Subscribe to publish your Life Soundtrack."
    - CTA: [Subscribe — $X/month] → payment flow → on success → [S5: Share Options]
    - Secondary: [Save Draft] → saves current pipeline output; returns to [S1: Essay Input]
  Exit → [S5: Share Options]

[S5: Share Options]
  Entry: User passed subscription gate
  Display: Two publish paths side by side
    Path A: [Share to Social]
    Path B: [Publish to Spotify as Creator]
  Exit A → [S6A: Social Share]
  Exit B → [S6B: Spotify Onboarding]

[S6A: Social Share]
  Entry: User selects Share to Social
  Display:
    - Platform selector: TikTok / Instagram Reels / X / Copy Link
    - Pre-filled caption using hook_lyric + curator_narration excerpt (editable)
    - TikTok hook zone timestamp shown as suggested trim point
    - Download audio button (if Suno song URL available from Method B)
  Actions:
    - [Post to TikTok] → opens TikTok share sheet with audio + caption pre-filled
    - [Copy Link] → copies song URL to clipboard
    - [Done] → return to [S1: Essay Input]
  Exit → [S1: Essay Input]

[S6B: Spotify Onboarding]
  Entry: User selects Publish to Spotify as Creator
  Sub-states:
    [S6B-1: Account Type Selection]
      Display: "Are you publishing as an Individual Artist or on behalf of a Label/Entity?"
      Options: [Individual Artist] / [Label / Entity]
      Exit → [S6B-2: Revenue Share Agreement]

    [S6B-2: Revenue Share Agreement]
      Display: Revenue share terms (platform cut / artist share / Life Soundtrack platform fee)
      Actions: [Agree & Continue] / [Back]
      Exit → [S6B-3: Credit Setup]

    [S6B-3: Credit Setup]
      Display: Fields for:
        - Artist display name
        - ISRC code (auto-generated if blank)
        - Songwriter credits (pre-filled with user name + "AI-assisted via Life Soundtrack")
        - Release date picker
      Actions: [Submit to Spotify] → triggers Spotify distributor API call
      Exit → [S7: Confirmation]

[S7: Confirmation]
  Entry: Social share completed OR Spotify submission sent
  Display:
    - Success message + song title
    - Curator narration card (shareable)
    - "Your Life Soundtrack for [date]" summary
    - [Create Another] → [S1: Essay Input]
    - [View My Library] → user song history (future feature)
```

---

## Open Items

- Spotify distributor API integration (DistroKid or TuneCore endpoint) — not yet selected; placeholder in [S6B-3].
- ISRC auto-generation logic — pending legal/rights review (Task List item 8).
- Revenue share percentage terms — pending business/legal finalization (Task List item 8).
- `comparable_artists` Spotify audience data is currently manually specified by Agent_4; automated Spotify API lookup planned for v2.1.

---

## Downstream

Agent_4 is the **terminal agent**. Its output feeds:

| Consumer | Fields Consumed |
|---|---|
| **UX Render Layer (Lovable)** | `ux_publish_spec`, `curator_narration_final`, `tiktok_hook_zone`, `final_artwork_prompt`, `quality_flag`, `global_title` |
| **Suno API / Copy-Paste UI** | `suno_payload` (via Agent_3; Agent_4 does not re-send — confirms delivery method only) |
| **Spotify Distributor API** | `global_title`, songwriter credits from `ux_publish_spec` [S6B-3] |
| **Social Share Layer** | `tiktok_hook_zone.hook_lyric`, `curator_narration_final`, song URL |
| **Orchestrator Run Log** | `quality_flag`, `final_confidence_score`, `global_title`, `chart_strategy.comparable_artists` |
