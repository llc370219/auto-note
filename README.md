# auto-note

Obsidian 自动笔记 Skill：**强制使用官方 Obsidian CLI** 读写 vault，禁止直接改库内 `.md` 文件。

## 安装

### Claude Code / Codex

```bash
git clone https://github.com/llc370219/auto-note.git ~/.claude/skills/auto-note
# 或 Codex
git clone https://github.com/llc370219/auto-note.git ~/.codex/skills/auto-note
```

### Grok

```bash
git clone https://github.com/llc370219/auto-note.git ~/.grok/skills/auto-note
```

### 手动复制

将本仓库 `SKILL.md` 放到对应 skills 目录下的 `auto-note/SKILL.md`。

## 使用

- 斜杠：`/auto-note`
- 自然语言（会自动匹配 description 触发词），例如：
  - 「记一下」「写一段记录」「归档到笔记」
  - 「更新状态」「写到 Obsidian」
  - `obsidian` / `notes` / `document this`

## 前置

1. 已安装 Obsidian 桌面端（含官方 CLI，macOS 常见 `/usr/local/bin/obsidian`）
2. 写库时 **Obsidian 保持运行**
3. 库根建议有 `AI使用笔记指南.md`（可选但推荐）

## 不要

- 不要 `npm install -g obsidian-cli`（会遮蔽官方命令）

## License

MIT
