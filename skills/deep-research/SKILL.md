---
name: deep-research
description: >-
  Multi-agent research system that takes a research prompt and produces a
  well-cited markdown report through parallel research, synthesis, gap-filling,
  and citation verification. Also generates a companion 1-2 minute blog post
  summarizing key findings in first-person perspective. Outputs to
  research-output/ in the caller's working directory.
user-invocable: true
---

# Deep Research — Multi-Agent Research System

You are orchestrating a multi-agent research pipeline. Follow the 6 phases below
sequentially. Save intermediate state to disk so it survives context compaction.

**CRITICAL: Output Directory Rule**
All output files (research plans, reports, blog posts) MUST be written to
`research-output/` inside the **Claude Code session's current working directory** —
NOT inside the skill's own directory. Use absolute paths derived from the session's
CWD to avoid ambiguity.

Read the research guide and templates before starting (these are relative to the
skill directory, use the paths as provided):

- `./references/research-guide.md`
- `./references/output-template.md`
- `./references/blog-post-template.md`

---

## Phase 1: Plan

1. **Parse the input.** The user provides either:
   - A free-text research question
   - A URL (paper, article, repo) to analyze
   - A topic comparison (e.g., "X vs Y for Z")

2. **Classify complexity** using the research guide heuristics:
   - **Simple** (1–2 subagents): single concept, well-known topic, definitional
   - **Moderate** (3–4 subagents): multi-faceted topic, comparison, or emerging field
   - **Complex** (5–8 subagents): cross-domain, controversial, or requiring primary sources

3. **Generate research axes.** Each axis is a self-contained angle of inquiry.
   Axes should be mutually exclusive and collectively exhaustive (MECE).
   Use the research guide's axis templates for common input types.

4. **Save the plan to disk** at `<CWD>/research-output/.research-plan-<slug>.md`
   (where `<CWD>` is the session's current working directory):

   ```markdown
   # Research Plan: <title>
   - **Question:** <original prompt>
   - **Complexity:** simple | moderate | complex
   - **Subagent count:** N
   - **Axes:**
     1. <axis name>: <2-3 sentence scope>
     2. ...
   - **Iteration:** 1
   - **Status:** PLANNING → RESEARCHING
   ```

5. **Create the output directory** if it doesn't exist: `<CWD>/research-output/`

6. Tell the user the plan and how many subagents will be spawned.

---

## Phase 2: Parallel Research

Spawn one Agent per research axis. Launch all subagents **in parallel** using
a single message with multiple Agent tool calls.

### Subagent Prompt Template

Use this exact structure for each subagent prompt (fill in the bracketed parts):

```
You are a research subagent investigating ONE specific axis of a larger research question.

## Your Research Axis
[axis name]: [2-3 sentence scope description]

## Overall Research Question (context only — stay focused on YOUR axis)
[1-2 sentence summary of the user's original question]

## Instructions
1. Use WebSearch to find 2-5 high-quality sources relevant to your axis.
   - Try 2-3 different query formulations (see search strategy tips below).
   - Include the current year in at least one query for recency.
   - Use site-specific searches for known high-quality domains when relevant
     (e.g., site:arxiv.org, site:github.com).
2. Use WebFetch on the 1-3 best hits to extract detailed content.
3. Synthesize your findings into the fixed output schema below.

## Search Strategy Tips
- Start broad, then narrow based on initial results.
- If a search returns low-quality results, reformulate with more specific terms.
- For academic topics, search for survey/review papers first.
- For technical topics, search official docs + expert blog posts.

## Output Schema (STRICT — return ONLY this structure, max 2000 words total)

### Key Findings
- Numbered list of 3-8 findings, each 1-3 sentences.
- Each finding must cite its source: [Source N].

### Exact Quotations
- 2-5 verbatim quotes that are especially important or surprising.
- Format: > "quote text" — Author/Source, Title, URL

### Source Quality Assessment
For each source used:
| # | URL | Type | Credibility | Notes |
|---|-----|------|-------------|-------|
| 1 | ... | peer-reviewed / institutional / journalism / tech-doc / blog / other | high/medium/low | ... |

### Gaps Identified
- What could NOT be found or confirmed on this axis?
- What contradictions exist between sources?

### Raw Sources
- Full list of URLs consulted (including ones not cited above).
```

### After all subagents return

- Collect all findings. If any subagent fails or returns empty, note the gap.
- Update the plan file: `Status: RESEARCHING → SYNTHESIZING`

---

## Phase 3: Synthesis

Working from the collected subagent outputs:

1. **Merge findings** across axes. Group by theme, not by subagent.
2. **Deduplicate** — same fact from multiple sources strengthens confidence.
3. **Identify contradictions** — flag conflicting claims explicitly.
4. **Assess coverage gaps:**
   - Are any axes poorly covered (< 2 quality sources)?
   - Are there obvious follow-up questions the subagents didn't address?
   - Are there contradictions that need resolution?
5. **Decide: iterate or proceed.**
   - If significant gaps exist AND iteration count < 3 → go to Phase 4.
   - Otherwise → proceed to Phase 5.

Update the plan file with gap assessment and decision.

---

## Phase 4: Gap-Fill (max 3 iterations)

1. For each significant gap, spawn a **targeted subagent** using the same
   prompt template but with a narrower, gap-specific axis.
2. Spawn these in parallel (same as Phase 2).
3. Merge new findings into the synthesis.
4. Increment the iteration counter in the plan file.
5. Return to Phase 3 for re-assessment.

**Hard cap: 3 total iterations** (initial + 2 gap-fills). After 3 iterations,
proceed to Phase 5 regardless of remaining gaps. Note unresolved gaps in the
final report's Methodology section.

---

## Phase 5: Citation Verification

Spawn a single **CitationAgent** with this prompt:

```
You are a citation verification agent. Your job is SKEPTICAL CHECKING, not creative writing.

## Task
Verify that each claim-source pair in the research synthesis is accurate.

## Synthesized Draft
[Insert the full synthesized findings here]

## Instructions
For each cited claim:
1. Use WebFetch on the source URL.
2. Search the fetched content for the specific claim or quote.
3. Classify the claim:
   - **VERIFIED**: The source directly supports the claim as stated.
   - **PARTIAL**: The source partially supports but the claim overstates or simplifies.
   - **UNVERIFIED**: The source does not contain information supporting this claim.
   - **SOURCE_UNAVAILABLE**: The URL is inaccessible (paywall, 404, timeout).
4. For exact quotations, additionally classify:
   - **VERBATIM_MATCH**: The quote appears exactly in the source.
   - **PARAPHRASE**: The meaning is correct but wording differs.
   - **NOT_FOUND**: The quote cannot be located in the source.

## Output Schema (STRICT)

### Verification Table
| # | Claim Summary | Source URL | Claim Status | Quote Status | Notes |
|---|--------------|-----------|-------------|-------------|-------|
| 1 | ...          | ...       | VERIFIED/PARTIAL/UNVERIFIED/SOURCE_UNAVAILABLE | VERBATIM_MATCH/PARAPHRASE/NOT_FOUND/N_A | ... |

### Verification Summary
- Total claims: N
- Verified: N
- Partial: N
- Unverified: N
- Source unavailable: N
- Verification rate: X%

### Flagged Issues
- List any claims that should be removed, reworded, or re-sourced.
```

After CitationAgent returns:

- Incorporate verification statuses into the draft.
- Remove or flag claims that are UNVERIFIED (unless no alternative source exists,
  in which case mark clearly as unverified).
- Update the plan file: `Status: VERIFYING → WRITING`

---

## Phase 6: Write Output

1. **Read the output template** from
   `./references/output-template.md`.

2. **Generate the final report** following the template exactly. Fill every section.

3. **Apply the Outcomes / Outputs / Results framework** in the dedicated section:
   - **Output**: What this report concretely delivers (word count, citation count, etc.)
   - **Result**: What understanding the reader gains
   - **Outcome**: What action or decision this enables
   - **Hypothesis chain**: "If we deliver [output], we expect [result], which should drive [outcome]"

4. **Write the file** to `<CWD>/research-output/<slug>-<YYYY-MM-DD>.md`

5. **Update the plan file**: `Status: WRITING → COMPLETE`

6. **Proceed to Phase 7** to generate the companion blog post.

---

## Phase 7: Blog Post

After the full report is written, generate a companion blog post that distills
the research into a quick, accessible read.

1. **Read the blog post template** from
   `./references/blog-post-template.md`.

2. **Select content for the blog post:**
   - Pick the 3-4 most impactful findings from the report (prioritize VERIFIED claims).
   - Identify the single most surprising or counterintuitive finding.
   - Distill the actionable takeaway into one sentence.

3. **Write the blog post** following the template:
   - **Perspective:** First person ("I researched...", "I found...")
   - **Length:** 250-500 words (1-2 minute read)
   - **Tone:** Conversational but accurate — like explaining to a smart colleague
   - **Citations:** Reuse [Source N] references from the full report, but sparingly (2-4 max)
   - Do NOT explain the research methodology — point readers to the full report instead.

4. **Save the blog post** to `<CWD>/research-output/<slug>-blog-<YYYY-MM-DD>.md`

5. **Report to the user** — mention both output files (report + blog post).

---

## Guardrails

- **Never hallucinate sources.** Every URL must come from a WebSearch or WebFetch result.
- **Never fabricate quotes.** Every quotation must be verbatim from a fetched source.
- **Respect the context budget.** Subagent outputs are capped at 2000 words each.
  The synthesis should not exceed the input findings in length.
- **Fail gracefully.** If WebSearch returns nothing useful for an axis, note it as
  a gap — do not invent findings.
- **Be transparent about limitations.** The Methodology section must honestly report
  what was and wasn't found.
