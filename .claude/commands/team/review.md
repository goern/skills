---
name: "Team: Review"
description: Spawn a 4-teammate research review team (critic + skeptic + editor + audience) to harden any research draft, report, or writeup before publishing
category: Agent Teams
tags: [agent-teams, research, review]
---

Spawn a multi-perspective review team against a research draft, report, or technical writeup.

**Input** (`$ARGUMENTS`): one of
- A path to the target file (e.g. `research-output/topic-2026-04-28.md`)
- A target plus the original research question, separated by `|` (e.g. `research-output/topic.md | What are the tradeoffs of X for Y?`)
- Empty — ask which file

## Pre-flight

1. **Verify agent teams enabled.** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` must be set (typically via `.envrc`). If not, refuse and tell the user to add it and reload the shell.

2. **Verify the target file exists.** Refuse otherwise.

3. **Determine the audience and the original question.**
   - If the user passed a question after `|`, use it verbatim.
   - Otherwise, look for a "Research Question" or "Question" field in the target's frontmatter or opening section.
   - If still unknown, ask the user once.

4. **Read the target end-to-end** before spawning. Lead must mediate, mediation needs context.

5. **Create a temp dir** at `<target-dir>/.review/` for teammate outputs.

## Team spawn

Create a team with four teammates. All read-only — none edit the target.

| Name | Subagent type | Model | Role |
|------|--------------|-------|------|
| `critic` | `research-critic` | opus | Adversarial review — load-bearing logical/factual breakage |
| `skeptic` | `research-skeptic` | opus | Evidence audit — citation quality, source diversity, claim-source fit |
| `editor` | `research-editor` | sonnet | Structure and clarity — argument reconstructability, narrative scaffolding |
| `audience` | `research-audience` | sonnet | Reader-fit — does the draft answer the question for the intended reader |

Spawn prompts (fill `<target>` and `<output-dir>`):

- **critic**: "Adversarially review `<target>`. Apply your load-bearing critique bar. Write to `<output-dir>/.review/critic.md`. Post path to the team mailbox addressed to `lead`. Coordinate with `skeptic` if you both flag the same paragraph."
- **skeptic**: "Audit the evidence base of `<target>`. Verify citations, check source mix, look for missed counter-evidence. Write to `<output-dir>/.review/skeptic.md`. Post path to the team mailbox addressed to `lead`."
- **editor**: "Review structure and clarity of `<target>`. Map the argument graph, flag TOC gaps and order errors. Write to `<output-dir>/.review/editor.md`. Post path to the team mailbox addressed to `lead`."
- **audience**: "Reader-fit review of `<target>`. Original question: `<question>`. Does the draft answer it for the intended reader? Write to `<output-dir>/.review/audience.md`. Post path to the team mailbox addressed to `lead`."

## Mediation loop (lead's job — i.e. you)

1. Wait for all four teammates to report. **Do not implement anything yourself** while they work.
2. Read each temp file.
3. Synthesize a single `## Team Review — <YYYY-MM-DD>` section appended to the target, containing:
   - **Convergent findings** — issues flagged by 2+ teammates. Load-bearing.
   - **Per-perspective findings** — flagged by one teammate. Lower priority but record.
   - **Patches applied** — file:line edits the lead made directly to the target. Default to applying any blocker or major-severity fix where the smallest-fix is unambiguous.
   - **Open questions** — stalemates, ambiguous fixes, decisions that need the human user.
   - **Strongest moves preserved** — what the team flagged as load-bearing wins. Do not regress these in revision.
4. Apply patches inline. Do not stop and ask the user for each one — the human-involvement budget is one decision at the end (accept synthesis or push back).
5. Delete `<target-dir>/.review/` after synthesis. Temp files served their purpose.

## Hard rules

- **Maximum one round.** This is fan-out review, not multi-round debate. If teammates disagree, surface as an open question — don't loop.
- **No teammate edits the target.** Synthesis is lead-only.
- **Convergence is the signal.** A critique flagged by one teammate is interesting; by two, load-bearing.
- **Lead applies patches directly when smallest-fix is unambiguous.** When ambiguous, list the patch as an open question instead of guessing.
- **Token budget warning** — four teammates plus mediation against a 5,000-word report is heavy. Surface this before spawning if the target is unusually large.

## Cleanup

After the synthesis section is written and patches applied, run team cleanup. Confirm with the user that the synthesis is acceptable before doing so.

## Argument: $ARGUMENTS
