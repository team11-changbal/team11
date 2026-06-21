# Life Soundtrack — Orchestration Overview
**Date:** 2026-06-20  
**Version:** v3.0  
**Status:** POC / Pitch Demo  
**Spec:** `orchestration_v3.md`

---

## Concept

**"Turn any life moment into a song you'll keep forever."**

Life Soundtrack is a two-way social music platform where users write a personal essay (10–15 sentences) about a life moment. A multi-agent AI pipeline analyzes the emotional tone, matches it to a historically and culturally grounded music style, and generates a fully produced song — lyrics in 5 languages, Suno-ready style tags, album artwork prompt, and a US chart strategy.

Music as communication medium, not entertainment.

---

## Agent Pipeline — Current Status

| Agent | Role | Skill File | Version | Status |
|---|---|---|---|---|
| Agent_1 | Lyricist & Mood Capturer | `Skills/Agent_1_skill.md` | v2.0 | Active (POC) |
| Agent_2 | Cultural & Musical Skill Matcher | `Skills/Agent_2_skill.md` | v2.0 | Active (POC) |
| Agent_3 | Suno API Architect | `Skills/Agent_3_skill.md` | v1.0 | **done!** — Suno API not yet connected |
| Agent_4 | UX Frontend | `Skills/Agent_4_skill.md` | v2.0 | Active — repo: `remix-of-life-soundtrack` |

---

## Data Flow

```
[User: Essay (10–15 sentences) + optional image/video]
        │
        ▼
  Agent_1 — analyzes emotional tone + historical date context
  → mood_analysis, mood_analysis_scores, history_match, lyrics (×5 languages), song_title
        │
        ├──────────────────────────────────┐
        ▼                                  ▼
  Agent_2                            Agent_3 (receives lyrics + history_match)
  matches mood → 1 of 13 genres      builds Suno API payload via regression (≥3 iterations)
  → genre, suno_style_tags,           → suno_payload, curator_narration,
    instrumentation, artwork_prompt,    confidence_score, regression_log
    artist_episode, match_rationale         │
        │                                  │
        └──────────────┬───────────────────┘
                       ▼
               Agent_4 — UX (remix-of-life-soundtrack)
               essay input → genre selection → generate → pick → publish
```

---

## Folder Structure

```
team11/
├── README.md
└── _orchestration_/
    ├── orchestration_v3.md          ← master spec
    ├── readme_draft.md              ← this file
    └── Skills/
        ├── Agent_1_skill.md         ← v2.0 active
        ├── Agent_2_skill.md         ← v2.0 active
        ├── Agent_3_skill.md         ← v3.0 active
        ├── Agent_4_skill.md         ← v2.0 active
        └── reference/
            ├── Agent_2_DB_skill.md         ← 13-genre Skills Database (full specs)
            ├── Agent_3_POC_skill.md        ← Agent_3 POC examples
            └── agent-alignment-2026-06-20.md ← UX ↔ Agent contract map
```

---

## Agent_2 Skills Database — 13 Genres

Agent_2 selects **exactly one genre** per run via 4-step matching:  
Emotional Quadrant → Historical Era → Lyric Hook Texture → Single Output

| ID | Genre | Era | Valence | Arousal |
|---|---|---|---|---|
| SKILLS_1 | Cool Jazz | 1945–1960 | − | Low |
| SKILLS_2 | Bebop | 1900–1945 | − | High |
| SKILLS_3 | Hard Bop | 1945–1975 | − | High |
| SKILLS_4 | Fusion Jazz | 1960–1975 | −/+ | Mid |
| SKILLS_5 | Acid Jazz | 1985–2000 | + | High |
| SKILLS_6 | Bossa Nova | 1945–1960 | +/− | Low |
| SKILLS_7 | Classical | Pre-1900 | −/+ | Low–High |
| SKILLS_8 | Modern Pop (Synthpop) | 1975–1985 | + | High |
| SKILLS_9 | Funk | 1960–1975 | +/− | High |
| SKILLS_10 | Disco | 1975–1985 | + | High |
| SKILLS_11 | Soul | 1945–1975 | −/+ | Mid |
| SKILLS_12 | Country | Any | −/+ | Mid |
| SKILLS_13 | Ballad Rock | 1975–Any | − | Mid–High |

Full specs (instrumentation, Suno style tags, artwork prompts, artist episodes) → `reference/Agent_2_DB_skill.md`

---

## UX Menu ↔ Agent Contract (Key Decisions)

| UX Element in `compose.tsx` | Agent Owner | Current State | Future State |
|---|---|---|---|
| Essay textarea | Agent_1 input | Active | No change |
| Image / video upload | Agent_1 input | Active | No change |
| "Generate one with AI" image field | Agent_2 → `artwork_prompt` | User types manually | Auto-filled by Agent_2 |
| Vibe prompt field | Agent_2 → `suno_style_tags` | User types manually | Auto-filled by genre selection |
| Genre selector (to be built) | Agent_2 output | Not yet built | Chip menu, 13 genres, pre-fills vibe prompt |
| "Generate 3 candidate songs" button | Agent_3 hidden layer | Static / always clickable | Gated on Agent_3 `confidence_score ≥ 0.75` |

Full UX ↔ Agent contract → `reference/agent-alignment-2026-06-20.md`

---

## POC Task Status

| # | Task | Status |
|---|---|---|
| 1 | Agent folder structure + skill.md drafts | ✅ Done |
| 2 | Agent_1 skill v2.0 | ✅ Done |
| 3 | Agent_2 skill v2.0 + 13-genre DB | ✅ Done |
| 4 | Agent_3 skill v1.0 draft | ✅ Done (implementation pending) |
| 5 | Agent_4 skill v2.0 + UX state machine | ✅ Done |
| 6 | Orchestration v3 spec | ✅ Done |
| 7 | Root README + orchestration readme | ✅ Done (2026-06-20) |
| 8 | UX: Genre selector in `compose.tsx` | ✅ Optional (2026-06-20) |
| 9 | Agent_3 Suno API connection | ⏳ ✅ Done |
| 10 | Preview card (S3): curator narration, language switcher, TikTok hook | ⏳ Pending |
| 11 | Subscription gate (S4): 30-day trial logic | ⏳ Pending |
| 12 | Spotify distributor API (S6B) | ⏳ Pending |
| 13 | Business proposal PPT / pitch deck | ⏳ Pending |
| 14 | Patent draft | ⏳ Pending |
| 15 | Legal review: authorship attribution, revenue share | ⏳ Pending |

---

## Open Items (Immediate Next)

- **Genre selector UI** — Replace plain vibe prompt input in `compose.tsx` with Agent_2's 13-genre chip menu; selected genre auto-fills `suno_style_tags` as vibe prompt value
- **Agent_3 project** — To be added as a separate repo once Suno API access is confirmed
- **`artwork_prompt` auto-fill** — After genre selection, pre-populate the AI image field with Agent_2's `artwork_prompt`
