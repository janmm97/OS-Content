# How I Built: video-production.md

## What This Document Is

A video production reference that teaches the pipeline how to write scene-by-scene production guides for video editors. It covers motion graphics trends, editing rhythm, text overlay conventions, pacing, platform-specific export specs, and AI-assisted post-production tools.

The pipeline reads this in Stages 3a and 3b when generating the Video Idea — the production guide that tells an editor exactly how to build the video.

---

## Why It Matters

Without this doc, the pipeline produces vague video ideas like "show the product being used with text overlays." With a strong video-production.md, it produces detailed scene plans with timing, shot types, overlay placements, and editing rhythm — something an editor can actually follow.

---

## Step 1: Research with Perplexity

Run these queries and compile the results:

**Motion graphics and visual trends:**
```
What are the dominant motion graphics and visual design trends in digital advertising in 2025-2026? Cover: authenticity vs polish, kinetic typography, color trends, transitions, and what's replacing the over-produced look of 2022-2023.
```

**Editing rhythm and pacing for short-form:**
```
What are the best practices for editing rhythm and pacing in short-form video ads in 2026? How do cut frequency, jump cuts, and natural pauses affect viewer retention? What's different about UGC editing vs produced commercial editing?
```

**AI-assisted post-production:**
```
What AI tools are being used for video post-production in 2026? Cover talking avatar tools (HeyGen, D-ID, Synthesia), AI video generation (Runway, Sora, Kling), upscaling, color grading, and motion graphics automation.
```

**Text overlays and captions:**
```
What are the best practices for on-screen text overlays and captions in short-form video ads in 2026? Cover: placement, timing, font style, animation, and how text reinforces the voiceover vs distracts from it.
```

**Platform-specific production specs:**
```
What are the technical production specifications for TikTok, Instagram Reels, and YouTube Shorts in 2026? Cover: aspect ratio, resolution, safe zones for text overlays, audio specs, and any platform-specific format requirements.
```

**UGC video production:**
```
How are brands producing UGC-style talking head ads at scale in 2026 using AI avatar tools? Cover talking avatar tools, background options, B-roll integration, and how to make AI avatars look authentic rather than synthetic.
```

---

## Step 2: Synthesize with Claude

```
I'm building a video production reference for a content pipeline. The pipeline reads this doc to write detailed scene-by-scene production guides for video editors. The pipeline produces two types of ads: UGC (talking head avatar) and Commercial Ad (produced footage + voiceover).

Here's my research: [paste research]

Write a comprehensive video production guide with these sections:

1. Motion Graphics and Visual Trends (2025–2026) — what's working, what's replaced the over-polished look, authenticity aesthetics
2. Editing Rhythm and Pacing — cut frequency, natural pauses, UGC vs produced edit style
3. Color Grading and Visual Finishing — tone, mood, platform-appropriate color
4. AI-Assisted Post-Production — talking avatar tools, AI video generation, when to use each
5. Text Overlays and Captions — placement rules, timing, animation style, safe zones
6. Short-Form Production Techniques — specific to 15–60 second formats
7. Platform-Specific Specs — TikTok, Reels, Shorts: aspect ratio, safe zones, audio, export
8. Scene Plan Format — how to write a clear scene-by-scene production guide for an editor

For section 8, provide a template scene plan format the pipeline can use consistently.

Write it as instructional guidance with specific rules and examples.
```

---

## Step 3: Add Your Production Style

After the base document is generated:

```
Add a section called "9. Our Production Style" covering:

- Default aesthetic: [describe your visual style — minimal, energetic, cinematic, raw/authentic, etc.]
- UGC setup: [describe the background, lighting, and setup for your talking avatar ads]
- Text overlay style: [font type, placement, animation, timing conventions]
- B-roll sources: [where your editors source B-roll — screen recordings, stock, original footage]
- Color palette: [your brand colors and how they appear in video]
- Editor handoff format: [what format you deliver assets to your editor in]
```

---

## Step 4: Build a Scene Plan Template

The most important output of this doc is a repeatable scene plan format. Add a dedicated section with a template:

```
Also create a "Standard Scene Plan Template" section with:

- A template for a 15-second UGC ad (scene breakdown with timing)
- A template for a 30-second Commercial Ad (scene breakdown with timing)
- Instructions for the editor on how to read and use this plan
- Notes on which elements are mandatory vs editorial judgment calls
```

---

## Step 5: Refine Against Real Video Ideas

After the pipeline generates video ideas for 10+ rows, evaluate each:

- Is the scene breakdown specific enough for an editor to follow without asking questions?
- Are the text overlay timings clearly tied to specific script beats?
- Does the overall aesthetic direction match your brand?
- For UGC: is the avatar brief detailed enough for a talking-avatar tool?

Add rules and examples to close any gaps.

---

## What a Good Version Includes

- Current visual and motion graphics trends (updated at least twice a year)
- Clear UGC vs Commercial Ad production differences
- A named scene plan template the pipeline uses every time
- Text overlay placement rules with safe zone references
- Platform export specs (aspect ratio, safe zones, audio)
- Talking avatar tool guidance for UGC ads
- Your production aesthetic and style rules in a dedicated section
