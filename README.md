<img src="https://xulioze.com/img/extended-context-guide.jpg" alt="notes in a screen" />

# 🤖 Extended Context
**Knowledge and Memory Architecture for Building Software with AI. Agents also need to take notes!**

Turn "vibe-coding" into traceable and scalable engineering.

**The problem:** AI agents (Cursor, GitHub Copilot, Claude Code) generate code without memory → lost decisions, constant hallucinations, work impossible to divide into tasks.

**The solution:** Separate reusable knowledge (rules) from temporary memory (memory). The same pattern your brain uses.

[🗂️ Architecture](docs/ARCHITECTURE.md) | [⚡ Quick Start](#quick-start) | [🎯 Real Examples](#real-examples)

---

## Quick Start

### Before vs After

**Without structure:**
```
"Implement WooCommerce automatic invoicing"
→ 200 lines of code
→ Where to start debugging? What decisions were made?
```

**With Extended Context:**
```
1. @analysis checkout.php      → Analyze existing code
2. @solution invoicing         → Documented proposal  
3. @backlog                    → 8 atomic tasks (2-4h each)
4. @execute [1.1]              → Implement first task
5. @commit                     → Conventional commit + update progress
```
→ Traceable work, documented decisions, systematic debugging

### Installation (5 minutes)

**Option A - Daily use (recommended):**
```bash
cd your-project
git clone https://github.com/arinspunk/extended-context.git temp
cp -r temp/{rules,memory} .cursor/
rm -rf temp
```

**Option B - Contributing to the project:**
```bash
cd your-project  
git clone https://github.com/arinspunk/extended-context.git .cursor
```

**Verify installation:**
```bash
# In Cursor, open chat (Cmd+L) and type:
What expert and guidelines do you have active?
```

✅ You should see: `wordpress-classic-themes.mdc`, `kiss.mdc`, etc.

---

## How It Works

### The Problem with Current Tools
LLMs have limited context windows (~200K tokens). Without structure:
- Each session starts from scratch
- The agent "forgets" previous decisions
- Impossible to trace why architecture X was rejected two weeks ago

### The Solution: Rules + Memory

```
.cursor/
├── rules/          # Reusable knowledge (persists between projects)
│   ├── experts/    # WordPress, React, Python… (alwaysApply: true)
│   ├── guidelines/ # JS architecture, BEM, project constraints… (alwaysApply: true)
│   └── utils/      # Invocable protocols (alwaysApply: false)
└── memory/         # Temporary experience (this project, this feature)
    └── YYYYMMDD-VV-description.md
```

**Rules = Your brain's skills** (JavaScript, WordPress, SOLID principles)  
**Memory = Recent experiences** (what you did yesterday, last week's decisions)

### Key Concept: alwaysApply

- `alwaysApply: true` → Cursor loads automatically (experts, critical guidelines)
- `alwaysApply: false` → User invokes with `@` as needed (protocols, utils)

**Practical example:**
```markdown
# rules/experts/wordpress-classic-themes.mdc
---
alwaysApply: true
---
PHP + vanilla JS specialist. Security: sanitize_*, nonces. 
Performance: transients (12h). Translation: __(), _e().
```

Cursor loads this expert in **every conversation** → all responses automatically follow WordPress best practices.

---

## Real Examples

### Case 1: Bug in PDF System (WordPress)
**Context:** Legacy PDF generation system with undocumented dependencies.

**Traditional workflow:**
- 2 hours reading code to understand architecture
- Change breaks email integration (undocumented)
- 4 additional hours of debugging

**With Extended Context:**
```bash
@analysis includes/pdf-generator.php
# → Generates: 20251015-01-analysis-pdf-generator.md
# Documents: WooCommerce dependencies, critical hooks, template system

@solution "separate PDF logic from email system"
# → Generates: 20251015-02-solution-refactor-pdfs.md
# Proposal: extract PDF_Generator class, maintain hook compatibility

@backlog
# → Generates: 20251015-03-backlog-refactor.md
# 6 atomic tasks, each with acceptance criteria
```

**Result:** 
- Refactor completed in 1 session
- Decisions documented for future maintenance
- Zero regressions (tests based on previous analysis)

### Case 2: Feature from Scratch with Complex Stack
**Context:** Automatic invoicing system in WooCommerce with WPML + ACF PRO integrations.

**Challenge:** Multiple critical dependencies, complex business logic, legal requirements.

**Workflow:**
```bash
@analysis woocommerce-checkout-flow
# Documents: available hooks, current integrations, legal constraints

@solution automatic invoicing
# Proposal: woocommerce_order_status_completed hook, IRPF validation, audit logs

@backlog
# 12 tasks divided into 3 phases: validation → implementation → testing

@execute Phase 1
# Implements validations and base structure

@commit
# Conventional commits + automatic backlog update
```

**Result:**
- Complete feature in 4 sessions (vs. 2 weeks estimated)
- Documentation of technical decisions for compliance
- New dev onboarding in 1 hour (reading memory/)

---

## Configuration for Your Project

### 1. Identify Your Stack (2 min)
Review `rules/experts/` and activate the matching expert:
- ✅ WordPress → `wordpress-classic-themes.mdc`
- ✅ React + TypeScript → `react-typescript.mdc`
- ✅ Python + FastAPI → `python-fastapi.mdc`

**Your stack not listed?** Copy the closest expert and adapt it.

### 2. Configure Project Constraints (3 min)
```bash
cp rules/guidelines/constraints-template.mdc \
   rules/guidelines/constraints-my-project.mdc
```

Complete:
- Technical stack (specific versions: PHP 7.4+, Node 18.x)
- Critical dependencies (ACF PRO, WooCommerce, libraries that WON'T change)
- Legacy systems you must respect

Change `alwaysApply: false` → `true`

### 3. Activate Base Principles (1 min)
In `rules/guidelines/kiss.mdc`:
```diff
---
- alwaysApply: false
+ alwaysApply: true
---
```

### 4. First Execution (5 min)
Test the workflow with a real file:
```bash
@analysis src/components/Header.tsx
```

✅ If it generates `memory/YYYYMMDD-01-*.md`, the system is operational.

---

## Migration from v1

This architecture is the evolution of the [v1 4-file system](https://xulioze.com/en/blog/guide-develop-software-agents.html). If you're coming from that system (`01-expert.md`, `02-analysis.md`, `03-plan.md`, `04-backlog.md`), here's the mapping to v2:

**Mapping v1 → v2:**
- `01-expert.md` → `rules/experts/{stack}.mdc` + `rules/guidelines/constraints-{project}.mdc`
- `02-analysis.md` → `memory/YYYYMMDD-VV-analysis-*.md` (generated automatically)
- `03-plan.md` → `memory/YYYYMMDD-VV-solution-*.md` (@solution protocols)
- `04-backlog.md` → `memory/YYYYMMDD-VV-backlog-*.md` (@backlog protocol)

**Migration benefits:**
- ✅ Modular and reusable knowledge between projects
- ✅ Chronological history without overwriting decisions
- ✅ Automated protocols (goodbye copy/paste prompts)

---

## Contributing

### How to Contribute

**New experts and protocols:**
Your stack not covered? Have specific workflows to automate? All contributions are welcome.

**Improvements to existing content:**
Share refinements, document edge cases, propose optimizations.

### Process

1. **Fork** this repo
2. **Create** your expert/protocol in branch: `git checkout -b expert/nextjs`
3. **Document** real use case in `docs/examples/`
4. **Submit** PR with description of problem it solves

**Acceptance criteria:**
- ✅ Includes real usage example
- ✅ Follows existing nomenclature (alwaysApply, structure)
- ✅ Documentation in English (Spanish welcome as translation)

---

## FAQ

**Q: Does it only work with Cursor?**  
A: No. The rules/memory architecture is tool-agnostic. You can adapt it to GitHub Copilot, Claude Code, or any tool with context injection capability. Cursor is the implementation example because it has a native rules system.

**Q: What if my project already has .cursor/?**  
A: Merge the directories. Previous backup: `cp -r .cursor .cursor.backup`

**Q: How do I scale to large teams?**  
A: Version `.cursor/rules/` in Git. Each dev maintains their local `memory/`. For shared decisions: create `rules/guidelines/team-decisions.mdc`.

**Q: How much does memory/ grow over time?**  
A: ~50-100 files per active project (6 months). They're plain text, ~20KB average. Total: <5MB. Git versions them efficiently.

## Troubleshooting

**Cursor doesn't load rules:**
```bash
# Verify YAML syntax
cat .cursor/rules/experts/wordpress.mdc | head -3
# Should show:
# ---
# alwaysApply: true
# ---
```

**@analysis doesn't generate file in memory/:**
```bash
# Verify permissions
ls -la .cursor/memory/
# Must be writable. If not: chmod -R u+w .cursor/memory/
```

**Agent ignores project constraints:**
```bash
# Verify constraints-*.mdc has alwaysApply: true
grep "alwaysApply" .cursor/rules/guidelines/constraints-*.mdc
```

---

## License

MIT License - Use, modify, distribute freely.

## Credits

Created by [Xúlio Zé](https://xulioze.com/en/cv.html).

---

## Support

- 🐛 **Bugs:** [GitHub Issues](https://github.com/arinspunk/extended-context/issues)
- 📧 **LinkedIn:** [Xúlio Zé](https://www.linkedin.com/in/xulioze/)
- 🦋 **Bluesky:** [@xulio-ze.bsky.social](https://bsky.app/profile/xulio-ze.bsky.social)

---

**⭐ If this system saves you time, leave a star to help other developers discover it 😊**