---
name: deep-research
description: >-
  Multi-agent research system that takes a research prompt and produces a
  well-cited markdown report through parallel research, synthesis, gap-filling,
  citation verification, multi-perspective team review, and a final humanizing
  pass. Produces three artifacts: the raw deep-research artifact (full citations,
  verification table, methodology), a publishable humanized report (Goern's
  voice, structure preserved), and a companion 1-2 minute first-person blog
  post. Outputs to research-output/ in the caller's working directory.
user-invocable: true
---

# Deep Research — Multi-Agent Research System

You are orchestrating a multi-agent research pipeline. Follow the 8 phases below
sequentially. Save intermediate state to disk so it survives context compaction.

**Output artifacts produced (in order):**

1. **Deep research artifact** — `<slug>-<YYYY-MM-DD>.md` — raw report with full
   citations, verification table, methodology. Preserved untouched after Phase 6.
2. **Publishable report** — `<slug>-publishable-<YYYY-MM-DD>.md` — humanized
   version of the artifact via the `write-like-goern` skill. Structure, headers,
   tables, and citations preserved verbatim.
3. **Blog post** — `<slug>-blog-<YYYY-MM-DD>.md` — 250–500 word first-person
   distillation.

**Human-involvement budget:** the pipeline is designed to run end-to-end with
minimal user prompts. Auto-apply unambiguous review patches, auto-proceed
between phases. Surface a decision to the user only when the team review
produces stalemates or when a phase fails irrecoverably.

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

4. **Write the file** to `<CWD>/research-output/<slug>-<YYYY-MM-DD>.md` —
   this is the **deep research artifact**. After Phase 6.5 review patches are
   applied, this file is preserved as-is for the rest of the pipeline.

5. **Update the plan file**: `Status: WRITING → REVIEWING`

6. **Proceed to Phase 6.5** for multi-perspective team review.

---

## Phase 6.5: Team Review (real agent team)

Spawn a 4-teammate review team against the artifact to harden it before publishing.

**Prerequisite:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` must be set (typically
via `.envrc`). If unset, skip Phase 6.5 with a one-line note in the plan file
and proceed to Phase 7. Do not block the pipeline.

### Team spawn

Create a team with four teammates, all read-only. None edits the artifact —
only the lead (you) applies patches.

| Name | Subagent type | Model | Role |
|------|--------------|-------|------|
| `critic` | `dr-critic` | opus | Adversarial — load-bearing logical/factual breakage |
| `skeptic` | `dr-skeptic` | opus | Evidence audit — citation quality, source diversity, claim-source fit |
| `editor` | `dr-editor` | sonnet | Structure and clarity — argument reconstructability, narrative scaffolding |
| `audience` | `dr-audience` | sonnet | Reader-fit — does the draft answer the question for the intended reader |

Create a temp dir at `<CWD>/research-output/.review-<slug>/` for teammate outputs.

Spawn prompts (fill `<artifact>`, `<question>`, `<output-dir>`):

- **critic**: "Adversarially review `<artifact>`. Apply load-bearing critique bar. Write to `<output-dir>/critic.md`. Post path to team mailbox addressed to `lead`."
- **skeptic**: "Audit evidence base of `<artifact>`. Verify citations, check source mix, look for missed counter-evidence. Write to `<output-dir>/skeptic.md`. Post path to lead."
- **editor**: "Review structure and clarity of `<artifact>`. Map argument graph, flag TOC gaps and order errors. Write to `<output-dir>/editor.md`. Post path to lead."
- **audience**: "Reader-fit review of `<artifact>`. Original research question: `<question>`. Does the draft answer it for the intended reader? Write to `<output-dir>/audience.md`. Post path to lead."

### Mediation loop (lead's job — i.e. you)

1. Wait for all four teammates to report. Do not implement anything yourself while they work.
2. Read each temp file.
3. Synthesize a `## Team Review — <YYYY-MM-DD>` section appended to the artifact, containing:
   - **Convergent findings** — issues flagged by 2+ teammates. Load-bearing.
   - **Per-perspective findings** — flagged by one teammate. Lower priority but record.
   - **Patches applied** — file:line edits the lead made directly to the artifact.
   - **Open questions** — stalemates, ambiguous fixes (these go to the user).
   - **Strongest moves preserved** — load-bearing wins flagged by the team.
4. **Auto-apply patches** for any blocker- or major-severity finding where the
   smallest-fix is unambiguous. Do not pause to confirm — the human-involvement
   budget is one decision at the end of the pipeline.
5. **Surface to user only** if open questions block the meaning of a section
   (e.g. unresolvable contradiction, unverifiable load-bearing source). Otherwise
   record as open questions and proceed.
6. Delete `<output-dir>` after synthesis. Temp files served their purpose.

### Hard rules

- **Maximum one round.** Fan-out review, not multi-round debate.
- **No teammate edits the artifact.** Lead-only synthesis.
- **Convergence is the signal.** Flagged by one teammate = interesting; by two = load-bearing.
- **Token budget warning** — four teammates against a 5,000-word artifact is heavy. If the artifact is unusually large, note it in the plan but proceed.

After review, update the plan file: `Status: REVIEWING → BLOGGING`.

---

## Phase 7: Blog Post

After Phase 6.5 review and patches, generate a companion blog post that distills
the research into a quick, accessible read.

1. **Read the blog post template** from
   `./references/blog-post-template.md`.

2. **Select content for the blog post:**
   - Pick the 3-4 most impactful findings from the artifact (prioritize VERIFIED claims).
   - Identify the single most surprising or counterintuitive finding.
   - Distill the actionable takeaway into one sentence.

3. **Write the blog post** following the template:
   - **Perspective:** First person ("I researched...", "I found...")
   - **Length:** 250-500 words (1-2 minute read)
   - **Tone:** Conversational but accurate — like explaining to a smart colleague
   - **Citations:** Reuse [Source N] references from the full report, but sparingly (2-4 max)
   - Do NOT explain the research methodology — point readers to the full report instead.

4. **Save the blog post** to `<CWD>/research-output/<slug>-blog-<YYYY-MM-DD>.md`

5. Update the plan file: `Status: BLOGGING → HUMANIZING`.

6. **Proceed to Phase 8** to produce the publishable humanized report.

---

## Phase 8: Humanize (publishable report)

Produce the **publishable report** — a humanized version of the deep research
artifact that reads like Goern wrote it, not like Claude wrote it for him.

**Critical:** the deep research artifact (`<slug>-<YYYY-MM-DD>.md`) is preserved
untouched. The publishable report is a **new file** — both artifacts coexist.

### Procedure

1. **Read the deep research artifact** end-to-end.

2. **Invoke the `write-like-goern` skill** with the artifact body as input.

   Pin the register explicitly in the invocation:

   > "Apply the **Critical / Analytical** register from `write-like-goern`.
   > This is a research report, not a casual note. Preserve formal structure;
   > tighten prose at sentence and paragraph level only."

3. **Hard constraints — the humanizer must NOT touch:**
   - All H1/H2/H3 headers (verbatim, no rewording, no lowercase conversion).
   - All tables (verification table, source quality table, etc.).
   - All code blocks and fenced quotations.
   - All citation markers `[Source N]` — must remain in identical positions.
   - The Methodology section (must stay neutral and technical).
   - The Verification Table (must stay verbatim).
   - URLs and reference list at the bottom.
   - The Outcomes / Outputs / Results framework section structure.

4. **Hard constraints — the humanizer SHOULD apply:**
   - Front-load the point in each section (move buried ledes up).
   - Strip throat-clearing, hedge walls, and filler ("It is important to note that...").
   - Tighten compound sentences doing too much work.
   - Strip AI-tells: "Das ist nicht X, das ist Y" constructions, staccato triadic
     punches, era-openings, empty pivots, inflated abstractions.
   - Run the tension-payoff check on each claim — if a sentence only adds rhythm,
     cut it.
   - Preserve normal capitalization (Critical/Analytical register, not casual).
   - English (or matching the artifact's input language).

5. **Save the publishable report** to
   `<CWD>/research-output/<slug>-publishable-<YYYY-MM-DD>.md`

   Add a frontmatter block at the top:

   ```markdown
   ---
   source-artifact: <slug>-<YYYY-MM-DD>.md
   humanized-via: write-like-goern (Critical/Analytical register)
   date: <YYYY-MM-DD>
   ---
   ```

6. **Verify structural fidelity.** Diff the publishable report's headers and
   tables against the artifact. If any header was renamed, any table was reformatted,
   or any `[Source N]` marker is missing, re-run Phase 8 with a stricter pin.

7. Update the plan file: `Status: HUMANIZING → COMPLETE`.

8. **Report to the user** — mention all three output files (artifact + publishable + blog post).

### Why two artifacts and not one

The deep research artifact is the *evidence record* — formal, neutral, defensible
under audit. The publishable report is the *communication layer* — Goern's voice,
ready to ship. Keeping them separate means the artifact stays trustworthy for
re-use (citations re-checked, claims re-verified) while the publishable version
can be more direct without sacrificing the underlying rigor.

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
- **Preserve the deep research artifact.** Phase 8 must never overwrite the
  artifact file produced by Phase 6. The publishable report is always a new file
  with the `-publishable-` infix in its name.
- **Preserve structural fidelity in the publishable report.** Headers, tables,
  citation markers, Methodology, and Verification Table must match the artifact
  byte-for-byte. The `write-like-goern` pass operates only on prose between
  these structural elements.
- **Skip Phase 6.5 gracefully if agent teams are disabled.** If
  `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` is not set, log a one-line note in the
  plan file and proceed to Phase 7. Do not block the pipeline.
