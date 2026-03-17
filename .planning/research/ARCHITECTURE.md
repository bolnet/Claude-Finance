# Architecture Research

**Domain:** Claude Code native skill — Python finance code generation and execution
**Researched:** 2026-03-17
**Confidence:** HIGH (sourced from installed plugin reference files on this machine, not training data)

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     User Invocation Layer                        │
│  /finance [natural language request]   /finance-analyst [req]   │
│  /finance-pm [req]  — slash commands in .claude/commands/       │
├─────────────────────────────────────────────────────────────────┤
│                     Skill Knowledge Layer                        │
│  .claude/skills/finance-analyst/SKILL.md                        │
│  .claude/skills/finance-pm/SKILL.md                             │
│  .claude/skills/finance/SKILL.md  (generalist)                  │
│  references/ — task routing rules, output templates             │
├─────────────────────────────────────────────────────────────────┤
│                     Code Generation Layer                        │
│  Claude generates Python code based on intent classification     │
│  Task types: stock-analysis | liquidity-model | investor-class  │
├─────────────────────────────────────────────────────────────────┤
│                     Execution Layer                              │
│  Bash tool — runs generated Python scripts                      │
│  Python environment: venv or system (auto-detected)             │
│  Dependencies: yfinance, pandas, numpy, matplotlib, sklearn     │
├─────────────────────────────────────────────────────────────────┤
│                     Output Layer                                 │
│  Charts → .finance-output/charts/*.png (saved files)           │
│  Tables → printed to terminal (pandas to_string / to_markdown)  │
│  Summary → plain-English interpretation printed to terminal     │
│  CSV inputs → read from working directory or path provided      │
└─────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Implementation |
|-----------|---------------|----------------|
| Slash command (`.claude/commands/finance.md`) | Entry point. Accepts `$ARGUMENTS`, defines allowed tools, routes to skill | Markdown file with YAML frontmatter |
| Skill (`SKILL.md`) | Domain knowledge: how to classify intent, what libraries to use, output conventions | Markdown file loaded by Claude automatically |
| Intent classifier (embedded in skill) | Maps natural language to task type: stock analysis / liquidity / investor | Instructions in SKILL.md, not code |
| Python code generator (Claude) | Writes complete, runnable Python scripts for each task type | Claude's generation, guided by skill instructions |
| Bash executor | Runs generated scripts, captures stdout/stderr, returns output | `allowed-tools: Bash(python3:*), Bash(pip:*)` |
| Output manager | Saves charts to `.finance-output/charts/`, prints tables + summaries | Conventions defined in SKILL.md |
| Dependency checker | Detects missing packages, installs them or prompts user | Shell check in command preamble: `!pip list` or `!python3 -c "import yfinance"` |
| Persona router | Determines which persona variant handles the request | Separate command files per persona (Phase 2+) |

## Recommended Project Structure

```
machine_learning_skill/
├── .claude/
│   ├── commands/
│   │   ├── finance.md              # /finance — generalist entry point (Phase 1)
│   │   ├── finance-analyst.md      # /finance-analyst — equity research persona (Phase 2)
│   │   └── finance-pm.md           # /finance-pm — portfolio manager persona (Phase 3)
│   └── skills/
│       ├── finance/
│       │   ├── SKILL.md            # Core skill knowledge (intent routing, output rules)
│       │   ├── references/
│       │   │   ├── task-types.md   # Task classification reference
│       │   │   ├── libraries.md    # Library usage patterns per task type
│       │   │   └── output-format.md# Output conventions: charts, tables, summaries
│       │   └── templates/
│       │       ├── stock-analysis.py.md      # Code template for stock workflows
│       │       ├── liquidity-model.py.md     # Code template for regression workflows
│       │       └── investor-classifier.py.md # Code template for classification workflows
│       ├── finance-analyst/
│       │   └── SKILL.md            # Analyst persona overlay (Phase 2)
│       └── finance-pm/
│           └── SKILL.md            # PM/Trader persona overlay (Phase 3)
└── .finance-output/
    ├── charts/                     # All matplotlib/seaborn charts saved here
    └── .gitkeep
```

### Structure Rationale

- **`.claude/commands/`:** One file per slash command. Each command is the user-facing entry point. File name = command name (`finance.md` → `/finance`).
- **`.claude/skills/finance/`:** The skill holds the domain knowledge Claude consults before generating code. Commands invoke tools; skills hold expertise. Separation allows skills to be reused across commands.
- **`references/`:** Supplementary markdown files loaded via `@path/to/file` syntax inside the skill. Keeps SKILL.md focused while providing detailed lookup tables.
- **`templates/`:** Python code templates stored as `.py.md` files. The skill instructs Claude to use these as starting points, reducing hallucination in library calls.
- **`.finance-output/charts/`:** Consistent output directory ensures charts are always findable. Using `matplotlib.use('Agg')` backend allows headless execution in terminal.

## Architectural Patterns

### Pattern 1: Command as Router, Skill as Brain

**What:** The slash command handles invocation mechanics (tools, arguments, context injection). The SKILL.md handles all domain logic (how to classify intent, how to write code, what to output).

**When to use:** Always. This separation keeps commands thin and skills reusable.

**Trade-offs:** Requires loading skill content at invocation time; adds a small latency but is unavoidable and expected in the Claude Code model.

**Example command structure:**
```markdown
---
description: Run finance analysis from natural language request
argument-hint: [request]
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read
---

Use the finance skill to handle this request: $ARGUMENTS

Environment check: !`python3 -c "import yfinance, pandas, sklearn" 2>&1 || echo "MISSING_DEPS"`

Working directory: !`pwd`
Available CSV files: !`ls *.csv 2>/dev/null || echo "none"`
```

### Pattern 2: Dynamic Context Injection Before Code Generation

**What:** Use `!`command`` syntax in the command file to inject live environment state before Claude sees the prompt. This tells Claude what Python packages are available, what CSV files are present, and what the working directory is — without Claude having to ask.

**When to use:** Always in the finance command. The injected context determines whether to use yfinance (live data) or read a CSV (user-provided data).

**Trade-offs:** Adds a shell invocation at command start. Acceptable — these are fast, cheap commands (`ls`, `python3 -c`).

**Example:**
```markdown
Available data: !`ls *.csv *.xlsx 2>/dev/null || echo "no local files"`
Python env: !`python3 -c "import sys; print(sys.executable)"`
Packages: !`pip list 2>/dev/null | grep -E "yfinance|pandas|sklearn|matplotlib"`
```

### Pattern 3: Write-Then-Execute for Python Scripts

**What:** Claude writes the Python script to a temp file, then executes it with Bash. This is preferable to inline `python3 -c "..."` for scripts longer than a few lines.

**When to use:** All finance workflows. Scripts are 50-200 lines; inline execution is fragile for that length.

**Trade-offs:** Leaves a temp script on disk. Convention: write to `.finance-output/last_run.py` so it's always findable and inspectable.

**Example flow in command:**
```markdown
1. Generate Python script appropriate for the request
2. Write script to `.finance-output/last_run.py` using Write tool
3. Execute: !`python3 .finance-output/last_run.py 2>&1`
4. Read and interpret the output
5. Summarize results in plain English
```

### Pattern 4: Persona as Skill Overlay (Phase 2+)

**What:** Each persona (analyst, PM/trader) gets its own SKILL.md that references the base `finance` skill but adds persona-specific framing, preferred tasks, and output emphasis.

**When to use:** Once the generalist `/finance` command is proven. Do not build persona variants until the shared code generation layer is solid.

**Trade-offs:** Some duplication between persona skills. Accept this — DRY is less important than clear persona boundaries.

**Example analyst SKILL.md framing:**
```markdown
---
name: finance-analyst
description: Use when user invokes /finance-analyst or describes an equity research task
---

This skill extends the finance skill for Financial Analyst workflows.
Priority tasks: stock analysis, valuation, DCF interpretation, sector comparison.
Default output: always include price chart + returns table + valuation summary.
```

## Data Flow

### Request Flow

```
User types: /finance "show me AAPL volatility over the last 6 months"
    |
    v
Command file loaded (.claude/commands/finance.md)
    |
    v
Dynamic context injected:
  !`ls *.csv` → "no local files"
  !`python3 -c "import yfinance"` → (success, no output)
    |
    v
Finance skill loaded (.claude/skills/finance/SKILL.md)
  Claude reads intent: stock analysis → yfinance fetch → volatility calculation
    |
    v
Code generation:
  Claude writes Python script targeting AAPL, 6-month window,
  rolling std dev of daily returns, matplotlib chart
    |
    v
Write tool: script → .finance-output/last_run.py
    |
    v
Bash tool: python3 .finance-output/last_run.py
  → chart saved to .finance-output/charts/AAPL_volatility.png
  → table printed to stdout
    |
    v
Claude reads output, writes plain-English summary:
  "AAPL's 30-day rolling volatility peaked at X% in [month]..."
    |
    v
User sees: table in terminal + summary text + file path to chart
```

### CSV Input Flow

```
User types: /finance "clean and explore my data" (with investor_data.csv present)
    |
    v
Dynamic context injected: !`ls *.csv` → "investor_data.csv"
    |
    v
Skill recognizes CSV presence → uses local file, not yfinance
    |
    v
Generated script reads CSV, runs ML 01 cleaning pipeline
  (outlier detection, categorical handling, visualization)
    |
    v
Charts → .finance-output/charts/
Tables → stdout
Summary → Claude interprets and explains
```

### Persona Routing Flow (Phase 2+)

```
User types: /finance-analyst "value MSFT using DCF"
    |
    v
finance-analyst command loads → references finance-analyst skill
    |
    v
Analyst skill overlay applied: valuation-focused framing
    |
    v
Code generation follows analyst conventions (DCF tables, sector context)
    |
    v
Output includes analyst-style summary (not just stats — interpretation)
```

## Skill File Format — Definitive Reference

### Command File Format (`.claude/commands/name.md`)

```markdown
---
description: Brief description shown in /help (under 60 chars)
argument-hint: [request]
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read
model: sonnet
---

[Prompt content — instructions for Claude]
[Use !`command` for dynamic context injection]
[Use $ARGUMENTS or $1 to reference user input]
[Use @path/to/file to inline file content]
```

**Key rules (sourced from frontmatter-reference.md on this machine):**
- `allowed-tools: Bash(python3:*)` — grants Python execution
- `allowed-tools: Bash(pip:*)` — grants package installation
- `!`command`` — runs command and injects output before Claude sees prompt
- `$ARGUMENTS` — full user input string after the slash command name
- `@file.md` — inlines file content into the prompt
- `model: sonnet` — appropriate for code generation tasks (haiku insufficient)

### Skill File Format (`.claude/skills/name/SKILL.md`)

```markdown
---
name: finance
description: Use when user asks for financial analysis, stock data, portfolio metrics,
             liquidity modeling, or investor classification. Activate for /finance command.
version: 1.0.0
---

[Skill knowledge content]
[Intent classification rules]
[Library usage guidance]
[Output format conventions]
[References to template files]
```

**Key rules (sourced from example-skill/SKILL.md and skills-reference.md):**
- Skills are loaded by Claude automatically based on `description` trigger matching
- Skills can reference supporting files via relative links: `[task-types.md](references/task-types.md)`
- Skills should not have `disable-model-invocation: true` unless side effects are dangerous
- `user-invocable: false` makes a skill background knowledge only (not invokable by user)

## Python Environment Handling

### Recommended Approach: Detect, Don't Assume

The command should inject the Python environment state before generating code:

```markdown
# In command file
Python check: !`python3 --version 2>&1`
Key packages: !`python3 -c "import yfinance, pandas, numpy, matplotlib, sklearn; print('OK')" 2>&1`
```

If packages are missing, Claude should:
1. Attempt `!pip install [package]` (if `Bash(pip:*)` is allowed)
2. If pip fails (permissions), instruct user to run `pip install -r requirements.txt` manually
3. Never silently continue with missing dependencies

### Virtual Environment Detection

```markdown
# Inject venv awareness
Venv: !`python3 -c "import sys; print('venv' if hasattr(sys, 'real_prefix') else 'system')" 2>&1`
```

If in a venv, use `python3`; if system Python, recommend venv creation as part of project setup.

### requirements.txt Convention

The skill repo should ship a `requirements.txt`:

```
yfinance>=0.2.0
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.0
scikit-learn>=1.3.0
```

The generalist command should detect if `requirements.txt` is present and reference it in setup instructions.

## Anti-Patterns

### Anti-Pattern 1: Inline Python in Bash for Long Scripts

**What people do:** `!python3 -c "import yfinance; df = yf.download('AAPL')..."` with full scripts inline.

**Why it's wrong:** Shell quoting breaks on apostrophes, newlines, and special characters in Python string literals. Scripts over 5 lines become unmaintainable.

**Do this instead:** Write the script to disk with the Write tool, then execute with Bash. Use `.finance-output/last_run.py` as the consistent temp file location.

### Anti-Pattern 2: One Monolithic Skill File

**What people do:** Put all three task types (stock, liquidity, investor), all three personas, and all library guidance into a single giant SKILL.md.

**Why it's wrong:** Exceeds effective context window for skill loading. Claude's attention degrades on very long instruction sets. Becomes unmaintainable.

**Do this instead:** Core SKILL.md stays under ~300 lines. Detailed task-specific logic goes in `references/task-types.md`, loaded via `@path` when needed. Persona overlays are separate skills.

### Anti-Pattern 3: Generating Code Without Environment Context

**What people do:** Let Claude generate Python code without knowing what packages are installed or what CSV files are available.

**Why it's wrong:** Claude hallucinates import paths, assumes packages like `yfinance` exist when they don't, or generates file paths that don't exist.

**Do this instead:** Always inject environment state with `!`command`` at the top of the command file. Claude uses this context to generate valid, runnable code.

### Anti-Pattern 4: Building Three Personas Before the Core Works

**What people do:** Immediately build analyst, PM, and generalist variants in parallel.

**Why it's wrong:** All three personas share the same Python code generation logic. Bugs in code generation affect all three. Fixing in three places is expensive.

**Do this instead:** Build the generalist `/finance` command first. When code generation is reliable, extract persona-specific framing as thin overlays.

### Anti-Pattern 5: Using `matplotlib.show()` in Generated Scripts

**What people do:** Generated Python scripts call `plt.show()` expecting an interactive window.

**Why it's wrong:** Claude Code runs in a terminal context. `plt.show()` either blocks or fails. No GUI is available.

**Do this instead:** Skill instructions must mandate `matplotlib.use('Agg')` and `plt.savefig('path.png')` in all generated code. The chart output convention is always file-based, never display-based.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| yfinance | `yf.download(ticker, period='6mo')` called in generated Python | No API key needed. Rate-limited. Wrap downloads in try/except |
| User CSV files | `pd.read_csv(path)` — path detected from `!ls *.csv` context injection | Support course-standard file names as defaults |
| matplotlib/seaborn | Headless backend (`Agg`). Save to `.finance-output/charts/` | Always enforce Agg backend in skill instructions |
| scikit-learn | Standard ML pipelines for regression (liquidity) and classification (investor) | Version-sensitive — skill should specify `sklearn.pipeline.Pipeline` pattern |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Command → Skill | Claude loads skill based on description match; command text references skill by name | Implicit activation or explicit "use the finance skill" reference in command |
| Skill → Reference files | `@path/to/references/task-types.md` inline in skill prompt | File inclusion at invocation time |
| Command → Python script | Write tool writes `.finance-output/last_run.py` | Consistent location for inspection and debugging |
| Python script → Output | stdout for tables/text; file writes for charts | Skill instructs Claude to always print table summaries to stdout |
| Output → User | Claude reads Bash output, synthesizes plain-English summary | Critical: Claude must interpret numbers, not just relay them |

## Build Order Implications

The architecture has a clear dependency graph that dictates phase ordering:

```
Phase 1: Core infrastructure
  ├── Python environment setup (requirements.txt)
  ├── Output directory conventions (.finance-output/)
  ├── Base SKILL.md (finance/SKILL.md) — intent classification
  └── Generalist command (/finance) — write-then-execute pattern

Phase 2: Task implementations (can be parallelized within phase)
  ├── Stock analysis workflow (yfinance + volatility + charts)
  ├── Data cleaning workflow (CSV + ML 01 pipeline)
  ├── Liquidity model (regression + ML 03 patterns)
  └── Investor classifier (classification + ML 05-06 patterns)

Phase 3: Persona variants
  ├── finance-analyst skill overlay + /finance-analyst command
  └── finance-pm skill overlay + /finance-pm command
```

**Why this order:**
- Environment and output conventions must exist before any task can run
- All task types share the write-then-execute pattern — build that once
- Intent classification in the base skill must be proven before adding persona framing on top
- Persona variants are thin layers over working task implementations — building them before tasks work creates rework

## Scaling Considerations

This skill is a single-user, local tool — scaling is not a concern in the traditional sense. The relevant "scaling" is increasing task type coverage:

| Phase | Scope | Architecture Adjustment |
|-------|-------|------------------------|
| Phase 1 (generalist) | 1 command, 1 skill, 3 task types | All task logic in one skill |
| Phase 2 (personas) | 3 commands, 3 skills | Persona skills reference base skill conventions |
| Phase 2+ (more tasks) | Additional task types | Add reference files; base SKILL.md stays stable |

The biggest scaling risk is SKILL.md growing too large. Mitigation: aggressively offload task-specific detail to `references/` files, keeping SKILL.md as a router with cross-links.

## Sources

- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md` — YAML frontmatter field definitions (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/examples/simple-commands.md` — Command patterns with Bash execution (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/claude-code-setup/skills/claude-automation-recommender/references/skills-reference.md` — Skill file format and invocation control (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/example-plugin/skills/example-skill/SKILL.md` — Skill frontmatter schema (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/advanced-workflows.md` — Multi-step workflow patterns, state management (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/interactive-commands.md` — AskUserQuestion tool, interactive patterns (HIGH confidence)
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/plugin-features-reference.md` — CLAUDE_PLUGIN_ROOT, namespaced commands (HIGH confidence)

---
*Architecture research for: Claude Code finance AI skill (slash command with Python execution)*
*Researched: 2026-03-17*
