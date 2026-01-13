# Repurposing Codex as Research Brainstorming Agent

A minimalist but efficient long-running “research brainstorming loop” that uses **Codex CLI** as the main agent and **Claude Code** as an optional coding specialist.

Each cycle:

- reads your plan (`.md`) + prior cycle memory
- performs **proof-of-work** (commands, artifacts, or web-backed evidence)
- writes a new report: `reports/cycle_XX.md`
- appends one-line memory to: `reports/INDEX.md`

This design keeps context small (no giant chat history) while enabling sustained discovery across many cycles.

## What this tool does

- Turn a Markdown research plan into **iterative, non-duplicative** improvements
- Force deeper investigation (not just ideas) via **proof-of-work**
- Optionally delegate coding-heavy work to **Claude Code**
- Save external evidence and downloads in `data_lake/externals/`

## Requirements

### 1) Codex CLI

You need Codex installed and working in your terminal.

Recommended Codex configuration

Place (or update) this in ~/.codex/config.toml before running cycles:

```
model = "gpt-5.2-codex" #recommended
model_reasoning_effort = "xhigh" #recommended

approval_policy = "never" #Fully automated runs (no interactive “type yes” prompts)

sandbox_mode = "workspace-write" #Allow edits inside the workspace

[features]
web_search_request = true
unified_exec = true

[sandbox_workspace_write]
network_access = true #Allow network access so the agent can download external artifacts

[shell_environment_policy]
inherit = "all" #Inherit full env so tools like `claude` can see your auth/config
ignore_default_excludes = true
```

Notes  
•	approval_policy = "never" is what makes this loop fully hands-off.  
•	sandbox_mode = "workspace-write" + network_access = true enables downloading artifacts during cycles.  

### 2) Claude Code (optional but recommended)

You may need Claude installed and authenticated, because Codex may launch it from scripts.

⸻

Repository layout
```
.
├── discover.sh
├── plan.md
├── reports/
│   ├── INDEX.md
│   ├── cycle_01.md
│   ├── cycle_02.md
│   └── ...
├── data_lake/
│   └── externals/
│       └── README.md
└── code_lake/
└── (scripts / utilities generated during cycles)

```
•	reports/INDEX.md is compact memory to avoid duplicates.  
•	data_lake/externals/ stores web-backed artifacts (links, tables, small downloads).  
•	code_lake/ stores generated scripts/tools.  


⸻

Quick start

1. Put your plan markdown in the repo root (default: plan.md)

2.	Create directories:
```

mkdir -p reports data_lake/externals code_lake

```
3.	Run 6 cycles:
```

CYCLES=6 START=1 PLAN=plan.md ./discover.sh

```
4.	Run 6 more cycles without colliding with prior runs:
```

CYCLES=6 START=7 SKIP_EXISTING=1 PLAN=plan.md ./discover.sh

```
Auto-start at the next unused cycle number (optional)

If you add AUTO_START logic to your script, you can do:
```

AUTO_START=1 CYCLES=6 PLAN=plan.md ./discover.sh

```
⸻

How Claude is used (by design)

Codex is the primary agent. Claude is used only for coding-heavy tasks (parsers, preprocessors, benchmarks).
