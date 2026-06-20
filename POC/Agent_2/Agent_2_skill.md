# Agent_2: Cultural & Musical Skill Matcher

**Version:** v2.0
**Status:** Active (POC)
**Source spec:** `orchestration_v3.md` (Section 3.2)
**Updated:** 2026-06-20
**Changelog v2.0:**
- Input fields expanded to full table format (field / type / required / description + example)
- Output schema aligned to Orchestration contract (downstream Agent_3 format)
- Skills Database (13 genres) embedded from `Agent_2_DB_skill.md`
- Matching Logic added: Primary Axis → Secondary Axis → Tie-Breaker → Single Genre Output

---

## Task

Match the mood and historical context derived by Agent_1 to the single most artistically synergistic genre from the embedded **SKILLS DATABASE** (13 genres). Apply the 4-step Matching Logic to resolve ambiguity and output exactly one primary genre with full production specs that Suno can act on directly.

## Core Rule

Do not output just a genre name. Draw on the genre's historical background and a representative artist episode (e.g., Charlie Parker's improvisation technique, Giorgio Moroder's synthesizer experiments) to produce concrete instrumentation, tempo, and mixing texture. The `suno_style_tags` field must be production-ready and within 1000 characters.

---

## Input

Receives Agent_1 output: `mood_analysis` + `history_match`.

### Input Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `mood_analysis` | string | **Y** | Emotional tone summary produced by Agent_1. Covers valence (positive/negative), arousal (high/low energy), and dominant feeling words. | `"Melancholic introspection with undercurrent of quiet hope; low arousal, high lyrical density"` |
| `history_match.date_event` | string | **Y** | The historical event Agent_1 matched to today's date or the essay's emotional thread. Used as the **Secondary Axis** in genre matching. | `"June 20, 1948 — West German Deutsche Mark currency reform (Wirtschaftswunder begins)"` |
| `history_match.lyric_hook` | string | **Y** | The metaphoric bridge Agent_1 created between the historical event and the essay's narrative. Informs tonal direction. | `"A new currency in my pocket, but the old debts written on my ribs"` |
| `mood_analysis_scores` | object | N | Optional structured scores if Agent_1 outputs them. Keys: `valence` (−1.0 to +1.0), `arousal` (0.0 to 1.0), `complexity` (0.0 to 1.0). Used to sharpen Primary Axis scoring. | `{"valence": -0.4, "arousal": 0.3, "complexity": 0.8}` |

### Input Schema (JSON)

```json
{
  "mood_analysis": "string",
  "history_match": {
    "date_event": "string",
    "lyric_hook": "string"
  },
  "mood_analysis_scores": {
    "valence": "number | null",
    "arousal": "number | null",
    "complexity": "number | null"
  }
}
```

---

## Matching Logic

Agent_2 resolves genre selection through 4 sequential steps. Each step narrows the candidate pool. The final output is always **exactly one primary genre**.

### Step 1 — Primary Axis: Emotional Valence × Arousal

Map `mood_analysis` to a 2×2 quadrant and pre-select the genre family:

| Quadrant | Valence | Arousal | Genre Family Candidates |
|---|---|---|---|
| **Q1 — Bright & Energetic** | Positive | High | Funk, Disco, Acid Jazz, Modern Pop (Synthpop) |
| **Q2 — Dark & Energetic** | Negative | High | Bebop, Hard Bop, Ballad Rock |
| **Q3 — Dark & Calm** | Negative | Low | Cool Jazz, Bossa Nova, Soul, Classical |
| **Q4 — Bright & Calm** | Positive | Low | Bossa Nova, Country, Fusion Jazz, Soul |

If `mood_analysis_scores` are provided, use numeric thresholds (valence < 0 = Negative; arousal < 0.5 = Low). If scores are absent, derive quadrant from descriptive keywords in `mood_analysis`.

### Step 2 — Secondary Axis: Historical Era Alignment

From the candidates remaining after Step 1, filter by how well the era of the `date_event` maps to the genre's historical origin period:

| Era of `date_event` | Preferred Genre Match |
|---|---|
| Pre-1900 / Classical Age | Classical |
| 1900–1945 (World Wars, Depression) | Bebop, Hard Bop |
| 1945–1960 (Post-war, Cold War) | Cool Jazz, Bossa Nova |
| 1960–1975 (Counterculture, Civil Rights) | Fusion Jazz, Soul, Funk, Hard Bop |
| 1975–1985 (Disco, MTV Era) | Disco, Modern Pop (Synthpop), Ballad Rock |
| 1985–2000 (Post Cold War, Rave) | Acid Jazz |
| Any era (universal narrative) | Country, Soul, Ballad Rock |

Score each remaining candidate +1 for era alignment. Eliminate candidates with 0 alignment score unless only one candidate remains.

### Step 3 — Tie-Breaker: Lyric Hook Texture

If two or more genres remain after Step 2 with equal scores, resolve by matching the dominant texture keyword in `lyric_hook` against genre texture signatures:

| Lyric Hook Texture Keyword | Preferred Genre |
|---|---|
| Urban / city / street / neon | Cool Jazz, Acid Jazz, Modern Pop |
| Nature / land / road / dust | Country, Bossa Nova |
| Body / physical / pulse / beat | Funk, Disco, Hard Bop |
| Memory / time / past / echo | Soul, Bossa Nova, Classical |
| Space / cosmos / machine / electric | Fusion Jazz, Modern Pop, Disco |
| Freedom / justice / resistance | Hard Bop, Soul, Funk, Ballad Rock |
| Intimacy / whisper / breath | Cool Jazz, Bossa Nova, Soul |
| Chaos / fracture / speed | Bebop, Ballad Rock |

Select the genre whose texture signature has the greatest overlap with the hook's keywords.

### Step 4 — Output Single Primary Genre

Output exactly one genre. Populate all output fields from the matched entry in the **SKILLS DATABASE** below. Include a `match_rationale` field explaining which axis/tie-breaker resolved the selection.

---

## Output

### Output Field Specification

| Field | Type | Required | Description | Example |
|---|---|---|---|---|
| `genre` | string | **Y** | Single matched genre name (English canonical name from Skills DB) | `"Fusion Jazz"` |
| `genre_id` | string | **Y** | Skills DB identifier | `"SKILLS_4"` |
| `tempo_bpm` | number | **Y** | Recommended BPM. Derived from genre DB entry + arousal score adjustment (±10 BPM). | `98` |
| `instrumentation` | array[string] | **Y** | Ordered list of key instruments for Suno. Most prominent instrument first. | `["Fender Rhodes electric piano", "distortion electric guitar", "analog synthesizer", "funk bassline"]` |
| `suno_style_tags` | string | **Y** | Production-ready comma-separated tag string for Suno `tags` field. Max 1000 chars. | `"Fusion jazz, jazz rock, fender rhodes electric piano, psychedelic atmosphere, mid-tempo"` |
| `artist_episode` | string | **Y** | 2–4 sentence narrative about a representative artist. Used by Agent_3 for curator narration seed. | `"Miles Davis entered the 1969 Bitches Brew sessions without sheet music..."` |
| `artwork_prompt` | string | **Y** | English image generation prompt (Midjourney / DALL-E ready) derived from Skills DB visual concept. | `"1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient"` |
| `match_rationale` | string | **Y** | Step-by-step explanation of which axes resolved genre selection. Required for Agent_3 regression log seeding. | `"Q3 (dark/calm) → era 1960–1975 aligned → lyric hook 'space/machine' texture → Fusion Jazz selected"` |
| `historical_context` | string | **Y** | 2–3 sentence background on the matched genre's historical era, for curator narration. | `"Late 1960s counterculture pushed jazz to embrace electric instruments..."` |

### Output Schema — Orchestration Contract (JSON)

This schema is the **canonical contract** passed to Agent_3. All fields must be present; `null` is not permitted.

```json
{
  "genre": "string",
  "genre_id": "string",
  "tempo_bpm": "number",
  "instrumentation": ["string"],
  "suno_style_tags": "string (max 1000 chars)",
  "artist_episode": "string",
  "artwork_prompt": "string",
  "match_rationale": "string",
  "historical_context": "string"
}
```

### Example Output

```json
{
  "genre": "Fusion Jazz",
  "genre_id": "SKILLS_4",
  "tempo_bpm": 98,
  "instrumentation": [
    "Fender Rhodes electric piano",
    "distortion electric guitar",
    "analog synthesizer swirls",
    "funk bassline",
    "complex ride cymbal"
  ],
  "suno_style_tags": "Fusion jazz, jazz rock, psychedelic jazz, fender rhodes electric piano, distortion electric guitar, funk bassline, analog synthesizer swirls, complex time signatures, cosmic atmosphere, mid-tempo, melancholic yet expansive",
  "artist_episode": "Miles Davis entered the 1969 Bitches Brew sessions without sheet music, telling the assembled musicians to 'just crash into each other.' Chick Corea and Herbie Hancock layered Fender Rhodes and synthesizer textures over each other like paint poured on a moving canvas — the result redefined what a jazz record could be.",
  "artwork_prompt": "1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient, lone figure silhouetted against galaxy, warm amber and deep violet tones",
  "match_rationale": "Mood mapped to Q3 (negative valence, low arousal). Date event 'June 20, 1948 Wirtschaftswunder' aligned to 1945–1960 era → Cool Jazz and Bossa Nova candidates. Lyric hook contained 'cosmic / machine / new currency' texture keywords → Fusion Jazz selected via Step 3 tie-breaker over Cool Jazz.",
  "historical_context": "Fusion Jazz emerged in the late 1960s as the Vietnam-era counterculture pressed jazz musicians to abandon acoustic tradition and embrace electric technology. Miles Davis's 1969 pivot permanently split jazz into 'before' and 'after,' making electric experimentation the language of a generation searching for new frameworks — mirroring any era of economic or social reinvention."
}
```

---

## SKILLS DATABASE — 13 Genres

*Embedded from `Agent_2_DB_skill.md`. This is the authoritative reference for all genre matching. Do not substitute genre data from outside this table.*

---

### SKILLS_1 — Cool Jazz

**Era:** Late 1940s–1950s. Post-WWII, early Cold War. Urban West Coast modernism.

**Historical Context:** As the shock of World War II faded and the Cold War settled in, audiences grew weary of bebop's ferocity and sought the city's intellectual calm. West Coast America developed a refined modernism and urban optimism that shaped this understated, cerebral sound.

**Artist Episode:** Miles Davis proved with *Birth of the Cool* (1949) that the silence between notes carries more power than the notes themselves. Chet Baker sang with a fragile, lyrical voice forged in solitude and narcotics, becoming the icon of melancholy urban youth.

**Tempo Range:** 80–110 BPM

**Instrumentation:** Muted trumpet, brushed snare drum, upright walking bass, sophisticated chord piano, occasional flugelhorn

**Suno Style Tags:** `Cool jazz, west coast jazz, subdued trumpet, brushed snare, walking bass, intimate, smoky vocals, sophisticated piano chords, mid-tempo, melancholic yet calm`

**Artwork Prompt:** `1950s cinematic, moody blue lighting, smoky jazz club aesthetic, film grain, rain-slicked New York street, trench-coated trumpet silhouette, low-fi monochrome`

**Primary Axis:** Q3 (dark/calm) | **Era Alignment:** 1945–1960 | **Texture:** Urban / intimacy / whisper

---

### SKILLS_2 — Bebop

**Era:** Early–mid 1940s. WWII dissolution of big bands; underground Harlem club scene.

**Historical Context:** The AFM recording strike (1942–1944) pushed bebop underground into Harlem clubs like Minton's Playhouse, away from mainstream media. It was the first jazz movement conceived as pure art for listening, not dancing — a radical break.

**Artist Episode:** Charlie Parker dismembered chord changes at lightning speed using only intuition and harmonic genius, never reading from sheet music. Dizzy Gillespie played a 45-degree-bent trumpet with cheeks inflated to their limit, physically embodying music at the edge of human capacity.

**Tempo Range:** 180–300 BPM

**Instrumentation:** Alto/tenor saxophone, trumpet, upright bass, ride cymbal, minimal piano comping

**Suno Style Tags:** `Bebop, frenetic jazz, fast tempo, complex saxophone improvisation, virtuoso trumpet solos, exploding drum fills, walking double bass, polyrhythmic, unpredictable chord changes`

**Artwork Prompt:** `Abstract expressionism, dynamic neon paint splatters, cubism jazz art, high energy visual noise, fractured saxophone and trumpet shapes, Jackson Pollock drip style`

**Primary Axis:** Q2 (dark/energetic) | **Era Alignment:** 1900–1945 | **Texture:** Chaos / fracture / speed

---

### SKILLS_3 — Hard Bop

**Era:** Mid-1950s–early 1960s. Birth of the Civil Rights Movement; LP record era.

**Historical Context:** Black musicians pushed back against the white-dominated Cool Jazz scene by re-injecting African identity, gospel, and blues into jazz. The LP format enabled longer improvisational arcs, and the emotional stakes of civil rights gave the music moral urgency.

**Artist Episode:** Art Blakey drove the Jazz Messengers with thunderous, gospel-rooted drumming — he called jazz "music that sweeps the dust away" — and detonated the stage with raw African rhythmic energy that shook audiences to their core.

**Tempo Range:** 120–180 BPM

**Instrumentation:** Tenor saxophone, trumpet, Hammond organ, heavy snare, walking bass with gospel feel

**Suno Style Tags:** `Hard bop, soulful jazz, gospel inflected, heavy blues scale, powerful driving drums, gritty tenor sax, intense hard-driving rhythm section, passionate, earthy`

**Artwork Prompt:** `Vintage Blue Note records cover style, high-contrast duotone, gritty monochrome portrait, mid-century graphic design, deep red and black, bold typography`

**Primary Axis:** Q2 (dark/energetic) | **Era Alignment:** 1945–1960 / 1960–1975 | **Texture:** Freedom / resistance / body

---

### SKILLS_4 — Fusion Jazz

**Era:** Late 1960s–1970s. Vietnam War counterculture; electric instrument revolution.

**Historical Context:** As rock electrified the youth, jazz made a historic decision to abandon acoustic instruments and absorb Fender Rhodes, synthesizers, and distortion. The result was a psychedelic collision of jazz harmony and rock rhythm that defined a generation seeking new frameworks.

**Artist Episode:** Miles Davis walked into the 1969 *Bitches Brew* sessions without scores, telling musicians to simply "crash into each other." Chick Corea and Herbie Hancock built cosmic sound effects on synthesizers and Fender Rhodes that made the record feel like transmissions from another dimension.

**Tempo Range:** 90–130 BPM

**Instrumentation:** Fender Rhodes electric piano, distortion electric guitar, analog synthesizer, electric bass with funk feel, complex ride cymbal

**Suno Style Tags:** `Fusion jazz, jazz rock, psychedelic jazz, fender rhodes electric piano, distortion electric guitar, funk bassline, analog synthesizer swirls, complex time signatures, cosmic atmosphere`

**Artwork Prompt:** `1970s sci-fi book cover art, psychedelic space landscape, airbrush synthwave surrealism, cosmic neon gradient, lone figure against galaxy backdrop`

**Primary Axis:** Q3–Q4 (variable/calm-to-mid) | **Era Alignment:** 1960–1975 | **Texture:** Space / cosmos / machine / electric

---

### SKILLS_5 — Acid Jazz

**Era:** Late 1980s–early 1990s. Fall of the Berlin Wall; UK rave culture; sampler technology.

**Historical Context:** British club DJs began sampling 1970s funk and jazz vinyl over house and hip-hop beats as rave culture exploded post-Cold War. The result married the harmonic sophistication of jazz with the body-first imperative of dance music.

**Artist Episode:** Jamiroquai's Jay Kay wore towering indigenous headdresses and delivered groove-drenched environmental anthems, reinterpreting vintage jazz harmony and 1970s string arrangements into a sound that dominated clubs worldwide through the 1990s.

**Tempo Range:** 95–120 BPM

**Instrumentation:** Rhodes electric piano riffs, slap bass guitar, hip-hop breakbeat, bright horn section, smooth lead vocals

**Suno Style Tags:** `Acid jazz, funk groove, urban retro-soul, slap bass, Rhodes electric piano riffs, bright horn section, hip hop breakbeat, danceable jazz, smooth vocals, 90s club vibe`

**Artwork Prompt:** `90s street fashion aesthetic, graffiti pop art, vivid acid colors, VHS lo-fi texture, urban disco club, acid green and purple palette`

**Primary Axis:** Q1 (bright/energetic) | **Era Alignment:** 1985–2000 | **Texture:** Urban / street / pulse

---

### SKILLS_6 — Bossa Nova

**Era:** Late 1950s–early 1960s. Brazilian economic optimism; Brasília construction era.

**Historical Context:** Rio de Janeiro's beach-side middle-class intelligentsia stripped samba of its frenzy and merged it with the harmonic sophistication of Cool Jazz, creating a whispering, intimate sound born of national optimism and afternoon shade.

**Artist Episode:** Antônio Carlos Jobim composed *The Girl from Ipanema* watching the same beautiful stranger walk past his Ipanema café each morning. João Gilberto perfected the bossa nova technique in his room alone — plucking nylon strings so softly that the microphone had to lean in to hear him.

**Tempo Range:** 75–105 BPM

**Instrumentation:** Nylon-string acoustic guitar, whispering warm vocals, soft flute, gentle piano comping, subtle brush percussion

**Suno Style Tags:** `Bossa nova, brazilian samba rhythm, nylon-string acoustic guitar, whispering warm vocals, soft flute, gentle piano comping, swaying rhythm, beach vibe, romantic, nostalgic`

**Artwork Prompt:** `Minimalist pastel watercolor, Rio de Janeiro sunset, retro South American travel poster aesthetic, warm sun rays, palm silhouettes, sand and ocean haze`

**Primary Axis:** Q3–Q4 (calm, variable valence) | **Era Alignment:** 1945–1960 | **Texture:** Nature / intimacy / whisper / memory

---

### SKILLS_7 — Classical (Classicism)

**Era:** 1750s–1820s. Enlightenment; French Revolution; American independence.

**Historical Context:** The bourgeois class rose as royal patronage collapsed. Music shed baroque ornamentation and established the sonata form — balanced, clear, universally beautiful — as the new democratic aesthetic. Music became the domain of human triumph rather than aristocratic decoration.

**Artist Episode:** Mozart left court servitude to become history's first freelance musician, opening a popular music market in Vienna. Beethoven, going deaf, encoded the spirit of the French Revolution — liberté, égalité, fraternité — directly into symphonic architecture, elevating music to a vessel for the human spirit.

**Tempo Range:** 60–140 BPM (highly variable by movement)

**Instrumentation:** Symphonic string section, grand piano arpeggios, oboe, French horn, timpani, balanced melody lines

**Suno Style Tags:** `Classical era, symphonic orchestra, sonata form, elegant string quartet, grand piano arpeggios, balanced melody, majestically structured, staccato violins, timpani accent`

**Artwork Prompt:** `Neoclassical oil painting, hyper-detailed marble architecture, dramatic chiaroscuro lighting, dark academic aesthetic, candlelit orchestra hall`

**Primary Axis:** Q3–Q4 | **Era Alignment:** Pre-1900 | **Texture:** Memory / structure / freedom / time

---

### SKILLS_8 — Modern Pop (80s Synthpop)

**Era:** 1980s. MTV launch (1981); Walkman era; Reaganomics consumerism.

**Historical Context:** MTV transformed music from sound into spectacle. The cassette and Walkman made music personal and portable. The Reagan-era economic boom and conspicuous consumption created the commercial pop-star industrial system built on maximum catchiness and visual impact.

**Artist Episode:** Michael Jackson's *Thriller* video introduced cinematic narrative to music promotion and broke racial barriers on MTV. Quincy Jones used the Jupiter-8 synthesizer and LinnDrum machine to engineer pop perfection — every element scientifically calculated for maximum impact.

**Tempo Range:** 110–135 BPM

**Instrumentation:** Analog synthesizer bassline, LinnDrum drum machine, bright electric guitar hooks, reverb-drenched snare, layered backing vocals

**Suno Style Tags:** `1980s synthpop, retro pop, vintage drum machine LinnDrum, glowing synthesizer bassline, bright electric guitar hooks, reverb-drenched snare, catchy anthem chorus, energetic lead vocals`

**Artwork Prompt:** `Synthwave grid background, neon wireframe, 1980s retro-futurism, glowing vector lines, cyberpunk palette, neon pink and cyan explosion`

**Primary Axis:** Q1 (bright/energetic) | **Era Alignment:** 1975–1985 | **Texture:** Urban / space / neon / machine

---

### SKILLS_9 — Funk

**Era:** Mid-1960s–1970s. MLK assassination; Black Power movement.

**Historical Context:** Following the assassination of Martin Luther King Jr. in 1968, the Black community channeled grief and defiance into the most physically confrontational groove ever recorded — melody was secondary; the bassline and the beat were the message.

**Artist Episode:** James Brown fined band members for missing a single beat — his perfectionism was a statement. He invented 'The One' beat, where every instrument accents the first downbeat together like a collective fist, laying the rhythmic foundation for hip-hop and disco alike.

**Tempo Range:** 90–120 BPM

**Instrumentation:** Slap bass guitar, staccato rhythm guitar scratching, punchy brass horns, tight snare on the one, percussive vocal grunts

**Suno Style Tags:** `Funk, heavy bass groove, slap bass guitar, staccato rhythm guitar scratching, punchy brass horns, percussive vocal grunts, infectious dance syncopation, raw energy, the one beat`

**Artwork Prompt:** `1970s blaxploitation movie poster style, retro funk comic art, distressed textures, warm orange and brown tones, Afro silhouette, bell-bottom era`

**Primary Axis:** Q1 (bright/energetic) or Q2 (dark/energetic) | **Era Alignment:** 1960–1975 | **Texture:** Body / pulse / resistance / freedom

---

### SKILLS_10 — Disco

**Era:** Mid-to-late 1970s. Oil shock recession; post-Vietnam fatigue; underground minority club culture.

**Historical Context:** New York's marginalized communities — Black, Latino, LGBTQ+ — gathered in underground discotheques to escape economic collapse and social exhaustion. The genre broke into the mainstream through *Saturday Night Fever* (1977) and permanently altered popular music's relationship to dance.

**Artist Episode:** Giorgio Moroder built Donna Summer's *I Feel Love* using only synthesizers and a drum machine — no live musicians — completing the world's first 100% electronic dance track. The blueprint he created became the DNA of techno, house, and all EDM.

**Tempo Range:** 120–140 BPM

**Instrumentation:** Four-on-the-floor kick drum, galloping disco bassline, sweeping orchestral strings, wah-wah guitar, falsetto backing vocals

**Suno Style Tags:** `Disco, four-on-the-floor drum beat, galloping disco bassline, sweeping orchestral strings, funky wah-wah guitar, falsetto backing vocals, mirror ball euphoria, uptempo dance`

**Artwork Prompt:** `1970s disco club dance floor, flashing floor lights, spinning mirrorball, glittering glitter effect, cinematic light flares, saturated jewel tones`

**Primary Axis:** Q1 (bright/energetic) | **Era Alignment:** 1975–1985 | **Texture:** Body / escape / space / neon

---

### SKILLS_11 — Soul

**Era:** Late 1950s–1960s. Civil Rights Act; Motown era; gospel secularization.

**Historical Context:** Gospel's explosive vocal tradition merged with R&B's secular rhythm as African Americans channeled both grief and liberation into a form of music that crossed racial barriers on the charts. Detroit's Motown label proved Black music could conquer white America commercially.

**Artist Episode:** Aretha Franklin transformed Otis Redding's *Respect* into an anthem for the Black women's civil rights movement — her voice was not singing but testifying, a lion's roar for freedom rising from the deepest part of the human chest.

**Tempo Range:** 70–110 BPM

**Instrumentation:** Deep emotional lead vocals, gospel choir backing, warm Hammond organ, expressive brass section, rhythm and blues rhythm guitar

**Suno Style Tags:** `Vintage soul, motown sound, deep emotional vocals, gospel choir backing, warm hammond organ, expressive brass section, rhythm and blues foundation, passionate, heartfelt`

**Artwork Prompt:** `Vintage analog photo, motown recording studio aesthetic, warm sepia lighting, dust and scratches film effect, spinning vinyl record close-up, large broadcast microphone`

**Primary Axis:** Q3 or Q2 | **Era Alignment:** 1945–1960 / 1960–1975 | **Texture:** Memory / freedom / intimacy / body

---

### SKILLS_12 — Country

**Era:** 1920s origin–1950s (Nashville Sound established). Great Depression; rural-to-urban migration.

**Historical Context:** Scottish/Irish immigrant folk tradition met American Southern blues to narrate the hardship of working-class migration from farm to factory. Country has always been about the dignity of loss — and the American highway as both escape and homecoming.

**Artist Episode:** Johnny Cash always wore black onstage. He traveled to Folsom Prison in California to perform for inmates, embodying country's core ethic: stand with the marginalized, the lonely, and the forgotten. His rawness was not aesthetic — it was moral.

**Tempo Range:** 80–120 BPM

**Instrumentation:** Twangy acoustic guitar, pedal steel guitar slide, fiddle solos, honest baritone vocals, simple rhythm section with brushed drums

**Suno Style Tags:** `Country music, nashville sound, twangy acoustic guitar, slide pedal steel guitar, fiddle solos, storytelling lyrics, honest baritone vocals, folksy rhythm, nostalgic rustic vibe`

**Artwork Prompt:** `Americana aesthetic, dust bowl rustic landscape, golden hour lighting, cinematic country road oil painting, worn acoustic guitar and cowboy hat, vintage pickup truck`

**Primary Axis:** Q3–Q4 | **Era Alignment:** Any (universal narrative) | **Texture:** Nature / land / road / memory / dust

---

### SKILLS_13 — Ballad Rock / Power Ballad

**Era:** Late 1970s–1980s. Arena rock stadium era; radio airplay battles; MTV power ballad format.

**Historical Context:** As rock bands filled stadiums, they needed radio-friendly formats to expand beyond core audiences. Fusing classical strings, lyrical piano, and theatrical high-range vocals with heavy electric guitar created the Power Ballad — a format designed for mass emotional release at scale.

**Artist Episode:** Journey's *Don't Stop Believin'* opened with a cinematic keyboard intro and built to a soaring electric guitar solo, narrating the dreams of kids on a midnight train. It became one of the best-selling digital singles of all time decades after its release — proof that emotional architecture never expires.

**Tempo Range:** 70–120 BPM (slow intro, building to climactic sections)

**Instrumentation:** Grand emotional piano intro, soaring electric guitar solos, heavy powerful drums, dramatic string arrangement, passionate high-pitched lead vocals

**Suno Style Tags:** `Power ballad, arena rock, melodic rock, soaring electric guitar solos, grand emotional piano intro, heavy powerful drums, dramatic string arrangement, passionate high-pitched vocals, epic crescendo`

**Artwork Prompt:** `Epic arena rock stadium concert, dramatic lens flare, crowd silhouette with glowing lights, high contrast rock aesthetic, backlit stage silhouette, thousands of raised lighters`

**Primary Axis:** Q2–Q3 | **Era Alignment:** 1975–1985 / any | **Texture:** Chaos / freedom / memory / intimacy

---

## Genre Quick-Reference Matrix

| ID | Genre | Valence | Arousal | Era | Texture Keywords |
|---|---|---|---|---|---|
| SKILLS_1 | Cool Jazz | − | Low | 1945–1960 | urban, intimacy, whisper |
| SKILLS_2 | Bebop | − | High | 1900–1945 | chaos, speed, fracture |
| SKILLS_3 | Hard Bop | − | High | 1945–1975 | resistance, body, freedom |
| SKILLS_4 | Fusion Jazz | −/+ | Mid | 1960–1975 | space, cosmos, machine |
| SKILLS_5 | Acid Jazz | + | High | 1985–2000 | urban, street, pulse |
| SKILLS_6 | Bossa Nova | +/− | Low | 1945–1960 | nature, whisper, memory |
| SKILLS_7 | Classical | −/+ | Low–High | Pre-1900 | structure, time, freedom |
| SKILLS_8 | Modern Pop | + | High | 1975–1985 | neon, machine, urban |
| SKILLS_9 | Funk | +/− | High | 1960–1975 | body, resistance, pulse |
| SKILLS_10 | Disco | + | High | 1975–1985 | body, escape, neon |
| SKILLS_11 | Soul | −/+ | Mid | 1945–1975 | memory, intimacy, freedom |
| SKILLS_12 | Country | −/+ | Mid | Any | land, road, nature, dust |
| SKILLS_13 | Ballad Rock | − | Mid–High | 1975–Any | chaos, memory, intimacy |

---

## Open Items

- Numeric `mood_analysis_scores` from Agent_1 are optional in v2.0; full numeric pipeline planned for v2.1.
- Era scoring weights (Step 2) may be refined once Agent_3 regression logs accumulate training data.
- Cross-language lyric hook texture analysis (non-English hooks) is not yet standardized; English hook translation assumed for matching.

---

## Downstream

Output (full JSON object above) feeds **Agent_3** as:
- `suno_style_tags` → Suno API `tags` field
- `artwork_prompt` → Album art generation (Midjourney/DALL-E)
- `artist_episode` + `historical_context` → Curator narration seed
- `match_rationale` → Agent_3 regression log iteration_1 baseline
