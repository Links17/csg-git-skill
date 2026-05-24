# Git 版本与发布规范

本规范参考 [SemVer 2.0.0](https://semver.org/lang/zh-CN/) 与 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/) 制定，用于统一团队的版本号、分支、提交信息与发布流程，确保协作透明、升级平滑。

---

## 1. 概述

- 适用范围：团队所有需要版本发布的代码仓库（服务端、Desktop、SDK、库等）。
- 核心目标：
  - 版本号有明确语义，调用方可据此评估升级风险。
  - 分支职责清晰，发布过程可追溯、可回滚。
  - 提交信息结构化，便于自动生成 Changelog 与版本号。

---

## 2. 版本号规范（SemVer）

### 2.1 版本号格式

标准版本号采用 `X.Y.Z`：

```text
主版本号 (Major) . 次版本号 (Minor) . 修订号 (Patch)
```

| 位 | 含义 | 触发条件 | 兼容性 |
| --- | --- | --- | --- |
| X | 主版本号 | 不兼容的破坏性变更 | 不向下兼容 |
| Y | 次版本号 | 新增功能 | 向下兼容 |
| Z | 修订号 | 缺陷修复 | 向下兼容 |

### 2.2 主版本号 X — 破坏性变更

- 定义：API、数据结构或系统架构发生不兼容修改。
- 典型场景：
  - 数据库表结构变更导致旧代码无法运行。
  - 接口参数删减、更名或逻辑彻底重构。
  - 升级后需要手动迁移数据或调整配置文件。
- 规则：X 增加时，Y 和 Z 必须置 0。

### 2.3 次版本号 Y — 功能迭代

- 定义：新增功能且保持向下兼容。
- 典型场景：
  - 新增接口或功能模块，不影响已有功能。
  - 显著的性能优化或体验改进。
  - 标记旧接口为 `Deprecated`（弃用但仍可用）。
- 规则：Y 增加时，Z 必须置 0。

### 2.4 修订号 Z — 缺陷修复

- 定义：向下兼容的问题修复（Bug Fix / Hotfix）。
- 典型场景：
  - 修复生产环境 Bug。
  - 修复安全漏洞。
  - 不影响发布物的纯文档/注释更新不强制升 Z（按是否需要重新发布判定）。

### 2.5 预发布版本（Pre-release）

在正式发布前，通过连字符附加阶段标识，对应不同测试阶段：

| 标识 | 含义 | 部署环境 | 来源分支 | 示例 |
| --- | --- | --- | --- | --- |
| alpha | **内部提测版**，提交给 QA 测试 | 测试 | `dev` | `1.2.0-alpha.1` |
| beta | 公测版，功能基本完整，用于收集反馈 | 预发 / 灰度 | `release/*` | `1.2.0-beta.1` |
| rc | 发行候选版，无新问题即转正式版 | 预发 | `release/*` | `1.2.0-rc.1` |

> 同一目标版本 `X.Y.Z` 在提测阶段允许多次重测，序号递增（`alpha.1`、`alpha.2` …），无需占用新的 `X.Y.Z`。

---

## 3. 分支规范

采用简化版 Git Flow，长期分支 + 临时分支组合：

### 3.1 长期分支

| 分支 | 用途 | 部署环境 | 对应 Tag | 允许直接提交 |
| --- | --- | --- | --- | --- |
| `main` | 稳定分支，仅存放可发布代码，所有正式 Tag 打在此分支 | 生产 | `vX.Y.Z` / `vX.Y.Z-rc.N` | 否（仅合并） |
| `dev` | **提测分支**，合并后自动部署测试环境并打提测 Tag | 测试 | `vX.Y.Z-alpha.N` | 否（仅合并） |

### 3.2 临时分支

| 分支前缀 | 用途 | 来源分支 | 合并目标 |
| --- | --- | --- | --- |
| `feature` 或 `feat`/vX.Y.Z/* | 指定目标版本的新功能开发 | `dev` | `dev` |
| `fix/vX.Y.Z/*` | 指定目标版本的普通缺陷修复 | `dev` | `dev` |
| `perf/vX.Y.Z/*` | 指定目标版本的性能优化 | `dev` | `dev` |
| `refactor/vX.Y.Z/*` | 指定目标版本的重构 | `dev` | `dev` |
| `release/vX.Y.Z` | 发布准备（联调、回归、修小问题） | `dev` | `main` + `dev` |
| `hotfix/vX.Y.Z/*` | 生产紧急修复 | `main` | `main` + `dev` |

### 3.3 命名约定

- 全部使用小写，不使用大写。
- 功能、修复、优化类分支统一携带目标版本号：`<type>/vX.Y.Z/<description>`。
- **功能类前缀**：`feature` 或 `feat` 均可（`feat` 与 Conventional Commits 的 `feat` 对齐；`feature` 与 Git Flow 用词一致）。
- **描述段 `<description>`**：推荐用 `-` 连接（如 `app-common`），亦允许 `_`（如 `app_common`），同一仓库内建议统一一种风格。
- 推荐携带任务编号或简短描述，便于追溯。

示例：

```text
feature/v1.2.0/user-login
feat/v1.4.1/app-common
feat/v1.4.1/app_common
fix/v1.2.0/login-token-expire
perf/v1.2.0/list-query-cache
release/v1.2.0
hotfix/v1.1.3/payment-timeout
```

**合规正则（校验用）**：

```text
^(feat|feature|fix|perf|refactor|hotfix)/v\d+\.\d+\.\d+(?:/[a-z0-9][a-z0-9\-_]*)?$
```

> `feature` / `feat` + `vX.Y.Z/*` 合并到 `dev` 表示进入提测流程；在该 commit 上打出 `vX.Y.Z-alpha.N` Tag 并部署到测试环境后，才算正式产出本次提测版本（CI 自动或开发手动均可）。

---

## 4. 提交信息规范（Conventional Commits）

### 4.1 格式

```text
<type>(<scope>): <subject>

<body>

<footer>
```

- `type`：变更类型，见下表。
- `scope`：可选，影响范围（模块、文件、组件名）。
- `subject`：简短描述，建议不超过 50 字。
- `body`：可选，详细说明动机和实现要点。
- `footer`：可选，关联 issue 或破坏性变更说明。

### 4.2 type 取值

| type | 含义 | 影响版本号 |
| --- | --- | --- |
| `feat` | 新功能 | Minor (Y) |
| `fix` | 缺陷修复 | Patch (Z) |
| `perf` | 性能优化 | Patch (Z)，引入新能力时为 Minor |
| `refactor` | 重构（不影响功能） | 不升版本 |
| `docs` | 文档变更 | 不升版本（除非影响发布物） |
| `test` | 测试相关 | 不升版本 |
| `build` | 构建系统或依赖变更 | 视情况 |
| `ci` | CI/CD 配置变更 | 不升版本 |
| `chore` | 杂项（不影响源码与测试） | 不升版本 |
| `revert` | 回滚提交 | 视被回滚的提交 |

### 4.3 破坏性变更

通过以下任意方式声明，将触发 Major (X) 升级：

- 在 type/scope 后加 `!`：`feat(api)!: remove deprecated v1 endpoints`
- 在 footer 添加 `BREAKING CHANGE:` 段落，描述变更与迁移方式。

### 4.4 示例

```text
feat(auth): 支持飞书扫码登录

接入飞书开放平台 OAuth，新增 /api/auth/lark 接口。

Closes #128
```

```text
fix(order): 修复并发下单导致库存超卖

使用乐观锁替换原有事务隔离方案。
```

```text
feat(api)!: 移除 v1 用户接口

BREAKING CHANGE: /api/v1/user 已下线，请迁移至 /api/v2/user。
```

---

## 5. Tag 与发布

### 5.1 Tag 命名

- 正式版本：`vX.Y.Z`，如 `v1.0.2`，打在 `main`。
- 提测版本：`vX.Y.Z-alpha.N`，如 `v1.2.0-alpha.3`，打在 `dev`。
- 预发版本：`vX.Y.Z-beta.N` / `vX.Y.Z-rc.N`，打在 `release/*` 或 `main`。
- Tag 一旦推送禁止删除或覆盖，重测请递增序号。

### 5.2 创建 Tag

统一使用带注释的 annotated tag，便于记录发布信息：

```bash
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

### 5.3 Changelog

- 每次发布需要更新 `CHANGELOG.md`，按版本号倒序排列。
- 推荐基于 Conventional Commits 自动生成（如 `standard-version`、`release-please`、`semantic-release`）。

---

## 6. 发布流程

完整生命周期：**开发 → 提测（dev / alpha）→ 发布准备（release / rc）→ 生产（main / 正式 Tag）**。

```text
feature/*  --MR-->  dev  --tag-->  vX.Y.Z-alpha.N   (测试环境)
                     │
                     └──切出──> release/vX.Y.Z --tag--> vX.Y.Z-rc.N  (预发环境)
                                         │
                                         └──合并到 main --tag--> vX.Y.Z   (生产)
```

### 6.1 提测流程（合并到 `dev`）

**目的**：将完成自测的功能合并到 `dev`，自动部署到测试环境并打提测 Tag，交付 QA 测试。

#### 6.1.1 提测前置条件（MR Checklist）

提交合并到 `dev` 的 MR/PR 时，作者必须确认：

- [ ] 已基于最新 `dev` rebase / merge，无冲突。
- [ ] 完成开发自测，关键路径手工验证通过。
- [ ] 单元测试 / 集成测试本地已通过（如有）。
- [ ] 关联需求 / 任务卡（如 Jira / 飞书任务）已在 MR 描述中给出。
- [ ] 涉及配置、表结构、依赖变更已在 MR 描述说明。

> Reviewer Code Review 与 CI（lint / 测试 / 构建）不作为合并到 `dev` 的硬性门槛；若仓库已配置则按其结果流转，开发自行决定是否等候。

#### 6.1.2 提测操作步骤

1. 开发在 `feature/*`（或 `fix/*`、`perf/*` 等）完成自测后，提交 MR 合并到 `dev`。
2. 合并后部署到测试环境（CI 自动或人工触发均可）；如启用了 CI 检查（lint / 单测 / 构建），按其结果决定是否需要补救。
3. 在合入 `dev` 的 commit 上打 Tag `vX.Y.Z-alpha.N`：
   - `X.Y.Z` 为本次迭代计划发布的目标版本，由迭代负责人在 `package.json` / `VERSION` 文件中预先维护。
   - `N` 为该目标版本下的提测序号，按提测顺序递增（CI 自动或开发手动均可，重测时不可复用旧序号）。
4. 在 MR 评论或群里贴出提测 Tag（如 `v1.2.0-alpha.3`）通知 QA 开始测试。
5. QA 在测试环境按该 Tag 进行验证，所有 Bug 关联到该 Tag 提单。

#### 6.1.3 提测期发现问题

- 在原 `feature/*` 分支修复（不要直接在 `dev` 上改）。
- 修复后重新走 MR → 合并到 `dev` → 自动产出新的 `vX.Y.Z-alpha.(N+1)` → 通知 QA 复测。
- 不允许覆盖或删除旧的 alpha Tag，保留每次提测的可追溯快照。

#### 6.1.4 提测 Tag 命名建议

| 目标版本 | 第 1 次提测 | 重测 | 通过后转预发 |
| --- | --- | --- | --- |
| `1.2.0` | `v1.2.0-alpha.1` | `v1.2.0-alpha.2`、`v1.2.0-alpha.3` … | `v1.2.0-rc.1` |

### 6.2 常规发布流程（提测通过后）

1. QA 通过最终一版 `vX.Y.Z-alpha.N`，由迭代负责人从该 commit 切出 `release/vX.Y.Z`。
2. 在 `release/*` 完成预发回归、文档与 `CHANGELOG.md` 更新，必要时打 `vX.Y.Z-rc.N` 部署预发环境。
3. 预发验证通过后，合并 `release/*` 到 `main`，在 `main` 打 Tag `vX.Y.Z`，CI 触发生产部署。
4. 回合 `release/*` 到 `dev`，保持两条长期分支同步。
5. 删除 `release/*` 临时分支。

### 6.3 紧急修复（Hotfix）

1. 从 `main` 最新 Tag 切出 `hotfix/vX.Y.(Z+1)-xxx`。
2. 修复并自测后提 MR 合并到 `main`，打 Tag `vX.Y.(Z+1)`，触发生产部署。
3. 同步合并回 `dev`，避免修复在下个迭代丢失。
4. 紧急修复无需走完整提测流程，但必须在 MR 描述中说明修复方案、影响范围与自测结论；Reviewer / CI 由仓库自行决定是否启用。

### 6.4 版本升级示例

| 变更类型 | 版本变化 |
| --- | --- |
| 修复 Bug | `0.0.1 → 0.0.2` |
| 新增功能 | `0.0.2 → 0.1.0` |
| 不兼容更新 / 架构重构 | `0.1.0 → 1.0.0` |
| 一次完整迭代 | `1.2.0-alpha.1 → 1.2.0-alpha.2 → 1.2.0-rc.1 → 1.2.0` |

---

## 7. 约束与建议

- 禁止向 `main`、`dev` 直接 push，必须通过 Merge Request / Pull Request。
- 合并到 `dev` 后触发测试环境部署，并在合入 commit 上打 `vX.Y.Z-alpha.N` Tag（CI 自动或开发手动均可）。
- 一个 PR 聚焦单一目标，提交信息符合本规范。
- 任何破坏性变更必须在 PR 描述与 Changelog 中显式说明迁移方式。
- 提测 Tag（`*-alpha.*`）与正式 Tag（`vX.Y.Z`）一旦推送均禁止删除或覆盖。
- 目标版本号（`X.Y.Z`）由迭代负责人在迭代启动时确定并写入版本文件；alpha 序号按提测顺序递增（CI 自动或开发手动）。
- Reviewer 与 CI **不是**硬性门槛；仓库自行决定是否启用，启用后建议遵守对应仓库的合并策略。
