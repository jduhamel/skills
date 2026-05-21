---
name: running-impl-notes
description: While implementing tasks, maintain a running implementation-notes.html (or .md) that captures design decisions, deviations, tradeoffs, and open questions as work proceeds — not as an end-of-task report.
triggers:
  - running notes
  - impl notes
  - implementation notes
  - track decisions
  - dec/dev/tr/oq
  - running implementation-notes.html
argument-hint: "[optional path or filename, default hydra-implementation-notes.html]"
---

# Running Implementation Notes Skill

## Purpose

Keep a living log alongside the code while implementing a task. The log is not a post-hoc summary — it's updated *while* the work happens so the user can read it at any point and understand:

- What choices Claude made when the spec was ambiguous
- Where the implementation diverged from the spec/plan, and why
- What alternatives were considered before settling
- What questions the user needs to answer

## When to Activate

Activate when the user kicks off implementation work and either:

1. Explicitly asks for a running notes file ("maintain a running implementation-notes.html", "track decisions as you go", "capture deviations/tradeoffs").
2. Sets a longer-form goal via `/goal` that includes design-quality concerns (decisions, deviations, tradeoffs, open questions).
3. Says "implement X..Y" for multi-step work where ambiguity is likely.

If the user has not asked, **do not** spin up a notes file uninvited. The skill is opt-in.

## File Format

Default to the file the user named. If they didn't name one:

- If there's already an `implementation-notes.html` (or `*-implementation-notes.html`) in the project root, append to it.
- Otherwise create `implementation-notes.md` (Markdown) at the project root.
- Use HTML only when the user explicitly asks for it (richer formatting + browser preview).

Whatever format, **four sections** are mandatory:

1. **Design decisions** — choices made where the spec/plan was ambiguous. Each entry: short title, date, scope (which task/day/file), rationale.
2. **Deviations** — places where the implementation intentionally departs from the spec/plan, and why. Mark each as "open" or "resolved".
3. **Tradeoffs** — alternatives considered, what was rejected and why, what was chosen.
4. **Open questions** — things the user needs to confirm or revise. Each one ends in a question mark and is actionable.

Each entry gets a stable ID prefix: `DEC-001`, `DEV-001`, `TR-001`, `OQ-001`. New entries get the next available number — never reuse IDs even after resolving.

## Workflow

1. **At task start:** check whether the notes file exists. If not, create the skeleton with the four sections. Add a short header that captures the current goal and date.
2. **While implementing:** every time you (Claude) make a non-obvious call, immediately append an entry to the right section. Don't batch — write the entry the moment the choice is made, so the file reflects your thinking timeline.
3. **At intermediate stops** (end of a sub-task, before committing): re-read the notes file briefly, fix anything that's now stale, and update the running status line at the top ("Status: P-2 in progress / P-1 shipped").
4. **At task end:** make sure every open deviation has either been resolved or explicitly captured as still-open. Open questions should be the cleanest list possible for the user to answer.

## Heuristics for what's worth writing down

Worth an entry:

- Picked option A over option B because of constraint X.
- Spec said one thing, real codebase implies another — went with the codebase.
- Library/framework's defaults don't fit; we override.
- A test boundary is fuzzy and you picked an interpretation.
- A field name, route shape, or schema decision that downstream code will depend on.
- Something you'd want to explain to a code reviewer who hasn't seen the conversation.

**Not worth an entry:**

- "I called Edit instead of Write" — process noise.
- "Test passed" — that's the commit message's job.
- "Renamed variable for clarity" — too small.

When in doubt, err on the side of writing it down. A two-line entry is cheap; reconstructing a decision three weeks later is not.

## Entry style

- Each entry has a header line with ID, title, and date.
- The body is 2–5 short paragraphs, not bullet soup. Reference real file paths and line numbers when relevant.
- Bold the key claim. Use `<code>` (HTML) or backticks (Markdown) for identifiers.
- For open questions, end with **Open:** and the actual question phrased so the user can answer "yes / no / option 2".

## Examples

```
> Joe: /goal implement P-1..P-4. Maintain a running implementation-notes.html
       capturing design decisions, deviations, tradeoffs, open questions.

Claude: [activates running-impl-notes skill]
        Reads existing hydra-implementation-notes.html.
        Begins P-1 work. After picking snapshot-history over patch-log
        for undo:
        - Adds DEC-010 to the notes file describing the choice
        - Adds TR-006 describing the alternatives considered
        Continues with P-2, etc., appending entries as decisions land.
```

```
> Joe: implement the new validation rule

Claude: [does NOT activate — single-step, no spec ambiguity]
```

## Notes

- The notes file is for the **user**, not for Claude's own memory. Write it the way you'd brief a teammate joining the project tomorrow.
- The four sections are non-negotiable. If you find yourself wanting a fifth (like "Lessons" or "Things to revisit"), fold it into "Open questions" or "Tradeoffs".
- Commit the notes file alongside the code change that introduced the entries. The commit message can be short ("…and log DEC-009 / TR-006 / OQ-005") — the diff is the audit trail.
- When the work is multi-PR, keep the notes file unified. Don't fork it per branch — it's the long-running narrative.
