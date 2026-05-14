# Automated Content Pipeline

An AI-powered content pipeline that takes a topic or source URL and produces a fully reviewed short-form video ad — transcript, voiceover audio, ad concept, video production guide, and image prompts — entirely through Notion with human approval at each stage.

Built to run on [Claude Routines](https://www.withone.ai), a cron job, or manually via the [One Agent](https://app.withone.ai/agents).

---

## What It Does

You drop a topic or URL into a Notion database. The pipeline handles the rest:

1. **Stage 0** — Scrapes the URL (if provided) and generates a Title + Topic/Angle for your approval
2. **Stage 1** — Writes a spoken script (UGC or Commercial Ad format)
3. **Stage 2** — Generates an ElevenLabs voiceover audio file and attaches it to the Notion row
4. **Stage 3** — Creates an Ad Idea, Video Production Guide, and Image Prompts (or Talking Avatar Brief for UGC)
5. **Stage 4** — Flags the row as ready for your video editor
6. **Stage 5** — Marks the row complete

At each stage, the pipeline sets Status = `Needs Approval` and sends a Slack notification with a link to the Notion row. You review and either Approve or request Changes — the pipeline picks it up on the next poll.

---

## Prerequisites

### 1. One CLI

Install and configure the [One CLI](https://www.withone.ai). This pipeline uses One as the unified interface to Notion, ElevenLabs, Slack, and Firecrawl.

```bash
# Install One CLI (see withone.ai for platform-specific instructions)
# Then connect your platforms:
one add notion
one add elevenlabs
one add slack
one add firecrawl
```

### 2. Connected Platforms

| Platform | Used For |
| --- | --- |
| **Notion** | Content database — reads rows, writes outputs, manages status |
| **ElevenLabs** | Text-to-speech — generates MP3 voiceover from the transcript |
| **Slack** | Notifications — sends a message when a stage needs approval |
| **Firecrawl** | Web scraping — used in Stage 0 to extract content from a source URL |

After connecting, run `one --agent list` to find your connection keys for each platform.

### 3. Notion Setup

**Create a Notion integration** at https://www.notion.so/my-integrations and share it with your database. Add the token to your `.env` file:

```
NOTION_TOKEN=secret_xxxxxxxxxxxx
```

**Create a Notion database** with these properties:

| Property | Type |
| --- | --- |
| `Name` | Title |
| `Status` | Status (values: `Draft`, `Processing`, `Needs Approval`, `Approved`, `Changes Requested`, `Done`, `Archived`) |
| `Stage` | Select (values: `Title/Angle`, `Transcript`, `Audio`, `Ad Idea`, `Video Output`, `Complete`) |
| `Topic/Angle` | Rich Text |
| `Transcript` | Rich Text |
| `Ad Idea` | Rich Text |
| `Video Idea` | Rich Text |
| `Images/Elements Needed` | Rich Text |
| `Notes` | Rich Text |
| `Ad Type` | Select (values: `UGC`, `Commercial Ad`) |
| `Link/Source` | URL |
| `ElevenLabs generated file` | Files & media |

### 4. ElevenLabs Voice IDs

Go to [ElevenLabs Voice Library](https://elevenlabs.io/voice-library) and choose two voices:
- One for UGC (conversational, personal)
- One for Commercial Ads (cinematic, polished)

Copy their Voice IDs — you'll need them in the skill configuration.

### 5. Reference Documents

The pipeline reads your brand and content docs at runtime to guide generation. Create these files in a `references/` folder:

| File | What to put in it |
| --- | --- |
| `references/content-strategist.md` | How you choose topics, develop angles, and write titles. Include your content pillars, target audience, and positioning. |
| `references/script-writer.md` | Your script writing frameworks, storytelling structures, CTA formulas, and funnel-stage guidance. |
| `references/short-form-tactics.md` | Short-form video best practices: hooks that work, pacing, scroll-stopping openers, platform-specific notes. |
| `references/elevenlabs-guide.md` | How to write for TTS: punctuation for pacing, word count targets per duration, sentence length guidance. |
| `references/brand-voice.md` | Your brand's tone, vocabulary, persona, what to avoid, and hook style guidance. |
| `references/video-production.md` | Your video editor's guide: scene structure, shot types, text overlay conventions, transitions, export specs. |
| `references/image-generation.md` | AI image generation framework: prompt templates, negative prompt guidance, tool recommendations (Midjourney, Flux, Runway, etc.). |
| `references/assets/ugc-avatar.png` | Your UGC avatar or talking-head image (used as reference for the video editor when Ad Type = UGC). |

The quality of the pipeline's output depends heavily on these docs. The more specific they are to your brand and style, the better the output.

---

## Setup

### Step 1: Clone this repo

```bash
git clone https://github.com/janmm97/OS-Content.git
cd OS-Content
```

### Step 2: Add your reference documents

Create the `references/` folder and add your brand, script, and production docs (see the table above).

### Step 3: Configure the skill

Open `SKILL.md` and replace all `YOUR_*` placeholders:

| Placeholder | Where to find it |
| --- | --- |
| `YOUR_NOTION_DATABASE_ID` | Notion database URL: `notion.so/workspace/<DATABASE_ID>?v=...` |
| `YOUR_NOTION_CONNECTION_KEY` | `one --agent list` → find your Notion key |
| `YOUR_ELEVENLABS_CONNECTION_KEY` | `one --agent list` → find your ElevenLabs key |
| `YOUR_UGC_VOICE_ID` | ElevenLabs Voice Library — copy the Voice ID |
| `YOUR_COMMERCIAL_VOICE_ID` | ElevenLabs Voice Library — copy the Voice ID |
| `YOUR_SLACK_CHANNEL_ID` | Right-click a Slack channel → Copy link → last segment of URL |

Also update the prompt framing in Stage 0.5 with your company name and description:
```
The content is for [Your Company] ([your-website.com]), [what your company does].
```

### Step 4: Create your `.env` file

In your project root:

```
NOTION_TOKEN=secret_xxxxxxxxxxxx
```

### Step 5: Run it

**Option A — Claude Routines (recommended)**

Set up a Claude Routine to run the skill on a schedule (e.g. every 5 minutes). Point it at `SKILL.md` in this repo.

**Option B — One Run**

Use [One Run](https://www.withone.ai) to schedule the skill as a recurring agent job.

**Option C — Manual / One CLI**

Run the skill manually anytime via the One CLI:

```bash
one --agent run skill SKILL.md
```

**Option D — Cron Job**

Set up a system cron to invoke the skill on your preferred interval.

---

## How to Use

1. Open your Notion database
2. Create a new row
3. Set **Ad Type** to `UGC` or `Commercial Ad`
4. Either:
   - Fill in **Topic/Angle** directly → pipeline picks it up on next poll and starts at Stage 1
   - Paste a URL into **Link/Source** → pipeline scrapes it and generates Title + Angle for your approval first (Stage 0)
5. The pipeline runs, sets Status = `Needs Approval`, and Slacks you a link
6. Review the output in Notion, then either:
   - Set Status = `Approved` → pipeline advances to the next stage
   - Add feedback to `Notes`, set Status = `Changes Requested` → pipeline revises and re-submits

---

## Pipeline Flow

```
URL/Topic added to Notion
       ↓
[Stage 0] Scrape URL → Generate Title + Angle
       ↓ (Approved)
[Stage 1] Generate Transcript
       ↓ (Approved)
[Stage 2] Generate ElevenLabs Audio
       ↓ (Approved)
[Stage 3a] Generate Ad Idea (auto-advances)
[Stage 3b] Generate Video Idea + Image Prompts
       ↓ (Approved)
[Stage 4] Flag as ready for video editor
       ↓ (Editor uploads video, sets Approved)
[Stage 5] Mark complete
```

At any stage, setting Status = `Changes Requested` + adding feedback to `Notes` sends it back for revision.

---

## Ad Types

**UGC** — Person-to-camera talking head. Produces a full conversational script + a Talking Avatar Brief (for tools like HeyGen, D-ID, or Synthesia). Best for TikTok/Reels authenticity.

**Commercial Ad** — Voiceover hook only (5–15 words) meant to play over visual footage. Produces AI image generation prompts for the visual assets. Best for cinematic/aspirational brand content.

---

## Powered by One CLI

This pipeline uses the [One CLI](https://www.withone.ai) to connect to Notion, ElevenLabs, Slack, and Firecrawl through a single unified interface — no custom API code required. One handles authentication, request building, and execution.

To explore what else One can do across 250+ platforms:

```bash
one --agent actions search <platform> "<what you want to do>"
```
