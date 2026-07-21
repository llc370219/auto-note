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
version: 1.0.0
user-invocable: true
allowed-tools: Read, Bash, Grep, Glob
---

# auto-note — Obsidian CLI 强制笔记 Skill

## 0. 硬性约束（违反即失败）

1. **强制使用官方 Obsidian CLI** 完成一切笔记的创建 / 追加 / 读取列表 / 删除 / 移动等库内操作。  
   - 命令：`obsidian`（通常为 `/usr/local/bin/obsidian` → Obsidian.app 内 `obsidian-cli`）  
   - **禁止**用 `Write`/`Edit`/`cat >`/`tee`/`python open()` 等直接改 vault 内 `.md` 来「写笔记」  
   - **禁止**安装或调用 npm 包 `obsidian-cli`（QA 工具，会遮蔽官方命令）  
2. Obsidian **桌面端必须在运行**；CLI 是遥控，不是无头库。  
3. 遵守库根 **`AI使用笔记指南.md`** 与 **`AGENTS.md`**（见 §2）。  
4. 有结论的笔记必须有短 **`## 成果总结`**（≤5 条）。  
5. **禁止**把 API Key、密码、Cookie 明文写入笔记。

若 CLI 不可用：先尝试启动 Obsidian 并等待 socket；仍失败则停止写库，向用户说明，**不得**回退为直接改文件。

---

## 1. 触发后立即执行

### 1.1 解析 vault

```bash
export VAULT="$(python3 -c '
import json
from pathlib import Path
p = Path.home() / "Library/Application Support/obsidian/obsidian.json"
data = json.loads(p.read_text())
vaults = data.get("vaults") or {}
# prefer open vault, else first
open_path = None
paths = []
for v in vaults.values():
    paths.append(v.get("path"))
    if v.get("open"):
        open_path = v.get("path")
print(open_path or (paths[0] if paths else ""))
')"
echo "VAULT=$VAULT"
```

若为空：询问用户 vault 路径，勿猜测错误盘符。

### 1.2 确保 CLI 与 App

```bash
command -v obsidian
# 若不是官方路径或 which 指向 npm，纠正：
# /usr/local/bin/obsidian 或 /Applications/Obsidian.app/Contents/MacOS/obsidian-cli

# App 未运行则拉起
pgrep -x Obsidian >/dev/null || open -a Obsidian
# 等待 CLI 可用（最多约 30s）
for i in $(seq 1 30); do
  obsidian version >/dev/null 2>&1 && break
  sleep 1
done
obsidian version
```

失败则报告，停止。

### 1.3 加载库规则（只读可用 CLI 或 Read 工具）

优先：

```bash
obsidian read path="AI使用笔记指南.md" 2>/dev/null || true
obsidian read path="AGENTS.md" 2>/dev/null || true
```

若 `read` 子命令不可用，可用 Read 工具读 `$VAULT/AI使用笔记指南.md`（**仅读规则**；写操作仍必须 CLI）。

---

## 2. 库结构与落点（与 AI使用笔记指南一致）

| 意图 | 落点 |
|------|------|
| 临时一笔、未定主题 | `收件箱/` 或既有 `Ai对话记录交接文档/`（若库尚未建收件箱，用后者并在成果总结注明） |
| 有目标的工作/课题 | `项目/<名>/` |
| 长期主题（工具、个人开发、作业） | `领域/` 或现有 `我的/...`（兼容旧路径，新内容优先新规范） |
| 可复用方法/模板 | `资源/` |
| 已结束 | `归档/`（默认只读，除非用户明确要求改） |

**项目内最少：**

| 文件 | 作用 |
|------|------|
| `00 导读.md` | 目标、范围、入口 |
| `01 状态.md` | 进度 / 阻塞 / 下一步 / 最近产物（常改） |
| `02 记录/YYYY-MM-DD 主题.md` | 过程记录 |

命名短、可扫；版本号写在正文，不堆进超长文件夹名。

---

## 3. 官方 CLI 用法（强制）

### 3.1 常用命令

```bash
# 版本 / 帮助
obsidian version
obsidian help

# 列表
obsidian files
obsidian files folder="项目/某课题"

# 创建（path 必须含 .md 的完整相对路径，最稳妥）
obsidian create path="项目/某课题/01 状态.md" content="# 标题\n\n## 成果总结\n\n- …\n"

# 追加
obsidian append path="项目/某课题/01 状态.md" content="\n## 更新\n- …"

# 读取（若支持）
obsidian read path="项目/某课题/01 状态.md"

# 删除（进废纸篓）
obsidian delete path="…"

# 多库
obsidian create path="…" vault="Vault名称"
```

### 3.2 path 铁律

- ✅ `path="项目/Foo/01 状态.md"` 一次写清文件路径  
- ❌ `name=` + 仅目录的 `path=` 叠用（易产生嵌套文件夹）  
- ❌ 绝对路径当 path  
- path 相对 **vault 根**

### 3.3 content 注意

- 多行：用 `$'...\n...'` 或先在 /tmp 拼好再传入时仍走 `obsidian create/append`  
- 转义引号，避免 shell 截断  
- 写入后立刻：`obsidian files` 或 `obsidian read` 校验

### 3.4 禁止清单

| 禁止 | 原因 |
|------|------|
| 直接写 `$VAULT/**/*.md` | 绕过索引/插件/CLI 约定 |
| `npm i -g obsidian-cli` | 假 CLI |
| 未开 App 时假装成功 | 误导用户 |
| 密钥明文 | 安全 |

---

## 4. 工作流（按用户意图选型）

### A. 快速一记（「写一段记录 / 记一下」）

1. 判定主题：用户指定路径 > 当前对话项目 > `收件箱/`  
2. CLI 创建或 append 一篇短笔记  
3. 文首 **成果总结**（哪怕 2～3 条）  
4. 校验 `files`/`read`  
5. 回复用户：相对路径 + 一句话结论  

### B. 项目状态更新（「更新状态 / 目前情况」）

1. 找到或创建 `项目/<名>/01 状态.md`  
2. 用 CLI 更新成果总结 + 状态表字段  
3. 可选：在 `02 记录/` 追加当日过程  

### C. 完整归档（「归档这次工作 / 交接」）

1. 确保 `00 导读`、`01 状态`  
2. `02 记录/` 写本次过程（参数、命令、报错原文、解法）  
3. 更新或创建 `交接文档.md`（路径、环境、下一步）  
4. 导读里补链接  
5. 全面校验  

### D. 只读查询（「笔记里有没有 / 找一下」）

1. 仅用 CLI `files` / `read` / `search`（若可用）  
2. 不写库  

意图不清时：默认 **A**，并在回复中说明落点；用户可纠正路径。

---

## 5. 笔记体例（写入 content 时遵守）

```markdown
# {短标题}

## 成果总结

- {结论 1}
- {产物路径}
- {下一步}（可选）

## 正文
…
```

过程类可增：参数表、命令、报错全文、数据位置。  
**成果总结永远在最前、尽量短。**

---

## 6. 与用户确认的时机

- 将覆盖已有正式导读/规范：先问  
- 写入 `归档/`：先问  
- 主题/项目名有歧义：先问  
- 快速一记且路径明确：**不必**多问，直接 CLI 写入并回报  

---

## 7. 完成后回复格式

```
已写入（Obsidian CLI）
- 路径：{vault 相对路径}
- 动作：create | append | update-status
- 成果总结：{一句话}
- 校验：ok / 失败原因
```

---

## 8. 故障排查

| 现象 | 处理 |
|------|------|
| `obsidian: command not found` | 查 App 内 CLI 路径并链到 PATH；勿装 npm 假包 |
| CLI 挂起/无响应 | `open -a Obsidian`，等 10～30s 重试 |
| create 后 files 不见 | 查 path 是否含 `.md`；查 vault 是否正确 |
| 权限/iCloud 延迟 | 重试 read；仍失败告知用户 |

---

## 9. 斜杠与别名

- 主命令：`/auto-note`  
- 同意图历史名：用户说 obsidian-record / 归档到库 等，同样走本 Skill，且 **仍强制 CLI**  

---

## 10. 自检清单（每次写库后默念）

- [ ] 是否 100% 通过 `obsidian` CLI 写入？  
- [ ] 是否未把密钥写入？  
- [ ] 是否有成果总结（需要时）？  
- [ ] 是否校验可见？  
- [ ] 是否告知用户路径？  
