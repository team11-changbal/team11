# Life Soundtrack — Team 11

> **"Turn any life moment into a song you'll keep forever."**

Music as a new communication medium — not entertainment.

---

## What It Is

Life Soundtrack is a two-way social music platform. Users write a personal essay (10–15 sentences) about a life moment — a graduation, a breakup, a quiet Tuesday morning — and the system generates a fully produced song: lyrics in 5 languages, a matched music style rooted in cultural and historical context, and a commercial production strategy targeting global streaming.

---

## System Architecture

```
[User: Essay + optional image/video]
        │
        ▼
  Agent_1 — Lyricist & Mood Capturer
  Analyzes emotional tone + historical date context
  → Outputs: lyrics (EN/KO/JA/ZH/ES), mood analysis, lyric hook
        │
        ▼
  Agent_2 — Cultural & Musical Skill Matcher
  Matches mood to 1 of 13 genres from the Skills Database
  → Outputs: genre, Suno style tags, instrumentation, artwork prompt
        │
        ▼
  Agent_3 — Suno API Architect  [pending]
  Builds optimized Suno payload via regression loop (≥3 iterations)
  → Outputs: suno_payload, curator narration, confidence score
        │
        ▼
  Agent_4 — UX Frontend (remix-of-life-soundtrack)
  React/TanStack app — essay input, genre selection, song preview, publish flow
  → repo: team11-changbal/remix-of-life-soundtrack
```

---

## Repository Structure

```
team11/
├── README.md                          ← this file
├── _orchestration_/
│   ├── orchestration_v3.md            ← current spec (v3 supersedes v2)
│   ├── orchestration_v2.md            ← retained for history
│   ├── readme.md                      ← project concept overview
│   ├── Tasks/
│   │   └── agent-alignment-2026-06-20.md   ← UX ↔ Agent contract map
│   ├── Skills/                        ← (genre DB, future expansions)
│   ├── Dokyu/                         ← Suno prompt samples, notes
│   ├── Juhye/                         ← team member notes
│   └── Lobster/                       ← team member notes
├── POC/
│   ├── Agent_1/
│   │   └── Agent_1_skill.md           ← Lyricist & Mood Capturer v2.0
│   ├── Agent_2/
│   │   └── Agent_2_skill.md           ← Cultural & Musical Skill Matcher v2.0
│   ├── Agent_3/
│   │   ├── Agent_3_skill.md           ← Suno API Architect v1.0
│   │   └── memory/
│   │       └── trained_index.md       ← regression session log index
│   └── Agent_4/
│       └── Agent_4_skill.md           ← UX + Global Music Director v2.0
└── Examples/
    ├── Agent_2_DB_skill.md            ← 13-genre Skills Database (reference)
    └── Agent_3_POC_skill.md           ← Agent_3 POC examples
```

---

## Agent Roles

| Agent | Role | Version | Status |
|---|---|---|---|
| Agent_1 | Lyricist & Mood Capturer | v2.0 | Active (POC) |
| Agent_2 | Cultural & Musical Skill Matcher | v2.0 | Active (POC) |
| Agent_3 | Suno API Architect | v1.0 | Pending — API not yet connected |
| Agent_4 | UX Frontend (remix-of-life-soundtrack) | v2.0 | Active (POC) |

---

## Agent_2 — 13-Genre Skills Database

Agent_2 selects exactly one genre per session via 4-step matching (Emotional Quadrant → Historical Era → Lyric Hook Texture → Single Output):

| # | Genre | Era | Mood |
|---|---|---|---|
| 1 | Cool Jazz | 1945–1960 | Dark / Calm |
| 2 | Bebop | 1900–1945 | Dark / Energetic |
| 3 | Hard Bop | 1945–1975 | Dark / Energetic |
| 4 | Fusion Jazz | 1960–1975 | Variable / Mid |
| 5 | Acid Jazz | 1985–2000 | Bright / Energetic |
| 6 | Bossa Nova | 1945–1960 | Calm / Variable |
| 7 | Classical | Pre-1900 | Variable |
| 8 | Modern Pop (Synthpop) | 1975–1985 | Bright / Energetic |
| 9 | Funk | 1960–1975 | Energetic |
| 10 | Disco | 1975–1985 | Bright / Energetic |
| 11 | Soul | 1945–1975 | Variable / Mid |
| 12 | Country | Any | Variable / Mid |
| 13 | Ballad Rock | 1975–Any | Dark / Mid–High |

Full genre specs (instrumentation, Suno style tags, artwork prompts, artist episodes) are in `Examples/Agent_2_DB_skill.md` and embedded in `POC/Agent_2/Agent_2_skill.md`.

---

## UX Flow (Agent_4 — remix-of-life-soundtrack)

1. **Essay Input** — User writes 10–15 sentences + optional image/video
2. **Genre Selection** — Agent_2 matches essay to one genre; Suno style tags auto-fill the vibe prompt (user can override)
3. **Generate** — Agent_3 builds Suno payload (static in POC; API-gated in production)
4. **Pick** — User picks one of 3 candidate songs to post
5. **Publish** — Share to social or publish to Spotify as creator (30-day free trial gate)

---

## Key Design Decisions

- **Vibe prompt = Agent_2 output.** The genre match from Agent_2 auto-fills the vibe prompt with production-ready Suno style tags. Users can type their own override, but the default is Agent_2's selection.
- **"Generate 3 candidate songs" is static in POC.** In production, this button is gated on Agent_3 returning a valid Suno payload with confidence score ≥ 0.75.
- **Agent_3 runs hidden.** Users see only the curator narration and confidence badge — not the regression loop.

---

## Versioning Policy

- Semantic versioning per agent (`v1.0`, `v2.0`, etc.)
- Minor bump: any change to input/output schema
- Major bump: new agent capability
- All pipeline runs logged to `POC/Agent_3/memory/` (once Agent_3 is live)
- Orchestration spec versioned in `_orchestration_/orchestration_vN.md`

---

## Status: POC / Pitch Demo

**Date:** 2026-06-20  
**Current phase:** Proof of Concept — internal demo  
See `_orchestration_/orchestration_v3.md` for full task list and pending items.
