---
name: mobbin-design-reference
description: Search Mobbin for real app UI screenshots and download them locally so Claude can visually analyze them. Use this skill whenever the user asks about UI/UX design patterns, wants to see how other apps handle a specific screen or flow, needs design inspiration or references, asks to compare UI approaches across apps, or mentions Mobbin. Trigger aggressively for any design-related question where seeing real-world examples would help — even if the user doesn't explicitly ask for screenshots.
---

# Mobbin Design Reference

Search Mobbin's library of real app screenshots, download them locally, and visually analyze them before responding to the user's design question.

## Why this skill exists

Claude can read images from local files but cannot view URLs directly. Without this skill, when the MCP tool returns image URLs, Claude tends to gloss over them and never actually looks at the image content. This skill ensures Claude **downloads and reads every image** so it can give informed, visually-grounded design feedback.

## Workflow

### 1. Understand the query

Figure out what the user is looking for. Extract:
- **Search query** — use the user's own words, or go broader. Never add specifics the user didn't mention — you'll bias results toward one pattern and miss the variety that makes this useful. For example, if the user asks for "login screens", search `login screen` — not `login screen with username and password`. The `deep` search mode already handles relevance; your job is to preserve breadth.
- **Platform** — `ios` or `web`. Infer from context:
  - Building a Next.js/React/web app → `web`
  - Building a SwiftUI/React Native/mobile app → `ios`
  - Unclear → ask the user
- **Limit** — default to `5`. If the user asks for more variety or explicitly requests more, increase (max 100).

### 2. Search Mobbin

Call `mcp__mobbin__search_screens` with the query, platform, and limit. The `mode` defaults to `deep` (AI-powered relevance scoring) — use this unless the user needs fast results.

### 3. Download all images in a single command

Speed matters — every separate tool call adds seconds of overhead. Download ALL images in **one Bash call** using background processes:

```bash
mkdir -p mobbin-screens && \
curl -sL -o "mobbin-screens/AppA-abcd1234.webp" "URL1" & \
curl -sL -o "mobbin-screens/AppB-efgh5678.webp" "URL2" & \
curl -sL -o "mobbin-screens/AppC-ijkl9012.webp" "URL3" & \
wait
```

Filename format: `<AppName>-<first8chars-of-id>.webp`

If any downloads fail (check with `ls -la mobbin-screens/` at the end), note which ones and move on.

### 4. Read all images in one turn

Issue **all Read tool calls in a single response** — Claude Code executes parallel Read calls within the same turn, so this takes the same time as reading one image. Do not read them one at a time in separate turns.

Do not skip this step. Do not summarize without reading. The whole point is to visually inspect the actual screenshots.

### 5. Present findings and respond

After reading all images, match your response to what the user actually asked for:

- **If the user wants patterns or analysis** → lead with the synthesis. A brief screen list with Mobbin links is useful for reference, but keep per-screen descriptions to one line. The user is here for the takeaways, not a narration of each screenshot.
- **If the user wants inspiration or references** → lead with the screens themselves, with enough description to be useful without clicking through.
- **If the user wants implementation help** → focus on the concrete details (spacing, colors, copy, layout) that inform code.

Include a Mobbin URL for each screen so the user can click through:
```
1. **N26** — Single-field login with biometric option. [View on Mobbin](https://mobbin.com/screens/...)
```

**Ground observations in what you actually see.** Reference specific details from the screenshots — copy text, element counts, colors, layout, button labels. Avoid generic UX platitudes unless tied to something visible. Describe what the screenshot shows, not what you think the app looks like from prior knowledge.

### 6. Handle "show me more" follow-ups

If the user wants more results after the initial batch:

1. Search again with a higher limit (e.g., 10-15)
2. Before downloading, check what's already in `mobbin-screens/` — match by the ID portion of the filename (the 8-char hex after the app name)
3. Skip downloading any images that already exist locally
4. Only read the **new** images — no need to re-read ones already discussed
5. Present the new screens alongside a note about which ones were already shown

This avoids duplicate downloads and wasted context while giving the user more variety.

### 7. Cleanup (optional)

The downloaded images remain in `mobbin-screens/` for the user to reference. If the folder is getting large or the user is done, offer to clean it up — but don't delete automatically.
