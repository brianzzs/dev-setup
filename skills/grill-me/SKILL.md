---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Each question should be formatted like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## Write the plan

Once confirmed, write the shared understanding to a lightweight file before implementing anything — this is what lets `tdd` and `code-review` pick it back up later, in this session or a new one, without an issue tracker or any other setup.

- Path: `.claude/plans/<slug>.md` at the repo root (create the directory if missing). `<slug>` is a kebab-case name for the feature/decision — derive it from the topic, or ask if it's unclear.
- Content:
  - `# <Title>` — one line on what this is.
  - `**Status:** confirmed <today's date>` — get the real date (e.g. `date +%F`), never hardcode or guess it.
  - `## Decisions` — every settled question as `- **Q:** ... **A:** ...`, in the order they were resolved. This is the trail of *why*, not just *what* — keep it, don't compress it into prose only.
  - `## Seams to implement` — the concrete public interfaces this plan implies, phrased the way the `tdd` skill's seam-confirmation step expects (e.g. "user can X given Y"). This section is what lets `tdd` skip re-asking for seam agreement.
  - `## Out of scope` — anything explicitly ruled out during the session, if applicable.
- Tell the user the path once written. It's a plain markdown file — committing it or gitignoring it is their call, not yours to decide.
- If a plan file for this slug already exists, overwrite it: the file reflects current confirmed state, not a history of revisions.