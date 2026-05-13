# How I Built: content-strategist.md

## What This Document Is

A comprehensive reference guide that teaches the pipeline how to think about content strategy — how to develop a sharp angle from a topic or source URL, how to write a compelling title, and how to evaluate whether an idea is worth producing.

The pipeline reads this doc in Stage 0 (when generating a Title + Topic/Angle from a scraped URL) and Stage 0b (when revising). It's the foundation everything else builds on.

---

## Why It Matters

Without this doc, Claude defaults to generic content strategy thinking. With a well-built content-strategist.md, the pipeline produces titles and angles that match your POV, your audience, and your content positioning — not a generic best-practice template.

---

## Step 1: Research with Perplexity

Run these queries one at a time. Copy the results into a working document.

**Core strategy frameworks:**
```
What are the most effective content strategy frameworks used by B2B SaaS companies in 2025-2026? Include frameworks like Jobs-to-Be-Done, StoryBrand, pillar-cluster, and others with examples of when to use each.
```

**Topic and angle development:**
```
How do professional content strategists develop a sharp content angle? What separates a topic from an angle? Include examples of weak vs strong angles and the frameworks used to sharpen them.
```

**Title writing:**
```
What are the best practices for writing high-performing content titles in 2026? Include data on which title formats (lists, how-tos, contrarian, result-first) perform best and why. Include examples.
```

**Content audits and planning:**
```
How do content teams run content audits and build editorial calendars in 2026? What metrics matter, what tools are used, and how do you identify content gaps vs content cannibalization?
```

**Audience research methods:**
```
What are the best methods for audience research in content strategy? Include qualitative and quantitative approaches, tools, and how to translate research into content decisions.
```

---

## Step 2: Synthesize with Claude

Paste all your Perplexity research into Claude and use this prompt:

```
I'm building a master reference document for a content pipeline. The pipeline uses this doc to make decisions about topic/angle development and title writing.

Here is all my research: [paste research]

Turn this into a structured, instructional reference document with these sections:

1. Role & Mindset — what it means to think like a content strategist
2. Core Responsibilities — content audits, editorial planning, audience research, messaging hierarchy, distribution
3. Content Frameworks — the key strategy frameworks with when-to-use guidance (Jobs-to-Be-Done, StoryBrand, pillar-cluster, PAS, BAB, etc.)
4. Topic & Angle Development — how to take a broad topic and sharpen it into a specific, defensible angle with a clear POV
5. Title Writing — the formats that work, with examples of weak vs strong titles and the principles behind each format
6. Content Calendar & Planning — how to maintain a pipeline with both evergreen and timely content

For each section, include concrete examples. Write it as instructional guidance, not descriptions. It will be read by an AI to make decisions.
```

---

## Step 3: Add Your Specifics

After Claude produces the base document, add a section specific to your brand:

```
Now add a section called "7. Our Content POV" that covers:

- Our target audience: [describe your audience in 2–3 sentences]
- The main tension we write into: [what is the core problem or frustration your content addresses]
- Content pillars: [list 3–5 topic areas you focus on]
- Content we avoid: [describe what's off-brand or outside scope]
- Our typical angle style: [examples of angles that have worked well]
```

---

## Step 4: Refine Against Pipeline Output

Run the pipeline on 5–10 test rows with URLs or topics. For each row where the Title or Topic/Angle misses:

1. Note what was wrong (too generic, wrong audience, off-brand tone, etc.)
2. Add a rule or example to the relevant section of the doc
3. Re-run and compare

Typical refinements:
- Add more examples of strong vs weak angles in Section 5
- Add "avoid these title formats" guidance with examples
- Clarify your audience segment in Section 7 if the pipeline is writing for the wrong person

---

## What a Good Version Includes

- At least 3 named content frameworks with when-to-use guidance
- A clear explanation of "topic vs angle" with before/after examples
- 6+ title format templates with examples
- A "3-point angle test": audience + tension + outcome
- Your brand's content pillars and POV
- Examples of content that's in-scope vs out-of-scope for your pipeline
