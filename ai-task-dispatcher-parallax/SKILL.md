---
name: ai-task-dispatcher-parallax
description: |
  AI Workflow - Pro(架构师)+ Flash(实现者)并行协作工作流。
  通过文档驱动通信,支持计划阶段对齐和执行阶段审核,避免返工。
  触发词:「规划」/「plan」→ Pro,「执行」/「做」/「execute」/「do」→ Flash。
triggers:
  - "规划"
  - "plan"
  - "架构"
  - "设计"
  - "执行"
  - "做"
  - "execute"
  - "do"
  - "实现"
  - "写代码"
  - "aiworkflow"
od:
  mode: prototype
  surface: code
  platform: desktop
  scenario: coding
---

# AI Workflow: Pro + Flash 并行协作协议（多 Worker 版）

## 核心理念

**文档驱动通信** — Pro 和 Flash 通过文件通信，天然解决上下文同步问题
**多 Worker 并行** — 支持 n 个 Pro + n 个 Flash 同时工作
**职责锁定** — 通过锁文件避免任务冲突
**两阶段对齐** — 执行前对齐 plan + 执行后对齐实现，返工风险大幅降低
**质量把关** — Flash 不能自己说 done，必须经过 Pro 审核

---

## 角色定义

### Pro(架构师 / DeepSeek-V4-Pro)

- 项目分析与架构设计
- 写架构代码:类型定义、接口、模块边界(**方法体只有签名+算法描述注释**)
- **产出模块计划文档** `{module}.plan.md`
- 回答 Flash 的 question,调整 plan
- **规划测试工程** → 输出 `TEST_PLAN.md`
- 审核实现代码,写 question `{module}.plan-done.q.md`
- 有最终决定权(分歧超过3轮时)

### Flash(实现者 / DeepSeek-V4-Flash)

- 读取 plan 文档,提出疑问 → 写 `{module}.plan.q.md`
- 确认 Pro 对 question 的回复,标记「已明确」
- **按 plan 实现方法体**(不改变签名)
- 执行测试,查看测试报告
- 根据测试报告修 Bug(最多3轮自动修复,超过上报 Pro)
- 实现测试代码(按 Pro 的 `TEST_PLAN.md`)
- 代码通过后写 `{module}.plan-done.md`(请求 Pro 审核)

---

## 文档命名规范

所有文档存放在 `aiworkflow/plan/` 目录:

```
plan/
  {module}.plan.md               # Pro 产出:模块计划(接口+算法描述)
  {module}.plan.q.md             # Flash 产出:对 plan 的疑问
  {module}.plan.q.done.md        # 疑问已解决(Flash 确认 Pro 回复后改名)
  {module}.plan-review.md        # Flash 产出:实现完成,请求 Pro 审核
  {module}.plan-done.q.md       # Pro 产出:审核实现代码发现的问题
  {module}.plan-done.q.done.md  # 审核全部通过(Flash 修复后 Pro 确认)
  TEST_PLAN.md                   # Pro 产出:整体测试规划
```

**标记规范**(在文档内用标题或粗体标记):
- `**[已回复]**` - Pro 已回复该条 question
- `**[已明确]**` - Flash 确认 Pro 回复,该条关闭
- `**[Pro决定]**` - 分歧超限,Pro 最终决定,强制推进
- `**[通过]**` - Pro 审核通过该项

---

## 并行工作流(修订版)

```
┌─────────────────────────────────────────────────────┐
│  阶段一:计划对齐(Plan Alignment)                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Pro 做计划,写框架代码(接口,无方法体)        │
│     → 产出 {module}.plan.md                        │
│     → 标注模块依赖顺序                              │
│                                                     │
│  2. Flash 读取 plan.md,有疑问写 question           │
│     → 产出 {module}.plan.q.md                      │
│     → 通知 Pro:「有 question 待回复」              │
│                                                     │
│  3. Pro 逐条回复 question,调整 plan                │
│     → 在 {module}.plan.q.md 原地更新,标记「已回复」│
│     → 必要时更新 {module}.plan.md                  │
│     → 通知 Flash:「question 已回复」                │
│                                                     │
│  4. Flash 确认 Pro 回复                            │
│     → 已明确的条目标记「已明确」                    │
│     → 全部确认 → 改名 {module}.plan.q.done.md      │
│                                                     │
│  5. 重复 2-4,最多 3 轮                          │
│     → 第3轮仍分歧 → Pro 标记「Pro决定」,强制推进   │
│                                                     │
└─────────────────────────────────────────────────────┘
                    ↓ 全部模块 plan.q.done.md 后进入阶段二
┌─────────────────────────────────────────────────────┐
│  阶段二:实现 + 测试(Implementation)               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  6. Pro 规划测试工程                                │
│     → 产出 TEST_PLAN.md(测试策略+用例设计)         │
│                                                     │
│  7. Flash 按 plan 实现代码 + 实现测试代码           │
│     → 按 {module}.plan.md 的签名和算法描述填代码    │
│     → 按 TEST_PLAN.md 编写测试用例                 │
│     → 执行测试,查看报告                            │
│     → 根据报告修 Bug(最多3轮自动修复)             │
│                                                     │
│  8. Flash 测试全部通过后,写审核请求                │
│     → 产出 {module}.plan-review.md                  │
│     → 通知 Pro:「实现完成,请求审核」               │
│                                                     │
└─────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────┐
│  阶段三:审核闭环(Review Loop)                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  9. Pro 审核实现代码                                │
│     → 没问题:标记「通过」                          │
│     → 有问题:写 {module}.plan-done.q.md            │
│     → 通知 Flash:「有审核问题待修复」                │
│                                                     │
│ 10. Flash 按 question 修复代码                      │
│     → 修复后通知 Pro:「已修复,请重新审核」         │
│                                                     │
│ 11. 重复 9-10,直到全通过                         │
│     → 改名 {module}.plan-done.q.done.md            │
│     → 模块完成 ✅                                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 模块依赖排序

Pro 在 `{module}.plan.md` 头部必须标注依赖顺序:

```markdown
# Module: auth

**依赖**: 无(可立即实现)
**执行顺序**: 1(最先实现)
```

```markdown
# Module: api

**依赖**: auth(必须先实现 auth)
**执行顺序**: 2(等 auth 的 plan.q.done.md 后再实现)
```

Flash **严格遵守执行顺序**,不实现依赖未就绪的模块。

---

## 分歧处理规则

| 轮次 | 处理方式 |
|-------|----------|
| 第1轮 | Flash 提问 → Pro 回复 → Flash 确认 |
| 第2轮 | 仍有分歧 → Pro 重新解释设计理由 |
| 第3轮 | 仍分歧 → Pro 标记 `**[Pro决定]**`,Flash 必须按 Pro 方案执行 |
| 实现审核 | 同理,最多3轮 → Pro 最终决定 |

---

## 与串行模式对比

| | 串行模式 | 并行工作流(本文档) |
|--|---------|---------------------|
| 通信方式 | 对话 | **文档驱动** |
| 返工风险 | 高(Pro 全设计完 Flash 才发现问) | **低(阶段一已对齐)** |
| 质量把关 | Flash 自测自说 done | **Pro 审核后才 done** |
| 循环上限 | 无 | **3轮 + Pro最终决定** |
| 适用场景 | 小项目(<3模块) | **大项目(>5模块),模块边界清晰** |

---

## 快速参考:文件清单

| 文件 | 写作者 | 读者 | 用途 |
|------|--------|------|------|
| `{module}.plan.md` | Pro | Flash | 模块计划(接口+算法) |
| `{module}.plan.q.md` | Flash | Pro | 对 plan 的疑问 |
| `{module}.plan.q.done.md` | Flash | - | 疑问全部解决 |
| `{module}.plan-review.md` | Flash | Pro | 实现完成,请求审核 |
| `{module}.plan-done.q.md` | Pro | Flash | 审核问题 |
| `{module}.plan-done.q.done.md` | Pro | - | 审核全部通过 |
| `TEST_PLAN.md` | Pro | Flash | 测试规划 |

---

## 触发词路由

| 用户说 | 路由到 | 角色 |
|--------|--------|------|
| 规划 / plan / 架构 / 设计 | DeepSeek-V4-Pro | Pro(架构师) |
| 执行 / 做 / execute / do / 实现 / 写代码 | DeepSeek-V4-Flash | Flash(实现者) |

**多轮对话**:自动保持角色一致性(规划阶段用 Pro,执行阶段用 Flash)。

---

## 多 Worker 并行协议

### 目录结构

```
aiworkflow/
├── plan/                    ← 任务目录
│   ├── {module}.plan.md    ← Pro 发布的任务
│   ├── pros/                ← Pro 身份注册目录
│   │   ├── pro1.identity   ← 身份注册文件(内容为空或含元数据)
│   │   └── pro2.identity
│   └── flashs/               ← Flash 身份 + 锁文件目录
│       ├── flash1.identity  ← Flash1 身份注册
│       ├── network.flash1.lock  ← Flash1 认领 network 任务
│       └── auth.flash2.lock   ← Flash2 认领 auth 任务
```

### 身份注册

**第一步:确认身份**

AI 开始工作时，被问到身份（如「你是谁」），回答自己是 Pro 还是 Flash:

```
我是架构师 Pro，我的 ID 是 pro1
```

Coordinator 会在 `plan/pros/` 或 `plan/flashs/` 目录下查找已有的身份文件，空闲 ID 即为当前 ID:

```python
# 获取空闲 ID
def get_free_id(role_dir, prefix):
    existing = sorted([f for f in os.listdir(role_dir) if f.startswith(prefix)])
    if not existing:
        return 1
    # 取最大编号 +1
    ids = [int(f.replace(prefix, '').replace('.identity', '')) for f in existing if f.endswith('.identity')]
    return max(ids) + 1 if ids else 1
```

**身份文件**: 空文件或含元数据:
```
pro1.identity 内容(可选):
id: pro1
role: architect
created: 2026-05-29 17:00:00
last_active: 2026-05-29 18:26:00
```

### 任务认领规则

#### Pro 发布任务

Pro 完成 `{module}.plan.md` 后:
1. 任务状态默认为 **pending**
2. 可被任何 Flash 认领

#### Flash 认领任务

Flash 需先检查任务是否已被认领:

```python
# 1. 检查是否有锁文件(表示已有人认领)
def is_claimed(module_name, flashs_dir):
    locks = [f for f in os.listdir(flashs_dir) 
            if f.startswith(module_name + '.') and f.endswith('.lock')]
    return len(locks) > 0

# 2. 认领时创建锁文件
def claim_task(worker_id, module_name, flashs_dir):
    lock_file = f"{flashs_dir}/{module_name}.{worker_id}.lock"
    with open(lock_file, 'w') as f:
        f.write(datetime.now().isoformat())  # 带时间戳，防死锁
    return True
```

**锁文件命名**: `{module}.{worker_id}.lock`
- 任务名在前，便于按任务 glob 查找
- 内容为时间戳，超过 2 小时可视为过期，清理后可重新认领

#### Flash 完成任务

任务完成后，删除锁文件:
```python
def release_task(worker_id, module_name, flashs_dir):
    lock_file = f"{flashs_dir}/{module_name}.{worker_id}.lock"
    if os.path.exists(lock_file):
        os.remove(lock_file)
    return True
```

### 回答带身份签名

AI 回答时必须在文档末尾带身份签名:

```markdown
# Pro1: network 模块设计

## 接口
...

-- Pro1
```

```markdown
# network 模块实现

```python
def authenticate():
    ...

-- Flash1
```

Coordinator 根据签名中的 `Pro1`、`Flash2` 等前缀:
1. 分发给对应的 Worker
2. 更新该 Worker 的 `last_active` 时间戳

### 任务生命周期

```
{module}.plan.md (发布) 
    → {module}.network.{flash1}.lock (认领)
    → {module}.plan-review.md (实现完成)
    → {module}.plan-done.q.md (审核问题)
    → {module}.plan-done.q.done.md (审核通过)
    → 删除锁文件 (完成)
```

### Coordinator 指令生成

Coordinator 根据锁文件和 plan 名生成下一步指令:

```python
def generate_instruction(plan_name, lock_files):
    # 1. 找到任务负责人
    for lock in lock_files:
        if lock.startswith(plan_name + '.'):
            worker = lock.split('.')[1]  # flash1, pro2 等
            return f"{worker} 继续 {plan_name} 任务"
    
    # 2. 无锁文件，说明是新任务或任务空闲
    return f"开放任务 {plan_name}，谁有空可以认领"
```


### 防僵死机制(可选)


锁文件带时间戳，超过 2 小时可清理:

```python
import timedelta

def is_lock_expired(lock_file, hours=2):
    with open(lock_file) as f:
        timestamp = datetime.fromisoformat(f.read().strip())
    return datetime.now() - timestamp > timedelta(hours=hours)

# 清理过期锁
flocks = [f for f in os.listdir('plan/flashs') if f.endswith('.lock')]
for f in flocks:
    if is_lock_expired(f'plan/flashs/{f}'):
        os.remove(f'plan/flashs/{f}')
```

