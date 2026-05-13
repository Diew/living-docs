# Living Docs

> Before doing anything else, ask the user exactly one question:
> **"Do you want to implement this system, or just explore it?"**
> - Implement → ask "Does this project have existing code?" then follow Path A or B
> - Explore → read and answer questions only, do not bootstrap anything

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

Contains: file ownership table, Task→Load mapping, Canonical Ownership table, Docs Naming Convention.

> Full template → Copy-Pasteable Registry Template below.

---

## Task → Load Mapping

The agent loads only the files relevant to the current task — not the entire `docs/` folder. The mapping is simple: each task type has a fixed set of files to load. If the task is unclear, open `ARCH_documentation-governance.md` first.

The authoritative task→load table lives in the **Copy-Pasteable Registry Template** below. That is the version the agent copies into `ARCH_documentation-governance.md` during bootstrapping.

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

Three domain-agnostic patterns enforced across all projects via `GUIDE_` and `ARCH_` documents:

- **3-Layer Separation**: Data/State → Processor/Helper → Orchestrator/Entry. Orchestrators must not contain business logic.
- **State & Storage Registry**: All global state identifiers (cache keys, env vars, DB tables) must be registered in an `ARCH_` document. No ad-hoc keys.
- **Namespace & Collision Governance**: Strict prefixes for shared resources. Priority hierarchy documented when resources conflict.

> Full rules and examples → Copy-Pasteable `GUIDE_developer.md` Template below.

---

## Execution Protocols

Protocols split across two files based on load frequency:

- **`agent.md`** (session-critical, loaded every session): Pre-Commit Checklist, TDD Decision Rule, Goal-Driven Execution, When to Stop and Ask, Common Mistakes, Communication Style.
- **`docs/GUIDE_developer.md`** (loaded on-demand): Zero-Loss Refactor Protocol, When to Extract a Function, When to Split a File, Module Structure Rules, Naming Conventions.

> Full rules → Copy-Pasteable `agent.md` Template and `GUIDE_developer.md` Template below.

### Key behaviors (summary)

**Goal-Driven Execution**: Before any multi-step task, state a brief plan: `[Step] → verify: [check]`. Never guess → build → fix → repeat.

**When to Stop and Ask**: Stop and ask **one** question when task scope is wider than described, contradicts a rule or `STUBBORN_FACT`, requires deleting content, is ambiguous between two valid interpretations, or lacks test coverage and requires TDD. Never ask more than one question per stop.

**TDD Decision Rule**: Use TDD for logic, data processing, routing, rendering, business rules. Skip for docs, copy, rename, formatting, cosmetic edits.

---

## Common Mistakes (Anti-Patterns)

8 session-critical guardrails copied verbatim into `agent.md` during bootstrapping: no unsolicited refactor, no adding to old changelog entries, no rule duplication, no loading unneeded files, no manual version bumps, no assuming without confirming, no unrelated edits in one pass, no skipping the registry.

> Full table → Copy-Pasteable `agent.md` Template below.

---

## Communication Style

Answer only what is asked. No intro, recap, filler, or padding. Pattern: `[thing] [action] [reason]. [next step].` Drop articles, pleasantries, and hedging. Use fragments. Auto-Clarity Override: revert to full sentences for security warnings and irreversible actions.

> Full rules and examples → Copy-Pasteable `agent.md` Template below.

---

## Standard Document Templates

### Copy-Pasteable Registry Template

> **Strict Rule**: Copy this entire template verbatim — every table, section, and row.

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
| `REF_template.md` | Blank template for creating new doc files | Rules, implementation content | Creating a new doc file |
| `REFACTOR_TODO.md` | Current refactor work plan and targets | Canonical rules | Refactor planning |

### Task → Load mapping

| Task | Load |
|------|------|
| General development | `agent.md` |
| Feature / Logic changes | `agent.md` + `GUIDE_developer.md` + relevant `LOGIC_*.md` |
| Implementation / Standards | + relevant `GUIDE_*.md` |
| Architecture / Data changes | + relevant `ARCH_*.md` |
| Interface / Output changes | + relevant `STANDARDS_*.md` |
| Reference lookup | + relevant `REF_*.md` |
| Refactor planning | `REFACTOR_TODO.md` |
| Documentation management | `ARCH_documentation-governance.md` |
| Reviewing past failures | + relevant `INCIDENT_*.md` |

---

## Canonical Ownership

Use one file as the source of truth for each rule group.

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

**Fixed names (no prefix, must not rename):** `agent.md`

**Fallback rule:** If a file is not in the registry → read prefix → load only if task matches → flag to user that registry needs updating.

---

## Maintenance Rules

**One rule above all: content exists in ONE file only. No duplication.**
**Compact, not incomplete: remove empty sections, never remove rules, edge cases, or reference rows.**

### Adding a new doc
1. Pick prefix from naming convention table.
2. Register in the Registry table above (mandatory).

### Moving content
1. Copy to destination → verify complete → delete from source → update any references.

### Editing existing files
- `agent.md`: session-critical rules and metadata only.
- `GUIDE_*`: coding standards + basic UI. No deep visual rules.
- `STANDARDS_*`: visual/interaction only. No coding standards.
- Registry table: update immediately when any file is added, renamed, or removed.

---

*v[x.y.z] — [YYYY-MM-DD]*
```

---

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

### Copy-Pasteable `GUIDE_developer.md` Template

> **Strict Rule**: Copy this entire template verbatim — every subsection, table, and rule. Do not summarize or omit any rows. Add project-specific content in the marked placeholders.

```markdown
# GUIDE_developer — Implementation rules for [Project Name]

---

## Rules

> Hard constraints. AI must follow unconditionally. Add project-specific rules here.

| Rule | Detail |
|------|--------|
| No unsolicited refactor | Never refactor working code unless explicitly asked |
| Git operations | No push or commit without explicit user approval |
| TDD | Write or update tests before implementation for anything affecting data, routing, rendering, or business logic |
| [project rule] | [what it means in practice] |

---

## Refactoring Standards

| Rule | Detail |
|------|--------|
| No unsolicited refactor | Never refactor working code unless explicitly asked — even if it violates standards above |

### When to Extract a Function

| Trigger | Action |
|---------|--------|
| Logic or template appears **2+ times** | Extract immediately — no exceptions |
| Function body exceeds **20 lines** | Extract inner logic into named helpers |
| Inline expression requires a comment to understand | Extract into a named function instead |
| Template string contains repeated HTML structure | Extract into a builder function |


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
| Orchestrators stay thin | Entry-point files contain only: data calls, cache logic, output rendering, event binding |
| Helpers are stateless | Helper functions must be pure — no side effects, no global reads |
| Shared helpers live in one place | If 2+ files need the same helper, extract to a shared `*-helpers` file — never duplicated |
| Import direction is one-way | Helpers never import from orchestrators. Processors never import from renderers |

### Zero-Loss Refactor Protocol

Use for any file split, large refactor, or move:

1. **Audit** first
2. **Create targets** before removing old code
3. **Bridge** with re-exports or adapters
4. **Verify** with typecheck and tests
5. **Cut** only after behavior is stable
6. **Verify again** after cleanup

Do not skip the bridge phase for large moves. It is the main guardrail against broken imports and partial refactors.

Other docs should name this protocol, not duplicate its full steps.

### Anti-Patterns (Banned)

| Pattern | Why it fails | Fix |
|---------|-------------|-----|
| Logic duplicated across files | Conflicts when one is updated | Extract to shared helper |
| One file doing data + rendering + events | Violates SRP, hard to cache | Split into processor / renderer / controller |
| Importing a full module for one constant | Unnecessary coupling | Move constant to shared constants file |
| [project-specific pattern] | [why] | [fix] |

---

## Architectural Defaults

### State & Storage Registry
If the system maintains state (e.g., Cache, Environment Variables, Database Tables, LocalStorage, Memory pools), it must be explicitly registered in an `ARCH_` document.
- **Rule**: No ad-hoc or undocumented keys/variables. All global state identifiers must be centralized to prevent collision and ensure predictable cache invalidation.

### Namespace & Collision Governance
Define strict naming boundaries for shared resources to prevent collision.
- **Resource Namespaces**: Use prefixes for global resources (e.g., API routes `/api/v1/`, Env vars `APP_DB_*`, UI tokens `.ui-`).
- **Priority Layering**: If resources stack or conflict (e.g., execution order, system ports, visual Z-indexes), define an absolute hierarchy in the documentation.

---

## Reference

Reference tables: [REF_developer-reference.md](./REF_developer-reference.md).

---

## Edge Cases

- **[case]**: [what to do]

---

*v[x.y.z] — [YYYY-MM-DD]*
```

---

### Copy-Pasteable `REF_developer-reference.md` Template

> **Strict Rule**: Copy this entire template verbatim — every table and row. Add project-specific tables below the Naming Conventions section.

```markdown
# REF_developer-reference — Developer reference tables

> Reference only. Rules live in `GUIDE_developer.md`.

---

## Reference

### Naming Conventions

| Type | Rule | Do | Don't |
|------|------|----|-------|
| Functions | `camelCase`, verb + noun | `getItemById`, `fetchSchema` | `doStuff`, `thing` |
| Variables | intent first, avoid generic names | `itemList`, `configData` | `data`, `tmp` |
| Booleans | prefix `is` / `has` / `can` / `should` | `isValid`, `hasItems` | `flag`, `state` |
| Event handlers | prefix `handle` + target + event | `handleSubmitClick`, `handleFilterChange` | `onClick`, `clickHandler` |
| Async functions | action-oriented, name what is fetched or saved | `fetchItemList`, `saveUserSettings` | `getData`, `loadStuff` |
| Data objects | context + subject + type | `UserAuthInfo`, `systemStateMap` | `payload`, `thingObject` |
| Files (logic) | responsibility-first, use role suffix when useful | `storage-manager.ts`, `board-renderer.ts` | `utils.ts`, `misc.ts` |

### Commands

| Command | Purpose |
|---------|---------|
| `[dev command]` | Local development |
| `[test command]` | Run tests |
| `[build command]` | Production build |
| `[bump command]` | Sync version across all files |

### Version

| Item | Detail |
|------|--------|
| Source of truth | `[e.g., package.json]` |
| Bump command | `[bump command]` |
| Files auto-updated | `[list of files]` |

### [Project-Specific Reference Tables]
> Add lookup tables, constants, endpoints, or other reference data here.

| Item | Value | Notes |
|------|-------|-------|
| [item] | [value] | [notes] |

---

*v[x.y.z] — [YYYY-MM-DD]*
```

---

### Copy-Pasteable `STANDARDS_*.md` Template

> Use for visual, interface, and IO specifications. One file per domain (e.g., `STANDARDS_ui-visual.md`, `STANDARDS_api.md`). Add project-specific rules and reference tables — do not leave placeholders empty; populate from the actual codebase.

```markdown
# STANDARDS_[domain] — [Visual / Interface / IO] specifications for [Project Name]

---

## Rules

> Hard constraints for this domain. AI must follow unconditionally.

| Rule | Detail |
|------|--------|
| No magic numbers | All values must come from tokens or shared constants |
| [domain rule] | [what it means in practice] |

---

## [Domain] Reference

### Color System / Design Tokens

| Token | Value | Usage |
|-------|-------|-------|
| `[token]` | `[value]` | [where it's used] |

### Component Standards

| Component | Class / Pattern | Purpose |
|-----------|----------------|---------|
| [component] | [class] | [purpose] |

### Layout

| Item | Value |
|------|-------|
| [layout item] | [value] |

---

## Edge Cases

> Only document cases that are non-obvious or have caused regressions.

- **[case]**: [what to do]

---

*v[x.y.z] — [YYYY-MM-DD]*
```

---

## Cross-Project Setup (Global LLM Wiki)

For projects connecting to a shared Global LLM Wiki across multiple repos:

- **Anchor path**: `../<global-llm-wiki>/index.md` (relative path from project root to the shared wiki repo).
- Local docs override global standards when there is a conflict.
- Document the anchor in `agent.md` under "Context Anchors".
- The Global LLM Wiki provides institutional memory; local docs provide project specifics.

---

## Copy-Pasteable `agent.md` Template

The entry point. The only file the agent is guaranteed to read at session start.

> **Strict Rule**: Copy this entire template verbatim — every section, table, and rule. Fill in project-specific values but do not omit any section.

```markdown
# Agent — [Project Name] (v[x.y.z]) — [YYYY-MM-DD]

> **Strict Rule**: Read this file at every session start.

## Project Setup
- **Project Name**: [Name]
- **Version**: [x.y.z] — use bump script only, never manually edit
- **Status**: [Active / Maintenance / Legacy]
- **Tech Stack**: [Core languages, frameworks, runtimes]
- **Context Anchors**: [Links to Global LLM Wiki or cross-project references]

## Documentation Priority
- `docs/` is the source of truth for behavior, architecture, and implementation rules.
- `agent.md` holds only: session-critical rules, quick-reference checklists, metadata.
- Do not duplicate detailed explanations here — link to `docs/` instead.
- **Scope unclear?** Open `docs/ARCH_documentation-governance.md` first — task→load mapping is there.

## Documentation Governance
- **Implementation Standards**: `docs/GUIDE_developer.md` — how we write code, refactor, and test.
- **Architecture & Data**: `docs/ARCH_technical-specs.md` — data models, routing, system boundaries.
- **Visual & IO Standards**: `docs/STANDARDS_*.md` — design tokens, output formats, interface specs.
- **Rule**: Never consolidate these files without explicit intent. Keep concerns isolated to prevent accidental regressions.

## AI Technical Governance (CRITICAL)

## TDD Decision Rule
- **Use TDD** for: logic, data processing, routing, rendering output, business rules.
- **Skip TDD** for: docs, copy, rename, formatting, cosmetic edits.

## Goal-Driven Execution
Verify → trace → build → confirm. Never guess → build → fix → repeat.
Before multi-step tasks, state a brief plan: `[Step] → verify: [check]`.

## Response Style (CRITICAL)
- Answer only what's asked. No intro, recap, outro, filler, or padding.
- Markdown only when it helps (tables, code blocks).
- Unsure → ask **one** question. No assumptions.

### Writing Rules
Drop: articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course/happy to), hedging. Fragments OK. Short synonyms (big not extensive, fix not "implement a solution for"). Technical terms exact. Code blocks unchanged. Errors quoted exact.

Pattern: `[thing] [action] [reason]. [next step].`

- ❌ "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..."
- ✅ "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:"
- "Why React component re-render?" → "New object ref each render. Inline object prop = new ref = re-render. Wrap in `useMemo`."
- "Explain database connection pooling." → "Pool reuse open DB connections. No new connection per request. Skip handshake overhead."

### Auto-Clarity
Revert to full sentences for: security warnings, irreversible action confirmations, multi-step sequences where fragment order risks misread. Resume terse after.

### Boundaries
Code blocks, commit messages, PR descriptions: write normal always.

## Common Mistakes
| Mistake | Prevention |
|---------|------------|
| Refactoring working code without being asked | Only refactor on explicit request |
| Adding work to an existing changelog entry | New task = new version header always |
| Duplicating a rule across multiple files | One file owns each rule — link, don't copy |
| Manual version bumps | Always use the bump script |
| Assuming without confirming | If unsure, ask one question |
| Changing unrelated files in the same edit | One edit = one concern |
| Skipping the registry when adding a doc | Register before use, always |

## Pre-Commit Protocol
1. **Test**: Run full test suite. All tests must pass.
2. **Bump**: Use bump script — never manually edit version numbers.
3. **Docs**: Add notes to new version header only. Never insert into old entries.
4. **Build**: Confirm production build passes.
5. **Clean**: Remove debug statements, fix TODOs, delete scratch files.
6. **Git**: No `push` or `commit` without explicit user approval per action.

## Milestones
- [x] v[x.y.z]: [Description of major milestone]
- [ ] v[x.y.z]: [Planned milestone]

## Commands
- `[dev command]`: local development.
- `[test command]`: run tests.
- `[build command]`: production build.
- `[bump command]`: sync version across all files. Never manually edit version numbers.

## Project Notes
> Add project-specific state, quick-reference data, or active constraints here (e.g., board status, feature flags, intentional quirks).

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

> **Strict Rule for Bootstrapping**: When executing a bootstrapping sequence, you **MUST NOT** summarize, omit, or paraphrase content from the master template. Every section, table, and rule specified in the bootstrap steps must be copied **verbatim** to ensure no institutional knowledge is lost.


### Path A: New Project (Greenfield)
1. **Initialize `agent.md`**: Create this file at the project root using the **Copy-Pasteable `agent.md` Template** from this document. The template includes Communication Style, Common Mistakes, TDD rules, Pre-Commit Protocol, and Goal-Driven Execution — fill in project-specific values but do not omit any section.
2. **Setup Folder**: Create `docs/` directory.
3. **Create Registry**: Create `docs/ARCH_documentation-governance.md` using the **Copy-Pasteable Registry Template** from this document. You **MUST** copy the entire template verbatim — every table, section, and row.
4. **Create Core Docs**: 
    - Create `docs/GUIDE_developer.md` using the **Copy-Pasteable `GUIDE_developer.md` Template** from this document. You **MUST** also extract and copy the entire `## Architectural Defaults (Domain-Agnostic)` section into it — including every subsection, table, and rule. Do not summarize or omit any rows.
    - Create `docs/REF_developer-reference.md` using the **Copy-Pasteable `REF_developer-reference.md` Template** from this document — including every row of the Naming Conventions table.
    - Create `docs/REF_template.md` using the **General Doc Template** from this document. This file is the reference template for all future doc creation — copy it verbatim.
    - Create `docs/STANDARDS_*.md` using the **Copy-Pasteable `STANDARDS_*.md` Template** for each visual/IO domain in the project.
5. **Register Everything**: Map all files in `docs/ARCH_documentation-governance.md`.

### Path B: Existing Project (Brownfield)

> **Warning**: Do not scan the entire codebase in one pass. Large codebases will exceed context limits and produce inaccurate docs. Use the module-by-module protocol below.

This path is **doc-only**. No code is written, moved, or modified at any point. The goal is to document what already exists — nothing more.

#### Step 0: Check for existing docs

Before anything else, check what already exists:

- **No `agent.md` or `docs/` found** → proceed to Step 1.
- **`agent.md` or `docs/` found (partial or full)** → audit existing files first. Read every existing doc, note what each contains, flag conflicts or gaps. Do not overwrite anything yet. Use findings to inform Steps 1–2 below — merge with templates, do not replace.

#### Step 1: Copy core templates (do this before auditing any code)

While context is fresh, copy all core templates verbatim from this document:

- Create `docs/GUIDE_developer.md` using the **Copy-Pasteable `GUIDE_developer.md` Template**. You **MUST** copy every section, table, and rule verbatim. Do not summarize or omit any rows.
- Create `docs/REF_developer-reference.md` using the **Copy-Pasteable `REF_developer-reference.md` Template** — including every row of the Naming Conventions table.
- Create `docs/REF_template.md` using the **General Doc Template** from this document. This file is the reference template for all future doc creation — copy it verbatim.
- Create `docs/STANDARDS_*.md` using the **Copy-Pasteable `STANDARDS_*.md` Template** for each visual/IO domain in the project.
- Create `docs/ARCH_documentation-governance.md` using the **Copy-Pasteable Registry Template**. Copy the entire template verbatim — every table, section, and row.
- Create `agent.md` using the **Copy-Pasteable `agent.md` Template**. Do not omit any section — fill in project-specific values in Steps 3–6 below.
- Create `docs/REFACTOR_TODO.md` using the **Refactor Work Plan Template** from this document. Leave all sections empty — this file is required even if no debt is found yet.

> If existing docs were found in Step 0: merge their content into the templates now. Existing rules take precedence over placeholder text. Do not discard institutional knowledge.

#### Step 2: Audit the codebase (module by module)

**Iteration unit**: one module, one feature area, or one layer (e.g., auth, data models, routing).

Repeat the following cycle for each unit until the entire codebase is covered:

1. **Audit** — Read the target module fully. Understand actual behavior, data flow, and edge cases. Do not touch any files yet.
2. **Create draft doc** — Write a `LOGIC_*.md` or `ARCH_*.md` based on what the code *actually does*, not what it should do. Flag any non-obvious decisions as `STUBBORN_FACT`. Flag anything uncertain as `[UNVERIFIED — needs human confirmation]`.
3. **Bridge** — Ask the human to confirm the `[UNVERIFIED]` items. Intent cannot be inferred from code alone — this is a conversation, not a code operation. This step is the main guardrail against documenting wrong behavior as correct.
4. **Verify** — Human approves the doc. Resolve all `[UNVERIFIED]` flags before proceeding.
5. **Register** — Add the approved doc to `ARCH_documentation-governance.md`.
6. **Verify again** — Check that the new doc does not conflict with any already-registered doc. If conflict found, resolve before moving to the next module.

#### Step 3: Fill in project-specific values

Once all modules are covered, fill in placeholders across all files created in Step 1:

- `agent.md` — Project name, version, tech stack, commands, milestones. Task→load mapping must reflect all registered docs.
- `docs/ARCH_documentation-governance.md` — Register all `LOGIC_*.md` and `ARCH_*.md` created in Step 2.
- `docs/STANDARDS_*.md` — Replace placeholder tokens with actual values from the codebase.

#### Step 4: Final checks

- **Flag Debt**: Fill in `REFACTOR_TODO.md` (created in Step 1) with any patterns that violate established standards found during the audit.
- **Lock State**: System is "Live" when all docs are registered, and `agent.md` task→load mapping is complete.

**Completion checklist:**
- [ ] `agent.md` exists and has no unfilled placeholders
- [ ] `docs/ARCH_documentation-governance.md` lists every doc in `docs/`
- [ ] Every `LOGIC_*.md` and `ARCH_*.md` has no `[UNVERIFIED]` flags remaining
- [ ] Task→load mapping covers all registered doc prefixes
- [ ] `REFACTOR_TODO.md` created (even if empty)

> **The system is live when the agent can start a fresh session, read only `agent.md`, and know exactly which files to load for any task — without asking.**

---

> **Final Goal**: A system where every developer (AI or human) shares the same perfect memory, enabling zero-loss handovers and instant project mobility.