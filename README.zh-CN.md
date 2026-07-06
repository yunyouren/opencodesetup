# OpenCode Setup Assistant

[English README](README.md)

这个仓库把 Anthropic Claude Code Setup 插件的核心思路迁移成了 **OpenCode 原生、默认只读** 的配置推荐器。

它的目标很简单:分析一个仓库里真实存在的信号,然后推荐最小、最稳妥的 OpenCode 配置,而不是自动改用户文件,也不是自动启用高风险 MCP 或插件。

## 它能做什么

当前推荐范围包括:

- `opencode.jsonc` 基线配置
- `AGENTS.md` 项目说明
- `.opencode/skills/*/SKILL.md`
- `.opencode/commands/*.md`
- `.opencode/agents/*.md`
- `opencode.jsonc` 里的 `mcp` 片段
- `opencode.jsonc` 里的 `permission` 片段
- `.opencode/plugins/*.ts` 仅在确有必要时建议,且必须附带风险提示

默认原则:

- **只输出建议,不自动写配置**
- **不自动启用 MCP**
- **不自动创建/启用 plugin**
- **先看真实信号,再做最小推荐**

## 当前能力

仓库当前提供:

- `/setup`：分析当前仓库并输出 OpenCode 配置建议
- `/setup-eval <fixture-path>`：用 fixture + checklist 做只读回归评估
- `setup-architect`：权限锁定的只读分析 subagent
- `opencode-automation-recommender`：推荐器核心 skill 与参考表

推荐输出分四层:

- `Baseline`：缺失的基础配置,例如 `opencode.jsonc`、`AGENTS.md`、保护性 permission
- `High confidence`：由明确仓库信号强支撑的推荐
- `Optional`：可选增强,有用但不是必需
- `Skipped`：已存在、风险过高、或没有真实信号支持的项

## 快速开始

### 1. 重启 OpenCode

这个项目使用项目级 OpenCode 配置。修改 `opencode.jsonc`、`AGENTS.md` 或 `.opencode/` 后,需要重启 OpenCode 才会重新加载。

### 2. 运行推荐器

在 OpenCode 中直接执行:

```text
/setup
```

也可以只看某一类:

```text
/setup mcp
/setup skills
/setup permissions
```

### 3. 手动审查 snippets

推荐器只打印建议。你确认后再把片段粘贴到目标仓库。

### 4. 查看本仓库自己的基线配置

这个仓库已经带了最小 root `opencode.jsonc` 和 `AGENTS.md`,目的是让项目自己也遵守它推荐给别人的基线:

- 默认不启用任何 MCP
- 默认不启用任何 plugin
- 对 OpenCode 自动脚手架文件做 deny/ask 保护
- 把 `claude-plugins-official/` 当只读上游参考

## 仓库结构

```text
.opencode/
  agents/
    setup-architect.md
  commands/
    setup.md
    setup-eval.md
  skills/
    opencode-automation-recommender/
      SKILL.md
      references/
docs/
  evals/
    setup-recommender-checklist.md
fixtures/
  empty-opencode/
  frontend-react/
  python-api/
  claude-plugin-source/
  existing-opencode-config/
opencode.jsonc
AGENTS.md
```

## 评估方式

这个仓库自带 fixtures 和 checklist,用来做回归验证,避免在真实项目上试错。

可运行:

```text
/setup-eval fixtures/empty-opencode
/setup-eval fixtures/frontend-react
/setup-eval fixtures/python-api
/setup-eval fixtures/claude-plugin-source
/setup-eval fixtures/existing-opencode-config
```

每次评估都应保持只读,并输出:

- fixture profile
- pass/fail expectations
- over-recommendations
- missing recommendations
- safety/schema issues
- failure 的源文件行号证据

## 安全边界

当前采用保守策略:

- 不读、不打印 `.env*`、`*.key`、`*.pem`、`credentials*`、`secrets*`、`id_rsa*`
- 不自动启用 MCP 或 plugin
- 本地 MCP 片段必须使用 `environment`,不能使用 `env`
- MCP snippets 默认 `"enabled": false`
- `opencode.jsonc` starter 必须带 `"$schema": "https://opencode.ai/config.json"`
- permission 示例采用 last-match-wins: 广泛 `*` 在前,具体规则在后
- `.opencode/package.json`、`.opencode/package-lock.json`、`.opencode/node_modules/`、`.opencode/.gitignore` 都视为 OpenCode 自动脚手架,不是用户意图

## 给维护者的说明

修改推荐器行为时,至少要做这些事:

1. 更新 `.opencode/skills/opencode-automation-recommender/` 下的 skill 或 references
2. 保持以下文件规则一致:
   - `.opencode/skills/opencode-automation-recommender/SKILL.md`
   - `.opencode/commands/setup.md`
   - `.opencode/agents/setup-architect.md`
   - `docs/evals/setup-recommender-checklist.md`
3. 重新跑 `/setup-eval` 覆盖所有 fixtures
4. 尽量在加新推荐规则前,先补一个 fixture

建议后续补充的 fixture:

- 真实 `.opencode/plugins/*.ts` plugin fixture
- docs-heavy repo fixture
- monorepo fixture
- security-sensitive fixture

## 上游参考

`claude-plugins-official/` 是只读上游 clone,用于参考原始 Claude Code Setup 插件思路,不会纳入本仓库历史。
