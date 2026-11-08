---
name: research
description: Investigate a question against high-trust primary sources and capture the findings as a Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork delegated to a background agent.
---

Spin up a **background agent** to do the research, so you keep working while it reads.

Its job:

1. Investigate the question against **primary sources** (official docs, source code, specs, first-party APIs), not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Write the findings to a single Markdown file, citing each claim's source.
3. Save it under the repo's research path. Default: `docs/research/NNNN-slug.md` — numbered sequentially, never renumbered, matching whatever numbering already exists. If the repo carries `docs/agents/artifact-paths.md`, that file wins. Never write it to `.scratch/` (that holds in-flight tickets) or to `.tmp/` (that holds throwaways). Say where you saved it.

Fetch pages, PDFs, and archives into `$TMPDIR` — `.tmp/` inside the checkout when `TMPDIR` is unset — never into the repo tree, and delete them when the record is written.
