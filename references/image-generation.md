# How I Built: image-generation.md

## What This Document Is

A comprehensive AI image and video generation reference covering prompt architecture, camera and lens language, lighting techniques, tool comparisons, and negative prompt strategies. It teaches the pipeline how to write high-quality, specific prompts for the visual assets needed in Commercial Ad production.

The pipeline reads this in Stages 3b and 3c when generating the Images/Elements Needed section — the AI image prompts for each visual asset in the ad.

---

## Why It Matters

Vague image prompts produce generic, unusable outputs. A good image-generation.md gives the pipeline the vocabulary of professional photography and cinematography — so it writes prompts that produce photorealistic, on-brand visuals rather than stock-photo-looking AI images.

---

## Step 1: Research with Perplexity

This doc is more technical than the others. Perplexity is useful for current tool comparisons and emerging techniques; the core prompting knowledge is synthesized from AI image generation communities and documentation.

**Tool landscape:**
```
What are the best AI image and video generation tools in 2025-2026 for creating photorealistic marketing assets? Compare: Midjourney v7, Flux 2, GPT Image 2, Stable Diffusion, Runway Gen-4, Kling 3, and Luma. Cover: photorealism quality, prompt adherence, speed, and best use cases for each.
```

**Prompt architecture:**
```
What is the optimal prompt structure for generating photorealistic AI images in 2026? Cover: element ordering, style anchors, subject description, environment, lighting, camera specs, and quality tags. What's changed in prompting technique from 2023 to 2026?
```

**Camera and lens language:**
```
What camera, lens, and photography terminology produces the most consistent results in AI image generation prompts? Cover: lens focal lengths (35mm, 85mm, etc.), aperture effects, camera models, shooting styles, and how to reference professional photography terms effectively.
```

**Lighting for photorealism:**
```
What lighting terminology and descriptions produce the most realistic results in AI image generation? Cover: natural light setups, studio lighting types (Rembrandt, split, butterfly), golden hour, and how to describe light direction, quality, and color temperature.
```

**Negative prompts:**
```
What are the most effective negative prompt strategies for AI image generation in 2026? What are the most common failure modes (hands, faces, text, backgrounds) and the best negative prompts to prevent each?
```

**Human and portrait realism:**
```
What techniques produce the most realistic human subjects in AI image generation? Cover: how to describe skin texture, facial expression, eye detail, and avoid the "AI face" look. What's improved in 2025-2026 for human generation?
```

---

## Step 2: Synthesize with Claude

```
I'm building an AI image generation reference for a content pipeline. The pipeline reads this doc to write high-quality image prompts for short-form video ad assets. It needs to produce photorealistic marketing visuals — lifestyle shots, product-in-context, environmental backgrounds, and human subjects.

Here's my research: [paste research]

Write a comprehensive AI image generation guide with these sections:

1. How AI Models Interpret Prompts — what models are trained on, why photography language works, prompt order effects
2. Prompt Architecture for Photorealism — the master template: [Style anchor] + [Subject] + [Environment] + [Lighting] + [Camera] + [Film/Style] + [Realism details] + [Composition]
3. Camera and Lens Language — focal lengths, aperture effects, camera models, shooting styles
4. Lighting Mastery — natural light, studio setups, golden hour, how to describe light precisely
5. Film, Texture, and Color Grading Language — film stocks, grain, color science references
6. Composition Fundamentals — rule of thirds, leading lines, depth, negative space
7. Human and Portrait Realism — skin, expression, eyes, avoiding the AI face look
8. Environment and Background Description — how to describe spaces precisely
9. Negative Prompts — common failure modes and the negative prompts that prevent them
10. Tool Comparison Matrix — when to use Midjourney, Flux, Runway, GPT Image, etc.
11. Master Prompt Template — a fill-in template the pipeline uses for every image

For section 11, create a fill-in template with labeled slots for each prompt component.

Write it as a technical reference with specific examples for every concept.
```

---

## Step 3: Build the Master Prompt Template

The most important section is the prompt template. Work with Claude to refine it for your specific ad types:

```
Now refine the Master Prompt Template for two specific use cases:

1. Lifestyle/Product-in-Context shots — showing someone using a tool, app, or product in a real setting
2. Environmental/Background shots — background scenes without a human subject

For each, provide:
- A filled-in example prompt
- The negative prompt to pair with it
- The recommended tool
- Any parameters to use (--ar, --style, etc.)
```

---

## Step 4: Add Your Brand's Visual Style

```
Add a section called "12. Our Visual Style" covering:

- Overall aesthetic: [cinematic, editorial, documentary, clean/minimal, warm/authentic, etc.]
- Color palette notes: [any brand colors that should appear, color temperature preference]
- Subject types we typically need: [people using product, workspaces, abstract concept visuals, etc.]
- What to avoid: [visual styles that feel off-brand — overly bright stock, generic office settings, etc.]
- Reference images: [describe the visual style in terms of reference photographers or films if helpful]
```

---

## Step 5: Test with Real Rows

Run the pipeline on 5 Commercial Ad rows and evaluate the image prompts it produces:

- Are the prompts specific enough to produce distinct, non-generic images?
- Do they use the Master Prompt Template format?
- Are tool recommendations appropriate for each asset type?
- Do the negative prompts address the likely failure modes?

For each weak prompt, trace it to a missing rule or example in the doc and add it.

---

## What a Good Version Includes

- The master prompt template: `[Style anchor] + [Subject] + [Environment] + [Lighting] + [Camera] + [Film/Style] + [Realism details] + [Composition]`
- Camera and lens reference list (35mm = environmental, 85mm = portrait, etc.)
- Lighting vocabulary (Rembrandt, split, golden hour, overcast diffused, etc.)
- Film stock references (Kodak Portra 400, Fuji Velvia, etc.)
- Negative prompt templates for: faces, hands, text, backgrounds, and generic AI look
- Tool comparison matrix with "best for" column
- Your brand's visual style rules
- At least 3 filled-in example prompts with paired negative prompts
