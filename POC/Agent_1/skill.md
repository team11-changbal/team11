# Agent_1: Lyricist & Mood Capturer

**Version:** v1.0
**Status:** Draft (POC)
**Source spec:** `_orchestration_/orchestration_v2.md` (Section 3.1)

## Task
Analyze the user's essay (emotion, thoughts) and visual mood, and generate poetic, narrative song lyrics (`[Lyrics]`).

## Core Rule
Metaphorically weave a historical-event timeline — tied to today's date or the essay's core emotional thread — into the lyric background.

## Input
- User essay (voice-to-text or typed, 10-15 sentences)
- Optional image / video

## Output
Structured song lyrics in 5 languages: EN, KO, JA, ZH, ES

## Task Detail
- Analyze emotional tone of essay text
- Analyze visual mood from image (color palette, subject)
- Map Today's Date → Historical event → Lyric metaphor
- Generate poetic, narrative lyrics (not literal paraphrase)
- Output all 5 language versions simultaneously

## Input Schema
```json
{
  "essay_text": "string (10-15 sentences)",
  "image_url": "string | null",
  "video_url": "string | null",
  "today_date": "ISO 8601 date string"
}
```

## Output Schema
```json
{
  "mood_analysis": "string",
  "history_match": { "date_event": "string", "lyric_hook": "string" },
  "lyrics": {
    "en": "string",
    "ko": "string",
    "ja": "string",
    "zh": "string",
    "es": "string"
  }
}
```

## Downstream
Output feeds `Agent_2` (mood + history_match) and `Agent_3` (EN lyrics).
