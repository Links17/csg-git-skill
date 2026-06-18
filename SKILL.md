---
name: csg-git-skill
description: Git 版本号、分支命名、提交信息与提测/发布/Hotfix 流程规范，基于 SemVer 2.0.0 + Conventional Commits + 简化 Git Flow，包含 dev 提测 alpha Tag 管控与 CSG 提测文档/Base 流程。当用户要为仓库制定 git 规范、判断版本号应升 X/Y/Z 哪一位、给分支取名（feature/fix/hotfix/release）、写 commit message、走提测流程合并到 dev、创建提测文档、写入提测 Base、创建服务子记录、通知测试负责人、走发布流程合到 main、打 Tag、生成 Changelog 时使用。
---

# Git 版本与发布规范

按 SemVer + Conventional Commits + 简化 Git Flow 管控版本、分支、提交与发布。完整规范见 [reference.md](reference.md)，本文件仅给出决策速查与硬性约束。

## 1. 版本号决策（SemVer `X.Y.Z`）

| 变更性质 | 升哪一位 | 例 |
| --- | --- | --- |
| 不兼容（删/改接口、表结构、需迁移） | X，Y/Z 清零 | `1.4.2 → 2.0.0` |
| 向下兼容的新功能 / 显著优化 / 标 Deprecated | Y，Z 清零 | `1.4.2 → 1.5.0` |
| 向下兼容的 Bug 修复 / 安全修复 | Z | `1.4.2 → 1.4.3` |
| 仅文档/注释，不影响发布物 | 不升 | — |

预发布后缀：`-alpha.N`（提测）/ `-beta.N`（公测）/ `-rc.N`（候选）。同一目标版本可多次重测，序号递增。

## 2. 分支命名

长期分支：`main`（生产）、`dev`（提测/测试环境）。两条均不可直接 push，只能 MR 合入。

临时分支统一格式 `<type>/vX.Y.Z/<description>`，全小写；描述段可用 `-` 或 `_` 连接（推荐 `-`）。

| 前缀 | 用途 | 来源 | 合并目标 |
| --- | --- | --- | --- |
| `feature` 或 `feat`/vX.Y.Z/* | 新功能 | `dev` | `dev` |
| `fix/vX.Y.Z/*` | 普通缺陷修复 | `dev` | `dev` |
| `perf/vX.Y.Z/*` | 性能优化 | `dev` | `dev` |
| `refactor/vX.Y.Z/*` | 重构 | `dev` | `dev` |
| `release/vX.Y.Z` | 发布准备 | `dev` | `main` + 回合 `dev` |
| `hotfix/vX.Y.Z/*` | 生产紧急修复 | `main` | `main` + 回合 `dev` |

示例：`feature/v1.2.0/user-login`、`feat/v1.4.1/app-common`、`feat/v1.4.1/app_common`、`hotfix/v1.1.3/payment-timeout`。

## 3. 提交信息（Conventional Commits）

格式：`<type>(<scope>): <subject>`，subject ≤ 50 字。

| type | 含义 | 版本影响 |
| --- | --- | --- |
| `feat` | 新功能 | Minor (Y) |
| `fix` | 缺陷修复 | Patch (Z) |
| `perf` | 性能优化 | Patch（引入新能力时 Minor） |
| `refactor` / `docs` / `test` / `ci` / `chore` | 重构 / 文档 / 测试 / CI / 杂项 | 不升版本 |
| `build` | 构建或依赖变更 | 视情况 |
| `revert` | 回滚 | 视被回滚提交 |

破坏性变更：type 后加 `!`（如 `feat(api)!: ...`）或在 footer 加 `BREAKING CHANGE:`，触发 Major (X)。

## 4. Tag 命名

| 场景 | Tag | 打在 |
| --- | --- | --- |
| 提测 | `vX.Y.Z-alpha.N` | `dev` |
| 预发候选 | `vX.Y.Z-beta.N` / `vX.Y.Z-rc.N` | `release/*` 或 `main` |
| 正式发布 | `vX.Y.Z` | `main` |

统一使用 annotated tag：`git tag -a v1.2.0 -m "Release v1.2.0" && git push origin v1.2.0`。**Tag 一旦推送禁止删除或覆盖**，重测请递增序号。

## 5. 流程速查

完整生命周期：

```text
feature/vX.Y.Z/*  --MR-->  dev  --tag-->  vX.Y.Z-alpha.N   (测试环境)
                            │
                            └──切出──> release/vX.Y.Z --tag--> vX.Y.Z-rc.N   (预发)
                                              │
                                              └──合 main --tag--> vX.Y.Z   (生产)
```

### 5.1 提测（合并到 `dev`）

**关键边界**：合并到 `dev` 仅完成代码进入测试分支；只有在该 commit 上打出 `vX.Y.Z-alpha.N` Tag，才视为正式提测版本。

MR 合并到 `dev` 前必须满足：

- [ ] 已基于最新 `dev` rebase / merge，无冲突
- [ ] 完成开发自测，关键路径手工验证通过
- [ ] 单测 / 集成测试本地通过（如有）
- [ ] MR 描述关联需求/任务卡 + 说明配置/表结构/依赖变更

> Reviewer Code Review 与 CI（lint / 测试 / 构建）不作为合并到 `dev` 的硬性门槛；若仓库已配置则按其结果流转，开发自行决定是否等候。

合并后流程：部署测试环境 → 在合入 commit 上打 `vX.Y.Z-alpha.N` Tag（CI 自动或开发手动）→ 通知 QA 用该 Tag 测试。提测期发现 Bug 必须回到原 `feature/*` / `feat/*` 分支修，**不要直接改 `dev`**，重新合入后递增为 `alpha.(N+1)`。

### 5.2 常规发布（提测通过后）

1. 从通过的 `alpha.N` commit 切 `release/vX.Y.Z`。
2. `release/*` 上完成预发回归、文档与 `CHANGELOG.md` 更新，必要时打 `-rc.N` 验证预发。
3. 合并 `release/*` → `main`，打 `vX.Y.Z`，CI 触发生产部署。
4. `release/*` 回合 `dev`，删除 `release/*` 分支。

### 5.3 紧急修复（Hotfix）

1. 从 `main` 最新 Tag 切 `hotfix/vX.Y.(Z+1)/xxx`。
2. 修复 + 自测后 MR 合 `main`，打 `vX.Y.(Z+1)`，触发生产部署。
3. 同步合回 `dev`。
4. 可跳过完整提测流程，但要在 MR 描述中说明修复方案、影响范围与自测结论。

## 6. 硬性约束

- `main`、`dev` 禁止直接 push，必须 MR/PR。
- 合并到 `dev` 后触发测试环境部署，并在合入 commit 上打 `vX.Y.Z-alpha.N` Tag（CI 自动或开发手动均可）。
- `*-alpha.*` / `vX.Y.Z` Tag 推送后禁止删除或覆盖。
- 目标版本号（`X.Y.Z`）由迭代负责人在迭代启动时写入 `package.json` / `VERSION` 文件；alpha 序号按提测顺序递增（CI 自动或开发手动）。
- 破坏性变更必须在 PR 描述与 Changelog 中显式说明迁移方式。
- 一个 PR 聚焦单一目标，提交信息符合本规范。
- Reviewer 与 CI **不是**硬性门槛；仓库自行决定是否启用，但启用后建议遵守对应仓库的合并策略。

## 7. CSG 提测文档与 Base

当用户要求“写提测文档”“写到提测 Base”“补服务子记录”“发给测试负责人”时，读取 [test-release.md](test-release.md)，按其中流程执行：

1. 收集项目、版本、域名、namespace、仓库、更新说明、提测人、测试人。
2. 生成本地 Markdown 提测文档。
3. 创建飞书 Docx 提测文档。
4. 写入提测 Base 主记录，并把“提测文档”更新为飞书文档链接。
5. 创建服务子记录并关联主记录。
6. 将提测总表和提测文档发送给指定测试负责人。

写 Base 前必须先读取真实字段结构；人员字段必须先查通讯录拿 open_id；不要猜字段名、用户 ID、表 ID 或记录 ID。

## 8. 何时读详细参考

- Git 版本、分支、Tag、发布流程：读 [reference.md](reference.md)
- 提测文档、飞书 Docx、Base 主记录/子记录、通知流程：读 [test-release.md](test-release.md)

需要完整规范、详细场景说明、CI 集成示例、Changelog 自动化工具选型等深度内容时再读。日常 Git 判断用本文件足够。

## 9. 安装本 Skill

```bash
npx skills add Links17/csg-git-skill -g -a cursor -a claude-code -a codex -y
```

详见仓库 [README.md](https://github.com/Links17/csg-git-skill#安装)。
