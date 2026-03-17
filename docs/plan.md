# Plan: `deep-research` Skill — Multi-Agent Research System

## Context

Inspired by [Anthropic's multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system), we want a Claude Code skill that takes a single research prompt and produces a **single, well-cited markdown file** through multiple iterations of parallel research, synthesis, and citation verification. Each report must frame findings using the **Outcomes / Outputs / Results** framework (from tabula.b4madservice.workers.dev).

## Skill Structure

```
~/.claude/skills/deep-research/
├── SKILL.md                            # Main orchestration (~400 lines)
└── references/
    ├── output-template.md              # Markdown template for the deliverable
    └── research-guide.md               # Heuristics: complexity, source quality, search strategy
```

No `scripts/` — the workflow is entirely prompt-driven using native tools (Agent, WebSearch, WebFetch).

## Architecture: 6-Phase Pipeline

```
User prompt
    │
    ▼
┌─────────────────────────┐
│ Phase 1: Plan           │  Parse input, classify complexity, generate research axes,
│                         │  save plan to disk (context survival)
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Phase 2: Parallel       │  Spawn 2-8 subagents via Agent tool (one per axis)
│          Research       │  Each: WebSearch (2-5x) → WebFetch (1-3x) → condensed findings
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐     ┌──────────────────┐
│ Phase 3: Synthesis      │────▶│ Phase 4: Gap-fill │──┐
│  Merge, dedup, assess   │     │ Targeted subagents│  │
│  gaps & contradictions  │◀────┘ for missing axes  │◀─┘
└──────────┬──────────────┘       (max 3 iterations)
           │ coverage sufficient
           ▼
┌─────────────────────────┐
│ Phase 5: Citation       │  Dedicated CitationAgent: WebFetch each source URL,
│          Verification   │  verify claims + verbatim quotes → verification table
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Phase 6: Write Output   │  Single markdown file using output-template.md
│                         │  → ./research-output/<slug>-<date>.md
└─────────────────────────┘
```

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **Subagents return fixed schema** (Key Findings, Quotations, Source Quality, Gaps) max 2000 words each | Context window budget — 8 subagents × 2000w = 16K, leaves room for synthesis |
| **Research plan saved to disk** (`./research-output/.research-plan-<slug>.md`) | Survives context compaction beyond 200K tokens (Anthropic's pattern) |
| **CitationAgent is a separate phase**, not inline | Separation of concerns: synthesis = creative coherence, verification = skeptical checking |
| **Max 3 iteration rounds** | Hard cap prevents infinite loops while allowing: initial → gap-fill → refinement |
| **Complexity classification** drives subagent count | Simple (1-2), Moderate (3-4), Complex (5-8) — avoids over-researching simple queries |
| **No scripts/** | Entire workflow is tool-based; a shell script would add indirection without determinism gains |

## Subagent Prompt Design

Each research subagent receives a **self-contained prompt** with:

- Role ("research subagent for one axis")
- Scope (one research axis, 2-3 sentences)
- Context (overall question, 1-2 sentences — NOT the full plan)
- Tool guidance (WebSearch 2-5x, then WebFetch 1-3x on best hits)
- Fixed output schema (Key Findings, Exact Quotations, Source Quality, Gaps, Raw Sources)
- Hard constraint: max 2000 words, condensed only, no raw HTML

Subagents do NOT receive: the full plan, other subagents' findings, or the output template.

## CitationAgent Design

Receives the synthesized draft. For each claim-source pair:

1. `WebFetch` the URL with targeted prompt: "Find the section supporting: [claim]"
2. Classify: **VERIFIED** / **PARTIAL** / **UNVERIFIED** / **SOURCE_UNAVAILABLE**
3. For quotes: check **VERBATIM MATCH** / **PARAPHRASE** / **NOT FOUND**
4. Return verification table

## Output Template (references/output-template.md)

The final markdown file contains:

1. **Executive Summary** — 2-3 paragraphs for the reader who reads nothing else
2. **Key Findings** — Numbered, with inline citations and verification badges
3. **Analysis** — Interpretive synthesis connecting findings
4. **Outcomes / Outputs / Results** — Dedicated section applying the framework:
   - Output: what this report delivers (e.g., "3000-word analysis with 12 verified citations")
   - Result: what understanding the reader gains (e.g., "clear picture of X with verified claims")
   - Outcome: what action this enables (e.g., "informed architecture decision for Y")
   - Hypothesis chain: "If we deliver [output], we expect [result], which should drive [outcome]"
5. **Quotations** — Exact quotes with author, source title, URL, section/page, verification status
6. **Sources / Bibliography** — All sources with credibility rating and verification status
7. **Methodology** — Subagents spawned, iterations, sources consulted, verification rate
8. **Verification Summary** — Table: total/verified/partial/unverified/unavailable

## Research Guide (references/research-guide.md)

Contains heuristics for:

- **Complexity classification** (simple/moderate/complex → subagent count)
- **Research axis generation** (paper URL: background, methodology, claims, reception, related work; broad topic: definition, history, current state, stakeholders, controversies, future)
- **Source quality hierarchy** (peer-reviewed > institutional > journalism > tech docs > expert blogs > general)
- **Search strategy** (2-3 query formulations per axis, include current year, use site-specific searches for known domains)
- **Context budget** (2000w/subagent, ~20K total findings, leaves room for template + orchestration)

## Files to Create

| File | Purpose | Lines (est.) |
|---|---|---|
| `~/.claude/skills/deep-research/SKILL.md` | Main skill: frontmatter + 6 phases + subagent/citation templates | ~400 |
| `~/.claude/skills/deep-research/references/output-template.md` | Markdown template for final output | ~80 |
| `~/.claude/skills/deep-research/references/research-guide.md` | Research heuristics and quality criteria | ~120 |

## Verification

After implementation, test with:

1. **Simple query**: `/deep-research "What is RLHF?"` — should spawn 1-2 subagents, complete in one iteration
2. **URL-based**: `/deep-research https://arxiv.org/abs/...` — should fetch and analyze the paper, verify citations
3. **Complex topic**: `/deep-research "Compare RAG vs fine-tuning for enterprise LLM deployment"` — should spawn 4-5 subagents, possibly iterate once
4. Check output file exists at `./research-output/`, contains all template sections, citations marked VERIFIED/UNVERIFIED
