# goern-skills

A collection of Claude Code skills by goern.

## Skills

### deep-research

Turns a single research prompt into a well-cited markdown report and a companion blog post through parallel multi-agent research, iterative synthesis, and citation verification.

Inspired by [Anthropic's multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system).

## How It Works

```
Prompt → Plan → Parallel Research (2-8 subagents) → Synthesis → Gap-Fill → Citation Verification → Report → Blog Post
```

1. **Plan** — classifies complexity, generates MECE research axes
2. **Research** — spawns parallel subagents, each searching and fetching independently
3. **Synthesize** — merges findings, deduplicates, identifies gaps and contradictions
4. **Gap-Fill** — spawns targeted subagents for missing coverage (max 3 iterations)
5. **Verify** — dedicated CitationAgent checks every claim against its source URL
6. **Write** — produces a single markdown file with verification badges
7. **Blog** — generates a 1-2 minute companion blog post summarizing key findings in first-person perspective

Each report uses the **Outcomes / Outputs / Results** framework and includes a full bibliography with credibility ratings and verification status.

## Installation

```bash
claude plugin marketplaces add https://github.com/goern/goern-skills
claude plugin install goern-skills
```

Or for development:

```bash
claude --plugin-dir /path/to/goern-skills
```

## Usage

```
/goern-skills:deep-research "What is RLHF?"
/goern-skills:deep-research "Compare RAG vs fine-tuning for enterprise LLM deployment"
/goern-skills:deep-research https://arxiv.org/abs/2401.12345
```

Output is written to `./research-output/`:

- Full report: `<slug>-<date>.md`
- Blog post: `<slug>-blog-<date>.md`

## Complexity Scaling

| Complexity | Subagents | When |
|------------|-----------|------|
| Simple     | 1–2       | Single concept, definitional |
| Moderate   | 3–4       | Comparison, multi-faceted, emerging field |
| Complex    | 5–8       | Cross-domain, controversial, primary sources needed |

## Output Sections

- Executive Summary
- Key Findings (with inline citations and verification badges)
- Analysis
- Outcomes / Outputs / Results (hypothesis chain)
- Quotations (verbatim, with verification status)
- Sources / Bibliography (with credibility ratings)
- Methodology (subagents, iterations, gaps)
- Verification Summary (verified / partial / unverified / unavailable)

## Project Structure

```
.claude-plugin/
  plugin.json             # Plugin manifest
skills/
  deep-research/
    SKILL.md              # 7-phase orchestration prompt
    references/
      output-template.md  # Report template
      blog-post-template.md # Blog post template (1-2 min read, first-person)
      research-guide.md   # Complexity heuristics, source quality, search strategy
```

## License

<https://spdx.org/licenses/GPL-3.0-or-later.html>
