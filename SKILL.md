---
name: git-commit-suggest
description: 根据工作区未提交变更拆分多批提交、产出可直接执行的 git add + git commit 定稿文案。支持 ch/en 语言参数与 openspec 文件夹名参数（读取 proposal.md + tasks.md 替代全量 diff，大幅节省 token）。在用户请求提交建议、拆分 commit、整理暂存区、或提到「分批提交」「commit 信息」「git commit suggestion」时使用。
---

# Git 提交分批与定稿 commit message

## 参数说明

调用格式：`/git-commit-suggest [ch|en] [openspec-name]`

| 参数 | 说明 | 示例 |
|------|------|------|
| `ch`（默认）| commit message 使用中文 | `/git-commit-suggest ch` |
| `en` | commit message 使用英文 | `/git-commit-suggest en` |
| `<openspec-name>` | openspec 文件夹名（支持部分匹配），读取 proposal + tasks 替代全量 diff | `/git-commit-suggest flow-selection-token` |

参数可组合：`/git-commit-suggest en flow-selection-token`

---

## 执行流程

### 有 openspec 参数时（推荐，节省 token）

1. **定位 openspec 文件夹**：在 `openspec/changes/` 和 `openspec/changes/archive/` 中按名称模糊匹配，找到对应目录。

2. **读取语义来源**（代替读大量 diff）：
   - `proposal.md`：Why / What Changes / Capabilities / Impact
   - `tasks.md`：任务清单与完成状态

3. **仅读文件范围**（不读完整 diff）：
   ```bash
   git status --short
   git diff --stat
   git diff --cached --stat
   ```

4. **按任务分组文件** → 生成提交批次。

### 无 openspec 参数时（全量读 diff）

1. 读变更范围：
   ```bash
   git status --short
   git diff --stat
   git diff --cached --stat
   ```
2. 对关键文件或不明确的变更补读 `git diff <path>` 了解语义。
3. 归类后生成提交批次。

---

## 分批原则（按优先级）

1. **不同关注点拆开**：公共模块（`genex-common-*`）与微服务实现独立成批，便于 review 与回滚。
2. **单一意图**：一个功能切面、一次重构、一组紧密 bugfix；避免大杂烩。
3. **机械性改动单独一批**：仅格式化、仅重命名等。
4. **新文件/删除文件**与对应逻辑同批；纯文档/配置可按 `docs`/`chore(config)` 单批。
5. **已暂存与未暂存分开说明**，避免误把无关文件打入同一提交。

---

## Commit message 规范

- **语言**：由 `ch`/`en` 参数决定；默认中文。
- **Subject**：50 字左右，动词开头，句末不加句号；可用 Conventional Commits 前缀，例如：
  - 中文：`feat(batch-message)：按 flowId 查询 WhatsApp Flow JSON`
  - 英文：`feat(batch-message): query WhatsApp Flow JSON by flowId`
- **Body（可选）**：第二条 `-m`，说明原因、影响面、调用方注意点；与 subject 同语言。
- **禁止**：`xxx`、`待填写`、`...`、`fix bug`、`更新代码` 等无信息占位符。

---

## 输出格式（必须遵守）

每批包含以下四要素，批次之间用分隔线，整段 bash 可直接复制执行：

```
## 批次 N：<简短主题>

**范围**：一句话说明本批语义

**文件**：
- path/to/file1
- path/to/file2

**命令**：
```bash
git add path/to/file1 path/to/file2
git commit -m "feat(module)：定稿中文或英文 subject"
# 需要 body 时追加：
# git commit --amend -m "feat(module)：subject" -m "body 说明"
```
```

批次数量以「逻辑可独立 review」为准，通常 2～6 批；变更极少时可合并为 1 批。

---

## 注意事项

- 不臆造 `git status` 中未出现的文件；路径以仓库根为准。
- 若存在密钥、token、本地环境文件，**单独一批标为「不建议提交」**并说明原因。
- 若用户指定「一条 commit 全提交」，合并为一批并说明取舍。
- openspec 的 `tasks.md` 中已标记 `[x]` 的任务对应本次变更，未标记 `[ ]` 的任务可能尚未实现，生成 commit 时不应包含。
