# How I Built: elevenlabs-guide.md

## What This Document Is

A technical writing guide specifically for generating high-quality AI voiceovers with ElevenLabs. It covers how to write text that sounds natural when spoken by an AI voice — punctuation as a pacing tool, sentence length targets, word counts per duration, SSML tags, voice settings, and common mistakes.

The pipeline reads this in Stages 1 and 1b to ensure every transcript is written in a way that will produce a professional-quality audio file.

---

## Why It Matters

Most people write transcripts like prose. ElevenLabs doesn't read prose — it reads scripts. A comma in the wrong place creates an unnatural pause. A 40-word sentence sounds robotic. Numbers written as digits get pronounced awkwardly. This doc prevents all of that.

---

## Step 1: Research with Perplexity

**Core TTS writing principles:**
```
How do you write scripts optimized for AI text-to-speech systems like ElevenLabs in 2025-2026? Cover: sentence length, punctuation for pacing, avoiding robotic delivery, and formatting best practices.
```

**SSML and ElevenLabs-specific:**
```
What SSML tags does ElevenLabs support and how do you use them for pacing and emphasis? Cover break tags, emphasis, phoneme tags, and how to use punctuation instead of SSML where possible.
```

**ElevenLabs voice settings:**
```
What do ElevenLabs voice settings do — stability, similarity boost, style, and speaker boost? How should you set these for different use cases: conversational UGC, cinematic voiceover, and narration?
```

**Model selection:**
```
What are the differences between ElevenLabs models (Turbo v2, Multilingual v2, v3) for short-form ad voiceovers? When should you use each and what are the tradeoffs in speed, quality, and expressiveness?
```

**Numbers, symbols, abbreviations:**
```
How should you format numbers, currency, symbols, and abbreviations in ElevenLabs scripts to ensure correct pronunciation? What are the common formatting mistakes that cause mispronunciation?
```

**Multi-speaker and word targets:**
```
For a 15-second and 30-second short-form video voiceover, what word count targets and sentence lengths work best with ElevenLabs? How do you pace a script to match a target duration?
```

---

## Step 2: Synthesize with Claude

You can supplement Perplexity research with the ElevenLabs documentation directly. The docs at elevenlabs.io cover SSML support, model differences, and voice settings in detail. Paste relevant sections alongside your Perplexity research.

```
I'm building a technical writing reference for a content pipeline that generates ElevenLabs TTS scripts. The pipeline will read this doc when writing transcripts to ensure they produce high-quality audio output.

Here's my research: [paste research + relevant ElevenLabs docs]

Write a comprehensive, instructional TTS writing guide with these sections:

1. Core Script Writing Principles — write conversationally, sentence length, read-aloud test
2. Punctuation as a Pacing Tool — how commas, periods, em-dashes, and ellipses affect delivery
3. SSML Break Tags and Pause Methods — when and how to use explicit pauses
4. SSML Phoneme Tags — how to control pronunciation of unusual words or brand names
5. Numbers, Symbols, Abbreviations — formatting rules for correct pronunciation
6. Voice Settings — what each parameter does and recommended settings for UGC vs commercial voiceover
7. Model Selection Guide — when to use each ElevenLabs model
8. Word Count and Pacing Targets — word counts for 15s, 30s, 60s durations
9. Common Mistakes — list of formatting issues that cause bad TTS output with fixes

Write it as a set of rules the pipeline can follow directly. Include examples for every rule.
```

---

## Step 3: Add Your Voice-Specific Rules

After the base document is generated, add your specific voice configurations:

```
Add a section called "10. Our Voice Configuration" covering:

- UGC voice: [voice name or ID, recommended stability/similarity settings]
- Commercial Ad voice: [voice name or ID, recommended settings]
- Brand name pronunciation: [if your brand name is unusual, add a phoneme note]
- Any words that get mispronounced: [list with correct phoneme tags or spelling workarounds]
- CTA phrasing: [exact wording of your standard CTA, formatted for TTS]
```

---

## Step 4: Test Against Real Audio

This doc is best refined by listening, not reading:

1. Run 5–10 transcripts through ElevenLabs
2. Listen for: unnatural pauses, rushed sentences, mispronounced words, flat delivery on emphasis points
3. Trace each issue to a missing rule in the guide and add it

Common discoveries:
- "And" before a list item needs a comma before it, not after
- Em-dashes create a longer pause than commas — good for dramatic beats
- Numbers should always be spelled out: "two hundred and fifty" not "250"
- Ellipses create a trailing-off effect — good for hooks, bad for CTAs

---

## What a Good Version Includes

- Word count targets: 30–35 words for 15s, 65–75 words for 30s
- Sentence length rule: 10–18 words per sentence
- A "read it aloud" quality check as a mandatory step before every script
- Punctuation guide with examples (comma, period, em-dash, ellipsis effects)
- Voice settings table per use case (UGC, commercial, narration)
- Numbers, symbols, and abbreviation formatting rules with examples
- Common mistakes list with before/after examples
