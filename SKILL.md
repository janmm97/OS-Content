---
name: content-pipeline
description: Content pipeline poller — processes Notion Content Pipeline rows through transcript → audio → ad idea → video output stages. Run every 5 minutes via Claude Routines, a cron job, or One Run.
---

# Content Pipeline Poller

You are an automation agent. When invoked, scan the Notion Content Pipeline database and process every eligible row. Work through all stages in sequence per run. Be silent on success — only report errors.

## Prerequisites

- [One CLI](https://www.withone.ai) installed and configured
- Connected platforms (via `one add <platform>`):
  - **Notion** — your content database
  - **ElevenLabs** — text-to-speech audio generation
  - **Slack** — stage completion notifications
  - **Firecrawl** — URL scraping for Stage 0
- A `.env` file in your project root containing:
  ```
  NOTION_TOKEN=secret_xxxxxxxxxxxx
  ```
  Get this from https://www.notion.so/my-integrations (Internal Integration Token). The integration must have access to your Content Pipeline database.
- Reference documents in place (see **Reference Documents** section below)

## Configuration

Replace all `YOUR_*` placeholders below with your actual values before running.

- **Database ID:** `YOUR_NOTION_DATABASE_ID`
  - Find this in your Notion database URL: `notion.so/your-workspace/<DATABASE_ID>?v=...`
- **Notion connection key:** `YOUR_NOTION_CONNECTION_KEY`
  - Run `one --agent list` and find your Notion connection key
- **ElevenLabs connection key:** `YOUR_ELEVENLABS_CONNECTION_KEY`
  - Run `one --agent list` and find your ElevenLabs connection key
- **Voice ID (UGC):** `YOUR_UGC_VOICE_ID`
  - Find voice IDs at https://elevenlabs.io/voice-library
- **Voice ID (Commercial Ad):** `YOUR_COMMERCIAL_VOICE_ID`
- **ElevenLabs model:** `eleven_turbo_v2`
- **Slack channel ID:** `YOUR_SLACK_CHANNEL_ID`
  - Right-click a channel in Slack → Copy link — the ID is the last segment (e.g. `C0B2NLE2B4J`)
- **`Link/Source` property:** A URL field on each database row. When populated (and `Title` / `Topic/Angle` are both empty), Stage 0 scrapes the URL with Firecrawl to extract use case or reference material, then generates a `Title` and `Topic/Angle` for human approval before the transcript pipeline begins.

## Notion Database Schema

Your Notion database must have the following properties:

| Property | Type | Notes |
| --- | --- | --- |
| `Name` | Title | The content title |
| `Status` | Status | Values: `Draft`, `Processing`, `Needs Approval`, `Approved`, `Changes Requested`, `Done`, `Archived` |
| `Stage` | Select | Values: `Title/Angle`, `Transcript`, `Audio`, `Ad Idea`, `Video Output`, `Complete` |
| `Topic/Angle` | Rich Text | The content angle/POV |
| `Transcript` | Rich Text | The spoken script |
| `Ad Idea` | Rich Text | Marketing concept |
| `Video Idea` | Rich Text | Production guide |
| `Images/Elements Needed` | Rich Text | Image prompts or avatar brief |
| `Notes` | Rich Text | Reviewer feedback |
| `Ad Type` | Select | Values: `UGC`, `Commercial Ad` |
| `Link/Source` | URL | Source URL for Stage 0 scraping |
| `ElevenLabs generated file` | Files | Audio file attachment |

## Reference Documents

Create these files in your project's `references/` folder. They are read by the pipeline at each stage to guide generation. The content you put here defines your brand voice, content strategy, and production style — these are **your** docs.

| File | Purpose |
| --- | --- |
| `references/content-strategist.md` | Topic development, angle selection, title writing frameworks |
| `references/script-writer.md` | Script structure, storytelling frameworks, CTA guidance |
| `references/short-form-tactics.md` | Short-form video best practices, hooks, pacing |
| `references/elevenlabs-guide.md` | ElevenLabs-optimized writing (punctuation, pacing, word targets) |
| `references/brand-voice.md` | Your brand's tone, persona, vocabulary, and what to avoid |
| `references/video-production.md` | Video editor guide: scene structure, shot types, overlays |
| `references/image-generation.md` | AI image generation prompts framework and tool recommendations |
| `references/assets/ugc-avatar.png` | Your UGC avatar image (for talking-head UGC ads) |

## Action IDs (do not change)

These are universal One CLI action identifiers — they work for all users.

- Notion query: `conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ`
- Notion update page: `conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg`
- ElevenLabs TTS: `conn_mod_def::GJ2bFPL5uv0::ij7bSi4TSmSHXxwq8pU-zg`

## Slack Notification Pattern

After every stage that sets Status = `Needs Approval`, send a Slack notification to your channel. Use this pattern each time — look up the action once per run and reuse it:

```bash
# Look up action ID (do once per run, cache in memory):
one --agent actions search slack "send message"
one --agent actions knowledge slack <ACTION_ID>

# Send notification:
one --agent actions execute slack <ACTION_ID> <SLACK_CONNECTION_KEY> \
  -d '{"channel": "YOUR_SLACK_CHANNEL_ID", "text": "<MESSAGE>"}'
```

**Constructing the Notion page URL:** Remove all hyphens from `<ROW_ID>`:
`NOTION_URL = https://www.notion.so/<ROW_ID_NO_HYPHENS>`

**Extract `ROW_TITLE`** from `properties["Name"].title[*].plain_text` — available in every query result row.

**Message format per stage:**

| Stage | Message |
| --- | --- |
| 0 — Title/Angle generated | `✅ *Title & Topic/Angle ready for review*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 0b — Title/Angle revised | `✅ *Title & Topic/Angle revised*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 1 — Transcript generated | `✅ *Transcript ready for review*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 1b — Transcript revised | `✅ *Transcript revised*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 2 — Audio generated | `✅ *Audio ready for review*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 2b — Audio revised | `✅ *Audio revised*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 3b — Ad Idea + Video + Images generated | `✅ *Ad Idea, Video Idea & Images ready for review*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 3c — Ad Idea revised | `✅ *Ad Idea revised*\n>{ROW_TITLE}\n{NOTION_URL}` |
| 4 — Video Output stage set | `✅ *Ready for video production*\n>{ROW_TITLE}\n{NOTION_URL}` |

If the Slack notification fails, log the error but do not fail the stage — the Notion row is already updated successfully.

---

## State Machine

Process rows in this order each run. Always skip rows with Status = `Processing`, `Needs Approval`, `Done`, or `Archived`. For `Changes Requested`, only process rows where Stage = `Title/Angle`, `Transcript`, `Audio`, or `Ad Idea` — skip all other `Changes Requested` rows.

| Status | Stage | Condition | Action |
| --- | --- | --- | --- |
| `Draft` | (empty) | `Link/Source` not empty AND `Title` + `Topic/Angle` both empty | Stage 0: Scrape URL, generate Title + Topic/Angle |
| `Changes Requested` | `Title/Angle` | — | Stage 0b: Revise Title/Angle using Notes |
| `Approved` | `Title/Angle` | — | Stage 0c: Auto-generate transcript immediately (no reset needed) |
| `Draft` | (empty) OR `Draft` \| `Transcript` (recovery) | `Topic/Angle` not empty (user filled it in, or manual recovery) | Stage 1: Generate transcript |
| `Changes Requested` | `Transcript` | — | Stage 1b: Revise transcript using Notes |
| `Approved` | `Transcript` | — | Stage 2: Generate audio |
| `Changes Requested` | `Audio` | — | Stage 2b: Revise audio — optionally update transcript first, then regenerate |
| `Approved` | `Audio` | — | Stage 3a: Generate ad idea only (auto-advances, no approval) |
| `Draft` | `Ad Idea` | — | Stage 3b: Generate video idea + images/elements |
| `Changes Requested` | `Ad Idea` | — | Stage 3c: Revise ad idea, video idea, and images/elements using Notes |
| `Approved` | `Ad Idea` | — | Stage 4: Set stage to Video Output, await editor upload |
| `Approved` | `Video Output` | — | Stage 5: Mark complete |
| `Changes Requested` | `Video Output` | — | No pipeline action — editor reads Notes, re-uploads clip, sets Status = Approved |

---

## Stage 0: Title & Topic/Angle Generation

**Trigger:** `Status = Draft`, `Stage = empty`, `Link/Source` not empty, `Title` (page Name) and `Topic/Angle` both empty.

This stage fires when a user has pasted a source URL into `Link/Source` but hasn't yet written a title or angle. It scrapes the URL, reads the content strategy reference doc, generates a `Title` and `Topic/Angle`, and sets the row to `Needs Approval` for human review. Once approved, Stage 0c resets the row so Stage 1 picks it up on the next run.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Stage", "select": {"is_empty": true}},
        {"property": "Topic/Angle", "rich_text": {"is_empty": true}},
        {"property": "Link/Source", "url": {"is_not_empty": true}},
        {"or": [
          {"property": "Status", "status": {"equals": "Draft"}},
          {"property": "Status", "status": {"is_empty": true}}
        ]}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 0.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 0.2 Extract fields

- `LINK_SOURCE = properties["Link/Source"].url`
- `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

### 0.3 Read content-strategist.md

Read this file in full before generating:
- `references/content-strategist.md`

Focus especially on sections covering Topic & Angle Development and Title Writing.

### 0.4 Scrape Link/Source with Firecrawl

Look up the correct Firecrawl action and scrape the URL:

```bash
one --agent actions search firecrawl "scrape url"
# Read the knowledge doc for the matched action ID before executing:
one --agent actions knowledge firecrawl <ACTION_ID>
# Then scrape:
one --agent actions execute firecrawl <ACTION_ID> \
  <FIRECRAWL_CONNECTION_KEY> \
  -d '{"url": "<LINK_SOURCE>"}'
```

Extract the main body text from the scraped response. This is `SCRAPED_CONTENT` — the raw source material for generation.

If scraping fails (URL unreachable, access denied, etc.), write an error to Notes and restore Status = Draft. Do not proceed.

### 0.5 Generate Title and Topic/Angle

Using `SCRAPED_CONTENT`, `AD_TYPE`, and the principles from `content-strategist.md`, generate:

- **Title:** A sharp, compelling content title. Apply the Title Writing principles from the reference doc — choose the format (list, how-to, contrarian, result-first, etc.) that best fits the use case. Must be specific, have a clear POV, and avoid vague superlatives.
- **Topic/Angle:** One to two sentences with a defined point of view and a specific tension or insight. Not a subject area — a position. Apply the Topic & Angle Development principles from the reference doc.

Prompt framing:
> Source material (scraped from {LINK_SOURCE}): {SCRAPED_CONTENT}
> Ad Type: {AD_TYPE}
> Using the content strategy principles in the reference doc, generate:
> 1. A Title that is specific, compelling, and follows the title-writing best practices from the reference doc
> 2. A Topic/Angle that has a clear point of view with a defined tension or insight — not just a subject description
> The content is for [Your Company] ([your-website.com]), [brief description of what your company does]. Match the angle to the use case described in the source material.

### 0.6 Write Title and Topic/Angle to Notion

Write `Title` to the Notion page's `Name` (title) property and `Topic/Angle` as a rich_text property. Split `Topic/Angle` into 1900-character chunks if needed.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Name": {"title": [{"type": "text", "text": {"content": "<TITLE>"}}]},
      "Topic/Angle": {"rich_text": [<TOPIC_ANGLE_CHUNKS>]},
      "Stage": {"select": {"name": "Title/Angle"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 0.7 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 0 message format with `ROW_TITLE` and `NOTION_URL`.

### 0.8 Error handling

If any step from 0.1–0.6 fails:

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Draft"}},
      "Stage": {"select": null},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 0 error: <ERROR_MESSAGE>"}}]}
    }
  }'
```

---

## Stage 0b: Title/Angle Revision

**Trigger:** `Status = Changes Requested`, `Stage = Title/Angle`

The reviewer has left feedback in `Notes`. Re-generate Title and Topic/Angle incorporating their instructions.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Changes Requested"}},
        {"property": "Stage", "select": {"equals": "Title/Angle"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 0b.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 0b.2 Read fields

Extract:
- `EXISTING_TITLE = properties["Name"].title[*].plain_text` joined with space
- `EXISTING_TOPIC = properties["Topic/Angle"].rich_text[*].plain_text` joined with space
- `REVISION_NOTES = properties["Notes"].rich_text[*].plain_text` joined with space
- `LINK_SOURCE = properties["Link/Source"].url`
- `AD_TYPE = properties["Ad Type"].select.name` — treat null as `"Commercial Ad"`.

If `REVISION_NOTES` is empty, write an error to Notes and restore Status = `Changes Requested`.

### 0b.3 Read content-strategist.md

Read `references/content-strategist.md` — focus on Topic & Angle Development and Title Writing sections.

### 0b.4 Revise Title and Topic/Angle

Using the existing title/angle, the reviewer's notes, and the content-strategist.md principles, produce revised versions. Reviewer notes take precedence over defaults.

Prompt framing:
> Existing Title: {EXISTING_TITLE}
> Existing Topic/Angle: {EXISTING_TOPIC}
> Reviewer feedback: {REVISION_NOTES}
> Source URL: {LINK_SOURCE}
> Produce a revised Title and Topic/Angle that addresses the feedback while following the title-writing and topic/angle development principles from the content strategy reference doc.

### 0b.5 Write revised Title and Topic/Angle, clear Notes

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Name": {"title": [{"type": "text", "text": {"content": "<REVISED_TITLE>"}}]},
      "Topic/Angle": {"rich_text": [<REVISED_TOPIC_CHUNKS>]},
      "Notes": {"rich_text": []},
      "Stage": {"select": {"name": "Title/Angle"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 0b.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 0b message format.

### 0b.7 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 0b error: <ERROR_MESSAGE>. Fix the issue and the pipeline will retry automatically."}}]}
    }
  }'
```

---

## Stage 0c: Auto-Generate Transcript After Title/Angle Approval

**Trigger:** `Status = Approved`, `Stage = Title/Angle`

The reviewer has approved the generated Title and Topic/Angle. Generate the transcript immediately in the same run — do not wait for a second poll cycle. Sets Status to `Needs Approval` with Stage = `Transcript` so the reviewer can approve it next.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Approved"}},
        {"property": "Stage", "select": {"equals": "Title/Angle"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 0c.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 0c.2 Read the Topic/Angle and Ad Type

Extract the plain text from `properties["Topic/Angle"].rich_text[*].plain_text` — join with space.

Also extract `AD_TYPE = properties["Ad Type"].select.name` — will be `"UGC"`, `"Commercial Ad"`, or null/empty. If null or empty, treat as `"Commercial Ad"`.

### 0c.3 Read reference documents

Read these files in full before generating:
- `references/script-writer.md`
- `references/short-form-tactics.md`
- `references/elevenlabs-guide.md`
- `references/brand-voice.md`

### 0c.4 Generate the transcript

Follow the exact same generation rules as Stage 1.4 — apply the correct rules for UGC vs Commercial Ad.

### 0c.5 Write transcript to Notion

Split the script into chunks of 1900 characters maximum.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Transcript": {"rich_text": [<CHUNKS_ARRAY>]},
      "Stage": {"select": {"name": "Transcript"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 0c.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the **Stage 1** message format (`✅ *Transcript ready for review*`).

### 0c.7 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Approved"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 0c error: <ERROR_MESSAGE>. Will retry automatically on next run."}}]}
    }
  }'
```

---

## Stage 1: Transcript Generation

**Trigger:** `Status = Draft` (or empty), `Stage = empty` AND `Topic/Angle` not empty. Also handles recovery: `Status = Draft`, `Stage = Transcript` (e.g. rows manually set to Transcript/Draft that have no transcript yet, or got stuck).

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Topic/Angle", "rich_text": {"is_not_empty": true}},
        {"or": [
          {"property": "Status", "status": {"equals": "Draft"}},
          {"property": "Status", "status": {"is_empty": true}}
        ]},
        {"or": [
          {"property": "Stage", "select": {"is_empty": true}},
          {"property": "Stage", "select": {"equals": "Transcript"}}
        ]}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 1.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 1.2 Read the Topic/Angle and Ad Type

Extract the plain text from `properties["Topic/Angle"].rich_text[*].plain_text` — join with space.

Also extract `AD_TYPE = properties["Ad Type"].select.name` — will be `"UGC"`, `"Commercial Ad"`, or null/empty. If null or empty, treat as `"Commercial Ad"`.

### 1.3 Read reference documents

Read these files in full before generating:
- `references/script-writer.md`
- `references/short-form-tactics.md`
- `references/elevenlabs-guide.md`
- `references/brand-voice.md`

### 1.4 Generate the transcript

Write the **spoken words only** for a short-form video ad on the topic: **{Topic/Angle text}**

Universal rules (all ad types):
- No [VISUAL:], [TEXT OVERLAY:], [CREATOR:], or any bracketed notes — those go in Video Idea later
- Output only the spoken script text, no commentary
- Write out numbers in full ("two hundred and fifty" not "250"); no symbols; use punctuation for pacing; use paragraph breaks for natural pauses
- When listing items in a series, always include "and" before the final item

---

**If `AD_TYPE = "UGC"`:**

This is a person-to-camera script. The speaker is sharing something from experience — not performing for an audience.

- Conversational, warm, and direct. Use contractions ("I've", "you're", "it's", "don't"). Short punchy sentences.
- Mirror your brand voice from `brand-voice.md`: friendly and direct, confident without arrogance, builder-first.
- Follow the persona hook guidance in `brand-voice.md` — it specifies which hook styles fit and which to avoid.
- Apply the best-fit storytelling structure from `script-writer.md`. Follow short-form pacing: each sentence is a micro-beat. Use paragraph breaks for natural rhythm and TTS pauses.
- Always end with a CTA. Match aggressiveness to funnel stage per `script-writer.md`. One CTA only. Action verbs.
- ElevenLabs word targets: 30–35 words for 15s, 65–75 words for 30s. Sentences 10–18 words.

---

**If `AD_TYPE = "Commercial Ad"` (or null):**

This is a voiceover hook only — one or two lines maximum. It is not a full script. The hook will be placed over visuals; the visuals carry the rest of the story.

- Write a single punchy line (or at most two) that lands an emotion or a tension. No body copy, no CTA — just the hook.
- Stop someone mid-scroll with a feeling. Aspirational, emotional, human — not technical, not product-focused.
- Fragments are acceptable when they serve rhythm. 5 to 15 words total.
- Read it aloud — it should hit like a film trailer opener, not a product pitch.

### 1.5 Write transcript to Notion

Split the script into chunks of 1900 characters maximum. Build a `rich_text` array where each chunk is `{"type": "text", "text": {"content": "<chunk>"}}`.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Transcript": {"rich_text": [<CHUNKS_ARRAY>]},
      "Stage": {"select": {"name": "Transcript"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 1.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 1 message format.

### 1.7 Error handling

If any step from 1.1–1.5 fails, run:

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Draft"}},
      "Stage": {"select": null},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 1 error: <ERROR_MESSAGE>"}}]}
    }
  }'
```

---

## Stage 1b: Transcript Revision

**Trigger:** `Status = Changes Requested`, `Stage = Transcript`

The human reviewer has left feedback in the `Notes` field. Re-generate the transcript incorporating their instructions, then set Status back to `Needs Approval`.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Changes Requested"}},
        {"property": "Stage", "select": {"equals": "Transcript"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 1b.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 1b.2 Read Topic/Angle, existing Transcript, Notes, and Ad Type

Extract:
- `TOPIC = properties["Topic/Angle"].rich_text[*].plain_text` joined with space
- `EXISTING_TRANSCRIPT = properties["Transcript"].rich_text[*].plain_text` joined with empty string
- `REVISION_NOTES = properties["Notes"].rich_text[*].plain_text` joined with space
- `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

If `REVISION_NOTES` is empty, write an error and skip — there is nothing to revise against.

### 1b.3 Read reference documents

Read these files in full before generating:
- `references/script-writer.md`
- `references/short-form-tactics.md`
- `references/elevenlabs-guide.md`
- `references/brand-voice.md`

### 1b.4 Revise the transcript

Using the reference documents, the original topic, the existing transcript, and the reviewer's notes, produce a revised transcript that addresses all feedback in `REVISION_NOTES`.

Apply the same rules as Stage 1.4 (brand voice, hook, structure, CTA, ElevenLabs formatting). The revision notes take precedence — if they conflict with a default rule (e.g. "make the hook softer"), follow the reviewer's instruction.

Prompt framing:
> Original topic: {TOPIC}
> Existing transcript: {EXISTING_TRANSCRIPT}
> Reviewer feedback: {REVISION_NOTES}
> Produce a revised transcript that addresses the feedback while following all script and brand voice rules.

Output only the revised spoken script text, no commentary.

### 1b.5 Clear Notes and write revised transcript to Notion

Split the revised script into chunks of 1900 characters maximum.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Transcript": {"rich_text": [<CHUNKS_ARRAY>]},
      "Notes": {"rich_text": []},
      "Stage": {"select": {"name": "Transcript"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 1b.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 1b message format.

### 1b.7 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 1b error: <ERROR_MESSAGE>. Fix the issue and the pipeline will retry automatically."}}]}
    }
  }'
```

---

## Stage 2: ElevenLabs Audio Generation

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Approved"}},
        {"property": "Stage", "select": {"equals": "Transcript"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 2.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 2.2 Extract plain transcript text and Ad Type

Join all `properties["Transcript"].rich_text[*].plain_text` with empty string. Store as TRANSCRIPT_TEXT.

Also extract `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

Select the voice ID based on AD_TYPE:
- UGC → `VOICE_ID = YOUR_UGC_VOICE_ID`
- Commercial Ad (or null) → `VOICE_ID = YOUR_COMMERCIAL_VOICE_ID`

### 2.3 Generate audio via ElevenLabs

Use the One CLI with `--output` to save the binary audio to a temp file:

```bash
one --agent actions execute elevenlabs conn_mod_def::GJ2bFPL5uv0::ij7bSi4TSmSHXxwq8pU-zg \
  YOUR_ELEVENLABS_CONNECTION_KEY \
  --path-vars '{"voiceId": "<VOICE_ID>"}' \
  --skip-validation \
  --output "$env:TEMP/cp_audio_<ROW_ID>.mp3" \
  -d '{"text": "<TRANSCRIPT_TEXT>", "model_id": "eleven_turbo_v2", "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}}'
```

The response will be: `{"saved":true,"path":"...","size":...,"contentType":"application/octet-stream"}`

### 2.4 Create Notion file upload slot and send binary

Both steps must use the same `NOTION_TOKEN` — the One CLI uses its own integration token which would cause a 404 when sending the binary. Read the token from `.env` and do both calls with PowerShell:

```powershell
Add-Type -AssemblyName System.Net.Http

# Read token from .env in your project root
$envContent = Get-Content ".env" -ErrorAction Stop
$tokenLine = $envContent | Select-String "^NOTION_TOKEN=(.+)"
if (-not $tokenLine) { throw "NOTION_TOKEN not found in .env" }
$notionToken = $tokenLine.Matches[0].Groups[1].Value.Trim()

$audioPath = "$env:TEMP\cp_audio_<ROW_ID>.mp3"

# Create the file upload slot
$slotBody = '{"mode":"single_part","filename":"audio_<ROW_ID>.mp3","content_type":"audio/mpeg"}'
$slotClient = [System.Net.Http.HttpClient]::new()
$slotClient.DefaultRequestHeaders.Add("Authorization", "Bearer $notionToken")
$slotClient.DefaultRequestHeaders.Add("Notion-Version", "2026-03-11")
$slotContent = [System.Net.Http.StringContent]::new($slotBody, [System.Text.Encoding]::UTF8, "application/json")
$slotResponse = $slotClient.PostAsync("https://api.notion.com/v1/file_uploads", $slotContent).Result
$slotJson = $slotResponse.Content.ReadAsStringAsync().Result | ConvertFrom-Json
$slotClient.Dispose()
if (-not $slotResponse.IsSuccessStatusCode) { throw "Failed to create upload slot: $($slotJson | ConvertTo-Json)" }
$fileUploadId = $slotJson.id

# Send the binary
$fileStream = [System.IO.File]::OpenRead($audioPath)
$streamContent = [System.Net.Http.StreamContent]::new($fileStream)
$streamContent.Headers.ContentType = [System.Net.Http.Headers.MediaTypeHeaderValue]::new("audio/mpeg")
$multipart = [System.Net.Http.MultipartFormDataContent]::new()
$multipart.Add($streamContent, "file", "audio_<ROW_ID>.mp3")
$sendClient = [System.Net.Http.HttpClient]::new()
$sendClient.DefaultRequestHeaders.Add("Authorization", "Bearer $notionToken")
$sendClient.DefaultRequestHeaders.Add("Notion-Version", "2026-03-11")
$sendResponse = $sendClient.PostAsync("https://api.notion.com/v1/file_uploads/$fileUploadId/send", $multipart).Result
$sendBody = $sendResponse.Content.ReadAsStringAsync().Result
$fileStream.Close()
$sendClient.Dispose()
if (-not $sendResponse.IsSuccessStatusCode) { throw "Notion file send failed: $sendBody" }
```

### 2.5 Attach file to Notion row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "ElevenLabs generated file": {
        "files": [{"type": "file_upload", "file_upload": {"id": "<FILE_UPLOAD_ID>"}}]
      },
      "Stage": {"select": {"name": "Audio"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

Delete the temp file after upload:
```powershell
Remove-Item "$env:TEMP\cp_audio_<ROW_ID>.mp3" -Force -ErrorAction SilentlyContinue
```

### 2.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 2 message format.

### 2.7 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 2 error: <ERROR_MESSAGE>. Set Status = Approved to retry."}}]}
    }
  }'
```

---

## Stage 2b: Audio Revision

**Trigger:** `Status = Changes Requested`, `Stage = Audio`

The reviewer has left feedback in the `Notes` field. If the notes indicate the transcript needs changes, revise it first then regenerate. Otherwise regenerate audio from the existing transcript. Set Status back to `Needs Approval` when done.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Changes Requested"}},
        {"property": "Stage", "select": {"equals": "Audio"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 2b.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 2b.2 Read fields

Extract:
- `EXISTING_TRANSCRIPT = properties["Transcript"].rich_text[*].plain_text` joined with empty string
- `REVISION_NOTES = properties["Notes"].rich_text[*].plain_text` joined with space
- `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.
- `TOPIC = properties["Topic/Angle"].rich_text[*].plain_text` joined with space

If `REVISION_NOTES` is empty, write an error to Notes and restore Status = `Changes Requested` — there is nothing to revise against.

Select voice ID based on AD_TYPE:
- UGC → `VOICE_ID = YOUR_UGC_VOICE_ID`
- Commercial Ad (or null) → `VOICE_ID = YOUR_COMMERCIAL_VOICE_ID`

### 2b.3 Determine if transcript needs updating

Read the revision notes and decide:
- **If notes mention script, wording, phrasing, content, or transcript changes:** revise the transcript first (follow Stage 1b.3–1b.4 rules — read reference docs, apply feedback, produce revised transcript text). Use the revised text as `TRANSCRIPT_TEXT`.
- **If notes are only about voice, pacing, tone, or audio quality:** use `EXISTING_TRANSCRIPT` as `TRANSCRIPT_TEXT` unchanged.

### 2b.4 If transcript was revised — update it in Notion first

Split revised transcript into 1900-character chunks and write back:

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Transcript": {"rich_text": [<CHUNKS_ARRAY>]}
    }
  }'
```

Skip this step if the transcript was not changed.

### 2b.5 Regenerate audio via ElevenLabs

```bash
one --agent actions execute elevenlabs conn_mod_def::GJ2bFPL5uv0::ij7bSi4TSmSHXxwq8pU-zg \
  YOUR_ELEVENLABS_CONNECTION_KEY \
  --path-vars '{"voiceId": "<VOICE_ID>"}' \
  --skip-validation \
  --output "$env:TEMP/cp_audio_<ROW_ID>.mp3" \
  -d '{"text": "<TRANSCRIPT_TEXT>", "model_id": "eleven_turbo_v2", "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}}'
```

### 2b.6 Upload new audio to Notion (same as Stage 2.4)

Follow the same PowerShell upload steps as Stage 2.4 to create a file upload slot and send the binary. Use filename `audio_<ROW_ID>_rev.mp3` to distinguish from the original.

### 2b.7 Attach revised audio and clear Notes

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "ElevenLabs generated file": {
        "files": [{"type": "file_upload", "file_upload": {"id": "<FILE_UPLOAD_ID>"}}]
      },
      "Notes": {"rich_text": []},
      "Stage": {"select": {"name": "Audio"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

Delete the temp file:
```powershell
Remove-Item "$env:TEMP\cp_audio_<ROW_ID>.mp3" -Force -ErrorAction SilentlyContinue
```

### 2b.8 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 2b message format.

### 2b.9 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 2b error: <ERROR_MESSAGE>. Fix the issue and the pipeline will retry automatically."}}]}
    }
  }'
```

---

## Stage 3a: Ad Idea Generation

**Trigger:** `Status = Approved`, `Stage = Audio`

Generates the marketing concept only. Sets Status to `Draft` (not `Needs Approval`) so it auto-advances to Stage 3b on the next run without requiring human approval.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Approved"}},
        {"property": "Stage", "select": {"equals": "Audio"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 3a.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 3a.2 Extract transcript text and Ad Type

Join all `properties["Transcript"].rich_text[*].plain_text`.

Also extract `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

### 3a.3 Read reference documents

Read these files:
- `references/short-form-tactics.md`
- `references/video-production.md`

### 3a.4 Generate Ad Idea

Using the references and transcript, produce a high-level marketing concept. The content differs by Ad Type:

**If `AD_TYPE = "UGC"`:**
- Content format: person-to-camera, authentic, informal — feels like a real user story, not a brand campaign
- Hook style: relatable opener drawing on personal pain or experience; conversational, not polished
- Emotional trigger: trust, relatability, social proof
- Platform recommendation with UGC-specific rationale (TikTok/Reels for authenticity)
- CTA: soft and conversational — no hard sell
- Note: this ad will be delivered via a talking avatar (avatar file: `references/assets/ugc-avatar.png`)

**If `AD_TYPE = "Commercial Ad"` (or null):**
- Platform recommendation (TikTok / Reels / YouTube Shorts / Meta) with rationale
- Hook strategy (which hook type and why it works for this topic)
- Emotional trigger being activated
- Overall campaign concept (1–2 sentences)
- CTA recommendation

Output only this section — no visual direction here.

### 3a.5 Write Ad Idea to Notion

Split into 1900-character chunks.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Ad Idea": {"rich_text": [<AD_IDEA_CHUNKS>]},
      "Stage": {"select": {"name": "Ad Idea"}},
      "Status": {"status": {"name": "Draft"}}
    }
  }'
```

### 3a.6 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 3a error: <ERROR_MESSAGE>. Set Status = Approved to retry."}}]}
    }
  }'
```

---

## Stage 3b: Video Idea & Images/Elements Generation

**Trigger:** `Status = Draft`, `Stage = Ad Idea`

Generates the production guide and image prompts. Sets Status to `Needs Approval` for the human review of all three outputs together (Ad Idea from 3a + Video Idea + Images from 3b).

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Draft"}},
        {"property": "Stage", "select": {"equals": "Ad Idea"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 3b.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 3b.2 Extract transcript text and Ad Type

Join all `properties["Transcript"].rich_text[*].plain_text`.

Also extract `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

### 3b.3 Read reference documents

**If `AD_TYPE = "UGC"`:** Read:
- `references/video-production.md`

**If `AD_TYPE = "Commercial Ad"` (or null):** Read:
- `references/video-production.md`
- `references/image-generation.md`

Also view the avatar image for UGC: `references/assets/ugc-avatar.png`

### 3b.4 Generate Video Idea

Using the video-production reference and transcript, produce the complete production guide for the video editor. The content differs by Ad Type:

**If `AD_TYPE = "UGC"`:**
- Scene-by-scene breakdown with timing (e.g. 0–3s, 3–10s, etc.)
- For each scene: what the talking avatar is saying (synced to the audio), on-screen TEXT OVERLAY content and placement, any B-roll or screen recording cutaways to intercut
- Overall aesthetic direction: keep it authentic and lo-fi; the avatar's existing background may be kept or swapped with a simple solid/gradient background for focus
- Pacing notes: UGC style — natural pauses, no over-cut; let the avatar breathe
- CTA delivery: how the final CTA appears on screen (text overlay timing, any end card)

**If `AD_TYPE = "Commercial Ad"` (or null):**
- Scene-by-scene breakdown with timing (e.g. 0–3s, 3–10s, etc.)
- For each scene: CREATOR direction (what they say/do on camera), camera angle and shot type, TEXT OVERLAY content and placement
- Overall visual aesthetic (lighting style, color palette, setting)
- Pacing and edit rhythm notes
- CTA delivery (how it looks and sounds)

### 3b.5 Generate Images/Elements Needed (or Talking Avatar Brief)

**If `AD_TYPE = "UGC"`:** Generate a **Talking Avatar Brief** for the video editor:

**Avatar Reference**
File: `references/assets/ugc-avatar.png`
Describe the avatar's appearance, setting, and style as visible in the image.

**Scene Setup**
- Background recommendation (keep existing background, or composite a new one — specify which and why based on the ad concept)
- Lighting style to match the avatar's existing look
- Any wardrobe or prop changes needed (usually none)

**On-Screen Text Overlays**
List each text overlay with approximate timing, content, and placement (e.g. top-third, lower-third, center). Sync to the script beats from the transcript.

**B-Roll / Cutaway Suggestions**
Any supplementary clips or screen recordings to intercut with the avatar (e.g. product UI, testimonial screenshots, app demos). Mark as optional if the ad concept doesn't require them.

**Editor Instructions**
"Take the ElevenLabs audio file (from the ElevenLabs generated file property) and the avatar image (`references/assets/ugc-avatar.png`). Animate the avatar to lip-sync the audio using your preferred talking avatar tool (HeyGen, D-ID, Synthesia, etc.). Layer the text overlays and any B-roll per the scene plan above. Export as vertical 9:16 for TikTok/Reels."

**If `AD_TYPE = "Commercial Ad"` (or null):** Using the `image-generation.md` framework, identify every visual asset the video editor will need to generate with AI (background scenes, product close-ups, lifestyle shots, text overlays, graphic elements, etc.) and write one high-quality prompt per asset.

For each image/element:
- Label it clearly (e.g. **Hero Shot**, **Product Close-Up**, **Lifestyle Scene 1**)
- Write a prompt following the Master Prompt Template from the reference doc:
  `[Style anchor] + [Subject] + [Environment] + [Lighting] + [Camera] + [Film/Style] + [Realism details] + [Composition]`
- Include a negative prompt (50–100 words targeting the most likely failure modes for that image type)
- Recommend the best tool for that asset (Flux 1.1 Pro Ultra, Midjourney v7, Runway Gen-4.5, etc.) based on the Tool Comparison Matrix in the reference doc

Produce as many prompts as the ad actually needs — typically 2–6 images per ad.

### 3b.6 Write Video Idea and Images/Elements to Notion

Split each output into 1900-character chunks.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Video Idea": {"rich_text": [<VIDEO_IDEA_CHUNKS>]},
      "Images/Elements Needed": {"rich_text": [<IMAGES_CHUNKS>]},
      "Stage": {"select": {"name": "Ad Idea"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

Note: For UGC rows, `Images/Elements Needed` will contain the Talking Avatar Brief instead of AI image prompts.

### 3b.7 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 3b message format.

### 3b.8 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Draft"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 3b error: <ERROR_MESSAGE>. Will retry automatically on next run."}}]}
    }
  }'
```

---

## Stage 3c: Ad Idea Revision

**Trigger:** `Status = Changes Requested`, `Stage = Ad Idea`

The reviewer has left feedback in `Notes`. Revise the Ad Idea, Video Idea, and/or Images/Elements based on the feedback, then set Status back to `Needs Approval`.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Changes Requested"}},
        {"property": "Stage", "select": {"equals": "Ad Idea"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 3c.1 Lock the row

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{"properties": {"Status": {"status": {"name": "Processing"}}}}'
```

### 3c.2 Read fields

Extract:
- `TOPIC = properties["Topic/Angle"].rich_text[*].plain_text` joined with space
- `TRANSCRIPT = properties["Transcript"].rich_text[*].plain_text` joined with empty string
- `EXISTING_AD_IDEA = properties["Ad Idea"].rich_text[*].plain_text` joined with empty string
- `EXISTING_VIDEO_IDEA = properties["Video Idea"].rich_text[*].plain_text` joined with empty string
- `EXISTING_IMAGES = properties["Images/Elements Needed"].rich_text[*].plain_text` joined with empty string
- `REVISION_NOTES = properties["Notes"].rich_text[*].plain_text` joined with space
- `AD_TYPE = properties["Ad Type"].select.name` — `"UGC"`, `"Commercial Ad"`, or null. Treat null as `"Commercial Ad"`.

If `REVISION_NOTES` is empty, write an error to Notes and restore Status = `Changes Requested`.

### 3c.3 Read reference documents

Follow the same reference reads as Stage 3a.3 and Stage 3b.3 based on AD_TYPE:
- `references/short-form-tactics.md`
- `references/video-production.md`
- If `AD_TYPE = "Commercial Ad"` (or null): also read `references/image-generation.md`
- If `AD_TYPE = "UGC"`: also view `references/assets/ugc-avatar.png`

### 3c.4 Revise outputs based on notes

Using the references, transcript, existing content, and revision notes, produce updated versions. The revision notes take precedence over defaults.

Prompt framing:
> Topic: {TOPIC}
> Transcript: {TRANSCRIPT}
> Existing Ad Idea: {EXISTING_AD_IDEA}
> Existing Video Idea: {EXISTING_VIDEO_IDEA}
> Existing Images/Elements: {EXISTING_IMAGES}
> Reviewer feedback: {REVISION_NOTES}
> Revise whichever outputs the feedback addresses. If the notes only mention one output (e.g. "change the hook strategy"), revise that output and carry forward the others unchanged.

Apply the same generation rules as Stages 3a.4, 3b.4, and 3b.5.

### 3c.5 Write revised outputs and clear Notes

Split each output into 1900-character chunks.

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Ad Idea": {"rich_text": [<AD_IDEA_CHUNKS>]},
      "Video Idea": {"rich_text": [<VIDEO_IDEA_CHUNKS>]},
      "Images/Elements Needed": {"rich_text": [<IMAGES_CHUNKS>]},
      "Notes": {"rich_text": []},
      "Stage": {"select": {"name": "Ad Idea"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

### 3c.6 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 3c message format.

### 3c.7 Error handling

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Status": {"status": {"name": "Changes Requested"}},
      "Notes": {"rich_text": [{"type": "text", "text": {"content": "Stage 3c error: <ERROR_MESSAGE>. Fix the issue and the pipeline will retry automatically."}}]}
    }
  }'
```

---

## Stage 4: Await Video Output

**Trigger:** `Status = Approved`, `Stage = Ad Idea`

The ad idea, video idea, and image prompts are approved. Move the row to the Video Output stage so the editor knows to upload the final clip.

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Approved"}},
        {"property": "Stage", "select": {"equals": "Ad Idea"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 4.1 Set stage to Video Output

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Stage": {"select": {"name": "Video Output"}},
      "Status": {"status": {"name": "Needs Approval"}}
    }
  }'
```

The video editor uploads the final clip to the `Video Output` property, then sets Status = Approved to advance to completion.

### 4.2 Send Slack notification

Send a Slack notification per the Slack Notification Pattern above. Use the Stage 4 message format.

---

## Stage 5: Completion

**Trigger:** `Status = Approved`, `Stage = Video Output`

### Query rows

```bash
one --agent actions execute notion conn_mod_def::GJ5EnL2xERY::ib0N5v41TveieFZQUV-vuQ \
  YOUR_NOTION_CONNECTION_KEY \
  --skip-validation \
  --path-vars '{"dataSourceId": "YOUR_NOTION_DATABASE_ID"}' \
  -d '{
    "filter": {
      "and": [
        {"property": "Status", "status": {"equals": "Approved"}},
        {"property": "Stage", "select": {"equals": "Video Output"}}
      ]
    },
    "page_size": 10
  }'
```

For each result row:

### 5.1 Mark complete

```bash
one --agent actions execute notion conn_mod_def::GJ5En6Rb6Bg::HBiyT8YHSkWyd8Wg5qHFCg \
  YOUR_NOTION_CONNECTION_KEY \
  --path-vars '{"page_id": "<ROW_ID>"}' \
  -d '{
    "properties": {
      "Stage": {"select": {"name": "Complete"}},
      "Status": {"status": {"name": "Done"}}
    }
  }'
```

---

## Done

Output nothing on success. If any row errors, output a brief summary: `Row <ROW_TITLE> (<ROW_ID>): <STAGE> failed — <ERROR>`.
