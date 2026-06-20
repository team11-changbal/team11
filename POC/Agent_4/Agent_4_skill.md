# Agent_4: Global Music Director + UX (US Chart Candidate)

**Version:** v1.0
**Status:** Draft (POC)
**Source spec:** `_orchestration_/orchestration_v2.md` (Section 3.4)

## Input
Final music concept + lyrics + curator narration from Agent_3

## Output
US chart strategy, album artwork prompt, TikTok kill zone (15 sec), UX prototype spec

## Task
Propose final commercial tuning aimed at US TikTok virality and Spotify playlist placement.

## Output Detail
- Global English title and an AI-image-ready album artwork prompt (Midjourney/DALL-E format)
- TikTok challenge "kill zone" (15-second hook section) recommendation

## Task Detail
- Recommend mixing/mastering direction for TikTok viral + Spotify playlist + Billboard entry
- Generate AI image prompt (Midjourney/DALL-E format) for album artwork
- Identify the 15-sec TikTok "hook zone" within the song structure
- Define UX flow:
  - Trial: free for 30 days, full feature access
  - Post-30 days: subscription-required gate UI
  - Publish flow: social media share OR Spotify creator registration UI

## UX States
```
[Essay Input] -> [Preview Music] -> [Approve / Regenerate]
  -> [Share to Social]  OR  [Publish to Spotify as Creator]
       Spotify path -> [Account Type Selection] -> [Revenue Share Agreement] -> [Credit Setup]
```

## Notes
This is the spec driving the Lovable UI/UX prototype (Lobster's track). The trial/subscription gate and the dual publish path (social vs. Spotify creator onboarding) are new screens not in the earlier v1 flow draft and should be reflected in the next Lovable prompt revision.
