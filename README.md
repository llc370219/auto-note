# auto-note

Obsidian 自动笔记 Skill：**强制官方 Obsidian CLI**；库规则在二级目录 **`规范/`**。

## 库路径

- **禁止**写死 `obsidian-vault` 或其它臆造库名。  
- 运行时从 `obsidian vaults` / `~/Library/Application Support/obsidian/obsidian.json` 解析**当前打开**的 vault。  
- 规则文件（相对 vault 根）：
  - `规范/AI笔记使用规范.md`
  - `规范/AGENTS.md`
  - `规范/CLAUDE.md`

## 安装

```bash
# Claude Code
git clone https://github.com/llc370219/auto-note.git ~/.claude/skills/auto-note

# Codex
git clone https://github.com/llc370219/auto-note.git ~/.codex/skills/auto-note

# Grok
git clone https://github.com/llc370219/auto-note.git ~/.grok/skills/auto-note
```

已安装时更新：

```bash
cd ~/.claude/skills/auto-note && git pull
# 同步到其它 tools 目录同理
```

## 使用

- `/auto-note`
- 自然语言：记一下、写一段记录、归档、更新状态、obsidian、notes…

## 前置

1. Obsidian 桌面端 + 官方 CLI  
2. 写库时 App 保持运行  
3. 库内存在 `规范/AI笔记使用规范.md`（推荐）

## 不要

- `npm install -g obsidian-cli`（会遮蔽官方命令）

## License

MIT
