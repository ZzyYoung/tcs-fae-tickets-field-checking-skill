# Titan FAE Tickets Field Checking Skill

这是一个用于 Telechips TITAN Jira 票据审计与更新的 OpenClaw skill，目标系统是 `tcs.telechips.com`，覆盖 **FAE** 标签页字段、**Field** 标签页字段，以及 Reporter/Assignee 修正状态。

## 系统与范围

- TCS（`tcs.telechips.com`）是 Jira 部署域名；TITAN 是 Jira 实例/上下文。对这个 skill 来说，它们是同一个系统。
- 只审计客户相关范围票：所有 `TANCS*` 前缀以及 `TMRCR-*`。
- 跳过非范围项目，例如 `TMCF-*`、`TPCP-*`、`IM*`、`IS*`、`IG*`；这些不属于本 skill 的 FAE/Field 审计范围。
- 大批量只读检查时，使用 Jira REST API 分页获取，不要逐张打开工单。

## 这个 skill 做什么

这个 skill 在只检查报告中始终按顺序执行两个必做审查部分，并保留一个修改场景：

1. **字段完整性审查**
   - 检查 FAE Tab 和 Field Tab 的必填字段
   - 按 reporter / FAE 人员分组
2. **Reporter/Assignee TITAN 关联负责人审查**
   - 字段审查后，扫描 TCS 票本身 Reporter 和 Assignee 都是 TITAN 的票
   - 只保留页面中有 `Issue Links -> links to -> TITAN Issue` 的票
   - 打开关联 TITAN Issue，只报告其 Assignee 是 `Unassigned` 或中国 FAE 9 人之一的票
3. **修改更新场景**
   - 只有用户明确要求且权限清楚时，才更新 FAE 标签页字段

## Telechips Jira 工作流背景

客户在 `https://tcs.telechips.com` 的 TITAN Jira 创建工单后，系统会在 `TITAN_Customer` 项目下自动创建对应票，通常是 `TANCS-xxxx`。

自动创建的票默认 Reporter 和 Assignee 都是 TITAN 系统账号：

- Username: `titan`
- Email: `titan@telechips.com`

这只是占位账号。中国 FAE 团队收到票后必须手动修正 Reporter 和 Assignee：

- Reporter 在 99% 情况下应为中国 FAE 9 人之一。
- Assignee 大多数情况下是总部 AE/R&D 工程师。
- 少数情况下 Assignee 可以是中国 FAE 自己，表示 FAE 团队可直接处理。
- Assignee 不能保留 TITAN，也不能为空。

## 覆盖字段

### FAE 标签页
- `FAE_Label`
- `FAE Pattern`
- `Comment`

### Field 标签页
- `O/S`
- `Self Resolution`
- `Cause (Customer)`
- `Hardware Issue Pattern`
- `Software Issue Pattern`（只检查第一个）
- `FAE Person`
- `git/repo command`

不要审计 Field 标签页 `Labels`、`SDK Version (TITAN)`、`Ref. H/W version` 或上述列表之外的其他字段。

## 固定字段 ID 映射

### FAE 标签页

| 字段 | customfield_id | 类型 | "空" 判定 |
|---|---|---|---|
| `FAE_Label` | `customfield_15300` | array | null 或 `[]` |
| `FAE Pattern` | `customfield_15200` | option | null |
| `Comment` | `customfield_15201` | textarea/string | null 或 `''` |

### Field 标签页

| 字段 | customfield_id | 类型 | "空" 判定 |
|---|---|---|---|
| `O/S` | `customfield_10684` | option | null 或 `value === 'None'` |
| `Self Resolution` | `customfield_15009` | option | null 或 `value === 'None'` |
| `Cause (Customer)` | `customfield_15044` | option | null 或 `value === 'None'` |
| `Hardware Issue Pattern` | `customfield_15045` | option | null 或 `value === 'None'` |
| `Software Issue Pattern` | `customfield_15046` | option，仅第一个下拉框 | null 或 `value === 'None'` |
| `Labels` | `labels` | array | 忽略，不审计 |
| `FAE Person` | `customfield_15100` | user/string | null 或 `''` |
| `git/repo command` | `customfield_15101` | string | null 或 `''` |

`Labels` 仅用于保留映射信息。字段完整性审查中不要获取它，也不要作为缺失项汇报。

注意：Jira metadata 中有两个字段都叫 `git/repo command`（`customfield_15008` 和 `customfield_15101`），Field 标签页实际显示的是 `customfield_15101`。

## Reporter/Assignee TITAN 关联负责人审查

这个必做审查每次都在字段完整性审查之后执行，用来替换旧的宽泛修正规则。

候选 TCS 票 JQL：

```jql
reporter = "system.titan@telechips.com"
AND assignee = "system.titan@telechips.com"
ORDER BY created DESC
```

对每个候选票，检查 TCS 工单页面，只保留如下链接：

```text
Issue Links -> links to -> TITAN Issue: <linked-key>
```

然后打开关联 TITAN Issue，例如：

```text
https://telechips-itan.atlassian.net/browse/<linked-key>
```

只有当关联 TITAN Issue 的 Assignee 是 `Unassigned` 或中国 FAE 9 人之一时，才报告这张 TCS 票。关联 TITAN Issue 分配给总部 AE/R&D 或其他非中国 FAE 时跳过。

优先使用 REST/search API 分页；如果 REST/XML/export 被拦截，可以使用已登录浏览器页面/DOM 方法。不要读取或导出浏览器 Cookie，让浏览器登录态自然完成认证。

### 中国 FAE 团队

| Username | Display Name | Email |
|---|---|---|
| `williamTang` | William Tang | williamTang@telechips.com |
| `hmyang` | HongMing Yang | hmyang@telechips.com |
| `shzhzeng` | ShengZhou Zeng | shzhzeng@telechips.com |
| `zyzhong` | ZhiYong Zhong | zyzhong@telechips.com |
| `jingoust` | 박정진 (JJ PARK) | jingoust@telechips.com |
| `Chris.Hsieh` | Chris Hsieh | Chris.Hsieh@telechips.com |
| `richard.li` | Richard Li | richard.li@telechips.com |
| `simon.sun` | Simon Sun | simon.sun@telechips.com |
| `junkai.he` | Junkai He | junkai.he@telechips.com |

## REST API 审计

批量只读审计使用 Jira search API：

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

为了更快导出缺失字段，可以先用 JQL 预筛选。option 字段必须同时检查 `IS EMPTY` 和 `= None`，例如 `Cause (Customer)` 使用 `cf[15044] IS EMPTY OR cf[15044] = None`。FAE Tab 的 `Comment` 是 `customfield_15201`，应使用 `cf[15201] IS EMPTY`，不要用 Jira 系统评论字段。

REST 结果必须用 `startAt=0,50,100...` 分页，直到收集数量达到 `total`。不要把 Jira 当前可见页或 XML/RSS 复制内容当成完整 filter 结果。给领导看的报告使用简洁汇总表：담당자、Missing 개수、Missing Ticket List、In-scope；即使要求 “Only Titan Issue”，也仍然按标准客户范围包含 `TANCS*` 和 `TMRCR-*`。只有用户明确要求一次性的 TANCS-only 报告时，才排除 `TMRCR`。

认证方式：

1. 浏览器 Console：在 `https://tcs.telechips.com` 打开 DevTools，粘贴 `SKILL.md` 中的审计脚本，浏览器会自动带 cookie。
2. Personal Access Token (PAT)：如果 TITAN 支持。
3. `JSESSIONID` cookie：从 DevTools 复制后作为 Cookie header 使用，例如 `export JIRA_COOKIE='JSESSIONID=...'`。

## 使用前提

使用前必须满足：

- 你已经登录 Telechips TITAN Jira
- 登录态存在于 OpenClaw 可控制的同一个 Chrome 浏览器中
- Jira 列表页或筛选页能在该浏览器中打开
- 批量处理时保持浏览器稳定

## 固定入口

固定从这里开始：

`https://tcs.telechips.com/secure/Dashboard.jspa`

## 关键规则

- 检查或修改 FAE 相关内容前，必须先点 **FAE**
- 审计模式下，必须同时检查 **Field** 和 **FAE** 两个标签页
- 审计结果必须限制在上面列出的 `TANCS*` 和 `TMRCR` 票号前缀内
- 不要把 Field 标签页 `Labels` 汇报为缺失
- `FAE_Label` 是标签选择器，不是普通文本输入框
- 创建或选择标签时，要等待候选项出现并选中目标项，再点击更新
- 大批量只读检查时，优先使用 Jira REST API 分页方式，不逐张点击工单

## 标签示例

- `safellink` → 仅用于 **TCC5110** 和 **SDM** 相关票
- `CarPlay` → 用于 CarPlay 相关票
- `notification` → 用于通知/公告类票，且需要新建标签时
- 其他标签必须根据票的实际内容决定

## 建议的检查汇报格式

建议检查结果包含：

- 检查时间
- 检查条件时间范围
- 使用的 JQL / filter
- 总页数 / 总票数
- 因不属于客户范围或没有 FAE 标签页而跳过的票
- 任一标签页存在缺失字段的票
- Reporter/Assignee TITAN 关联负责人审查结果，即使结果为 0 也要体现

## 最近更新

- 用必做的 Reporter/Assignee TITAN 关联负责人审查替换旧的宽泛修正规则
- FAE 标签页维持 3 个字段：`FAE_Label`、`FAE Pattern`、`Comment`
- Field 标签页范围限定为 7 个检查字段，并明确排除 `Labels`、`SDK Version (TITAN)` 和 `Ref. H/W version`
- 明确 TCS 和 TITAN 在本 skill 中指同一个 Jira 系统
- 增加固定 customfield ID 映射和 REST API 快速审计说明
- 增加必做的 Reporter/Assignee TITAN 关联负责人审查，用于找出仍使用 TITAN 系统账号且关联负责人需要中国 FAE 处理的票
- 保留并延续当前的双标签页审计模型，也就是同时检查 **Field** 和 **FAE** 两个标签页
- 明确要求汇报结果区分缺失项来自 Field 标签页还是 FAE 标签页

## 仓库包含文件

- `SKILL.md` — 主 skill 说明
- `references/usage-en.md` — 英文使用说明
- `references/usage-zh.md` — 中文使用说明
- `references/field-tab-fields.md` — Field 标签页说明
- `jira-fae-tickets.skill` — 打包后的 skill 压缩包

## English version

See: [README.md](README.md)
