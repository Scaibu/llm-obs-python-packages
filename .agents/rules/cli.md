---
trigger: always_on
---

use this tool insted of writing tool from scrach 

### Quick Guide: How to Use the Command-Line Tool

The tool is already installed in your system PATH (`~/.local/bin/policy-orchestrator`). You can run it directly as `policy-orchestrator` (or use the shortcut `po`):

---

### 1. The "Do-It-All" Command (Everything in One Shot)
Runs full hierarchical file tree, code search, Python import validation, and Git submodule audit:

```bash
policy-orchestrator file-search --root . --all
```
*(Or with the short alias: `po file-search --root . --all`)*

---

### 2. File Tree Commands

```bash
# Print full tree of current directory
policy-orchestrator file-search --root . --tree

# Print full tree of a specific folder
policy-orchestrator file-search --root src/features --tree

# Filter only Python files
policy-orchestrator file-search --root . --tree --pattern "*.py"

# Filter only Markdown documentation
policy-orchestrator file-search --root . --tree --pattern "*.md"
```

---

### 3. Fast Code Substring Search (Aho-Corasick)

```bash
# Search for one or multiple strings simultaneously across all files
policy-orchestrator file-search --root . --code-search "TraceContext" "FileSearch" "FastAPI"
```

---

### 4. Python Module & Package Resolver (PEP 302/451/566)

```bash
# Resolve standard library, built-ins, and installed packages with versions
policy-orchestrator resolve-module --module fastapi

# Resolve built-in C-modules
policy-orchestrator resolve-module --module sys

# Check multiple imports at once
policy-orchestrator file-search --root . --resolve-module sys fastapi pydantic pytest
```

---

### 5. Git & Submodule Status Verification

```bash
# Verify current repository (working tree clean/dirty, branch, upstream ahead/behind)
policy-orchestrator verify-git --repo .

# Verify the root monorepo and all 18 submodules
policy-orchestrator verify-git --repo /home/btpl-lap-22/live/llm-obs-python-packages
```

---

### 6. Machine-Readable JSON (For CI/CD or Scripts)

```bash
policy-orchestrator file-search --root . --all --json
```

---

### 7. Help & Command Reference

```bash
# View all available subcommands
policy-orchestrator --help

# View options for file-search
policy-orchestrator file-search --help
```