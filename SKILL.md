---
name: auto-note
description: >
  Auto-write and maintain Obsidian notes strictly via the official Obsidian CLI
  (never raw filesystem edit of vault .md for note ops). Use whenever the user
  mentions notes, journaling, logging work, archiving, handoff, vault, knowledge
  base, or Obsidian—broad triggers below. Also use on /auto-note.
  Triggers (CN/EN, non-exhaustive): obsidian, Obsidian, vault, 笔记, 记笔记,
  写笔记, 写一段记录, 记一下, 记一笔, 记下来, 记录一下, 记录这次, 做个记录,
  写个记录, 工作记录, 过程记录, 操作记录, 实验记录, 会议记录, 会话记录,
  归档, 存档, 整理笔记, 更新笔记, 追加笔记, 补记, 备忘, 备忘录, 日记,
  日志, log, journal, note, notes, take notes, write a note, jot down,
  document this, document it, capture this, save to notes, put in notes,
  写到笔记, 写到obsidian, 写进库, 放进笔记, 同步到笔记, 沉淀, 复盘写入,
  交接, 交接文档, 状态更新, 更新状态, 目前情况, 导读, 知识库, second brain,
  双链, wiki, 知识管理, PKM, 记到库里, 帮我记, 帮我写进笔记, 帮我归档,
  记到 Obsidian, 用 Obsidian 记, auto-note, /auto-note.
version: 1.2.0
user-invocable: true
allowed-tools: Read, Bash, Grep, Glob
---

# auto-note — Obsidian CLI 强制笔记 Skill

## 0. 硬性约束（违反即失败）

1. **强制使用官方 Obsidian CLI** 完成一切笔记的创建 / 追加 / 列表 / 删除 / 移动等库内操作。  
   - 命令：`obsidian`（通常 `/usr/local/bin/obsidian` → App 内 `obsidian-cli`）  
   - **禁止**用 Write/Edit/`cat >`/`python open()` 等直接改 vault 内 `.md` 写笔记  
   - **禁止**安装或调用 npm 包 `obsidian-cli`  
2. Obsidian **桌面端必须在运行**。  
3. **AI 使用 Obsidian 时强制读取** **`AI笔记使用规范/AI使用笔记指南.md`**（及摘要 `AGENTS.md`），见 §1.3。  
4. **进入某项目时强制读取** 该项目 AI 指引（见 §1.4），再读 `00 导读` / `01 状态`；未完成读取链禁止写库。  
5. 有结论的笔记必须有短 **`## 成果总结`**（≤5 条）。  
6. **禁止**密钥/密码/Cookie 明文入笔记。  
7. **禁止写死库路径/库名**（如 `obsidian-vault`、臆造文件夹名）。必须以本机当前打开的 vault 为准（见 §1.1）。

CLI 不可用：启动 App → 等待 → 仍失败则停止，**不得**回退为直接改文件。

---

## 1. 触发后立即执行

### 1.1 解析 vault（唯一合法来源）

```bash
# 1) 优先：官方 CLI 列出的库
obsidian vaults verbose 2>/dev/null

# 2) 回退：obsidian.json 中 open=true 的 path
export VAULT="$(python3 -c '
import json
from pathlib import Path
p = Path.home() / "Library/Application Support/obsidian/obsidian.json"
if not p.exists():
    raise SystemExit("")
data = json.loads(p.read_text())
vaults = data.get("vaults") or {}
open_path, paths = None, []
for v in vaults.values():
    path = v.get("path") or ""
    if path:
        paths.append(path)
    if v.get("open") and path:
        open_path = path
print(open_path or (paths[0] if paths else ""))
')"
export VAULT_NAME="$(basename "$VAULT")"
echo "VAULT=$VAULT"
echo "VAULT_NAME=$VAULT_NAME"
```

规则：

- **不要**假设库名是 `obsidian-vault`、`Obsidian`、或任何未经验证的名字。  
- 多库时：用 `obsidian vaults` 的名称；写命令加 `vault="$VAULT_NAME"`（以 CLI 显示名为准）。  
- 解析失败：问用户，勿猜路径。

### 1.2 确保 CLI 与 App

```bash
command -v obsidian
pgrep -x Obsidian >/dev/null || open -a Obsidian
for i in $(seq 1 30); do
  obsidian version >/dev/null 2>&1 && break
  sleep 1
done
obsidian version
```

### 1.3 加载库规则（强制 · AI 使用 Obsidian）

```bash
# 一级目录 `AI笔记使用规范/` — 使用 Obsidian 时强制读取
obsidian read path="AI笔记使用规范/AI使用笔记指南.md" vault="$VAULT_NAME"
obsidian read path="AI笔记使用规范/AGENTS.md" vault="$VAULT_NAME"
```

`read` 不可用时，可用 Read 工具**只读**规则文件：

`$VAULT/AI笔记使用规范/AI使用笔记指南.md`  
（**仅读规则**；写库仍必须 CLI。）

### 1.4 项目 AI 指引（进入某项目时强制）

任务落在 `项目/<名>/` 或用户点名某项目时，在写库前按序尝试（命中即停）：

```bash
PROJ="<项目名>"
obsidian read path="项目/$PROJ/00 AI指引.md" vault="$VAULT_NAME" \
  || obsidian read path="项目/$PROJ/00 AI接入与项目导读.md" vault="$VAULT_NAME" \
  || obsidian read path="项目/$PROJ/AGENTS.md" vault="$VAULT_NAME" \
  || echo "WARN: 缺少项目 AI 指引，建议补建 00 AI指引.md"
obsidian read path="项目/$PROJ/00 导读.md" vault="$VAULT_NAME" 2>/dev/null || true
obsidian read path="项目/$PROJ/01 状态.md" vault="$VAULT_NAME" 2>/dev/null || true
```

然后用三句话复述：目标 · 可写范围 · 下一步，再执行。

---

## 2. 库结构与落点

| 意图 | 落点 |
|------|------|
| 库级规则 | `AI笔记使用规范/`（本 Skill 强制遵守处；**使用 Obsidian 必读**） |
| 临时一记 | `收件箱/` 或既有 `Ai对话记录交接文档/` |
| 有目标的工作 | `项目/<名>/` |
| 长期主题 | `领域/` 或兼容旧路径 `我的/...` |
| 可复用知识 | `资源/` |
| 已结束 | `归档/`（默认只读） |

**项目内最少：** `00 AI指引.md`（有 AI 协作则必须）· `00 导读.md` · `01 状态.md` · `02 记录/…`

---

## 3. 官方 CLI 用法（强制）

```bash
obsidian version
obsidian files
obsidian files folder="项目/某课题"
obsidian create path="项目/某课题/01 状态.md" content="# 标题\n\n## 成果总结\n\n- …\n"
obsidian append path="项目/某课题/01 状态.md" content="\n## 更新\n- …"
obsidian read path="项目/某课题/01 状态.md"
obsidian delete path="…"
# 多库时：
obsidian create path="…" vault="$VAULT_NAME"
```

**path 铁律**

- ✅ `path="AI笔记使用规范/AI使用笔记指南.md"` / `path="项目/Foo/01 状态.md"`（含 `.md` 的完整相对路径）  
- ❌ `name=` + 仅目录 `path=` 叠用  
- ❌ 绝对路径当 path  
- path 相对 **当前 vault 根**

**禁止**

| 禁止 | 原因 |
|------|------|
| 直接写 vault 内 `.md` | 绕过 CLI |
| `npm i -g obsidian-cli` | 假 CLI |
| 硬编码错误库名/路径 | 写错库 |
| 密钥明文 | 安全 |

---

## 4. 工作流

### A. 快速一记

判定路径 → CLI create/append → 成果总结 → 校验 → 回报相对路径。

### B. 状态更新

更新 `项目/<名>/01 状态.md`；可选 `02 记录/` 追加。

### C. 完整归档

`00 导读` + `01 状态` + `02 记录` + `交接文档.md`。

### D. 只读查询

仅 `files` / `read` / `search`，不写库。

意图不清默认 **A**。

---

## 5. 笔记体例

```markdown
# {短标题}

## 成果总结

- {结论}
- {产物路径}
- {下一步}

## 正文
…
```

---

## 6. 确认时机

覆盖导读/规范、写入归档、主题歧义 → 先问。路径明确的快速一记可直接写。

---

## 7. 完成后回复

```
已写入（Obsidian CLI）
- vault：{名称}（动态解析，非写死）
- 路径：{相对路径}
- 动作：create | append | update-status
- 成果总结：{一句话}
- 校验：ok / 失败原因
```

---

## 8. 故障排查

| 现象 | 处理 |
|------|------|
| command not found | 用 App 内官方 CLI；勿装 npm 假包 |
| 无响应 | `open -a Obsidian`，等 10–30s |
| 写到错误库 | 重跑 §1.1，核对 `vaults verbose` |
| create 后不可见 | path 是否含 `.md`；vault 名是否正确 |

---

## 9. 斜杠

`/auto-note`；同意图的「归档 / obsidian-record」等同样强制 CLI。

---

## 10. 自检

- [ ] 100% 经 `obsidian` CLI 写入？  
- [ ] vault 为动态解析结果？  
- [ ] 已强制读取 `AI笔记使用规范/AI使用笔记指南.md`？  
- [ ] 若进项目：已读项目 AI 指引 + 导读 + 状态？  
- [ ] 无密钥明文？  
- [ ] 有成果总结（需要时）？  
- [ ] 已校验并告知路径？  
