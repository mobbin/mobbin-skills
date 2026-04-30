---
name: mobbin-search
description: Search Mobbin for real app UI screenshots and visually analyze them. Required before calling the `search_screens` MCP tool — this skill defines how to plan searches, respond, and build HTML evidence boards when the screens are the answer. Use whenever the user asks about UI/UX design patterns, wants to see how other apps handle a screen or flow, needs design inspiration or references, asks to compare UI approaches across apps, mentions Mobbin, or whenever `search_screens` would be relevant. Trigger aggressively for any design-related question — even if screenshots aren't explicitly requested.
---

# Mobbin Search

Pull real app screenshots from Mobbin and visually analyze them before answering the user's design question.

## Why this skill exists

Design questions are best answered with concrete references, not generic advice. This skill routes those questions through Mobbin's library of curated app screens so your response is grounded in what real apps actually do rather than prior assumptions.

## Workflow

### 1. Plan the search

Before calling any tool, figure out:

- **Query terms** — use the user's own words, or go broader. Never add specifics the user didn't mention — you'll bias results toward one pattern and miss the variety that makes this useful. E.g. "login screens" → search `login screen`, not `login screen with username and password`. The `deep` search mode handles complex queries already; don't prematurely split into multiple searches and don't narrow with unrequested specifics.
- **Platform** — `ios` or `web`. Infer from context:
  - Next.js/React/web app → `web`
  - SwiftUI/React Native/mobile app → `ios`
  - Unclear → ask the user before searching.
- **Limit** — default to `5`. Bump up (max ~15) only if the user asks for variety or specifies a count.

Don't decide about a board yet — that comes after you've seen the results.

### 2. Announce the plan, then proceed

State the plan in one line, then call `search_screens` in the same turn. The announcement is for expectation alignment, not permission — the user can redirect if they disagree, but don't pause for approval.

Example: "Searching 20 iOS login screens."

Keep it terse. Don't explain the tool.

### 3. Call `search_screens`

This is the tool to use for design reference. It runs the search server-side, fetches every matching image, and returns them to you as inline image content blocks — no download step, no Read step, no URL emission. You see the screens directly in the tool result.

The response contains:
- A metadata text block: `{ screens: [{index, id, app_name, mobbin_url, image_url, platform}], failed: [...] }`
- N image blocks, one per entry in `metadata.screens`, in the same order

The `index` field correlates each image with its app. `image_url` is a debug/sanity-check reference — you don't need it for analysis.

### 4. Decide how to respond

After reading the screens, pick one of two modes based on what the user actually needs. Ground every observation in what you actually see — reference specific copy, element counts, colors, button labels. Avoid generic UX platitudes.

**Mode A — Direct answer.** The query is concrete and the answer lives in a short explanation + 1-3 screens. Just reply. Lead with the screens for inspiration asks, with concrete implementation details (spacing, copy, layout) for build-help asks. One-line notes per screen with Mobbin URLs.

Link format:
```
1. **N26** — Single-field login with biometric option. [View on Mobbin](https://mobbin.com/screens/...)
```

If there's a natural follow-up ("want more results?", "go deeper on any of these?"), make it the final line of your message — see the question-last rule below.

**Mode B — Offer a board.** The query benefits from dwelling: pattern extraction across a batch, side-by-side comparison of several apps, walkthrough of a flow, or inspiration browse with enough screens to warrant it. Ask the user what they want before building anything.

The ask is deliberately minimal — the user should see a clear recommendation with a couple of easy-to-scan escapes, not a paralysis-inducing menu of equal-weight choices. Shape it as:

1. **Preamble — one short sentence.** A single observation that motivates your recommendation (e.g. "These 15 screens split into 3 clear approaches"). No findings list, no summary of patterns, no bullets describing what you saw. Save all of that for the board once the user picks. Front-loading findings buries the question *and* gives away the answer the board is supposed to carry.
2. **Options as a short numbered list — exactly 3 items.** Numbered lists scan faster than prose when there are choices to make.
   1. Your recommendation, clearly labelled (`**Recommended** — ...`) and described in plain English with a short reason it fits.
   2. "Describe your own layout (and any styling guidelines or other skills to apply)."
   3. "Just a text summary."
   
   Three items only — don't tack on additional named structures. If the user wants something different, they'll describe it under option 2.
3. **Question last.** The final line is a real question the user can answer ("Which would you like?" / "Sound good?"). Nothing after it. The user should reach the end and know you're waiting on them.

Tone is consultative: one clear recommendation with easy escapes, not a neutral menu.

Example:
> These 15 login screens split into 3 clear approaches.
>
> 1. **Recommended** — a board grouped by pattern so you can scan the approaches side by side
> 2. Describe your own layout (and any styling or other skills to apply)
> 3. Just a text summary
>
> Which would you like?

Wait for the user to pick before building.

### 5. Build the board

Only after the user has picked a structure.

The board is the main artifact; the terminal text is just a signpost pointing to it. **Do not duplicate the board's content in the terminal** — that's the trap to avoid.

Self-contained static HTML file. No frameworks, no external CSS, no build step. How you embed the screens depends on where the board will render — see "Choose the delivery target" below.

Structure the board for what the user picked. Pattern grouping → sections with thumbnail rows beneath each. Comparison → fixed columns per app, same rows across (copy, CTA, field count, etc.). Walkthrough → horizontal sequence in flow order. Tile → loose grid, app name + thumbnail, minimal chrome. Legibility over template-matching — adapt to the query.

If the user specified styling guidelines or pointed to other skills (e.g. `design-principles`), apply them.

Regardless of structure:
- **Light theme by default** — clean whites/off-whites for the page, dark text, subtle borders and dividers. Agents often overreach with dark themes and pick illegible colour combinations (low-contrast greys, muddy accents); a light theme is more forgiving and lets the screenshots carry the visual weight. Override if the user specifies a theme or points at a design system.
- **Let visuals carry the meaning, not words.** The screens are the explanation — your chrome should recede. Prefer visual cues — grouping, whitespace, consistent thumbnail sizes, subtle labels/tags, colour coding, simple callouts drawn on images — over paragraphs or long bullet lists. Section headers and one-line captions are fine; a wall of bullets is a tell that you've defaulted to text when you should've defaulted to layout. When you catch yourself writing a list explaining what screens show, try to rework it as visual structure instead.
- Link every screen back to its Mobbin URL so the user can click through
- Include the app name on each screen
- Consider click-to-enlarge — a plain `<a href>` to `image_url` opening in a new tab. Anchors aren't restricted by `img-src`, so use the CDN URL here even in chat mode (don't inline a second base64 copy in the href).

**Choose the delivery target.** Before writing the file, check where it'll render:

```bash
if [ -d /mnt/user-data/outputs ]; then
  # Hosted environment (claude.ai chat/app). Artifact CSP blocks
  # external img-src — must inline screens as base64.
  TARGET=chat
else
  # Local environment (Claude Code, Desktop). Browser loads CDN
  # URLs directly — keep the fast path.
  TARGET=local
fi
```

**If `TARGET=local`:** embed screens via `<img src>` using `image_url` from the search response — Mobbin CDN URLs work directly and expire roughly an hour later, fine for immediate viewing. Save to `./.mobbin/<slug>-<YYYYMMDD-HHMM>.html` — relative to the current working directory, not `/tmp`. Slug is a short readable description of the query (`ios-login-patterns`, `fintech-dashboards`). Create the parent directory if missing. Open in the browser after writing (`open <path>` on macOS).

**If `TARGET=chat`:** inline screens as base64 data URIs, since Mobbin CDN URLs are blocked and will silently fail to load. For each `image_url`:

```bash
curl -sL "<image_url>" -o /tmp/screen-N.webp
base64 -w 0 /tmp/screen-N.webp
```

Embed as `<img src="data:image/webp;base64,...">`. Mobbin serves WebP. If `file --mime-type` reports otherwise, match the actual type in the data URI prefix. Save to `/mnt/user-data/outputs/<slug>.html` and call `present_files` to surface it in the conversation.

The board renders inline either way — the only difference is whether the bytes travel by reference (URL) or by value (base64). Detection is a heuristic on filesystem layout; if the sandbox structure ever changes, this is the line to revisit.

**Then a short terminal message — that's it:**
- One-line headline finding
- 2-3 bullets of top-level takeaways
- Path to the board, plus a note on how it surfaces — opened in the browser (local) or presented inline (chat)

The depth lives in the board; the terminal is a signpost.

### 6. Handle "show me more" follow-ups

If the user wants more results, call `search_screens` again — higher limit, or a refined query. The tool is cheap to re-call; no dedup logic needed on your end.
