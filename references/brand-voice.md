# How I Built: brand-voice.md

## What This Document Is

A brand voice reference that tells the pipeline exactly who is speaking, how they speak, and what that sounds like in practice. It covers core voice attributes, tone by channel, persona-specific hook guidance, and what to avoid.

The pipeline reads this in Stages 1 and 1b to ensure every transcript sounds like your brand — not a generic AI-generated script.

---

## Why It Matters

This is the document that makes the pipeline's output sound like *you*. Without it, Claude defaults to clean, neutral, professional copy that could belong to anyone. With a specific brand voice doc, the scripts have a recognizable POV and consistent personality.

---

## The Source Material

Unlike the other reference docs, brand-voice.md can't be built primarily through external research — it has to come from inside your company. The research phase here is internal, not Perplexity-based.

Gather these before writing:

1. **Existing content** — your top-performing social posts, email subject lines, and ad copy. What do the good ones have in common?
2. **Customer language** — support tickets, Slack messages, sales call transcripts. How does your audience actually describe their problems?
3. **Competitor voice** — how do competitors talk? You want to be clearly different.
4. **Founder voice** — if UGC ads feature a founder or key persona, record a few minutes of them speaking naturally. Transcribe it. That's your voice source.

---

## Step 1: Identify Voice Attributes

Ask Claude to help you define your voice by analyzing your existing content:

```
Here are 10 examples of our best-performing content: [paste examples]

Analyze the voice and tone across these examples. Identify:
1. The 4–5 core voice attributes (e.g. direct, confident, builder-first, warm, etc.)
2. What this voice is NOT (the opposite or adjacent voices we avoid)
3. 3 example sentences that sound like us and 3 that don't, per attribute

Format this as the beginning of a brand voice guide.
```

---

## Step 2: Research Voice Frameworks with Perplexity

```
What are the most effective brand voice frameworks used by successful B2B SaaS companies? Cover: how to define a brand voice, how to document it for AI use, and how to make it distinct from competitors. Include examples of strong brand voice docs.
```

```
What are the best practices for writing a brand voice guide that an AI can actually follow? What makes a brand voice guide specific enough to produce consistent output vs too vague to be useful?
```

---

## Step 3: Build the Full Document with Claude

```
I'm building a brand voice reference document for a content pipeline. The pipeline uses this doc when generating short-form video scripts to ensure they sound like our brand.

Our voice analysis: [paste from Step 1]
Framework research: [paste from Step 2]
Examples of content that sounds like us: [paste 5–10 examples]
Our target audience: [describe your audience]

Write a comprehensive brand voice guide with these sections:

1. Who We're Talking To — detailed audience description (not demographics, but mindset, goals, and frustrations)
2. Core Voice Attributes — 4–5 attributes, each with: definition, examples of what it sounds like, and examples of what it doesn't sound like
3. Tone by Channel — how the voice shifts across short-form video, email, social posts, and paid ads
4. UGC Persona Guidelines — specific guidance for first-person scripts: who the speaker is, their backstory, hook styles that fit, hook styles to avoid
5. Language Rules — specific words we use, specific words we avoid, formatting preferences
6. CTA Style — how we invite action (casual, specific, platform-native)
7. What We Avoid — a clear list of voice anti-patterns

Write it as instructional guidance an AI can directly follow, with examples throughout.
```

---

## Step 4: Add UGC Persona Details

The pipeline uses the brand voice doc specifically to calibrate the UGC persona. Add a dedicated section:

```
Now add a detailed "UGC Creator Persona" section covering:

- Who this person is: [role, background, relationship to the product]
- How they speak naturally: [formal/informal, technical/plain, fast/measured]
- Their relationship to the audience: [peer, mentor, power user, founder, etc.]
- Hook styles that fit this persona: [specific hook types from the hook framework]
- Hook styles that break character: [what would feel inauthentic for this person]
- Sample phrases that sound like them
- Sample phrases that don't
```

---

## Step 5: Refine Against Real Scripts

Run 10 UGC transcripts through the pipeline and evaluate each for brand alignment:

- Does this sound like someone from our company would actually say it?
- Would our target audience trust this voice?
- Are there any phrases that feel generic or corporate?

For each miss, identify the gap in the brand-voice.md and add a specific rule or example to close it. Vague guidance like "be authentic" doesn't help — concrete examples do.

---

## What a Good Version Includes

- 4–5 named voice attributes, each with positive AND negative examples
- A clear audience description focused on mindset and goals (not demographics)
- Explicit UGC persona section: who speaks, how they speak, what they avoid
- Hook style guidance specific to your persona (which hook types fit, which don't)
- A "what we avoid" list with examples (corporate jargon, over-promising, passive voice, etc.)
- Your standard CTA phrasing, formatted exactly as it should appear in scripts
- Tone shifts by channel — the voice in a TikTok UGC differs from a paid Reels commercial
