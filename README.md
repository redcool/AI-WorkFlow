# AI Workflow Coordinator

AI Workflow 项目管理与自动化监控脚本 / A Pro + Flash dual-model collaboration workflow manager

---

## 简介 / Introduction

AI Workflow Coordinaltor 是一个用于管理 Pro（架构师）与 Flash（实现者）双模型协作的工作流监控器。它监控 `plan/` 目录，自动检测文件状态变化并生成下一步行动指令。

AI Workflow Coordinator is a workflow monitor for Pro (architect) + Flash (implementer) dual-model collaboration. It monitors the `plan/` directory, detects file state changes, and generates next-step action instructions.

---

## 目录结构 / Directory Structure

```
项目根目录/
├── aiworkflow/
│   ├── plan/              ← 监控目标目录 / Monitored directory
│   │   ├── module1.plan.md
│   │   ├── module1.plan.q.md
│   │   ├── ...
│   └── scripts/            ← 脚本目录 / Scripts directory
│       ├── coordinator.py  ← 主监控脚本 / Main monitor script
│       ├── run_coordinator.bat ← Windows 启动脚本 / Windows launcher
│       ├── latest_action.txt ← 最新的行动指令 / Latest action instruction
│       ├── action_history.log ← 历史记录 / History log
│       └── coordinator_state.json ← 状态文件 / State file
```

---

## 使用方法 / Usage

### 方法一：双击运行 / Method 1: Double-click

在 `scripts/` 目录下双击 `run_coordinator.bat`，监控器将自动运行。

Double-click `run_coordinator.bat` in the `scripts/` directory to start monitoring.

### 方法二：命令行运行 / Method 2: Command line

```bash
cd 项目路径/aiworkflow/scripts
python coordinator.py
```

---

## 支持的文件类型 / Supported File Types

| 文件名 | 含义 | 行动 |
|--------|------|------|
| `module.plan.md` | Pro 完成的计划 | 让 Flash 阅读并提问 |
| `module.plan.q.md` | Flash 的提问 | Pro 回答问题 |
| `module.plan.q.done.md` | 提问已解决 | 进入实现阶段 |
| `module.plan-done.q.md` | Flash 实现完成，等待 Pro 审核 | Pro 审核代码 |
| `module.plan-done.q.done.md` | 审核通过 | 继续下一个模块 |
| `plan-review.md` | Flash 自审报告 | Pro 最终审核 |

---

## License

MIT