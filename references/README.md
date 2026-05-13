# References — How to Build Your Master Docs

This folder contains step-by-step guides explaining how to build each reference document the content pipeline reads at runtime. The actual content of these docs is intentionally left out — they should reflect **your brand, your audience, and your style**.

The pipeline is only as good as the docs you feed it. These guides explain the research process, the prompts, and the tools used to build each one.

---

## The Build Stack

| Tool | Role |
| --- | --- |
| **Perplexity** | Research — gather current best practices, platform data, and frameworks |
| **Claude** | Synthesis — turn research into structured, pipeline-ready reference docs |
| **One CLI** | Orchestration — connect Perplexity and other tools via `one --agent actions` |

### Connecting Perplexity via One CLI

```bash
# Connect Perplexity to your One account
one add perplexity

# Search for research
one --agent actions search perplexity "search"
one --agent actions knowledge perplexity <ACTION_ID>
one --agent actions execute perplexity <ACTION_ID> <CONNECTION_KEY> \
  -d '{"query": "your research query here"}'
```

---

## Reference Documents

| File | What It Drives in the Pipeline |
| --- | --- |
| [content-strategist.md](content-strategist.md) | Stage 0 — Title and Topic/Angle generation |
| [script-writer.md](script-writer.md) | Stages 1, 1b, 2b — Transcript generation and revision |
| [short-form-tactics.md](short-form-tactics.md) | Stages 1, 3a — Script style and ad concept |
| [elevenlabs-guide.md](elevenlabs-guide.md) | Stages 1, 1b — TTS-optimized script formatting |
| [brand-voice.md](brand-voice.md) | Stages 1, 1b — Persona, tone, and hook style |
| [video-production.md](video-production.md) | Stages 3a, 3b, 3c — Video idea and scene planning |
| [image-generation.md](image-generation.md) | Stages 3b, 3c — AI image prompt generation |
| [assets/ugc-avatar.png](assets/) | Stage 3b — Talking avatar reference for UGC ad type |

---

## General Process

Every reference document was built the same way:

1. **Research with Perplexity** — run targeted queries to pull current best practices, platform data, stats, and frameworks
2. **Synthesize with Claude** — paste the research and ask Claude to structure it into a dense, pipeline-readable reference doc
3. **Refine with real runs** — run the pipeline on test rows, read the outputs, and edit the reference doc until the pipeline produces what you want
4. **Version when major platform changes happen** — these docs should be updated at least quarterly as platforms evolve

The goal is a doc that gives Claude enough context to make good decisions without you having to prompt it every time.
