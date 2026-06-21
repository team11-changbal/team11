# Agent_1: Lyricist & Mood Capturer

**Version:** v2.0
**Status:** Active (POC)
**Source spec:** `orchestration_v3.md` (Section 3.1)
**Updated:** 2026-06-20
**Changelog v2.0:**
- Input fields expanded to full table format (field / type / required Y/N / description + example)
- Output schema aligned to Orchestration contract (downstream Agent_2 + Agent_3 format)
- Task logic formalized into 5 sequential steps
- Mood analysis scoring guidelines added
- Historical event mapping rules clarified

---

## Task

Analyze the user's essay (emotion, thoughts) and optional visual input, then generate poetic, narrative song lyrics in 5 languages. Metaphorically weave a historical-event timeline — tied to today's date or the essay's core emotional thread — into the lyric background. Output must not be a literal paraphrase of the essay; it must be transformed into a narrative voice.

## Core Rule

Every output must include a `history_match`: a real historical event anchored to `today_date` (or the essay's dominant emotion if no strong date event exists), and a `lyric_hook` — the metaphoric bridge between that event and the essay's emotional narrative. This becomes the cultural spine of the entire downstream pipeline.

---

## Input

Receives directly from the user (via the Orchestrator UI layer): essay text, optional image or video URL, and today's date.

### Input Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `essay_text` | string | **Y** | User's diary or personal essay. Must be 10–15 sentences. Voice-to-text or typed. The primary source of emotional analysis. | `"Today I stood at the window for an hour without knowing why. The city looked like it was holding its breath..."` |
| `image_url` | string \| null | N | URL of an optional image uploaded by the user. Used for color palette and subject analysis to augment mood detection. If null, visual mood is derived from text alone. | `"https://cdn.example.com/user/upload/morning_fog.jpg"` |
| `video_url` | string \| null | N | URL of an optional short video clip. Used for scene/tone extraction if image is absent. At most one of `image_url` or `video_url` should be non-null. | `"https://cdn.example.com/user/upload/rainy_walk.mp4"` |
| `today_date` | string | **Y** | ISO 8601 date string representing the date of the essay entry. Used to look up the historical event anchor. Must be accurate — it drives the `history_match` output. | `"2026-06-20"` |

### Input Schema (JSON)

```json
{
  "essay_text": "string (10–15 sentences)",
  "image_url": "string | null",
  "video_url": "string | null",
  "today_date": "ISO 8601 date string"
}
```

---

## Task Logic — 5 Sequential Steps

### Step 1 — Emotional Tone Analysis

Parse `essay_text` for:
- **Valence:** Is the dominant emotional charge positive, negative, or ambivalent?
- **Arousal:** Is the energy high (urgency, excitement, anger) or low (sadness, peace, fatigue)?
- **Complexity:** How many distinct emotional registers appear? Simple (one) vs. layered (two or more)?
- **Dominant feeling words:** Extract 3–5 keywords that best summarize the emotional texture (e.g., *longing, disorientation, quiet defiance*).

Produce a single-paragraph `mood_analysis` string combining all four dimensions. This is the direct input to Agent_2.

### Step 2 — Visual Mood Augmentation (if image or video provided)

If `image_url` or `video_url` is non-null:
- Analyze dominant color palette (warm/cool, saturated/muted, light/dark).
- Identify subject matter (urban landscape, nature, interior, portrait, etc.).
- Merge visual mood into `mood_analysis` with a brief qualifier (e.g., *"Visual input — muted grey-green palette, foggy urban exterior — reinforces low-arousal melancholy"*).

If both are null, append: *"No visual input provided; mood derived from text only."*

### Step 3 — Historical Event Mapping (Date → Event → Metaphor)

Using `today_date`:
1. Identify one historically significant event that occurred on this calendar date in any year (prioritize events with strong emotional or social resonance — wars ending, revolutions, scientific breakthroughs, cultural milestones).
2. Assess whether the event's emotional register (triumph, grief, rupture, renewal) aligns with or productively contrasts the essay's `mood_analysis`.
3. Craft a `lyric_hook`: a short metaphoric phrase (one sentence, max 20 words) that bridges the historical event to the essay's emotional narrative. This is not exposition — it is a poetic leap.

If no strong calendar event exists for `today_date`, fall back to the essay's dominant emotional theme and select a historically resonant parallel event (any date) that mirrors it. Document the fallback in `history_match.date_event`.

### Step 4 — Lyric Generation

Generate song lyrics that:
- Are **poetic and narrative**, not a summary or literal retelling of the essay.
- Have **clear song structure**: minimum Verse 1, Pre-Chorus, Chorus, Verse 2, Bridge.
- Embed the `lyric_hook` metaphor — the historical event should be felt, not stated.
- Are singable: natural stress patterns, internal rhyme or assonance where appropriate, no forced rhymes.
- Run 80–150 words per language version (suitable for a 3–4 minute song).

### Step 5 — Multilingual Output

Translate and culturally adapt lyrics into all 5 languages simultaneously. Translation must preserve:
- Emotional register (not word-for-word literal).
- Singability (syllable count and stress patterns adjusted per language).
- Cultural idiomatic expression (metaphors adapted if they do not land in the target language).

Languages: EN (source), KO (Korean), JA (Japanese), ZH (Simplified Chinese), ES (Spanish).

---

## Output

### Output Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `mood_analysis` | string | **Y** | Single-paragraph emotional summary. Includes valence, arousal, complexity, dominant feeling words, and (if applicable) visual mood augmentation note. Passed directly to Agent_2. | `"Melancholic introspection with undercurrent of quiet hope; low arousal, high lyrical complexity. Dominant feelings: longing, disorientation, stillness. Visual input (grey-green fog, urban) reinforces low-energy isolation."` |
| `mood_analysis_scores` | object | **Y** | Structured numeric scores derived from Step 1. All values are floats. Passed to Agent_2 for precise quadrant matching. | `{"valence": -0.45, "arousal": 0.28, "complexity": 0.75}` |
| `history_match` | object | **Y** | Contains the historical event anchor and the metaphoric lyric hook. Passed to both Agent_2 and Agent_3. | See sub-fields below |
| `history_match.date_event` | string | **Y** | Description of the historical event matched to `today_date`, including the year. If fallback was used, note it. | `"June 20, 1948 — West German Deutsche Mark currency reform launched the Wirtschaftswunder (Economic Miracle)"` |
| `history_match.lyric_hook` | string | **Y** | One-sentence metaphoric bridge between the historical event and the essay's emotional narrative. Max 20 words. | `"A new currency in my pocket, but the old debts are still written on my ribs."` |
| `song_title` | string | **Y** | Proposed English song title. Evocative, not literal. Passed to Agent_3 for the Suno API `title` field. | `"The Weight of New Coins"` |
| `lyrics` | object | **Y** | Song lyrics in 5 languages. Each value is the full lyric text with structure tags: [Verse 1], [Pre-Chorus], [Chorus], [Verse 2], [Bridge]. | See sub-fields below |
| `lyrics.en` | string | **Y** | English lyrics (source version). | `"[Verse 1]\nI stood at the window like a debt unpaid..."` |
| `lyrics.ko` | string | **Y** | Korean lyrics (culturally adapted). | `"[Verse 1]\n나는 창가에 서서 지워지지 않는 빚처럼..."` |
| `lyrics.ja` | string | **Y** | Japanese lyrics (culturally adapted). | `"[Verse 1]\n窓の前に立ち、消えない借りのように..."` |
| `lyrics.zh` | string | **Y** | Simplified Chinese lyrics (culturally adapted). | `"[Verse 1]\n我站在窗前，像一笔还不清的债..."` |
| `lyrics.es` | string | **Y** | Spanish lyrics (culturally adapted). | `"[Verse 1]\nEstaba en la ventana como una deuda sin saldar..."` |

### Output Schema — Orchestration Contract (JSON)

This schema is the **canonical contract** passed to Agent_2 (mood fields) and Agent_3 (lyrics + history_match). All fields must be present; `null` is not permitted except in input.

```json
{
  "mood_analysis": "string",
  "mood_analysis_scores": {
    "valence": "number",
    "arousal": "number",
    "complexity": "number"
  },
  "history_match": {
    "date_event": "string",
    "lyric_hook": "string"
  },
  "song_title": "string",
  "lyrics": {
    "en": "string",
    "ko": "string",
    "ja": "string",
    "zh": "string",
    "es": "string"
  }
}
```

### Example Output

```json
{
  "mood_analysis": "Melancholic introspection with an undercurrent of quiet hope; low arousal, high lyrical complexity. Dominant feelings: longing, disorientation, stillness. Visual input (grey-green fog, urban exterior) reinforces low-energy isolation with a residual warmth at the horizon.",
  "mood_analysis_scores": {
    "valence": -0.45,
    "arousal": 0.28,
    "complexity": 0.75
  },
  "history_match": {
    "date_event": "June 20, 1948 — West Germany launched the Deutsche Mark, replacing the worthless Reichsmark and igniting the Wirtschaftswunder (Economic Miracle). Overnight, empty shop windows filled with goods.",
    "lyric_hook": "A new currency in my pocket, but the old debts are still written on my ribs."
  },
  "song_title": "The Weight of New Coins",
  "lyrics": {
    "en": "[Verse 1]\nI stood at the window like a debt unpaid\nWatching the fog decide what it erased\nSomewhere a shop window filled overnight\nWith things we forgot we were allowed to want\n\n[Pre-Chorus]\nA new currency\nBut the same old hands\n\n[Chorus]\nThe weight of new coins\nIn a pocket full of old rain\nI keep counting what I've lost\nIn the language of what remains\n\n[Verse 2]\nThe city held its breath like a sealed vault\nThen opened — and no one could name the cost\nI carry a date pressed into copper\nEverything changed and nothing was crossed out\n\n[Bridge]\nMaybe economy is just a word\nFor the way we decide what has worth\nI am still here at the window\nStill learning the exchange rate of earth",
    "ko": "[Verse 1]\n나는 창가에 서서 지워지지 않는 빚처럼\n안개가 무엇을 지울지 바라보았어\n어딘가 진열창엔 하룻밤 사이 물건들이 채워졌고\n우리가 원해도 된다는 걸 잊었던 것들이\n\n[Pre-Chorus]\n새로운 화폐\n하지만 같은 손들\n\n[Chorus]\n새 동전의 무게\n오래된 빗물로 가득 찬 주머니 속에\n나는 잃어버린 것들을 세고 있어\n남아있는 것들의 언어로\n\n[Verse 2]\n도시는 봉인된 금고처럼 숨을 참았다가\n열렸어 — 그리고 아무도 그 대가를 말하지 못했지\n나는 구리에 새겨진 날짜를 품고 다녀\n모든 것이 바뀌었지만 지워진 건 없었어\n\n[Bridge]\n어쩌면 경제란\n우리가 가치를 결정하는 방식의 이름일 뿐\n나는 아직 창가에 있어\n이 땅의 환율을 배우는 중",
    "ja": "[Verse 1]\n窓の前に立ち、消えない借りのように\n霧が何を消すのか見つめていた\nどこかのショーウィンドウは一夜で満たされ\n欲しがってよかったと忘れていたものたちで\n\n[Pre-Chorus]\n新しい通貨\nでも同じ手のひら\n\n[Chorus]\n新しいコインの重さ\n古い雨水が満ちたポケットの中\n失ったものを数え続ける\n残ったものの言葉で\n\n[Verse 2]\n街は封じられた金庫のように息をひそめ\n開いた — そして誰もその代償を言えなかった\n銅に刻まれた日付を胸に抱いている\n何もかも変わったのに、消されたものは何もない\n\n[Bridge]\nもしかしたら経済とは\n価値を決める方法の名前に過ぎない\n私はまだ窓の前にいる\nこの地の為替レートを学びながら",
    "zh": "[Verse 1]\n我站在窗前，像一笔还不清的债\n看着雾决定抹去什么\n某处橱窗在一夜之间被填满\n装着那些我们忘了自己可以渴望的东西\n\n[Pre-Chorus]\n新的货币\n却是同一双手\n\n[Chorus]\n新硬币的重量\n在装满旧雨水的口袋里\n我用剩下的语言\n反复数着失去的一切\n\n[Verse 2]\n城市像一个封存的金库屏住呼吸\n然后打开了 — 没有人能说出代价\n我随身带着压印在铜上的日期\n一切都变了，却没有什么被划去\n\n[Bridge]\n也许经济只是个词\n用来描述我们决定何为价值的方式\n我还站在窗前\n还在学习这片土地的汇率",
    "es": "[Verse 1]\nEstaba en la ventana como una deuda sin saldar\nViendo a la niebla decidir qué borrar\nEn algún lugar una vitrina se llenó de la noche a la mañana\nCon cosas que habíamos olvidado que podíamos querer\n\n[Pre-Chorus]\nUna moneda nueva\nPero las mismas manos\n\n[Chorus]\nEl peso de las monedas nuevas\nEn un bolsillo lleno de lluvia vieja\nSigo contando lo que perdí\nEn el idioma de lo que queda\n\n[Verse 2]\nLa ciudad contuvo el aliento como una caja fuerte sellada\nLuego se abrió — y nadie pudo nombrar el costo\nLlevo una fecha grabada en cobre\nTodo cambió y nada fue tachado\n\n[Bridge]\nQuizás la economía es solo una palabra\nPara la manera en que decidimos qué tiene valor\nSigo aquí en la ventana\nTodavía aprendiendo el tipo de cambio de la tierra"
  }
}
```

---

## Quality Rules

- Lyrics must NOT be a paraphrase of the essay. The essay is raw material; the lyric is the transformed artifact.
- `lyric_hook` must be a genuine metaphoric leap — if it reads like a caption or summary, rewrite it.
- All 5 language versions must be independently singable, not machine-translated word-for-word.
- `song_title` must be evocative and marketable — avoid generic titles like *"My Feelings"* or *"Today"*.
- `mood_analysis_scores` must be populated even if no image is provided; derive from text analysis alone.

---

## Downstream

| Receiver | Fields Passed |
|---|---|
| **Agent_2** | `mood_analysis`, `mood_analysis_scores`, `history_match` |
| **Agent_3** | `lyrics.en`, `history_match`, `song_title` |

Agent_2 and Agent_3 receive their respective slices simultaneously from the Orchestrator after Agent_1 completes.
