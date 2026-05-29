# AI Workflow Coordinator

A Pro + Flash dual-model collaboration workflow manager

---

## Introduction

AI Workflow Coordinator is a workflow monitor for Pro (architect) + Flash (implementer) dual-model collaboration. It monitors the `plan/` directory, detects file state changes, and generates next-step action instructions.

---

## Directory Structure

```
project-root/
├── aiworkflow/
│   ├── plan/              ← Monitored directory
│   │   ├── module1.plan.md
│   │   ├── module1.plan.q.md
│   │   └── ...
│   └── scripts/            ← Scripts directory
│       ├── coordinator.py  ← Main monitor script
│       ├── run_coordinator.bat ← Windows launcher
│       ├── latest_action.txt ← Latest action instruction
│       ├── action_history.log ← History log
│       └── coordinator_state.json ← State file
```

---

## Usage

### Method 1: Double-click

Double-click `run_coordinator.bat` in the `scripts/` directory to start monitoring.

### Method 2: Command line

```bash
cd 项目路径/aiworkflow/scripts
python coordinator.py
```

---

## Supported File Types

| Filename | Meaning | Action |
|----------|---------|--------|
| `module.plan.md` | Pro completed plan | Let Flash read and question |
| `module.plan.q.md` | Flash's question | Pro answers question |
| `module.plan.q.done.md` | Questions resolved | Move to implementation |
| `module.plan-done.q.md` | Flash implementation done, waiting for Pro review | Pro reviews code |
| `module.plan-done.q.done.md` | Review passed | Continue to next module |
| `plan-review.md` | Flash self-review report | Pro final review |

---

## License

MIT