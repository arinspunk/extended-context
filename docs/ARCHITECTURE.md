# Architecture: Extended Context

This document explains the design decisions, conceptual foundations, and system evolution from v1 to the current version.

## Table of Contents
- [The Fundamental Problem](#the-fundamental-problem)
- [Core Concepts: Prompt, Context, Memory](#core-concepts)
- [Evolution: From v1 to v2](#evolution-from-v1-to-current-version)
- [Knowledge/Memory Architecture](#knowledgememory-architecture)
- [Design Decisions](#design-decisions)
- [Limitations and Trade-offs](#limitations-and-trade-offs)
- [Universal Applicability](#universal-applicability)

---

## The Fundamental Problem

### The Perfect Prompt Illusion

Autonomous AI agents (Cursor, GitHub Copilot, Claude Code) created an illusory expectation: "if I write the perfect prompt, the agent will generate the entire feature in one go."

**Reality:**
- LLMs have limited context windows (~200K tokens)
- Without structure, each session starts from scratch
- The agent "forgets" previous decisions
- Impossible to trace why architecture X was rejected two weeks ago

### Vibe-Coding: Prototyping vs Production

**Vibe-coding** (trusting the agent to "understand the vibe" of the project) works for quick prototypes, but generates problems in real projects:

1. **Lost decisions**: Why did we choose this architecture two weeks ago?
2. **Constant hallucinations**: The AI invents APIs, mixes frameworks, ignores constraints
3. **Monolithic work**: Impossible to divide features into traceable tasks
4. **Continuous improvisation**: Each session starts from scratch

**Recent research** shows that [LLMs tend to generate more complex code than necessary](https://arxiv.org/html/2411.09916v1), reinforcing the need for structured constraints.

---

## Core Concepts

Before explaining the architecture, we need to establish three fundamental concepts:

### 1. Prompt
**Definition:** The instruction that the user sends to the LLM at a specific moment.

**Example:**
```
"Implement WooCommerce automatic invoicing"
```

**Characteristics:**
- Ephemeral (only exists in that conversation)
- Depends on available context
- No memory between sessions

### 2. Context
**Definition:** All the information the LLM can "see" when processing a prompt.

**Context composition:**
```
Total Context = System Prompt + Current conversation + Attached files + Active rules
```

**Example in Cursor:**
```
System Prompt:    "You are a WordPress expert..."
Conversation:     [Last 10 messages]
Files:            checkout.php, functions.php
Active rules:     wordpress-expert.mdc, kiss.mdc, my-project-constraints.mdc
```

**Physical limit:** ~200K tokens in current models (GPT-4, Claude Sonnet)

### 3. Memory
**Definition:** Persistent information that survives between sessions.

**Types of memory in AI-assisted development:**
- **Short-term memory**: Current conversation (automatically managed by the tool)
- **Medium-term memory**: Decisions from this feature/sprint (manual in most tools)
- **Long-term memory**: Project architecture, established patterns (non-existent without structure)

**The problem:** Most tools only have short-term memory. Extended Context implements all three.

---

## Evolution: From v1 to Current Version

### v1 System: Structured Workflow

**Original proposal:** [Guide: Developing Quality Software Assisted by AI Agents](https://xulioze.com/en/blog/guide-develop-software-agents.html)

**Structure:**
```
context/
├── 01-expert.md      # Expert profile
├── 02-analysis.md    # Code analysis
├── 03-plan.md        # Implementation plan
└── 04-backlog.md     # Pending tasks
```

**Workflow:**
```
Analysis → Solution → Backlog → Execution → Commit → Continue
```

**Main achievement:** Converted vibe-coding into traceable engineering.

### v1 Limitations

**1. Mixed knowledge**
`01-expert.md` combines general principles (KISS, DRY) with project-specific constraints (stack, versions):

```markdown
# 01-expert.md
## Principles
- KISS, DRY, SOLID
- Single Responsibility

## Project Stack
- WordPress 6.3
- ACF PRO 6.2.1
- WooCommerce 8.0
```

**Problem:** Impossible to reuse principles between projects without dragging irrelevant configuration.

**2. Duplication between projects**
If you work on 3 WordPress projects, you have "security: sanitize_*, nonces" copied three times. Update a best practice → manual synchronization in 3 places.

**3. No historical management**
v1 gives you three options, all bad:
- Overwrite `02-analysis.md` → lose previous decisions
- Accumulate analyses in the same file → grows uncontrollably
- Create multiple files → disorder without consistent nomenclature

**4. No modularity**
Switching from WordPress to React requires rewriting the entire `01-expert.md`. You can't maintain common modules (principles, patterns) and only swap the technical expert.

**5. Manual execution**
Each action required writing the complete prompt:
```
"Analyze this file following the 01-expert.md structure and save to 02-analysis.md"
"Read 04-backlog.md, execute tasks 1-3, update status"
```

### Current Version: Knowledge/Memory Separation

The current version solves the 5 problems through the fundamental conceptual separation:

**Knowledge** (reusable, persists between projects) vs **Memory** (temporary, project/feature-specific)

**Implementation in Cursor:**
- `rules/` = mechanism to inject knowledge
- `memory/` = directory to store memory
- `@` commands = syntax to invoke protocols

**Problems solved:**

1. **Modular knowledge** → `rules/experts/`, `rules/guidelines/` separated
2. **Reusability** → One WordPress expert for N projects
3. **Chronological history** → `memory/YYYYMMDD-VV-description.md`
4. **Composability** → Swap experts, maintain guidelines
5. **Automation** → Invocable protocols (`@analysis`, `@backlog`)

**Important note:** Although this guide uses Cursor as an example, the Knowledge/Memory separation is a universal pattern applicable to any AI tool.

---

## Knowledge/Memory Architecture

### Mental Model: Your Brain

The architecture replicates how your brain works:

**Permanent knowledge:**
- Languages you speak
- Frameworks you master
- Architecture principles

**Episodic memory:**
- What you did yesterday
- Last week's decisions
- Bugs you resolved this month

But also, your knowledge **is not monolithic**: you have specialized modules (JavaScript, WordPress, KISS) that you activate according to context.

### Implementation in Cursor

Cursor implements this separation through `rules/` and `memory/`:

```
.cursor/
├── rules/          # Reusable and modular knowledge
│   ├── experts/    # Technical roles (WordPress, React, Python...)
│   ├── guidelines/ # Principles and constraints (KISS, project-specific)
│   └── utils/      # Executable protocols (analysis, backlog, commit)
└── memory/         # Temporary experiences
    └── YYYYMMDD-VV-description.md
```

**Key concepts:**
- `rules/` = Vehicle to inject **knowledge** into context
- `memory/` = Storage of project **memory**
- This structure is Cursor-specific, but the Knowledge/Memory pattern is universal

### Rules: Reusable Knowledge

The `rules/` in Cursor are the mechanism to inject structured **knowledge**. In Extended Context, we organize rules into three categories:

**Experts** - Technical/domain profiles:

```markdown
# rules/experts/wordpress-classic-themes.mdc
---
alwaysApply: true
---
# WordPress Classic Expert
PHP + vanilla JS specialist. WordPress core APIs, theme dev.

## Stack
Backend: PHP 7.4+, WP Core APIs
Frontend: Vanilla JS, HTML5, CSS3

## Core Patterns
- Security: sanitize_*, esc_*, nonces
- Performance: transients (12h), conditional enqueue
- Translation: __(), _e() with text domain
```

**Guidelines** - Principles and constraints:

```markdown
# rules/guidelines/kiss.mdc
---
alwaysApply: true
---
# KISS Principle
- Direct solutions, avoid over-engineering
- Simple patterns, minimal dependencies
- Single responsibility per function

Rule: If you can't explain it in 2-3 sentences, it's too complex.
```

```markdown
# rules/guidelines/my-project-constraints.mdc
---
alwaysApply: true
---
# my-project Theme Project
Text domain: my-project v0.1.2
Stack: Node | SASS 1.67.0 | PHP 7.4+ | jQuery 3.x
Build: npm run watch:sass (never edit compiled CSS)

Critical: ACF PRO (18 groups), WooCommerce, WPML
```

**Utils** - Invocable protocols:

```markdown
# rules/utils/analysis.mdc
---
alwaysApply: false
---
# Analysis Protocol
Trigger: @file/@folder or "analyze"

Steps:
1. Acknowledge target
2. Analyze structure, patterns, dependencies
3. Save to .cursor/memory/YYYYMMDD-VV-analysis-*.md
4. Confirm completion
```

### Memory: Historical Record

Protocols (`rules/utils/`) automatically generate files in `memory/` following the nomenclature `YYYYMMDD-VV-description.md`:

```
memory/
├── 20250919-01-analysis-pdf-generator.md
├── 20250919-02-pdf-email-system-completed.md
├── 20251015-01-complete-analysis-theme.md
├── 20251015-02-solution-automatic-invoicing-399.md
└── 20251015-03-backlog-automatic-invoicing-399.md
```

**Advantages of chronological nomenclature:**
- Natural ordering by date
- Multiple entries per day (VV sequence: 01, 02, 03...)
- Immediate traceability
- Visible relationships by temporal proximity

**Key about context:**
Files in `memory/` store the project's **memory**. They are **disk references** that only occupy context when:
- The user invokes them: `@memory/20251015-03-backlog.md`
- A protocol automatically reads/writes them

**Practical implication:** You can accumulate hundreds of memory files without performance impact. The complete history persists for future consultation without context penalty.

---

## Design Decisions

### 1. Why `alwaysApply`

**Problem:** With 50+ available rules, which ones does Cursor automatically load in each conversation?

**Solution:** `alwaysApply` flag for granular control of what occupies context:

```markdown
---
alwaysApply: true   # Automatic loading → always consumes context
---
```

```markdown
---
alwaysApply: false  # On-demand invocation → only consumes context when invoked
---
```

**Decision criteria:**
- `true` → Knowledge you ALWAYS want active (base expert, KISS principles)
- `false` → Specific tools you use as needed (protocols @analysis, @backlog)

**Practical limit:** ~5-10 rules with `alwaysApply: true` to avoid saturating context.

**Important note:** `memory/` files **do not** use `alwaysApply` because they are never loaded automatically. They only occupy context when you invoke them explicitly (`@memory/file.md`) or when a protocol reads them.

### 2. Why YYYYMMDD-VV Nomenclature

**Alternatives considered:**

**Option A - Descriptive:**
```
analysis-pdf-generator.md
solution-refactor-pdfs.md
```
✗ No temporal order, difficult to trace when each decision was made

**Option B - Incremental:**
```
001-analysis.md
002-solution.md
```
✗ Numbers without meaning, impossible to associate with date

**Option C - Complete ISO 8601:**
```
20251015T143022-analysis.md
```
✗ Unnecessary precision, hinders human readability

**Chosen option - Chronological with daily sequence:**
```
20251015-01-analysis-pdf-generator.md
20251015-02-solution-refactor-pdfs.md
```
✓ Perfect balance: chronological order + readability + multiple entries/day

### 3. Why Separate Experts/Guidelines

**Rejected alternative:** A single `rules/my-project-config.mdc` file

**Problem:** Mixing technical expert with project constraints prevents reusability.

**Example:**

```markdown
# ✗ Monolithic (not reusable)
rules/wordpress-my-project.mdc
- WordPress expert
- KISS principles
- My Project constraints

# ✓ Modular (reusable)
rules/experts/wordpress-classic-themes.mdc    # Reusable in N projects
rules/guidelines/kiss.mdc                     # Reusable in any stack
rules/guidelines/my-project-constraints.mdc    # Project-specific
```

**Benefit:** You work on 5 WordPress projects → 1 expert, 1 KISS, 5 constraints.

### 4. Why Protocols in utils/

**Rejected alternative:** Manually writing prompts each time.

**Problem in v1:**
```
"Analyze includes/pdf-generator.php following 01-expert.md structure, 
document dependencies and critical hooks, save to 02-analysis.md with 
consistent format..."
```

**Solution v2:**
```
@analysis includes/pdf-generator.php
```

**Implementation:**
```markdown
# rules/utils/analysis.mdc
---
alwaysApply: false
---
Trigger: @file/@folder or "analyze"
1. Acknowledge → 2. Analyze → 3. Save to memory/ → 4. Confirm
```

**Benefit:** Reduces cognitive friction. Developer thinks "I need to analyze this file" → invokes `@analysis` → protocol handles the details.

### 5. Why Explicit KISS Principle

**Observation:** [LLMs tend to generate more complex code than necessary](https://arxiv.org/html/2411.09916v1).

**Without explicit KISS:**
```python
# LLM proposes
class ConfigurationManager:
    def __init__(self):
        self.strategies = {}
        self.observers = []
    
    def register_strategy(self, name, strategy):
        self.strategies[name] = strategy
    
    def notify_observers(self):
        for observer in self.observers:
            observer.update(self)
```

**With explicit KISS:**
```python
# LLM proposes
config = {
    'api_key': os.getenv('API_KEY'),
    'timeout': 30
}
```

**Result:** It's easier to ask the agent to add complexity to a simple solution than to waste time analyzing what's unnecessary in its proposal.

---

## Limitations and Trade-offs

### Current Limitations

**1. Cursor dependency (partial)**
- Implementation uses Cursor's native rules system
- Concepts (rules/memory) are portable to other tools
- Requires syntax adaptation (`@` commands)

**2. Context window still limited**
- Rules with `alwaysApply: true` permanently consume context
- With 10+ active rules simultaneously, you can saturate context (~200K tokens)
- **Important:** Files in `memory/` **DO NOT** automatically consume context. They only occupy tokens when:
  - You explicitly invoke them: `@memory/20251015-03-backlog.md`
  - A protocol reads/writes them
- You can have hundreds of files in `memory/` without context impact
- **Curation needed:** Deactivate unnecessary rules (`alwaysApply: false`), not archive memory

**3. No automatic synchronization**
- Each developer maintains their local `memory/`
- Shared decisions require manually moving to `rules/guidelines/`
- No tooling to "promote" memory to rule

**4. Initial learning curve**
- First-time setup: 15-20 minutes
- Internalize workflow: 2-3 days of use
- Create custom experts: requires understanding structure

### Design Trade-offs

**Trade-off 1: Automation vs Control**
- ✓ Protocols automate repetitive tasks
- ✗ Less transparency about what the agent does
- **Decision:** Prioritize automation, document protocols well

**Trade-off 2: Modularity vs Simplicity**
- ✓ Separate experts/guidelines → maximum reusability
- ✗ More files to maintain
- **Decision:** Modularity is worth it in projects >1 month

**Trade-off 3: Structure vs Flexibility**
- ✓ YYYYMMDD-VV nomenclature → chronological order, unlimited history
- ✗ Can't use completely custom names
- **Decision:** Consistency > absolute flexibility

**Trade-off 4: Active rules vs Available context**
- ✓ More rules with `alwaysApply: true` → more knowledge always present
- ✗ Less available context for project files
- **Decision:** Limit to 5-10 active rules, rest on-demand
- **Doesn't apply to memory:** Memory files don't consume context until invoked

---

## Universal Applicability

### Beyond Cursor

The **Knowledge/Memory** architecture is tool-agnostic. The fundamental pattern is:

```
Reusable knowledge + Temporary memory = Effective context
```

**Cursor implements this via:**
- `rules/` (knowledge) + `memory/` (memory) + `@` commands (invocation)

**Other tools implement it in different ways, but the concept is the same.**

### Adaptations to Other Tools

### GitHub Copilot
```
# .github/copilot-instructions.md
Expert: WordPress classic themes
Principles: KISS, DRY
Constraints: [see memory/project-constraints.md]
```

**Memory management:** Invoke memory files manually as needed, not loaded automatically.

### Claude Code (CLI)
```bash
# .claude/config.json
{
  "rules": ["wordpress-expert", "kiss", "my-project-constraints"],
  "memory_path": ".claude/memory/"
}
```

**Memory management:** Memory files are explicitly referenced in commands, don't passively consume context.

### Windsurf / Aider / Continue
Each tool has its own mechanisms to inject context. The Knowledge/Memory pattern adapts to all:

1. **Identify** how your tool injects context (config files, system prompts, etc.)
2. **Map Knowledge** to that mechanism (equivalent to `rules/`)
3. **Implement Memory** as reference files (equivalent to `memory/`)
4. **Create shortcuts** for common protocols (equivalent to `@` commands)

### Universal Principle

**Regardless of the tool:**
- Separate **Knowledge** (persists) from **Memory** (temporary)
- Modularize **Knowledge** (experts, guidelines, protocols)
- Automate repetition (invocable protocols)
- Document chronologically (YYYYMMDD-VV)

**The specific implementation (rules/, @commands, etc.) is secondary. The conceptual pattern is what matters.**

---

## Conclusion

Extended Context is not just a file structure. It's a **knowledge architecture** based on the universal pattern:

**Knowledge/Memory Separation**

This architecture:

1. **Extends LLM memory** without changing the model
2. **Modularizes expertise** for reusability between projects
3. **Automates workflows** without losing control
4. **Documents decisions** chronologically
5. **Adapts to any tool** with context injection capability

**Implementation in Cursor:**
- `rules/` = knowledge injection
- `memory/` = memory storage
- `@` commands = protocol invocation

**The pattern is universal, the implementation is tool-specific.**

Vibe-coding works for prototypes. For production, you need traceable engineering. This architecture provides the conceptual framework, regardless of the tool you use.

---

## References

- [Original post about Extended Context v1](https://xulioze.com/en/blog/guide-develop-software-agents.html)
- [Original post about Extended Context v0](https://xulioze.com/en/blog/extended-context.html)
- [Anthropic's prompt engineering guide](https://docs.anthropic.com/en/docs/prompt-engineering)
- [LLMs and Code Complexity (arXiv)](https://arxiv.org/html/2411.09916v1)
- [Cursor Rules Documentation](https://docs.cursor.com)

---

**Questions about design decisions?** Open a [GitHub Issue](link) or participate in [Discussions](link).