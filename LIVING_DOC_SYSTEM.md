# Living Docs

> Read this file. Then ask exactly one question before doing anything else:
> **"Does this project have existing code?"**
> - Yes → follow Path B (Brownfield) in the Bootstrapping Guide.
> - No → follow Path A (Greenfield) in the Bootstrapping Guide.
> Do not assume. Do not proceed without an answer.

This document defines the **Living Docs** system — a self-maintaining documentation architecture for technical projects. Read this file first, then split the project into the correct docs and files.

---

## What this is

**Living Docs** is the markdown memory layer for a project.

- **Self-Maintaining**: same agent changes code and updates docs.
- **Persistent**: knowledge survives across sessions and models.
- **Structured**: one file owns one rule.

---

## Three Layers

### 1. Entry point — `agent.md`

One file at the project root. The agent reads this first in every session without being asked. It gives enough context to act without replacing the full docs.

**Mandatory Metadata Schema:**

```markdown
# [Project Name] — Agent Context (v[x.y.z]) — [YYYY-MM-DD]

## Project Setup
- **Project Name**: [Name]
- **Version**: [Semantic version, linked to bump script]
- **Status**: [Active / Maintenance / Legacy]
- **Tech Stack**: [Core languages, frameworks, runtimes]
- **Context Anchors**: [Links to Global LLM Wiki or cross-project references]

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

A single source of truth for doc management.

**The Registry Rule**: If a file is not in this registry, it does not exist for the agent. Every new doc must be registered before use.

#### Copy-Pasteable Registry Template

```markdown
# ARCH_documentation-governance — Registry & Loading Guide

---

## Registry & Loading Guide

> **If a file is not here, register it before using it.**

| File | Contains | Must NOT contain | Load when |
|------|----------|-----------------|-----------|
| `agent.md` | Session-critical rules, quick-reference, project metadata | Detailed implementation, full doc content | **Always** |
| `GUIDE_developer.md` | Implementation standards, naming rules, patterns, versioning, pre-commit | Feature-specific logic, data structures | Technical work |
| `ARCH_documentation-governance.md` | System's source of truth for doc management | Implementation content | Managing docs |
| `ARCH_technical-specs.md` | Core architecture, data models, state flows, system boundaries | Formatting rules, UI tokens | Structural work |
| `STANDARDS_interface.md` | Output formats, API specs, visual/CLI standards, IO rules | Business logic, algorithms | Interface/IO work |
| `REF_developer-reference.md` | Naming examples, lookup tables, layout tables, reference data | Implementation content | Checking conventions / lookup |
| `REFACTOR_TODO.md` | Current refactor work plan and targets | Canonical rules | Refactor planning |

### Task → Load mapping

| Task | Load |
|------|------|
| General development | `agent.md` |
| Feature / Logic changes | `agent.md` + `GUIDE_developer.md` + relevant `LOGIC_*.md` |
| Architecture / Data changes | + `ARCH_technical-specs.md` |
| Interface / Output changes | + `STANDARDS_interface.md` |
| Refactor planning | `REFACTOR_TODO.md` |
| Documentation management | `ARCH_documentation-governance.md` |
| Reference lookup | relevant `REF_*.md` |

---

## Canonical Ownership

| Area | Canonical file |
|------|----------------|
| Session-critical rules | `agent.md` |
| Implementation standards | `GUIDE_developer.md` |
| Docs registry / load mapping | `ARCH_documentation-governance.md` |
| Project-specific logic | `LOGIC_*.md` |

---

## Docs Naming Convention

| Prefix | Scope |
|--------|-------|
| `GUIDE_` | Implementation rules and standards |
| `ARCH_` | System architecture, data flow, structure |
| `LOGIC_` | Feature behavior, algorithms, business rules |
| `STANDARDS_` | Interface specs, output formats, visuals |
| `REF_` | Reference tables, constants, lookup data |
| `INCIDENT_` | Incident post-mortems, regression logs |

**Fixed names:** `agent.md`

---

*v[x.y.z] — [YYYY-MM-DD]*
```

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

Use these rules to keep docs aligned with code and to prevent duplicate or stale guidance.

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

Docs are updated **on explicit human request**, not automatically after every task. When asked, the agent must:

- **Sync**: Update docs that own the changed logic.
- **Register**: Add new docs to the governance registry.
- **Enforce**: Move rules to their correct owner if misplaced. Delete duplicates.

The agent must **never update docs silently** as a side effect of a code task unless explicitly told to. Unsolicited doc edits risk scope creep and unreviewed changes to the source of truth.

### 4. Adding a New Doc

1. Pick prefix from naming convention table.
2. Create the file using the Standard Document Template.
3. Register it in `ARCH_documentation-governance.md` before using it.

### 5. Moving Content Between Files

1. Copy to destination → verify complete → delete from source → update any cross-references.

### 6. Hyperlinking Standard

- Use relative Markdown links for all cross-references: `[Title](FILENAME.md#header)`.
- Never use absolute paths.
- Link to specific headers wherever possible to reduce search time.

---

## Architectural Defaults (Domain-Agnostic)

Use `GUIDE_` and `ARCH_` documents to enforce these patterns:

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

### Code Naming Conventions (Domain-Agnostic)

| Type | Rule | Do | Don't |
|------|------|----|-------|
| Functions | `camelCase`, verb + noun | `getItemById`, `fetchSchema` | `doStuff`, `thing` |
| Variables | intent first, avoid generic names | `itemList`, `configData` | `data`, `tmp` |
| Booleans | prefix `is` / `has` / `can` / `should` | `isValid`, `hasItems` | `flag`, `state` |
| Event handlers | prefix `handle` + target + event | `handleSubmitClick`, `handleFilterChange` | `onClick`, `clickHandler` |
| Async functions | action-oriented, name what is fetched or saved | `fetchItemList`, `saveUserSettings` | `getData`, `loadStuff` |
| Data objects | context + subject + type | `UserAuthInfo`, `systemStateMap` | `payload`, `thingObject` |
| Files (logic) | responsibility-first, use role suffix when useful | `storage-manager.ts`, `board-renderer.ts` | `utils.ts`, `misc.ts` |

> Naming rules live here as the bootstrap-friendly default. Put longer examples and edge cases in `REF_developer-reference.md` or a project-specific `REF_*.md`.

### Module Structure Rules

| Rule | Detail |
|------|--------|
| One responsibility per file | A file does one thing: builds one section, processes one data type, or holds one group of helpers |
| Orchestrators stay thin | Entry-point files contain only: data calls, cache logic, output rendering, trigger/event handling |
| Responsibility-first naming | Filenames must reflect their role: `*-controller.ts`, `*-renderer.ts`, `*-processor.ts` |
| Helpers are stateless | Helper functions must be pure — no side effects, no global reads |
| Shared helpers live in one place | If 2+ files need the same helper, extract to a shared `*-helpers` file — never duplicated |
| Import direction is one-way | Helpers never import from orchestrators. Processors never import from renderers |

### Goal-Driven Execution

Before any multi-step task, state a brief plan: `[Step] → verify: [check]`.

Transform vague tasks into verifiable goals. Never guess → build → fix → repeat.

### When to Stop and Ask

Stop before acting and ask **one** question when any of the following is true:

| Trigger | Reason |
|---------|--------|
| Task scope is wider than described — would touch files not mentioned | Risk of unintended side effects |
| Task contradicts a rule in `docs/` or a `STUBBORN_FACT` | Cannot resolve conflict without human intent |
| Task requires deleting or overwriting existing content | Irreversible — must confirm |
| Task is ambiguous enough that two valid interpretations produce different outcomes | Guessing here = regression |
| Code to be modified has no test coverage and task requires TDD | Cannot verify safety without tests first |

Never ask more than one question per stop. Pick the most blocking unknown.

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

**Pattern**: `[thing] [action] [reason]. [next step].`

### Rules for Terse Communication

1.  **Drop articles & fillers**: a, an, the, just, really, basically, actually, simply.
2.  **Drop pleasantries**: "Sure!", "I'd be happy to", "Certainly", "Of course", "Great question".
3.  **No hedging**: "I think", "It might be", "Perhaps". If unsure, ask one question.
4.  **Use fragments**: "Bug in auth. Fix:" instead of "I found a bug in the auth module and I am going to fix it."

### Example Comparison

| User Request | ❌ Bad Response | ✅ Good Response |
|--------------|----------------|------------------|
| Add a new user field | "Sure! I can definitely help with that. I will start by adding the 'age' field to the User model..." | "User model updated. Added 'age' (int). Migration generated. Ready to apply." |
| Why is this failing? | "It seems like the error is caused by a null pointer. You might want to check the initialization..." | "Null pointer in `AuthService.ts:42`. `currentUser` undefined. Fix: initialize in constructor." |
| Refactor this function | "I have carefully reviewed the code and I think we should extract this part to a helper. Does that sound okay?" | "Function exceeds 20 lines. Logic for 'tax_calc' extracted to helper. Tests pass. Ready to cut." |

**Auto-Clarity Override**: Revert to full sentences for security warnings, irreversible action confirmations, and complex multi-step sequences.

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

## Cross-Project Setup (Global LLM Wiki)

For projects connecting to a shared Global LLM Wiki across multiple repos:

- **Anchor path**: `../<global-llm-wiki>/index.md` (relative path from project root to the shared wiki repo).
- Local docs override global standards when there is a conflict.
- Document the anchor in `agent.md` under "Context Anchors".
- The Global LLM Wiki provides institutional memory; local docs provide project specifics.

---

## 4. Layer One: The Agent — `agent.md`

The entry point. The only file the agent is guaranteed to read at session start.

#### Copy-Pasteable `agent.md` Template

```markdown
# Agent — [Project Name]

> **Strict Rule**: Read this file at every session start.

## System Role
You are a Co-Developer. You maintain institutional memory via documentation. You do not hallucinate; if a rule is missing, you ask to document it.

## Communication Style
- Mode: **Terse** (as defined in `LIVING_DOC_SYSTEM.md`)
- Filler: Disabled
- Padding: Disabled

## Context Anchors
- **Primary Registry**: `docs/ARCH_documentation-governance.md`
- **Global Wiki**: `[relative/path/to/wiki]` (Optional)

## Project DNA
- **Framework**: [e.g., Python/FastAPI, Node/NextJS, Rust/CLI]
- **Primary Pattern**: [e.g., Modular Monolith, Hexagonal, Scripting]
- **Naming Rule**: [e.g., camelCase for UI, snake_case for Logic]

## Task → Load Mapping
> **Load sequence**: `agent.md` -> [Target Files]

| Task | Files to Load |
|------|---------------|
| Feature Work | `GUIDE_developer.md`, relevant `LOGIC_*.md` |
| UI / Styling | `STANDARDS_*.md` |
| Bug Fix | `INCIDENT_*.md` (if related), relevant Logic |
| Governance | `ARCH_documentation-governance.md` |

---

## Technical Standards
- **Version Control**: [e.g., Squash-and-Merge, Linear History]
- **Code Hygiene**: [e.g., max 50 lines per function, No God Modules]
- **TDD**: [e.g., Tests first for every bug fix]

---

*v[x.y.z] — [YYYY-MM-DD]*
```

---

## The Maintenance Loop (Doc Sweeps)

Documentation drift is the enemy. To prevent it, the agent performs a **Doc Sweep** only when instructed.

1. **Trigger**: User requests "Doc Sweep" or "Commit Prep".
2. **Action**: The agent compares the current implementation against the registered rules.
3. **Outcome**:
    - If code changed behavior: Update the corresponding `LOGIC_*.md`.
    - If a new pattern emerged: Update `GUIDE_developer.md`.
    - If a bug was complex: Create an `INCIDENT_*.md`.
4. **Final Step**: Increment the version number in the footer of edited docs.

---

## Bootstrapping Guide

### Path A: New Project (Greenfield)
1. **Initialize `agent.md`**: Use the template above. Define the project DNA.
2. **Setup Folder**: Create `docs/` and `docs/ARCH_documentation-governance.md`.
3. **Create Core Docs**: 
    - Create `docs/GUIDE_developer.md` (Standard practices).
    - Create `docs/REF_developer-reference.md` (Copy the reference-table pattern from the template in this file).
4. **Register Everything**: Map all files in `docs/ARCH_documentation-governance.md`.

### Path B: Existing Project (Brownfield)

> **Warning**: Do not scan the entire codebase in one pass. Large codebases will exceed context limits and produce inaccurate docs. Use the module-by-module protocol below.

This path is **doc-only**. No code is written, moved, or modified at any point. The goal is to document what already exists — nothing more.

This path mirrors the structure of the Zero-Loss Refactor Protocol, applied to documentation bootstrapping instead of code. Work one module at a time. Each iteration is independently valid and human-approved before proceeding.

**Iteration unit**: one module, one feature area, or one layer (e.g., auth, data models, routing).

Repeat the following cycle for each unit until the entire codebase is covered:

1. **Audit** — Read the target module fully. Understand actual behavior, data flow, and edge cases. Do not touch any files yet.
2. **Create draft doc** — Write a `LOGIC_*.md` or `ARCH_*.md` based on what the code *actually does*, not what it should do. Flag any non-obvious decisions as `STUBBORN_FACT`. Flag anything uncertain as `[UNVERIFIED — needs human confirmation]`.
3. **Bridge** — Ask the human to confirm the `[UNVERIFIED]` items. Intent cannot be inferred from code alone — this is a conversation, not a code operation. This step is the main guardrail against documenting wrong behavior as correct.
4. **Verify** — Human approves the doc. Resolve all `[UNVERIFIED]` flags before proceeding.
5. **Register** — Add the approved doc to `ARCH_documentation-governance.md`.
6. **Verify again** — Check that the new doc does not conflict with any already-registered doc. If conflict found, resolve before moving to the next module.

Once all modules are covered:

7. **Anchor File** — Create `agent.md` to capture project DNA using the approved docs as the source of truth.
8. **Flag Debt** — Create `REFACTOR_TODO.md` for patterns that violate established standards.
9. **Lock State** — System is "Live" when every module has an approved doc, all docs are registered, and `agent.md` task→load mapping is complete.

> **The system is live when the agent can start a fresh session, read only `agent.md`, and know exactly which files to load for any task — without asking.**

---

> **Final Goal**: A system where every developer (AI or human) shares the same perfect memory, enabling zero-loss handovers and instant project mobility.
