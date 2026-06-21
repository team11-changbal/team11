# Agent-to-Agent Alignment: UX Menu ↔ Skill Contract
**Date:** 2026-06-20  
**Version:** v1.0  
**Status:** Active (POC)  
**Owner:** Dokyu  
**Source specs:** `orchestration_v3.md`, `POC/Agent_1–4/skill.md`

---

## Purpose

This document maps every UX menu / input field in `remix-of-life-soundtrack` (Agent_4's frontend) to the exact agent skill contract that owns, produces, or consumes it. Use this as the canonical reference when aligning UX changes to backend agent changes.

---

## Pipeline Overview

```
[User: Essay + optional image/video]
        │
        ▼
  Agent_1  ──────────────────────────────────────────┐
  Lyricist & Mood Capturer (v2.0)                    │
  Output: mood_analysis, history_match, lyrics(×5)   │
        │                                             │
        ▼                                             │
  Agent_2  ─────────────────────────────────┐        │
  Cultural & Musical Skill Matcher (v2.0)   │        │
  Output: genre, suno_style_tags,            │        │
          instrumentation, artwork_prompt    │        │
        │                                   │        │
        ▼                                   │        │
  Agent_3  (PENDING — API not yet live)     │        │
  Suno API Architect (v1.0)                 │        │
  Output: suno_payload, curator_narration,  │        │
          regression_log, confidence_score  │        │
        │                                   │        │
        ▼                                   ▼        ▼
  Agent_4: remix-of-life-soundtrack UX (receives all above)
```

---

## UX Menu → Agent Contract Map

### Screen: `/compose` — AI Compose tab

| UX Element | Field / State | Agent Owner | Role | Status |
|---|---|---|---|---|
| **Essay textarea** | `caption` (state) | **Agent_1** input | `essay_text` — primary emotional source for mood + lyric generation | Active |
| **Image / video upload** | `uploadedPath`, `uploadedType` | **Agent_1** input | `image_url` / `video_url` — optional visual mood augmentation | Active |
| **"Generate one with AI" image field** | `imagePrompt` (state) | **Agent_2** → Agent_4 | Pre-filled by Agent_2 `artwork_prompt` output after genre match. User may override. Until Agent_2 API is live, user types manually. | Static now → Agent_2 auto-fill later |
| **Vibe prompt field** (currently plain input) | `vibePrompt` (state) | **Agent_2** output | Populated by Agent_2 `suno_style_tags` (production-ready Suno tag string, max 1000 chars) after genre match. User may override. User does NOT type this — it is a result of Agent_2's selection from 13-genre Skills DB. | Static now → Agent_2 selection result later |
| **Genre selector** (to be added) | Genre chip / dropdown | **Agent_2** output | UI representation of Agent_2's matched genre (one of 13 from Skills DB). Clicking a genre pre-fills `vibePrompt` with that genre's `suno_style_tags`. | To be built |
| **Visibility selector** | `visibility` | Agent_4 UX only | `"public"` / `"private"` — not passed to any upstream agent | Active |
| **"Generate 3 candidate songs" button** | triggers `generateCandidates()` | **Agent_3** (hidden) | In POC: calls static generation (AI compose without Suno). In production: gated on Agent_3 API availability — button enabled only after Agent_3 returns `suno_payload` with `confidence_score ≥ 0.75`. | Static/clickable now → Agent_3 gated in production |

---

## Agent_1 Contract (UX inputs it consumes)

**Skill file:** `POC/Agent_1/Agent_1_skill.md` v2.0

| Agent_1 Input Field | UX Source | compose.tsx state |
|---|---|---|
| `essay_text` | Essay textarea | `caption` |
| `image_url` | Image upload | `uploadedPath` (when `uploadedType === "image"`) |
| `video_url` | Video upload | `uploadedPath` (when `uploadedType === "video"`) |
| `today_date` | Auto-filled (ISO date) | `new Date().toISOString().slice(0, 10)` passed in `genMut` |

**Agent_1 outputs passed downstream:**

| Output Field | Passed To | Purpose |
|---|---|---|
| `mood_analysis` | Agent_2 | Primary axis for genre matching |
| `mood_analysis_scores` `{valence, arousal, complexity}` | Agent_2 | Numeric quadrant matching |
| `history_match.date_event` | Agent_2, Agent_3 | Historical era alignment for genre |
| `history_match.lyric_hook` | Agent_2, Agent_3 | Tie-breaker texture keyword matching; curator narration seed |
| `song_title` | Agent_3 | Suno API `title` field |
| `lyrics.en` | Agent_3 | Suno API `prompt` field |
| `lyrics.ko/ja/zh/es` | Agent_4 UX | Language switcher in Preview screen (S3) |

---

## Agent_2 Contract (what it produces for the UX)

**Skill file:** `POC/Agent_2/Agent_2_skill.md` v2.0  
**Skills DB:** 13 genres (SKILLS_1 Cool Jazz → SKILLS_13 Ballad Rock)

Agent_2 runs after Agent_1 completes. It selects exactly **one genre** via 4-step matching logic (Emotional Quadrant → Historical Era → Lyric Hook Texture → Single Output).

| Agent_2 Output Field | UX Destination | compose.tsx target |
|---|---|---|
| `genre` | Genre badge / chip displayed in UI | Display label in genre selector |
| `genre_id` | Internal reference | `SKILLS_1` … `SKILLS_13` |
| `suno_style_tags` | **Vibe prompt field** (auto-filled, user-editable) | `vibePrompt` state |
| `instrumentation` | Displayed as instrument tags in candidate card | `SongCard` instrumentation chips |
| `tempo_bpm` | Displayed in candidate card metadata | `SongCard` BPM display |
| `artwork_prompt` | **"Generate one with AI" image field** (auto-filled) | `imagePrompt` state |
| `artist_episode` | Curator narration seed → Agent_3 | Not yet shown in UX (future: Preview card) |
| `historical_context` | Curator narration seed → Agent_3 | Not yet shown in UX (future: Preview card) |
| `match_rationale` | Agent_3 regression log baseline | Not shown in UX (transparent layer) |

### Agent_2 Genre Options (13 genres from Skills DB)

| ID | Genre | Valence | Arousal | Era | Key Suno Style Tags |
|---|---|---|---|---|---|
| SKILLS_1 | Cool Jazz | − | Low | 1945–1960 | cool jazz, west coast jazz, subdued trumpet, brushed snare, walking bass |
| SKILLS_2 | Bebop | − | High | 1900–1945 | bebop, frenetic jazz, fast tempo, complex saxophone improvisation |
| SKILLS_3 | Hard Bop | − | High | 1945–1975 | hard bop, soulful jazz, gospel inflected, heavy blues scale |
| SKILLS_4 | Fusion Jazz | −/+ | Mid | 1960–1975 | fusion jazz, jazz rock, fender rhodes electric piano, psychedelic jazz |
| SKILLS_5 | Acid Jazz | + | High | 1985–2000 | acid jazz, funk groove, urban retro-soul, slap bass, hip hop breakbeat |
| SKILLS_6 | Bossa Nova | +/− | Low | 1945–1960 | bossa nova, nylon-string acoustic guitar, whispering warm vocals, beach vibe |
| SKILLS_7 | Classical | −/+ | Low–High | Pre-1900 | classical era, symphonic orchestra, sonata form, elegant string quartet |
| SKILLS_8 | Modern Pop (Synthpop) | + | High | 1975–1985 | 1980s synthpop, retro pop, vintage drum machine, glowing synthesizer bassline |
| SKILLS_9 | Funk | +/− | High | 1960–1975 | funk, heavy bass groove, slap bass guitar, punchy brass horns |
| SKILLS_10 | Disco | + | High | 1975–1985 | disco, four-on-the-floor drum beat, galloping disco bassline, sweeping orchestral strings |
| SKILLS_11 | Soul | −/+ | Mid | 1945–1975 | vintage soul, motown sound, deep emotional vocals, gospel choir backing |
| SKILLS_12 | Country | −/+ | Mid | Any | country music, nashville sound, twangy acoustic guitar, slide pedal steel guitar |
| SKILLS_13 | Ballad Rock | − | Mid–High | 1975–Any | power ballad, arena rock, soaring electric guitar solos, grand emotional piano intro |

---

## Agent_3 Contract (hidden layer, pending API)

**Skill file:** `POC/Agent_3/Agent_3_skill.md` v1.0  
**Status:** Not yet connected. Static generation runs instead.

| Agent_3 Input | Source |
|---|---|
| `lyrics.en` | Agent_1 |
| `history_match` | Agent_1 |
| `song_title` | Agent_1 |
| `suno_style_tags` | Agent_2 |
| `artwork_prompt` | Agent_2 |
| `artist_episode` + `historical_context` | Agent_2 |

**Agent_3 output that gates the UX:**

| Field | UX Gate |
|---|---|
| `suno_payload` | "Generate 3 candidate songs" button activates only after this is ready |
| `final_confidence_score` | If `< 0.75`, show "Review Required" warning before enabling Generate button |
| `delivery_method` | `method_a` = copy-paste UI; `method_b` = live Suno API spinner |
| `curator_narration` | Displayed in Preview card (S3 state) |

---

## Agent_4 UX State Machine Summary

Per `POC/Agent_4/Agent_4_skill.md` v2.0:

| State | Screen | Entry Condition | Key UX Action |
|---|---|---|---|
| S1 | Essay Input | App launch / new session | User writes essay, uploads media, selects genre (Agent_2 result) |
| S2 | Generating | Essay submitted | Shows agent pipeline progress steps |
| S3 | Preview Music | Pipeline complete, `quality_flag = "approved"` | Shows curator narration, lyrics, genre, TikTok hook zone, artwork |
| S4 | Publish Gate | User clicks Approve | Subscription check (free trial ≤ 30 days passes through) |
| S5 | Share Options | Passed gate | Social share OR Spotify publish paths |
| S6A | Social Share | User selects Social | TikTok / Instagram / X share with pre-filled caption + hook |
| S6B | Spotify Onboarding | User selects Spotify | Account type → Revenue share → Credit setup → Submit |
| S7 | Confirmation | Published | Success card + "Create Another" |

---

## Key Design Decisions (POC Phase)

1. **Vibe prompt is an Agent_2 output, not a user input.** The genre selection from Agent_2's 13-genre Skills DB automatically populates the vibe prompt with `suno_style_tags`. The field remains editable so users can override or type their own free-form style. This is the bridge until Agent_3 is live.

2. **"Generate 3 candidate songs" is static now.** It triggers existing static generation. In commercial release, this button is gated: it becomes active only after Agent_3 returns a valid `suno_payload` with `confidence_score ≥ 0.75`.

3. **"Generate one with AI" image field is pre-filled by Agent_2's `artwork_prompt`.** When Agent_2 pipeline is connected, the image description auto-fills. Until then, users type manually. The field should be read-only (pre-filled) with an optional override.

4. **Agent_3 runs hidden.** Users never see Agent_3's regression loop. They only see the final `curator_narration` in the Preview card and the confidence score badge.

---

## File Change Log

| Date | File | Change | Author |
|---|---|---|---|
| 2026-06-20 | `_orchestration_/Tasks/agent-alignment-2026-06-20.md` | Created — initial Agent 1–4 UX alignment document | Dokyu |
| 2026-06-20 | `POC/Agent_1/Agent_1_skill.md` | Updated to v2.0 — full input/output schema, 5-step task logic | team11 |
| 2026-06-20 | `POC/Agent_2/Agent_2_skill.md` | Updated to v2.0 — 13-genre Skills DB embedded, 4-step matching logic, full output contract | team11 |
| 2026-06-20 | `POC/Agent_3/Agent_3_skill.md` | Drafted v1.0 — Suno payload schema, regression log structure | team11 |
| 2026-06-20 | `POC/Agent_4/Agent_4_skill.md` | Updated to v2.0 — full UX state machine, mixing/mastering direction, TikTok hook zone logic | team11 |
| 2026-06-20 | `_orchestration_/orchestration_v3.md` | Created v3 — supersedes v2; full 4-agent spec with data flow contracts | team11 |

---

## Open Items

- [ ] Agent_2 genre selector UI in `compose.tsx` — replace plain vibe prompt input with genre chip menu (13 options); auto-fill `suno_style_tags` on selection; field remains user-editable
- [ ] Agent_2 `artwork_prompt` → pre-fill `imagePrompt` state in `compose.tsx` after genre selected
- [ ] Agent_3 API connection — gate "Generate 3 candidate songs" button on `suno_payload` + `confidence_score` availability
- [ ] Preview card (S3 state) — add curator narration display, language switcher (EN/KO/JA/ZH/ES), TikTok hook zone highlight
- [ ] Subscription gate (S4) — 30-day free trial logic vs. paywall modal
- [ ] Spotify distributor API integration (S6B) — DistroKid or TuneCore endpoint TBD
- [ ] Agent_3 regression memory logging to `POC/Agent_3/memory/` on each run
