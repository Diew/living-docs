# Living Docs

> Read this file. Then build the system described in it.

This document defines the **Living Docs** system — a self-maintaining documentation architecture for any technical project. An agent that reads this file should be able to set up the full system from scratch for any new codebase, research corpus, or automation suite.

---

## What this is

**Living Docs** is a set of markdown files that the agent reads before acting and updates after completing work. It acts as the "institutional memory" of the project.

- **Self-Maintaining**: The agent that changes the code/data is the same agent that updates the docs. It stays true because the writer and the worker are the same.
- **Persistent**: Knowledge survives across sessions and different AI models.
- **Structured**: Rules are partitioned to prevent context bloat. One file owns each rule.

---

## Three Layers

### 1. Entry point — `agent.md`

One file at the project root. The agent reads this first in every session without being asked. It defines the **Awareness Layer** — enough context to act, not enough to replace the full docs.

**Mandatory Metadata Schema:**

```markdown
# [Project Name] — Agent Context (v[x.y.z]) — [YYYY-MM-DD]

## Project Setup
- **Project Name**: [Name]
- **Version**: [Semantic version, linked to bump script]
- **Status**: [Active / Maintenance / Legacy]
- **Tech Stack**: [Core languages, frameworks, runtimes]
- **Context Anchors**: [Links to global wiki or cross-project references]

## Documentation Priority
- `docs/` is the source of truth for behavior, architecture, and implementation rules.
- `agent.md` holds only: session-critical rules, quick-reference checklists, metadata.
- Do not duplicate detailed explanations here — link to `docs/` instead.
- **Scope unclear?** Open `docs/ARCH_documentation-governance.md` first.

## Commands
- `[dev command]`: local development.
- `[test command]`: run tests.
- `[build command]`: production build.
- `[bump command]`: sync version across all files.

## Milestones
- [x] v[x.y.z]: [Description of major milestone]
- [ ] v[x.y.z]: [Planned milestone]
```

**Rule**: If a rule exists in `agent.md` AND in a `docs/` file, the `docs/` file is the source of truth. `agent.md` holds a high-level echo only — enough to trigger awareness, not enough to replace the full rule.

---

### 2. Doc Layer — `docs/`

A folder of markdown files. Each file owns exactly one concern. The agent reads only the files relevant to the current task.

**Naming Convention:**

| Prefix | Scope | Content |
|--------|-------|---------|
| `GUIDE_` | Standards | How we write code, handle errors, or format data. |
| `ARCH_` | Structure | High-level system design, data flow, schemas. |
| `LOGIC_` | Behavior | Deep-dive into specific features, algorithms, business rules. |
| `STANDARDS_` | Visuals/IO | Interface specs, CSS, CLI outputs, API formats. |
| `REF_` | Facts | Lookup tables, constants, hardware/API endpoints. |
| `INCIDENT_` | History | Post-mortems and known regressions to prevent repeating. |

**Fixed names (no prefix, must not rename):** `agent.md`

**Fallback rule**: If a file is not in the registry → read prefix → load only if task matches → flag to user that registry needs updating.

**Size Rule (Expansion Protocol)**:
When a file exceeds **300 lines** or contains diverse content types (Logic/Visual/Data), follow this protocol:

1. **Global vs. Local Distinction**
   - **Global Rules**: Apply across the entire project (e.g., shared tokens, base interfaces). Must stay in global files (`STANDARDS_`, `ARCH_`, `REF_`).
   - **Local Rules**: Unique to the module. Must stay in the module's scope.
2. **Flat Sub-Document Naming**: Split into smaller flat files in `docs/` root (no nested folders):
   - `LOGIC_feature.md` — Core logic, algorithms, primary formulas.
   - `LOGIC_feature-ui.md` — Feature-specific visual standards and layout.
   - `LOGIC_feature-data.md` — Data structures, fields, API specs.
3. **Cross-Referencing**: The primary file remains the entry point and links to its sub-documents.

**Ownership Rule**: One file owns each rule. No duplication across files. Linking is preferred over copying.

---

### 3. Registry — `ARCH_documentation-governance.md`

A single file acting as the system's "Source of Truth" for doc management.

**Required sections:**

1. **Registry Table** — Every file mapped to: what it contains, what it must NOT contain, and when to load it.
2. **Task → Load Mapping** — Which files to load per task type.
3. **Canonical Ownership** — Which file is the single source of truth for each rule group.
4. **Naming Convention Table** — Prefix reference.
5. **Maintenance Rules** — How to add, move, or edit docs.

**The Registry Rule**: If a file is not in this registry, it does not exist for the agent. Every new doc must be registered before use.

---

## Task → Load Mapping

The agent uses this table to decide which files to read. A version of this table belongs in `agent.md`.

| Task | Load |
|------|------|
| General technical work | `agent.md` only |
| Implementation / Standards | + relevant `GUIDE_*.md` |
| Deep UI / Visuals / IO | + relevant `STANDARDS_*.md` |
| Data / Structure / Routing | + relevant `ARCH_*.md` |
| Feature-specific behavior | + relevant `LOGIC_*.md` |
| Reference lookup | + relevant `REF_*.md` |
| Documentation management | `ARCH_documentation-governance.md` |
| Reviewing past failures | relevant `INCIDENT_*.md` |

If the task is unclear, open `ARCH_documentation-governance.md` first.

---

## Canonical Ownership

Use one file as the source of truth for each rule group. No rule should appear in full in more than one file.

| Area | Canonical file | Notes |
|------|----------------|-------|
| Session-critical rules | `agent.md` | Fast startup rules only |
| Implementation standards | `GUIDE_developer.md` | TDD, refactor, naming, module structure |
| Docs registry / load mapping | `ARCH_documentation-governance.md` | File ownership and doc routing |
| Data / API / routing | `ARCH_technical-specs.md` | SSOT and data flow rules |
| Domain logic | `LOGIC_*.md` | Feature-specific behavior |
| Visual / IO standards | `STANDARDS_*.md` | Design tokens, output formats |
| Work plans | Separate `REFACTOR_TODO.md` | Task list only, not a canonical rule source |

---

## Core Maintenance Protocols

### 1. Intent Logging ("The Why")

Code/data tells you *how* it works. Docs must tell you *why* it was built that way.

- Record **trade-offs** and **rejected alternatives**.
- Flag intentional quirks as `STUBBORN_FACT` to prevent "fixes" that break things.
- Log errors that were kept intentionally (e.g., a "wrong" value that is correct for business reasons).

Format:
```
> STUBBORN_FACT: [what it is] — [why it must stay this way]
```

### 2. Conflict Resolution (Code vs. Docs)

If the codebase and the documentation disagree:

1. **Code is primary truth** for behavior (what the system actually does).
2. **Docs are primary truth** for intent (what the system is supposed to do).
3. If they drift: determine if the code is buggy (fix code) or the doc is stale (fix doc). Never leave them unresolved.

### 3. Maintenance Cycle (Doc Sweep)

After completing any task, the agent must:

- **Sync**: Update docs that own the changed logic.
- **Register**: Add new docs to the governance registry.
- **Enforce**: Move rules to their correct owner if misplaced. Delete duplicates.

### 4. Adding a New Doc

1. Pick prefix from naming convention table.
2. Create the file using the Standard Document Template.
3. Register it in `ARCH_documentation-governance.md` before using it.

### 5. Moving Content Between Files

1. Copy to destination → verify complete → delete from source → update any cross-references.

---

## Architectural Defaults (Domain-Agnostic)

Regardless of the tech stack (Web, Backend, Data Science, Scripts), the following architectural patterns must be enforced via `GUIDE_` and `ARCH_` documents:

### 1. The 3-Layer Separation
Strictly separate responsibilities into three isolated layers:
- **Data/State Layer**: Manages raw data, database queries, API payloads, or file I/O.
- **Processor/Helper Layer**: Pure, stateless functions that transform data.
- **Orchestrator/Entry Layer**: The thin glue that connects data to processors and outputs the result. Orchestrators must not contain complex business logic.

### 2. State & Storage Registry
If the system maintains state (e.g., Cache, Environment Variables, Database Tables, LocalStorage, Memory pools), it must be explicitly registered in an `ARCH_` document.
- **Rule**: No ad-hoc or undocumented keys/variables. All global state identifiers must be centralized to prevent collision and ensure predictable cache invalidation.

### 3. Namespace & Collision Governance
Define strict naming boundaries for shared resources to prevent collision.
- **Resource Namespaces**: Use prefixes for global resources (e.g., API routes `/api/v1/`, Env vars `APP_DB_*`, UI tokens `.ui-`).
- **Priority Layering**: If resources stack or conflict (e.g., execution order, system ports, visual Z-indexes), define an absolute hierarchy in the documentation.

---

## Execution Protocols

### Pre-Commit Checklist

1. **Test**: Run full test suite. All tests must pass.
2. **Bump**: Use the project's bump script — never manually edit version numbers.
3. **Docs**: Add changelog notes to the new version header only. Never insert into old entries.
4. **Build**: Confirm production build passes.
5. **Clean**: Remove debug statements, fix TODOs, delete scratch files.
6. **Git**: No `push` or `commit` without explicit user approval per action.

### Refactoring (Zero-Loss Protocol)

Use for any file split, large refactor, or move:

1. **Audit** — Read existing code fully before touching it.
2. **Create targets** — New structure in place before removing old code.
3. **Bridge** — Re-exports or adapters connect old to new.
4. **Verify** — Typecheck + tests pass.
5. **Cut** — Remove old code only after behavior is confirmed stable.
6. **Verify again** — Run full suite after cleanup.

Do not skip the bridge phase. It is the main guardrail against broken imports.

### When to Extract a Function

| Trigger | Action |
|---------|--------|
| Logic appears **2+ times** | Extract immediately — no exceptions |
| Function body exceeds **20 lines** | Extract inner logic into named helpers |
| Expression requires a comment to understand | Extract into a named function |
| Template/string contains repeated structure | Extract into a builder function |

### When to Split a File

| Trigger | Action |
|---------|--------|
| File exceeds **200 lines** | Review — split if multiple responsibilities |
| File exceeds **400 lines** | Split mandatory — one responsibility per file |
| File contains 2+ unrelated concept groups | Split regardless of line count |
| A function is reused across 2+ files | Move to a shared helper file |

### Module Structure Rules

| Rule | Detail |
|------|--------|
| One responsibility per file | A file does one thing: builds one section, processes one data type, or holds one group of helpers |
| Orchestrators stay thin | Entry-point files contain only: data calls, cache logic, output rendering, trigger/event handling |
| Helpers are stateless | Helper functions must be pure — no side effects, no global reads |
| Shared helpers live in one place | If 2+ files need the same helper, extract to a shared `*-helpers` file — never duplicated |
| Import direction is one-way | Helpers never import from orchestrators. Processors never import from renderers |

### Goal-Driven Execution

Before any multi-step task, state a brief plan: `[Step] → verify: [check]`.

Transform vague tasks into verifiable goals. Never guess → build → fix → repeat.

### TDD Decision Rule

- **Use TDD for**: logic, data processing, routing, rendering output, business rules.
- **Skip TDD for**: docs, copy, rename, formatting, cosmetic edits.

---

## Common Mistakes (Anti-Patterns)

These apply to all projects using this system:

| Mistake | Why it fails | Prevention |
|---------|-------------|------------|
| Refactoring working code without being asked | Breaks stable behavior, wastes context | Only refactor on explicit request |
| Adding work to an existing changelog entry | Corrupts version history | New task = new version header always |
| Duplicating a rule across multiple files | Creates conflicts when one is updated | One file owns each rule — link, don't copy |
| Loading files not needed for the current task | Context bloat, slower reasoning | Follow the Task → Load Mapping table |
| Manual version bumps | Causes version drift across files | Always use the bump script |
| Assuming without confirming | Destroys trust, causes regressions | If unsure, ask one question |
| Changing unrelated files in the same edit | Scope creep, harder to review | One edit = one concern |
| Skipping the registry when adding a doc | Doc becomes invisible to AI | Register before use, always |

---

## Communication Style

Answer only what is asked. No intro, recap, filler, or padding. Fragments acceptable. Short synonyms preferred. Technical terms exact. Code blocks unchanged. Errors quoted exactly.

Pattern: `[thing] [action] [reason]. [next step].`

- Drop: articles (a/an/the), filler words (just/really/basically/actually), pleasantries.
- Use full sentences for: security warnings, irreversible actions, multi-step sequences.
- Always use professional writing for: code comments, commit messages, documentation files.

**Good:**
- `Bug in auth middleware. Token expiry uses < not <=. Fix:`
- `Function extracted. Tests pass. Ready to cut old code.`

**Bad:**
- `Sure! I'd be happy to help you with that! The issue you're experiencing is likely...`

---

## Standard Document Templates

### General Doc Template (`REF_template.md` pattern)

Every document in `docs/` should follow this structure:

```markdown
# [PREFIX_filename] — [One-line purpose]

> **Compact, not incomplete.** Remove sections with no content. Never remove rules, edge cases, or reference rows to save space.

---

## Rules
> Hard constraints. AI must follow these unconditionally.

| Rule | Detail |
|------|--------|
| [rule] | [what it means in practice] |

---

## Reference
> Lookup tables. No prose.

| Item | Value | Notes |
|------|-------|-------|

---

## Edge Cases
> Only document cases that are non-obvious or have caused bugs.

- **[case]**: [what to do]

---

*v[x.y.z] — [YYYY-MM-DD]*
```

### Incident Report Template (`INCIDENT_` pattern)

Use for any failure, regression, or non-obvious recovery:

```markdown
# Incident Report: [Short title]

**Date:** [YYYY-MM-DD]
**Status:** [Resolved / Ongoing]
**Scope:** [Affected systems or features]

## 1. What Happened?
[Brief description of the failure and its visible impact.]

## 2. Root Causes

### A. [Root Cause Name]
[Technical explanation with code snippets if relevant.]
**Impact:** [What broke and why.]

## 3. Lessons Learned & Prevention
1. [Actionable rule to prevent recurrence.]

## 4. Recovery Actions
- [Exact commands or steps taken to restore stability.]
```

### Refactor Work Plan Template (`REFACTOR_TODO.md` pattern)

```markdown
# Refactor TODO — [Project Name]

> Working note for current refactoring tasks. Canonical rules: `GUIDE_developer.md`.

---

## ✅ Phase [N]: [Phase Title] (Done)
- [x] [Task description] (Done)

---

## 🚀 Phase [N+1]: [Phase Title] (In Progress)

Objective: [One sentence goal.]

- [ ] **1. [Major task]**
  - [ ] [Sub-task]

---

## 🧊 On Hold / Backlog
- [ ] [Deferred task with reason]

---

## Safety & Verification
1. Zero-Loss Refactor Protocol: No behavioral changes allowed.
2. Test before and after every major edit.
3. Build must pass before commit.
```

---

## Cross-Project Setup (Global Wiki)

For projects using a shared knowledge base across multiple repos:

- **Anchor path**: `../<global-wiki>/index.md` (relative path from project root to the shared wiki repo).
- Local docs override global standards when there is a conflict.
- Document the anchor in `agent.md` under "Context Anchors".
- The global wiki provides institutional memory; local docs provide project specifics.

---

## Bootstrapping a New Project

Run this sequence exactly when setting up Living Docs for a new project:

1. **Create `agent.md`** at project root. Populate with: metadata schema, documentation priority rules, command reference, task→load mapping, milestones.
2. **Create `docs/`** folder at project root.
3. **Create `docs/ARCH_documentation-governance.md`** with: registry table, task→load mapping, canonical ownership table, naming convention, maintenance rules.
4. **Create `docs/GUIDE_developer.md`** with: implementation rules, refactoring standards, module structure, anti-patterns.
5. **Create initial domain docs** extracted from existing logic or spec. Follow naming conventions. One concern per file.
6. **Register everything** in `ARCH_documentation-governance.md` before use.
7. **Verify**: The system is ready when the agent knows exactly which files to load for any task type.

> The system is alive when every rule has exactly one owner, every file is registered, and the agent can start a new session and know the full project state by reading `agent.md` alone.
