# goern-skills

A collection of Claude Code skills, agents, and slash commands by goern.

## Components

### Skill: deep-research

Turns a single research prompt into three artifacts — a deep-research artifact, a publishable humanized report, and a companion blog post — through parallel multi-agent research, iterative synthesis, citation verification, multi-perspective team review, and a final humanizing pass.

Inspired by [Anthropic's multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system).

### Agents: dr-{critic,skeptic,editor,audience}

Four single-shot reviewer agents that audit a research draft from orthogonal angles:

| Agent | Role |
|-------|------|
| `dr-critic` | Adversarial — load-bearing logical and factual breakage |
| `dr-skeptic` | Evidence audit — citation quality, source diversity, claim-source fit |
| `dr-editor` | Structure and clarity — argument reconstructability, narrative scaffolding |
| `dr-audience` | Reader-fit — does the draft answer the question for the intended reader |

These ship as reusable subagent types. Use them directly via the `Agent` tool or spawn the whole team with `/team:review`.

### Command: /team:review

Spawns a 4-teammate review team (`dr-critic` + `dr-skeptic` + `dr-editor` + `dr-audience`) against any research draft, report, or technical writeup. Synthesizes convergent findings, auto-applies unambiguous patches, surfaces stalemates as open questions.

```text
/team:review research-output/topic-2026-04-28.md
/team:review research-output/topic.md | What are the tradeoffs of X for Y?
```

Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` (see Requirements below).

## How `deep-research` Works

```text
Prompt → Plan → Parallel Research (2-8 subagents) → Synthesis → Gap-Fill →
Citation Verification → Write Artifact → Team Review → Blog Post → Humanize
```

1. **Plan** — classifies complexity, generates MECE research axes
2. **Research** — spawns parallel subagents, each searching and fetching independently
3. **Synthesize** — merges findings, deduplicates, identifies gaps and contradictions
4. **Gap-Fill** — spawns targeted subagents for missing coverage (max 3 iterations)
5. **Verify** — dedicated CitationAgent checks every claim against its source URL
6. **Write artifact** — produces the deep-research artifact with verification badges
6.5. **Team review** — `dr-critic` + `dr-skeptic` + `dr-editor` + `dr-audience` review the artifact, lead synthesizes and auto-applies patches (skipped gracefully if agent teams are not enabled)
7. **Blog post** — generates a 1-2 minute first-person companion blog post
8. **Humanize** — produces a publishable report via the `write-like-goern` skill (Critical/Analytical register, structure preserved)

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

## Requirements

The `/team:review` command and Phase 6.5 of `deep-research` use Claude Code's experimental agent teams feature. To enable it, set the environment variable in your shell:

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Or via direnv (recommended) — drop a `.envrc` in the project where you call `deep-research`:

```bash
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> .envrc
direnv allow
```

If unset, the deep-research skill will skip Phase 6.5 with a one-line note in the plan file and proceed without team review. The pipeline still produces the deep-research artifact, the blog post, and the publishable report — just without the multi-perspective hardening pass.

The Phase 8 humanize pass requires the [`write-like-goern`](https://github.com/goern/dotfiles) skill (or any compatible skill named `write-like-goern`) to be available. If absent, the skill notes the gap and skips humanization.

## Usage

```text
/goern-skills:deep-research "What is RLHF?"
/goern-skills:deep-research "Compare RAG vs fine-tuning for enterprise LLM deployment"
/goern-skills:deep-research https://arxiv.org/abs/2401.12345
```

Output is written to `./research-output/`:

- Deep research artifact: `<slug>-<date>.md`
- Publishable report: `<slug>-publishable-<date>.md`
- Blog post: `<slug>-blog-<date>.md`

## Complexity Scaling

| Complexity | Subagents | When |
|------------|-----------|------|
| Simple     | 1–2       | Single concept, definitional |
| Moderate   | 3–4       | Comparison, multi-faceted, emerging field |
| Complex    | 5–8       | Cross-domain, controversial, primary sources needed |

## Output Sections (deep-research artifact)

- Executive Summary
- Key Findings (with inline citations and verification badges)
- Analysis
- Outcomes / Outputs / Results (hypothesis chain)
- Quotations (verbatim, with verification status)
- Sources / Bibliography (with credibility ratings)
- Methodology (subagents, iterations, gaps)
- Verification Summary (verified / partial / unverified / unavailable)
- Team Review (convergent findings, applied patches, open questions) — when Phase 6.5 ran

## Project Structure

```text
.claude-plugin/
  plugin.json             # Plugin manifest
  marketplace.json        # Marketplace entry
agents/
  dr-critic.md            # Adversarial reviewer
  dr-skeptic.md           # Evidence auditor
  dr-editor.md            # Structure reviewer
  dr-audience.md          # Reader-fit reviewer
commands/
  team/
    review.md             # /team:review slash command
skills/
  deep-research/
    SKILL.md              # 8-phase orchestration prompt
    references/
      output-template.md
      blog-post-template.md
      research-guide.md
```

## License

<https://spdx.org/licenses/GPL-3.0-or-later.html>
