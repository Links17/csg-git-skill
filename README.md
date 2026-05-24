# csg-git-skill

面向 AI 编码助手的 **Git 版本与发布规范** Skill：SemVer 2.0.0、Conventional Commits、简化 Git Flow，含 `dev` 提测 `alpha` Tag 管控。

- 速查：[`SKILL.md`](SKILL.md)
- 完整规范：[`reference.md`](reference.md)
- 仓库：<https://github.com/Links17/csg-git-skill>

## 支持平台

通过 [npx skills](https://github.com/vercel-labs/skills) 安装到：

| 平台 | `--agent` | 全局路径（`-g`） |
| --- | --- | --- |
| Cursor | `cursor` | `~/.cursor/skills/csg-git-skill/` |
| Claude Code | `claude-code` | `~/.claude/skills/csg-git-skill/` |
| Codex | `codex` | `~/.codex/skills/csg-git-skill/` |

## 安装

需要 Node.js（可执行 `npx`）。

### 三平台一键安装（全局，推荐）

```bash
npx skills add Links17/csg-git-skill -g -a cursor -a claude-code -a codex -y
```

安装到本机已检测到的全部 Agent：

```bash
npx skills add Links17/csg-git-skill -g -a '*' -y
```

### 按平台单独安装

```bash
npx skills add Links17/csg-git-skill -g -a cursor -y
npx skills add Links17/csg-git-skill -g -a claude-code -y
npx skills add Links17/csg-git-skill -g -a codex -y
```

### 项目级安装（提交到仓库，团队共享）

在项目根目录执行（去掉 `-g`）：

```bash
npx skills add Links17/csg-git-skill -a cursor -a claude-code -a codex -y
```

### 从本地克隆安装（开发调试）

```bash
git clone https://github.com/Links17/csg-git-skill.git
cd csg-git-skill
npx skills add . -g -a cursor -a claude-code -a codex -y
```

### 其他选项

- 预览仓库内 skill：`npx skills add Links17/csg-git-skill --list`
- 复制而非符号链接（如 Windows symlink 受限）：加上 `--copy`
- 完整 URL：`npx skills add https://github.com/Links17/csg-git-skill -g -y`

## 验证

```bash
npx skills list -g
```

确认 `csg-git-skill` 已出现在 Cursor / Claude Code / Codex 对应目录后，**重启或新开** Agent 会话。

- **Cursor**：Agent 聊天输入 `/csg-git-skill`，或描述「提测 alpha tag」「分支命名」等
- **Claude Code**：`/csg-git-skill` 或自然语言触发
- **Codex**：`$csg-git-skill` 或匹配 `description` 的任务

## 更新与卸载

```bash
# 更新
npx skills update csg-git-skill -g -y

# 卸载（全部已安装 Agent）
npx skills remove csg-git-skill -g -a '*' -y
```

## 规范概要

| 主题 | 要点 |
| --- | --- |
| 版本号 | `X.Y.Z`；提测 `vX.Y.Z-alpha.N`，发布 `vX.Y.Z` |
| 长期分支 | `main`（生产）、`dev`（提测）；禁止直接 push |
| 临时分支 | `feature/vX.Y.Z/*`、`fix/*`、`release/vX.Y.Z`、`hotfix/*` 等 |
| 提交信息 | `<type>(<scope>): <subject>`；破坏性变更用 `!` 或 `BREAKING CHANGE:` |

细节见 [`SKILL.md`](SKILL.md) 与 [`reference.md`](reference.md)。

## 许可

与仓库默认许可一致；Skill 内容可随团队规范自行 fork 修改。
