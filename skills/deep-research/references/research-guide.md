# Research Guide — Heuristics and Quality Criteria

## Complexity Classification

Classify the research prompt to determine subagent count.

| Complexity | Subagents | Criteria | Examples |
|------------|-----------|----------|----------|
| **Simple** | 1–2 | Single concept, well-documented, definitional or explanatory | "What is RLHF?", "Explain CAP theorem" |
| **Moderate** | 3–4 | Multi-faceted, comparison, emerging field, or requires cross-referencing | "RAG vs fine-tuning for enterprise", "State of WebAssembly in 2026" |
| **Complex** | 5–8 | Cross-domain, controversial, requires primary sources, policy analysis, or deep technical comparison | "Impact of EU AI Act on open-source LLM deployment", "Compare 5 vector databases for production RAG" |

### Signals that increase complexity

- Comparison of 3+ alternatives → +1 subagent per extra alternative
- Requires historical context + current state → +1 subagent
- Involves policy, legal, or regulatory dimensions → +1 subagent
- Cross-domain (e.g., technical + business + legal) → complex

### Signals that decrease complexity

- Single well-known concept with canonical sources → simple
- User provides a specific URL to analyze → simple (but may add axes for context)
- Narrow, well-scoped question → simple

---

## Research Axis Templates

### For a paper or article URL

1. **Background & Context** — What problem does this address? What came before?
2. **Methodology & Approach** — How was it done? What are the key techniques?
3. **Claims & Results** — What are the main findings? Are they well-supported?
4. **Reception & Criticism** — How has the community responded? Known limitations?
5. **Related & Subsequent Work** — What builds on or competes with this?

### For a broad topic

1. **Definition & Scope** — What is it? What are the boundaries?
2. **History & Evolution** — How did it develop? Key milestones?
3. **Current State** — Where is it today? Who are the key players?
4. **Stakeholders & Use Cases** — Who uses it and why? Real-world applications?
5. **Controversies & Limitations** — What's debated? Known problems?
6. **Future Directions** — Where is it heading? Open research questions?

### For a comparison (X vs Y)

1. **X Deep-Dive** — Architecture, strengths, weaknesses, typical use cases
2. **Y Deep-Dive** — Architecture, strengths, weaknesses, typical use cases
3. **Head-to-Head Comparison** — Performance, cost, complexity, ecosystem
4. **Decision Framework** — When to choose X vs Y, hybrid approaches
5. **Real-World Case Studies** — Who chose X/Y and why? Outcomes?

### For a "how to" or technical question

1. **Canonical Approach** — Official docs, recommended practices
2. **Alternative Approaches** — Community solutions, workarounds, newer methods
3. **Pitfalls & Gotchas** — Common mistakes, edge cases, known issues

---

## Source Quality Hierarchy

Rank sources by type. Prefer higher-ranked sources when available.

| Rank | Type | Description | Examples |
|------|------|-------------|----------|
| 1 | **Peer-reviewed** | Academic papers, published research | arXiv (with peer review), ACM, IEEE, Nature |
| 2 | **Institutional** | Reports from established organizations | NIST, government agencies, research labs |
| 3 | **Journalism** | Investigative or technical journalism | Ars Technica, The Verge, MIT Tech Review |
| 4 | **Tech documentation** | Official product/project docs | docs.python.org, Kubernetes docs |
| 5 | **Expert blogs** | Posts by recognized domain experts | Personal blogs of known researchers, staff eng blogs |
| 6 | **General web** | Everything else | Medium posts, Stack Overflow, forum threads |

### Quality signals (positive)

- Author has verifiable credentials in the domain
- Publication has editorial review process
- Claims include citations to primary sources
- Data, methodology, or code is available for verification
- Recent publication date (within 2 years for fast-moving fields)

### Quality signals (negative)

- No author attribution
- No citations or references
- Promotional or sponsored content
- Outdated information without noting the date
- Aggregator or content farm

---

## Search Strategy

### Query formulation

For each research axis, generate 2–3 search queries:

1. **Broad query** — the axis topic in natural language
2. **Specific query** — include technical terms, author names, or project names
3. **Recency query** — add the current year or "2025 2026" for recent developments

### Site-specific searches

Use `site:` when you know where high-quality content lives:

- Academic: `site:arxiv.org`, `site:scholar.google.com`
- Technical: `site:github.com`, `site:stackoverflow.com`
- Institutional: `site:nist.gov`, `site:openai.com/research`
- News: `site:arstechnica.com`, `site:technologyreview.com`

### When initial searches fail

- Broaden the terms (remove specifics, use synonyms)
- Search for survey/review papers that cover the topic
- Search for the topic in a different language or domain
- Search for experts in the field, then search for their publications

---

## Context Budget

Respect these limits to keep the pipeline within context window constraints:

| Component | Budget |
|-----------|--------|
| Single subagent output | max 2,000 words |
| All subagent outputs combined | ~16,000 words (8 × 2,000) |
| Synthesis (pre-verification) | should not exceed total findings length |
| CitationAgent input | synthesis + verification instructions |
| Final report | no hard limit, but aim for 3,000–5,000 words |

### If budget is exceeded

- Prioritize findings by source quality and relevance
- Compress lower-priority findings into bullet points
- Move detailed evidence to an appendix section
- Never cut the Verification Summary or Sources table
