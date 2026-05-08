# Living Docs

I didn't set out to build a system. I just got tired of repeating myself.

Every new AI coding session, I'd paste the same context. Here's the project structure. Here's the naming convention. Here's why we don't touch that module. Here's the version bump command. The agent would do the work, and then the next session I'd do it all again. The knowledge was in my head, not anywhere the agent could find it.

So I wrote it down. An `agent.md` at the project root. A few rules, some metadata, a checklist. The agent read it at the start of each session and we got moving faster.

Then I noticed something: the agent wasn't just reading the doc. It was improving it. After fixing a bug, it would update the relevant section. After adding a feature, it would register the new module. After hitting a sharp edge in the codebase, it would write a warning so the next session wouldn't hit it again.

I hadn't asked it to do that. It just made sense.

## Quick Start (Bootstrapping a New Project)

To initialize this system in a brand new repository, give your AI agent the following prompt:

> **"Read the master template at `https://raw.githubusercontent.com/Diew/living-docs/main/LIVING_DOC_SYSTEM.md` and execute the 'Bootstrapping a New Project' sequence to initialize this current directory."**

Alternatively, download `LIVING_DOC_SYSTEM.md` and attach it directly to your agent if it does not support URL fetching.

The agent will generate the required tripartite structure (`agent.md`, the `docs/` folder, and the governance registry) and enforce the domain-agnostic rules for your new codebase.

## What it became

Over time, one file wasn't enough. The logic was real and it was growing. Domain rules belonged somewhere separate from data rules. Architecture decisions didn't belong next to domain-specific constants. So I split things out.

Now the system looks like this:

**`agent.md`** — the entry point. Session-critical rules, project metadata, quick-reference checklists. Short by design. The agent reads this every session without being asked.

**`docs/`** — the wiki. Every file owns exactly one concern. `GUIDE_developer.md` for coding standards. `ARCH_technical-specs.md` for data and routing. `STANDARDS_*.md` for domain-specific rules (e.g., API boundaries, UI tokens). `LOGIC_*.md` for feature-specific behavior. Each file has a clear prefix so both humans and agents know what's inside before opening it.

**`ARCH_documentation-governance.md`** — the schema. A registry of every doc file, what it contains, what it must not contain, and when to load it. A task-to-file mapping so the agent knows exactly which docs to read for any given task. This file is what makes the agent a disciplined maintainer rather than a chatbot that happens to have access to some markdown files.

The rule that holds it together: **one file owns each rule. No duplication.** If a rule exists in two places, you have two sources of truth, which means you have none.

## The part I didn't expect

The docs compound.

When the agent fixes a routing bug, it updates `ARCH_technical-specs.md`. When it adds a new feature, it registers the new doc in the governance registry. When it discovers an edge case — an intentional API timeout, a specific architectural quirk, or a critical data pipeline sequence that must not be touched — it writes it down. The next session, that knowledge is already there.

This is different from a skill file or a static prompt. A skill file is a snapshot. It's accurate when you write it and gradually wrong after that. The living doc is a contract: the agent is responsible for keeping it true. It doesn't drift because the thing that updates the code also updates the doc.

After enough sessions, the docs start to feel like institutional memory. Not my memory, not the agent's memory — the project's memory.

## The architecture, restated simply

Three layers:

**The codebase** — immutable from the doc system's perspective. Source of truth for what the code actually does.

**The docs** — LLM-maintained markdown files. The agent reads them before acting and updates them after. Governed by naming conventions, ownership rules, and a registry that prevents duplication.

**The schema** — `agent.md` plus the governance doc. Tells the agent how the doc system works, which file owns what, and what workflows to follow. This is the config that makes the whole thing disciplined.

## Why it works

The tedious part of maintaining documentation isn't the writing. It's the bookkeeping — keeping things in sync, updating cross-references, catching when a rule in one file contradicts a rule in another. Humans abandon docs because the maintenance cost compounds faster than the value. An agent doesn't get bored, doesn't forget to update a reference, and can touch five files in one pass.

The human's job is to design the structure, make judgment calls, and ask the right questions. The agent's job is everything else.

One deliberate constraint: **the agent does not update docs automatically.** Doc updates happen when the human explicitly asks — "update the docs", "run a doc sweep", "register this new file".

The reason is a failure chain. If the agent updates docs automatically after every code task, and that code task was wrong, the docs now reflect wrong behavior. The next session, the agent reads those docs and treats the wrong behavior as intended. It compounds. The human is the checkpoint — you confirm the code is correct first, then instruct the agent to update the docs. That sequence is what keeps the system trustworthy over time.

## What I'd call it now

I built this without a name for it. I thought I was just writing good documentation.

After reading Karpathy's LLM Wiki gist, I wanted to apply the concept. I asked an AI to compare his system with my existing documentation workflow. The AI's conclusion: *same architecture, different domain.* 

His pattern: raw sources stay immutable, an LLM-maintained wiki sits between you and the raw material, a schema doc tells the agent how to maintain it. 
My pattern: the codebase stays immutable, an LLM-maintained doc layer sits between the agent and the code, a governance doc tells the agent how to maintain it.

That confirmation was the trigger. I decided to extract the governance rules from the chaotic main project into this standalone, domain-agnostic template.

The name I'd give it now: **Living Docs**. Not a skill file. Not a prompt. A persistent, compounding artifact that gets more accurate with every task the agent completes — because the agent that changes the code is the same agent that keeps the record.

---

*This system grew out of the chaotic iteration of my main project. By having the AI itself audit the messy codebase, it successfully extracted the structural patterns into a simple web-development starter kit called [vt-template](https://github.com/Diew/vt-template). But the true breakthrough wasn't the folder structure—it was the domain-agnostic AI governance rules that the AI helped formalize along the way: Living Docs.*
