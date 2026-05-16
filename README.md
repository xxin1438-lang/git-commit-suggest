# git-commit-suggest

根据工作区未提交变更，自动拆分提交批次并生成可直接执行的 `git add` + `git commit` 定稿命令。

Automatically splits uncommitted changes into logical batches and generates ready-to-run `git add` + `git commit` commands.

---

## 目录 / Table of Contents

- [中文说明](#中文说明)
- [English](#english)

---

## 中文说明

### 解决什么问题

手动写 commit message 容易出现两个问题：

1. **写得太泛**：`fix bug`、`更新代码`、`update` 等无法表达实际意图的提交信息
2. **一锅粥**：把不相关的改动打成一个大 commit，难以 review 和回滚

这个 skill 读取当前工作区的变更，按语义逻辑拆批，每批给出可以直接粘贴执行的完整命令。标题和说明均为**定稿文案**，不使用占位符。

### 快速开始

```
/git-commit-suggest                          # 中文，读全量 diff
/git-commit-suggest en                       # 英文，读全量 diff
/git-commit-suggest <openspec-name>          # 中文，OpenSpec 模式（推荐）
/git-commit-suggest en <openspec-name>       # 英文，OpenSpec 模式
```

### 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ch` | 语言 | ✅ 默认 | commit message 使用中文 |
| `en` | 语言 | — | commit message 使用英文 |
| `<openspec-name>` | 字符串 | — | openspec 文件夹名，支持部分匹配 |

参数顺序不限，可任意组合。

### OpenSpec 模式（推荐）

当你的改动对应一个 OpenSpec 变更时，传入文件夹名即可启用高效模式：

```
/git-commit-suggest en flow-selection-token
```

**Skill 会做什么：**

1. 在 `openspec/changes/` 和 `openspec/changes/archive/` 中按名称模糊匹配文件夹
2. 读取 `proposal.md`（Why / What Changes / Impact）和 `tasks.md`（任务清单）
3. 仅运行 `git diff --stat` 了解文件范围，**不读完整 diff**
4. 按 `tasks.md` 中已完成（`[x]`）的任务分组文件，生成对应批次

> `tasks.md` 中未完成（`[ ]`）的任务不会出现在 commit 中。

**Token 对比：**

| 场景 | 全量 diff 模式 | OpenSpec 模式 |
|------|--------------|--------------|
| 50 个文件改动 | 数千 token（读所有 diff） | 数百 token（读 proposal + stat） |
| 语义准确度 | 依赖代码可读性 | 直接来自 spec，更准确 |

### 分批原则

1. **公共模块独立**：`genex-common-*` 独立成批，便于跨服务 review 与回滚
2. **单一意图**：一个功能切面一个 commit，避免大杂烩
3. **机械改动单独**：仅格式化、仅重命名等单独一批
4. **新增/删除文件**与对应逻辑同批；纯文档/配置可按 `docs`/`chore` 单批
5. **已暂存与未暂存分开说明**，不混淆暂存区

### Commit Message 规范

- **Subject**：50 字左右，动词开头，句末不加句号，遵循 Conventional Commits 前缀
- **Body（可选）**：第二条 `-m`，说明原因、影响面、调用方注意点

**中文示例：**

```
feat(batch-message)：新增按 flowId 查询 WhatsApp Flow JSON 接口
fix(business)：FLOW trace 改用 tenantId 与 flowToken 关联
chore(config)：新增 db-schema-diff 配置示例与 gitignore 规则
```

**禁止：** `xxx`、`待填写`、`...`、`fix bug`、`更新代码` 等无信息占位符。

### 输出格式示例

```
## 批次 1：flowToken → flowId 映射持久化

**范围**：在发送 Flow 模版/互动消息时建立映射并持久化，供 webhook 回调归因使用

**文件**：
- genex-business-service/.../FlowTokenMappingMapper.java
- genex-business-service/.../FlowTokenMappingService.java

**命令**：
```bash
git add genex-business-service/.../FlowTokenMappingMapper.java \
        genex-business-service/.../FlowTokenMappingService.java
git commit -m "feat(business)：新增 flowToken → flowId 映射持久化与 DAO 层" \
           -m "发送 Flow 消息时落库映射关系，webhook 回调缺少 flowId 时通过 flowToken 反查恢复"
```
```

---

## English

### What Problem It Solves

Manual commit messages often suffer from two issues:

1. **Too vague**: `fix bug`, `update code`, `changes` — convey no real intent
2. **Monolithic commits**: unrelated changes lumped together, hard to review or revert

This skill reads your working tree, splits changes into semantically coherent batches, and produces fully composed, ready-to-paste `git add` + `git commit` commands — no placeholders, no filler wording.

### Quick Start

```
/git-commit-suggest                          # Chinese (default), full diff
/git-commit-suggest en                       # English, full diff
/git-commit-suggest <openspec-name>          # Chinese, OpenSpec mode (recommended)
/git-commit-suggest en <openspec-name>       # English, OpenSpec mode
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ch` | language | ✅ default | commit messages in Chinese |
| `en` | language | — | commit messages in English |
| `<openspec-name>` | string | — | OpenSpec folder name, partial match supported |

Order doesn't matter; all combinations work.

### OpenSpec Mode (Recommended)

When your changes correspond to an OpenSpec change, pass the folder name to enable token-efficient mode:

```
/git-commit-suggest en flow-selection-token
```

**What the skill does:**

1. Locates the folder by partial name under `openspec/changes/` and `openspec/changes/archive/`
2. Reads `proposal.md` (Why / What Changes / Impact) and `tasks.md` (task checklist)
3. Runs only `git diff --stat` to see which files changed — **no full diff reading**
4. Groups files by completed tasks (`[x]`) from `tasks.md` and generates batches

> Incomplete tasks (`[ ]`) in `tasks.md` are excluded from the generated commits.

**Token comparison:**

| Scenario | Full diff mode | OpenSpec mode |
|----------|---------------|--------------|
| 50 changed files | Thousands of tokens (reads all diffs) | Hundreds of tokens (proposal + stat only) |
| Semantic accuracy | Depends on code readability | Comes directly from the spec |

### Batch Principles

1. **Isolate shared modules**: `genex-common-*` changes get their own batch for easier cross-service review and rollback
2. **Single intent per commit**: one feature slice, one refactor, one related bugfix set
3. **Mechanical changes separate**: format-only or rename-only changes go in their own batch
4. **New/deleted files** go with their corresponding logic; pure docs/config can be `docs`/`chore` batches
5. **Distinguish staged vs unstaged** to avoid polluting the staging area

### Commit Message Convention

- **Subject**: ~50 chars, starts with a verb, no trailing period, follows Conventional Commits prefixes
- **Body (optional)**: second `-m`, explains why, scope of impact, or caller-side notes

**English examples:**

```
feat(batch-message): add WhatsApp Flow JSON query API by flowId
fix(business): use tenantId and flowToken for FLOW trace attribution
chore(config): add db-schema-diff config examples and gitignore rules
```

**Never use:** `xxx`, `TBD`, `...`, `fix bug`, `update code`, or other uninformative placeholders.

### Output Format

```
## Batch 1: flowToken → flowId mapping persistence

**Scope**: Persist token-to-ID mapping when sending Flow messages for webhook attribution

**Files**:
- genex-business-service/.../FlowTokenMappingMapper.java
- genex-business-service/.../FlowTokenMappingService.java

**Commands**:
```bash
git add genex-business-service/.../FlowTokenMappingMapper.java \
        genex-business-service/.../FlowTokenMappingService.java
git commit -m "feat(business): add flowToken → flowId mapping persistence and DAO layer" \
           -m "Persists token mapping on Flow message send; webhook handler falls back to flowToken lookup when flowId is absent"
```
```

### Prerequisites

No installation needed — runs entirely on `git` commands and file reads available in any Claude Code session.
