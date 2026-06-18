# CSG 提测文档与 Base 流程

用于把一次提测整理成本地文档、飞书 Docx、提测 Base 主记录、服务子记录，并通知测试负责人。

## 1. 必填信息

开始前收集这些字段：

| 字段 | 说明 | 示例 |
| --- | --- | --- |
| 项目 | Base 的“项目”选项 | `lerobot-demo` |
| 版本 | 提测版本 | `v0.1` |
| 域名 | 测试域名 | `https://lerobot.seeed.cc` |
| Namespace | K8s namespace 或项目空间 | `sensecraft-ai` |
| 仓库 | 代码仓库 HTTPS URL | `https://iteam-gitlab.seeed.cn/sensecraft-ai/sensecraft-lerobot-demo` |
| 更新说明 | 本次提测核心变更 | `支持 Wio Terminal 控制机械臂` |
| 提测人 | 人员姓名，需解析 open_id | `阙和荣` |
| 测试人 | 人员姓名，需解析 open_id | `阙和荣` |
| 通知对象 | 要发送链接的人 | `张殿晶`、`王宗明` |
| Base 链接 | 提测总表 URL | `/base/{base_token}?table={table_id}&view={view_id}` |

不要猜 `base_token`、`table_id`、字段名、用户 ID 或 record ID；全部从链接、字段结构和通讯录命令读取。

## 2. 本地 Markdown 提测文档

默认路径：

```text
docs/test-release-<version>.md
```

模板：

```markdown
# <项目名> <版本> 提测文档

## 基本信息

| 项目 | 内容 |
| --- | --- |
| 项目名称 | <项目名> |
| 版本号 | <版本> |
| 域名 | <域名> |
| Namespace | <namespace> |
| 仓库 | <仓库 URL> |
| 提测人 | <提测人> |
| 测试人 | <测试人> |

## 更新说明

<更新说明>。

## 提测范围

- Web 站点访问。
- 核心功能页面。
- 本次新增/变更流程。

## 建议测试项

1. 访问测试域名，确认首页可正常打开。
2. 进入核心页面，确认页面加载正常。
3. 验证本次更新说明对应功能。
4. 验证异常/回退/停止流程。

## 风险与注意事项

- 浏览器、权限、外设、环境变量或端口占用等限制。
- 真实设备测试前确认安全条件。

## 回归结论

待测试填写。
```

## 3. 创建飞书 Docx 提测文档

创建文档前必须读取对应 lark-doc skill 指南：

```bash
lark-cli skills read lark-doc references/lark-doc-create.md
lark-cli skills read lark-doc references/lark-doc-xml.md
lark-cli skills read lark-doc references/style/lark-doc-style.md
lark-cli skills read lark-doc references/style/lark-doc-create-workflow.md
```

推荐用 XML 创建结构化 Docx：

```bash
lark-cli docs +create --api-version v2 --as user --content '<title>项目 v0.1 提测文档</title>...'
```

文档结构建议：

- `<callout>` 开头概括版本和核心更新。
- `<table>` 展示项目、版本、域名、namespace、仓库、提测/测试人。
- `<checkbox>` 展示测试项。
- `<callout>` 展示风险与注意事项。

创建成功后记录：

```text
doc_url = https://.../docx/...
doc_token = 返回的 document_id
```

## 4. 写入提测 Base 主记录

从 Base URL 解析：

```text
base_token = /base/ 后的 token
table_id = table= 参数
view_id = view= 参数，仅用于定位视图，写记录通常不必传
```

先读取字段结构：

```bash
lark-cli base +field-list --base-token <base_token> --table-id <table_id> --as user
```

常见主表字段：

| 字段 | 类型 | 写入规则 |
| --- | --- | --- |
| 时间 | datetime | `YYYY-MM-DD HH:mm:ss` |
| 项目 | select | 若选项不存在，先 field-update 追加 |
| 更新环境 | select | 用户未提供时留空，不要猜 |
| 版本 | text | 版本号 |
| 提测文档 | text | 飞书 Docx URL |
| 提测状态 | select | 默认 `待提测` |
| 测试人 | user multi | `[{"id":"ou_xxx"}]` |
| 提测人 | user multi | `[{"id":"ou_xxx"}]` |
| 内容 | text | 更新说明 |
| 备注 | text | 域名、namespace、仓库等补充信息 |

人员字段写入前必须解析 open_id：

```bash
lark-cli contact +search-user --query "<姓名>" --as user
```

如果“项目”单选没有目标选项，先读取现有字段，再用全量 `field-update` 保留原选项并追加新选项：

```bash
lark-cli base +field-update \
  --base-token <base_token> \
  --table-id <table_id> \
  --field-id <项目字段 ID> \
  --json '{"name":"项目","type":"select","multiple":false,"options":[...原选项...,{"name":"<项目>","hue":"Green","lightness":"Lighter"}]}' \
  --yes \
  --as user
```

创建主记录：

```bash
lark-cli base +record-batch-create \
  --base-token <base_token> \
  --table-id <table_id> \
  --json '{"fields":["时间","项目","版本","提测文档","提测状态","测试人","提测人","内容","备注"],"rows":[["2026-06-18 18:26:00","lerobot-demo","v0.1","https://.../docx/...","待提测",[{"id":"ou_tester"}],[{"id":"ou_submitter"}],"更新说明","域名：...\nNamespace：...\n仓库：..."]]}' \
  --as user
```

保存返回的 `record_id`，后续子记录关联要用。

## 5. 创建并关联服务子记录

先从主表字段中找到“服务子记录”的 link_table，例如：

```text
服务子记录 -> link_table = tbl...
```

读取子表字段：

```bash
lark-cli base +field-list --base-token <base_token> --table-id <child_table_id> --as user
```

常见子表字段：

| 字段 | 类型 | 写入规则 |
| --- | --- | --- |
| 仓库 | text | 仓库 HTTPS URL |
| tag | text | 版本或 tag，例如 `v0.1` |
| 关联版本 | link | `[{"id":"<主记录 record_id>"}]` |
| 类型 | select | 如 `前端`、`后端`，按真实选项填写 |
| 镜像版本号 | text | 无独立镜像时可用版本号 |

创建子记录：

```bash
lark-cli base +record-batch-create \
  --base-token <base_token> \
  --table-id <child_table_id> \
  --json '{"fields":["仓库","tag","关联版本","类型","镜像版本号"],"rows":[["https://repo","v0.1",[{"id":"rec_xxx"}],"前端","v0.1"]]}' \
  --as user
```

创建后回查主记录，确认“服务子记录”显示新子记录 ID：

```bash
lark-cli base +record-get --base-token <base_token> --table-id <table_id> --record-id <record_id> --as user
```

## 6. 通知测试负责人

先解析通知对象 open_id：

```bash
lark-cli contact +search-user --query "<姓名或拼音>" --as user
```

发送提测总表和提测文档：

```bash
lark-cli im +messages-send \
  --user-id <open_id> \
  --markdown '项目 v0.1 提测信息：

- [提测总文档 / Base](https://...)
- [v0.1 提测文档](https://.../docx/...)

更新说明：...' \
  --as user
```

需要通知多人时，逐个发送；不要猜同名人员，搜索命中多条时先让用户确认。

## 7. 完成回报

最终回复用户时给出：

- 飞书 Docx 链接。
- Base 主记录 ID。
- 服务子记录 ID。
- 已通知人员及 message_id。
- 如果 `lark-cli` 输出 `_notice.update`，提示可在后续执行 `lark-cli update`。
