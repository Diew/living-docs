# Living Docs

> Read this file. Then build the system described in it.

This document defines the **Living Docs** system — a self-maintaining documentation architecture for codebases. An agent that reads this file should be able to set up the full system from scratch for any new project.

---

## What this is

**Living Docs** is a set of markdown files that the agent reads before acting and updates after completing work. The docs compound over time — every task makes them more accurate, not less.

It is not a static skill file. A skill file is a snapshot written by a human that drifts the moment the code changes. Living Docs is maintained by the same agent that changes the code. It stays true because the writer and the worker are the same.

The human designs the structure and makes judgment calls. The agent does the bookkeeping.

---

## Three layers

### 1. Entry point — `agent.md`

One file at the project root. The agent reads this at the start of every session without being asked.

Contains only:
- Project metadata (name, version, stack)
- Session-critical rules (things the agent must know before taking any action)
- Task → load mapping (which doc files to read for which task types)
- Links to `docs/` for everything else

Does not contain: implementation details, full doc content, anything that belongs in a specific doc file.

**Rule**: If a rule is in `agent.md` and also in a `docs/` file, the `docs/` file is the source of truth. `agent.md` holds a high-level echo at most — enough to trigger awareness, not enough to replace the full rule.

### 2. Doc layer — `docs/`

A folder of markdown files. Each file owns exactly one concern. The agent reads only the files relevant to the current task — not all of them.

**Naming convention:**

| Prefix | Scope | Load when |
|--------|-------|-----------|
| `GUIDE_` | Implementation rules, coding standards | Code / workflow tasks |
| `ARCH_` | System architecture, data flow, API schemas | Data / backend / structural tasks |
| `LOGIC_` | Business logic, algorithms, feature behavior | Feature-specific tasks |
| `STANDARDS_` | Visual / design specs, CSS tokens | Deep UI / animation tasks |
| `REF_` | Reference tables, lookup charts | Lookup tasks |
| `INCIDENT_` | Post-mortems, regression logs | Reviewing past failures |

**Fixed name**: `agent.md` — never rename, never prefix.

**Ownership rule**: One file owns each rule. No duplication across files. If the same rule exists in two places, one of them is wrong.

**Size rule**: When a file exceeds 300 lines or contains multiple unrelated content types, split it. Example: `LOGIC_archive.md` → `LOGIC_archive.md` + `LOGIC_archive-ui.md` + `LOGIC_archive-data.md`. The primary file remains the entry point and links to its sub-documents.

### 3. Registry — `ARCH_documentation-governance.md`

A single file that contains:
- A registry table of every doc file: what it contains, what it must not contain, when to load it
- The naming convention
- Maintenance rules (how to add, move, and edit docs)
- Canonical ownership (which file is the source of truth for each rule area)

**The registry rule**: If a file is not in the registry, it does not exist for the agent. Every new doc must be registered before use.

---

## Task → load mapping

The agent uses this table to decide which files to read. Put this table in `agent.md`, using the project's actual filenames.

Example pattern:

| Task | Load |
|------|------|
| General code | `agent.md` only |
| Web / UI features | relevant `GUIDE_*.md` |
| Deep UI (animation, visual effects) | + relevant `STANDARDS_*.md` |
| Data / backend / routing | + relevant `ARCH_*.md` |
| Feature-specific logic | + relevant `LOGIC_*.md` |
| Docs changes | `ARCH_documentation-governance.md` |

If the task is unclear, open `ARCH_documentation-governance.md` first. It contains the full mapping for the project.

---

## How the agent maintains the docs

After completing any task, the agent must:

1. **Update the relevant doc file** — if the task changed how something works, the doc that owns that rule must reflect the change.
2. **Register new files** — if a new doc was created, add it to the registry table in `ARCH_documentation-governance.md`.
3. **Enforce ownership** — if a rule was added to the wrong file, move it to the correct one and delete the duplicate.
4. **Never duplicate** — copying a rule to a second file is always wrong. Link instead.

The agent does not need to be asked to do this. It is part of completing the task.

---

## Protocols the agent must follow

### Pre-commit
1. Run full test suite. All tests must pass.
2. Bump version using the project's bump script — never manually.
3. Add changelog notes to the new version header only. Never modify old entries.
4. Run production build. No regressions.
5. Remove debug statements, address TODOs, delete scratch files.
6. No `push` or `commit` without explicit user approval per action.

### Refactoring
Follow **verify → trace → build → confirm**.
- Never refactor working code unless explicitly asked.
- Never consolidate files without explicit intent.
- Split functions at 2+ repetitions or 20+ lines.
- Split files at 200 lines (review) or 400 lines (mandatory).
- Zero-loss protocol: create new structure → bridge old to new → verify → cut old.

### Goal-driven execution
Before any multi-step task, state a brief plan: `[Step] → verify: [check]`. Transform vague tasks into verifiable goals before starting. Never guess → build → fix → repeat.

### TDD decision
- Use TDD for: logic, data processing, routing, rendering output, business rules.
- Skip TDD for: docs, copy, rename, formatting, cosmetic edits.

---

## Communication style

Answer only what is asked. No intro, recap, filler, or padding. Fragments are acceptable. Short synonyms preferred. Technical terms exact. Code blocks unchanged. Errors quoted exactly.

Pattern: `[thing] [action] [reason]. [next step].`

- Drop: articles (a/an/the), filler words (just/really/basically/actually), pleasantries (sure/certainly/happy to), hedging.
- Use full sentences for: security warnings, irreversible action confirmations, multi-step sequences where fragment order risks misread.
- Always use normal professional writing (not terse) for: code comments, commit messages, PR descriptions, documentation files.

---

## Cross-project setup (global wiki pattern)

If using this system across multiple projects, the recommended layout is:

```
/Workspace/
├── wiki-dev/       ← global knowledge base
└── your-project/  ← active project
```

In `agent.md` of each project, add:

```markdown
## Global Wiki
- Anchor: `../wiki-dev/wiki/index.md`
```

The agent reads the global wiki index first to find relevant standards, then applies them to the local project. Local project docs override global standards when there is a conflict.

---

## Setting up the system for a new project

When starting a new project using this system:

1. **Create `agent.md`** at the project root. Populate with: project metadata, session-critical rules, task → load mapping, link to `docs/`.
2. **Create `docs/`** folder.
3. **Create `docs/ARCH_documentation-governance.md`** with an empty registry table. Register `agent.md` first.
4. **Add doc files as needed.** Follow the naming convention. Register each one immediately.
5. **On every task completion**: update relevant docs, register new files, remove duplicates.

The system starts minimal and grows with the project. Do not create doc files speculatively — create them when the content exists.

---

## Why this works

The maintenance cost of documentation is not the writing. It is the bookkeeping — keeping rules in sync, updating cross-references, catching contradictions. Humans abandon docs because the cost compounds faster than the value. An agent does not get bored, does not forget to update a reference, and can touch multiple files in one pass.

The critical property: the agent that changes the code also updates the docs. There is no gap between when the code changes and when the docs reflect it. Drift is structurally prevented.

After enough sessions, the docs become institutional memory. Not the human's memory. Not the agent's memory. The project's memory — available to any agent that reads it.

---

## What the agent should produce after reading this

For a new project:
- `agent.md` at project root
- `docs/ARCH_documentation-governance.md` with registry
- Any initial doc files the project's existing code requires

For an existing project with undocumented logic:
- Read the codebase
- Identify the rule areas (architecture, logic, standards, etc.)
- Create the appropriate doc files following the naming convention
- Populate each file with the rules extracted from the code
- Register everything in `ARCH_documentation-governance.md`
- Add the task → load mapping to `agent.md`

The system is ready when: the agent can start a fresh session, read only `agent.md`, and know exactly which files to load for any given task.
