# Living Docs

I didn't set out to build a system. I just got tired of repeating myself.

Every new AI coding session, I'd paste the same context. Here's the project structure. Here's the naming convention. Here's why we don't touch that module. Here's the version bump command. The agent would do the work, and then the next session I'd do it all again. The knowledge was in my head, not anywhere the agent could find it.

So I wrote it down. An `agent.md` at the project root. A few rules, some metadata, a checklist. The agent read it at the start of each session and we got moving faster.

Then I noticed something: after each task, I'd tell the agent to update the docs. And it would — accurately. It would update the relevant section after fixing a bug, register the new module after adding a feature, write a warning after hitting a sharp edge in the codebase so the next session wouldn't hit it again.

The key word is *tell*. The agent doesn't update docs automatically. That's a feature, not a limitation.

If the agent updated docs silently after every code task, and that task had a bug, the docs would now describe wrong behavior as intended. The next session would read those docs and treat the mistake as a rule. It compounds. The human is the checkpoint — confirm the code is correct first, then instruct the agent to update. That sequence is what keeps the system trustworthy over time.

## TL;DR

Two habits. That's it.

**Before starting a task** — tell the agent what you want and tell it to read the docs first. No need to specify which files — it looks those up itself from the registry. Being explicit makes sure it doesn't skip.

**After finishing** — tell the agent to write down what was built. It updates the docs, registers new files, and keeps the knowledge current for the next session.

---

## What it became

Over time, one file wasn't enough. The logic was real and it was growing. Domain rules belonged somewhere separate from data rules. Architecture decisions didn't belong next to domain-specific constants. So I split things out.

Now the system looks like this:

**`agent.md`** — the entry point. Session-critical rules, project metadata, quick-reference checklists. Short by design. Tell your agent to read this at the start of every session.

**`docs/`** — the wiki. Every file owns exactly one concern. `GUIDE_developer.md` for coding standards. `ARCH_technical-specs.md` for data and routing. `STANDARDS_*.md` for domain-specific rules. `LOGIC_*.md` for feature-specific behavior. Each file has a clear prefix so both humans and agents know what's inside before opening it.

**`ARCH_documentation-governance.md`** — the registry. Every doc file mapped to: what it contains, what it must not contain, and when to load it. A task-to-file mapping so the agent knows exactly which docs to read for any given task.

The rule that holds it together: **one file owns each rule. No duplication.** If a rule exists in two places, you have two sources of truth, which means you have none.

## How it actually works

**The agent reads before acting.** `agent.md` provides session-critical context. For deeper tasks, the governance registry tells it which files to load based on the task type — only the relevant ones, not the entire `docs/` folder. You don't need to name specific files; the registry handles the routing.

**The agent updates after completing work — when you ask.** At the end of a task, run a "doc sweep": instruct the agent to sync changed logic, register any new files, and enforce ownership rules. This is deliberate. The human confirms correctness first. Then the docs get updated.

**Intentional quirks get flagged.** Any non-obvious decision in the codebase — a "wrong-looking" value that's correct for business reasons, an intentional timeout, an architectural decision you'll regret touching — gets marked:

```
> STUBBORN_FACT: [what it is] — [why it must stay this way]
```

The agent treats these as hard constraints. It won't "fix" them.

## Example prompts

Because `ARCH_documentation-governance.md` contains a Task→Load Mapping, the agent knows which files to read for any given task — you don't have to tell it. Say what you want in plain language, the registry handles the routing.

| What you want | What you type |
|---------------|---------------|
| Start a task with correct context | `I want to add X, read the relevant docs first` |
| Audit codebase and log violations | `audit this folder, write down what needs fixing` |
| Update docs after a task | `run a doc sweep` |
| Bootstrap a new project | `Read LIVING_DOC_SYSTEM.md and execute the 'Bootstrapping' sequence` |
| Flag intentional quirks | `flag any STUBBORN_FACTs you find` |
| Split an oversized file | `refactor this file, zero-loss protocol` |

No need to name specific files. The agent looks them up itself.

---

## Limitations

**The human checkpoint is the system.** If you approve bad code and then run a doc sweep, the docs now describe wrong behavior as correct. The agent can't catch that — it trusts the human's confirmation. Garbage in, garbage out.

**Needs a closing step.** The agent registers new files, enforces ownership, and keeps the registry accurate — but only when you run a doc sweep. The system stays healthy on its own as long as that habit is consistent.

**TDD is built in.** For logic, data processing, routing, rendering, and business rules, the agent writes tests before writing code. The only exception is docs, copy, renaming, formatting, and cosmetic edits — those skip TDD.

**Sometimes you need to remind it.** In practice, for tasks that look simple the agent may skip reading the docs. A short reminder is enough — "read the docs first" — it will find the right files itself. No need to specify which ones.

---

## The architecture, restated simply

Three layers:

**The codebase** — source of truth for what the code actually does.

**The docs** — LLM-maintained markdown files. The agent reads them before acting, updates them after (on explicit instruction). Governed by naming conventions, ownership rules, and a registry that prevents duplication.

**The schema** — `agent.md` plus the governance doc. Tells the agent how the doc system works, which file owns what, and what workflows to follow.

## The First Question

When you give an agent the master template, it is instructed to stop and ask you exactly one question before doing anything:

> **"Does this project have existing code?"**

**Do not let it proceed without answering.** This choice determines whether the agent follows **Path A (Greenfield)** or **Path B (Brownfield)**. If you have an existing codebase, the agent must audit it module-by-module to extract rules from actual behavior before writing any documentation.

## Quick Start (Bootstrapping a New Project)

Give your AI agent the following prompt:

> **"Read the master template at `https://github.com/Diew/living-docs/blob/main/LIVING_DOC_SYSTEM.md` and execute the 'Bootstrapping a New Project' sequence to initialize this current directory. STICK TO THE TEMPLATES VERBATIM — DO NOT SUMMARIZE."**

The agent will generate the required structure (`agent.md`, the `docs/` folder, and the governance registry). If your agent doesn't support URL fetching, download `LIVING_DOC_SYSTEM.md` and attach it directly.

Two paths are available:
- **Path A** — New project with no existing code.
- **Path B** — Existing codebase with no docs. The agent audits the code first, extracts rules from actual behavior, and flags intentional quirks as `STUBBORN_FACT` before writing anything down.

### What you get after bootstrapping

A skeleton — not a finished system. The structure is ready; the content grows with your project.

```
agent.md                              ← session rules, commands, task→load mapping
docs/
  ARCH_documentation-governance.md   ← registry: what each file owns, when to load it
  GUIDE_developer.md                  ← coding standards, TDD, refactor protocol
  REF_developer-reference.md          ← naming conventions, directory structure
  REF_template.md                     ← blank template for new doc files
  STANDARDS_ui-visual.md              ← design tokens, layout rules (placeholders)
  REFACTOR_TODO.md                    ← technical debt tracker (empty)
  INCIDENT_*.md                       ← (optional) post-mortems and regression logs
```

Files like `LOGIC_*.md` and `INCIDENT_*.md` are created later — when you have a feature complex enough to document, or a bug worth remembering. Every session that changes the code can add to these files. That's what makes them living.

## What the agent does on its own

These behaviors are always-on. The agent applies them during any coding task without being asked — because the rules are already loaded from `docs/`.

**Live code warnings**
While writing or editing code, the agent watches for threshold violations and flags them immediately:

| Trigger | What the agent does |
|---------|---------------------|
| Logic appears 2+ times | Flags it, extracts into a named function |
| Function body exceeds 20 lines | Warns, extracts inner logic into named helpers |
| Expression requires a comment to understand | Extracts into a named function |
| Function is reused across 2+ files | Moves to a shared helper file |
| File exceeds 200 lines | Warns, reviews for split |
| File exceeds 400 lines | Warns, split is mandatory |
| File contains 2+ unrelated concept groups | Warns, split regardless of line count |

**When the agent stops and asks**
The agent will pause mid-task and ask one question — never more — when it hits any of these:

- Task scope is wider than described and would touch files not mentioned
- Task contradicts a rule in `docs/` or a `STUBBORN_FACT`
- Task requires deleting or overwriting existing content
- Task is ambiguous enough that two valid interpretations produce different outcomes
- Code has no test coverage and the task requires TDD

One question. The most blocking unknown. Then it waits for you.

**How the agent communicates**
The system defines a strict communication style. Answer only what is asked. No intro, no recap, no filler. Fragments are acceptable. The pattern is: `[thing] [action] [reason]. [next step].`

Good:
```
Bug in auth middleware. Token expiry uses < not <=. Fix:
Function extracted. Tests pass. Ready to cut old code.
```

Bad:
```
Sure! I'd be happy to help you with that! The issue you're experiencing is likely...
```

Full sentences are reserved for security warnings, irreversible actions, and multi-step sequences.

---

## What you can ask the agent to do

These workflows are triggered by explicit human instruction.

**Doc Sweep**
After completing a task, instruct the agent to sync the docs. It will update files that own the changed logic, register any new files in the governance registry, and move misplaced rules to their correct owner. Duplicates get deleted.

**Code Audit → Todo List**
Ask the agent to audit the codebase against the rules in `docs/`. It reads the governing files, checks the actual code, identifies everything that violates a rule, and writes it down as a todo list — the agent figures out where to put it. You get a prioritized list of what needs fixing without touching a single file yourself.

**Zero-Loss Refactor**
For any large refactor, file split, or move, the agent follows a strict 6-step protocol:
1. **Audit** — read existing code fully before touching anything
2. **Create targets** — new structure in place before removing old code
3. **Bridge** — re-exports or adapters connect old to new (this step cannot be skipped)
4. **Verify** — typecheck and tests pass
5. **Cut** — remove old code only after behavior is confirmed stable
6. **Verify again** — full suite after cleanup

The bridge phase is the main guardrail against broken imports. The agent will not skip it.

**Sanity Check**
Not a formal workflow in the system, but a common pattern in practice. If something feels off after implementing, ask the agent to check it against the docs. It will compare what was just written against the relevant `LOGIC_*.md` or `ARCH_*.md` and tell you exactly where it diverges — before anything breaks.

```
check this against the docs
does this match our logic?
```

**Conflict Resolution**
If the codebase and the docs disagree, the agent follows a clear rule: code is the source of truth for *behavior* (what the system actually does), docs are the source of truth for *intent* (what it's supposed to do). If they drift, the agent determines whether the code is buggy (fix the code) or the doc is stale (fix the doc) — and resolves it. It never leaves a conflict unresolved.

**Cross-Project Memory (Global LLM Wiki)**
If you run multiple repos, Living Docs supports a shared Global LLM Wiki — a separate repo that holds institutional knowledge that applies across all projects. Each project's `agent.md` anchors to it via a relative path (`../<global-llm-wiki>/index.md`). Local docs always override global ones when there's a conflict. This gives you one place to store shared conventions without duplicating them into every repo.

---

## Why it works

Maintaining documentation fails because the bookkeeping cost compounds faster than the value — keeping things in sync, catching contradictions, updating cross-references. Humans abandon it. An agent doesn't get bored, doesn't forget to update a reference, and can touch five files in one pass. The human's job is to confirm correctness and ask the right questions. The agent handles the rest — when asked.

## What I'd call it now

I built this without a name for it. I thought I was just writing good documentation.

After reading Karpathy's LLM Wiki gist, I asked an AI to compare his system with my existing documentation workflow. The AI's conclusion: *same architecture, different domain.*

His pattern: raw sources stay immutable, an LLM-maintained wiki sits between you and the raw material, a schema doc tells the agent how to maintain it.  
My pattern: the codebase stays immutable, an LLM-maintained doc layer sits between the agent and the code, a governance doc tells the agent how to maintain it.

The architecture is the same. The problem being solved is different.

LLM Wiki is a knowledge distillation system — it takes raw sources and helps an LLM maintain a structured understanding of them. Living Docs is a coding agent governance system — it defines how an agent should write code, when to refactor, how to handle conflicts, and how to keep institutional knowledge alive across sessions. TDD rules, zero-loss refactor protocols, extraction thresholds, conflict resolution between code and docs — none of that exists in LLM Wiki. It was built for a different job.

The name I'd give it now: **Living Docs**. Not a skill file. Not a static prompt. A persistent, compounding artifact that gets more accurate with every task the agent completes — because the agent that changes the code is the same agent that keeps the record.

---

*This system grew out of the chaotic iteration of my main project. By having the AI itself audit the messy codebase, it successfully extracted the structural patterns into a simple web-development starter kit called [vt-template](https://github.com/Diew/vt-template). But the true breakthrough wasn't the folder structure — it was the domain-agnostic AI governance rules that the AI helped formalize along the way: Living Docs.*