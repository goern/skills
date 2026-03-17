# Output Template — Deep Research Report

Use this template for the final markdown output. Fill every section.
Replace bracketed placeholders with actual content.

---

```markdown
# [Research Title]

> **Generated:** [YYYY-MM-DD]
> **Query:** [original user prompt]
> **Verification Rate:** [X]% ([verified]/[total] claims)
> **Sources Consulted:** [N]
> **Research Iterations:** [N]

---

## Executive Summary

[2-3 paragraphs summarizing the most important findings for the reader who reads
nothing else. Lead with the answer, not the process. Include the single most
surprising or important finding.]

---

## Key Findings

1. **[Finding title]** — [1-3 sentence description with inline citation.]
   [Source N] [VERIFIED ✅ | PARTIAL ⚠️ | UNVERIFIED ❓]

2. **[Finding title]** — [...]
   [Source N] [STATUS]

[Continue for all major findings, typically 5-12.]

---

## Analysis

[Interpretive synthesis connecting the key findings into a coherent narrative.
Address contradictions explicitly. Highlight what is well-established vs.
emerging vs. contested. This section adds analytical value beyond listing facts.
Typical length: 3-6 paragraphs.]

---

## Outcomes / Outputs / Results

### Output
[What this report concretely delivers. Be specific: word count, number of
verified citations, number of sources, coverage of axes.]

### Result
[What understanding or clarity the reader gains from this output. What questions
are now answered? What was previously unclear that is now resolved?]

### Outcome
[What action, decision, or next step this enables. How does this research change
what the reader can do?]

### Hypothesis Chain
> If we deliver **[output]**, we expect the reader to gain **[result]**,
> which should drive **[outcome]**.

---

## Quotations

> "[Exact verbatim quote]"
> — **[Author/Organization]**, *[Source Title]*, [URL]
> Section/Page: [location in source]
> Verification: [VERBATIM_MATCH ✅ | PARAPHRASE ⚠️ | NOT_FOUND ❌]

[Repeat for 3-8 key quotations.]

---

## Sources / Bibliography

| # | Title | Author/Org | URL | Type | Credibility | Verification |
|---|-------|-----------|-----|------|-------------|-------------|
| 1 | [title] | [author] | [url] | [peer-reviewed/institutional/journalism/tech-doc/blog/other] | [high/medium/low] | [VERIFIED/PARTIAL/UNVERIFIED/SOURCE_UNAVAILABLE] |

[Include ALL sources consulted, not just those cited in findings.]

---

## Methodology

- **Research approach:** [Brief description of overall strategy]
- **Subagents spawned:** [N] (axes: [list axis names])
- **Iterations performed:** [N] (initial + [N-1] gap-fills)
- **Total sources consulted:** [N]
- **Sources fetched (full content):** [N]
- **Unresolved gaps:** [List any axes or questions that remain poorly covered]
- **Limitations:** [Known constraints: paywalled sources, recency gaps, etc.]

---

## Verification Summary

| Metric | Count |
|--------|-------|
| Total claims verified | [N] |
| ✅ Verified | [N] |
| ⚠️ Partial | [N] |
| ❓ Unverified | [N] |
| 🚫 Source unavailable | [N] |
| **Verification rate** | **[X]%** |

| Metric | Count |
|--------|-------|
| Total quotes checked | [N] |
| ✅ Verbatim match | [N] |
| ⚠️ Paraphrase | [N] |
| ❌ Not found | [N] |
```
