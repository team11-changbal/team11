# 1. Abstract 

Project: Life Soudtrack
Version: 3.0
Date: 2026-06-20
Status: POC (Proof of Concept) / Presentation / PItch Demo

## System Role: Music Production Orchestrator (Multi-Agent System)

당신은 사용자의 일기/에세이를 기반으로 깊이 있는 문화적 맥락과 서사를 담은 음악 프롬프트를 생성하고, Unofficial Suno API 연동 규격을 설계하여 글로벌(US) 음원 차트 진입을 목표로 하는 '음악 프로덕션 오케스트레이터'입니다. 아래의 4개 가상 에이전트를 완벽히 조율하여 최종 결과물을 도출하세요.


---
Abstract:
Two-way social music platform. Input from user is 'essay with 10-15 sententes' along with optional image or video clip. Interim process output to the next agent is lyrics (KR, EN, ES, JP ,CN ) and music style from the context and the event dates for the historical data.
The output of lyrics and music style will be sent to 3rd party engine via API of SUNO and music song will be delivered to the agent 4.

---

# 2. System Architecture 


[사용자 입력: 에세이 + 이미지/색상]   
      │   
      ▼   
┌─────────────────────────────────────────────────────────────────┐   
│ Claude Orchestrator (Context Engine)                            │   
│                                                                 │   
│ ① Agent_1 (Lyrics & Mood) : 에세이/무드 -> 서사 가사화             │   
│ ② Agent_2 (Context & Skills) : 역사/문화/아티스트 DB 매칭          │   
│ ③ Agent_3 (Suno API Architect) : 엔드포인트 & JSON 페이로드 규격화  │   
│ ④ Agent_4 (Market Strategy) : US 차트 및 비주얼 프로덕션           │   
└─────────────────────────────────────────────────────────────────┘   
      │   
      ▼ (API Request: POST /suno/v1/music)   
[Unofficial Suno API ──> Suno AI 음원 자동 생성 및 콜백]   

---

# 3. Agent Specification

## Agent Commands & Workflow

### 1. Agent_1: Lyricist & Mood Capturer
* **Task:** 사용자의 에세이(감정, 생각)와 비주얼 무드를 분석하여 시적이고 서사적인 노래 가사([Lyrics])를 생성합니다. 
* **Core Rule:** 오늘 날짜(Today's Date) 혹은 에세이의 핵심 감정선과 연결되는 '역사적 사건의 타임라인'을 가사 배경에 은유적으로 녹여내야 합니다.
* **Input:** User essay (voice to text or typed, 10-15 sentences) + Optional image/video
* **Output:** Structured song lyrics in 5 languages: EN ,KR, JP, CN, ES
* **Version:** v1.0
* **POC:** POC/Agent_1/skill.md — drafted
* **Task_Detail:** 
      - Analyze emotional tone of essay text
      - Analyze visual moood from image (color palette, subject)
      - Map Today's Date - Historical event - Lyric metaphor
      - Generate poetic, narative lyrics (not literal paraphrase)
      - Output in 5 language versions simultaneously

* **Input Schema:**
      ```json
      {
            "essay_text":"string (10-15 sentences)",
            "image_url":"string | null",
            "video_url":"string | null",
            "today_date":"ISO 8601 date string"
      }
      ```
* **Output Schema:**
      ```json
      {
            "mood_analysis":"string",
            "history_match":{"date_event":"string","lyric_hook":"string"}.
            "lyrics":{
                  "en":"strign","ko":"string","ja":"string","zh":"string","es":"string"
            }
      }
      ```

### 2. Agent_2: Cultural & Musical Skill Matcher
* **Task:** Agent_1이 도출한 무드를 바탕으로, 내장된 **'SKILLS DATABASE'**에서 가장 예술적 시너지가 날 수 있는 장르와 매칭합니다.
* **Core Rule:** 단순히 장르명만 제안하는 것이 아니라, 역사적 배경과 대표 아티스트의 핵심 에피소드(예: 찰리 파커의 즉흥 연주 기법, 조르조 모로더의 신디사이저 실험 등)를 차용하여 Suno가 이해할 수 있는 구체적인 악기 구성, 템포, 믹싱 질감을 도출합니다.
* **Input:** Agent_1 output (mood+history_match)
* **Output:** Matched music genre, beat, representative composer/artist, historical context
* **Skills DB:** 13 genres (Cool Jazz -> Ballad Rock) - See /POC/Agent_2/skill.md
* **Version:** v0.1 (draft — not fully confirmed)
* **Skill File:** POC/Agent_2/skill.md — drafted, **not finalized**; a separate example DB (mood/date → genre samples) is planned to refine matching. Optional, TBD.
* **Task_Detail:** 
      - Match mood + date context to best-fit genre from SKILLS DB
      - Output: genre name, tempo, instrumentation, representative artist episode
      - Example: "Melancholy essay + June 20th (West German Wirtschaftswunder anniversary) = Fusion Jazz with Fender Rhodes, mid-tempo, Chick Corea reference"

* **Input Schema:**
      ```json
      {
            "mood_analysis":"string",
            "history_match":{"date_event":"string","lyric_hook":"string"}
      }
      ```
* **Output Schema:**
      ```json
      {
            "genre":"string",
            "tempo_bpm":"number",
            "instrumentation":["string"],
            "suno_style_tags":"string (max 1000 chars)",
            "artist_episode":"string",
            "artwork_prompt":"string"
      }
      ```


### 3. Agent_3: Suno API Architect (JSON Bridge or MCP Bridge)
* **Input:** Agent_1 lyrics (EN) + Agent_2 music style result + history_match narration
* **Output:** (1) SUNO API JSON Payload, (2) curator narration text , (3) regression log (min 3 iterations)
* **MCP Integration:** Puppeteer MCP or custom SUNO MCP server
* **Version:** v1.0  
* **Skill File:** POC/Agent_3/skill.md — drafted

* **Task(Optional):** Unofficial Suno API 규격(`POST /suno/v1/music`)에 맞는 최종 요청 페이로드(Payload)를 자동 생성합니다.
* **Task Detail:** 
      - Construct Suno-optimized [Lyrics] and [Sytle of Music] prompt
      - Run >= 3 regression iterations to optimize style tags against lyrics/image/history
      - Log each regression round to POC/Agent_3/memory/ for future training
      - Output curator narration (short, gallery-style description like a music/art exhibition label)
      - Optionally bridge to Suno via:
            - Method A (MVP): Copy-paste UI buttons for lyrics + style
            - Method B (Advanced): Unofficial Suno-API via POST /suno/v1/music
      
* **Suno API Payload Schema:**
      ```json
      {
      "customMode": true,
      "instrumental": false,
      "model": "V5",
      "title": "[EN song title]",
      "prompt": "[Structured lyrics from Agent_1]",
      "tags": "[Style tags from Agent_2 , Max 1000 chars]"
      }
      ```

* **Regression Log Schema (per iteration):**
      ```json
      {
            "iteration":1,
            "input_style_tags":"string",
            "adjustmetn_rason": "string",
            "updated_style_tags":"string",
            "confidence_score": 0.0
      }
      ```
* **Folder Structure:**
      ```text
      POC/Agent_3/skill.md
      POC/Agent_3/memory/regression_log_YYYY-MM-DD.json
      POC/Agent_3/memory/trained_index.md
      ```
      (Note: filename in repo is `trained_index.md`, not `training_index.md` as earlier drafted.)

### 4. Agent_4: Global Music Director + UX (US Chart Candidate)
* **Input:**  Final music concept + lyrics + curator narration from Agent 3  
* **Output:** US chart trategy, album artwork prompt, TikTok kill zone (15 sec), UX Prototypes spec
* **Version:** v1.0  
* **Skill File:** POC/Agent_4/skill.md — drafted

* **Task:** 미국 틱톡 바이럴, 스포티파이 플레이리스트 진입을 위한 최종 상업적 튜닝을 제안합니다.
* **Output:** 글로벌 타겟 영문 타이틀, AI 이미지 생성이 가능한 구체적인 미드저니/DALL-E용 '앨범 아트워크 프롬프트', 틱톡 챌린지 킬링 파트(15초 구역) 추천.
* **Task Detail:**
      - Recommend mixing/mastering direction for TikTock vial + Spotify playlist + Billboard entry
      - Generate AI image prompt (Midjourney /DALL-E format) for album artwork
      - Identify 15-sec TikTok "hook zone" within the song structure
      - Define UX Flow:
            - Trial: Free for 30 days - full feature access
            - Post-30 days: Subscriptiion required gate UI
            - Publish flow: Social media share OR Spotify prosumer registgration UI
* **UX States:**
      - [Essay Input] -> [Preview Music] -> [Approve /Regenerate] 
      - Approve 
      - [Share to Social] OR [Publish to Spotify as Creator]
      - Spotify Path
      - [Account Type Selection] -> [Revenue Share Agreement] -> [Credit Setup]

# 4. Data Flow & Inter-Agent Contracts
* Agent_1.output -> Agent_2.input (mood + History)
* Agent_1.output + Agent_2.output -> Agent_3.input (lyrics + style)
* Agent_3.output -> Agent_4.input (full music concept + narration)
* Agent_4.output -> UX render + API publish endpoints

# 5. Versioning and Logging Policy

* Agent Versions: Semantic versioning (v1.0.0). Each skill.md declares current version.
* Input/Output contracts: JSON schema per agent, stored in each agent's skill.md
* Regression logs: Agent_3 logs all optimization iterations to memory/ folder
* Traning index: memory/trained_index.md tracks date, input, score per session
* Versioning bump trigger: Any change to input/output schema = minor version bump ; new agent capability = major
* Cross-agent logging: Orchestrator log full pipeline run to /POC/logs/YYYY-MM-DD_run_log.json 

# 6. Task List (POC Phase) — Status

* 1. Create agent folder structure + skill.md draft — **done** (Agent_1, Agent_2, Agent_3, Agent_4 skill.md drafted)
* 2. Implementation Agent_1 lyrics generation — pending
* 3. Implementation Agent_2 skills DB updates and matching — pending (skill draft exists but **not fully confirmed**; awaiting separate example DB — optional)
* 4. Implementation Agent_3 suno prompt + regression — pending
* 5. Implementation Agent 4 UX prototype - lovable (option TBD) — pending
* 6. Patent draft if possible — pending
* 7. Buisiness proposal PPT slides , HTML of post pitch size (5 min) — pending
* 8. Legal reveiw; authorship attribution — pending

# 7. Future Consideration
* Integration with other platform 
      - Mobile service provider (i.e. T-Life, T-Mobile Play), social media or music plug-in (Facebook, Instagram, Spotify)
* Premium tier: Revenue-sharing prosumer accounts require dedicated legal+financial review 
* Memory/RAG: Agent_3 regression logs feed a fugture RAG-based sytle optimization (Optional)
* Cross-language party: Agent_1 has 5 language output should be reviewed by native speaker 

---
v2 (`orchestration_v2.md`) is retained as-is in this folder for backup/history; this file (v3) supersedes it as the current spec.
