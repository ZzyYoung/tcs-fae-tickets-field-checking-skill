# 使用指南 — 中文

## 快速开始

1. 确认 Chrome 已打开，TITAN 已登录。
2. 提供审计用的 JQL（reporter、日期范围）。
3. 只检查类审计必须先做字段完整性审查，再做必需的 Reporter/Assignee TITAN 关联负责人审查。
4. 字段完整性审查：Claude 会同时检查 Field 标签页和 FAE 标签页，但不检查 Field 标签页 Labels。
5. 关联负责人审查：Claude 只保留 Reporter=TITAN 且 Assignee=TITAN 的 TCS 票，再沿着 `Issue Links -> links to -> TITAN Issue` 打开关联 TITAN Issue；仅当关联 TITAN Issue Assignee 是 Unassigned 或中国 FAE 时报告。
6. 修改更新模式：Claude 只修改 FAE 标签页字段（需明确指示）。

## 系统与项目范围

- TCS（`tcs.telechips.com`）是 Jira 部署域名；TITAN 是 Jira 实例/上下文。对这个 skill 来说二者是同一个系统。
- 只审计客户相关范围票号前缀：所有 `TANCS*`，以及 `TMRCR`。
- 跳过 `TMCF-*`、`TPCP-*`、`IM*`、`IS*`、`IG*` 以及其他非客户范围项目。
- 大批量审计时，使用 Jira REST API search 分页（`maxResults=50`），不要逐张点击工单。

## 审计范围

### Field 标签页（7 个检查字段）
- O/S（操作系统）
- Self Resolution（自主解决情况）
- Cause (Customer)（客户原因分类）
- Hardware Issue Pattern（硬件问题模式）
- Software Issue Pattern（软件问题模式，只检查第一个）
- FAE Person（负责 FAE）
- git/repo command（Git/Repo 命令）

不要审计 Field 标签页 Labels、SDK Version (TITAN)、Ref. H/W version 或其他范围外字段。

### FAE 标签页（3 个字段）
- FAE_Label
- FAE Pattern
- Comment

## 常用 JQL

**FAE 标签页缺失字段：**
```jql
(FAE_Label = empty OR cf[15200] = empty OR cf[15201] IS EMPTY)
AND created >= 2025-01-01
AND reporter in ("user@telechips.com")
ORDER BY created DESC
```

**Field 标签页缺失字段：**
```jql
(
  cf[10684] IS EMPTY OR cf[10684] = None OR
  cf[15009] IS EMPTY OR cf[15009] = None OR
  cf[15044] IS EMPTY OR cf[15044] = None OR
  cf[15045] IS EMPTY OR cf[15045] = None OR
  cf[15046] IS EMPTY OR cf[15046] = None OR
  cf[15100] IS EMPTY OR
  cf[15101] IS EMPTY
)
AND created >= 2025-01-01
AND reporter in ("user@telechips.com")
ORDER BY created DESC
```

**按 reporter 和日期范围：**
```
created >= 2025-01-01 AND created <= 2026-04-17 AND reporter in ("user@telechips.com") order by created DESC
```

**当前用户：**
```
created >= 2025-01-01 AND reporter in (currentUser()) order by created DESC
```

**按项目：**
```
project = TANCS5 AND created >= 2025-01-01 AND reporter in ("user@telechips.com") order by created DESC
```

如果 JQL 范围较宽，获取结果后仍然必须按 `TANCS*` 和 `TMRCR` 前缀过滤，再生成报告。

**Reporter/Assignee TITAN 关联负责人审查候选票：**
```jql
reporter = "system.titan@telechips.com"
AND assignee = "system.titan@telechips.com"
ORDER BY created DESC
```
然后逐张打开候选票，只保留存在 `Issue Links -> links to -> TITAN Issue: <key>` 的票；再打开关联 TITAN Issue，仅当其 Assignee 是 `Unassigned` 或中国 FAE 9 人之一时报告。

## REST API 快速审计

使用：

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

必须用 `startAt` 分页直到收集数量达到响应里的 `total`。不要依赖 Jira 当前 UI 页或 XML/RSS 复制内容作为完整 filter 数据。

必取字段：

```text
summary,customfield_10684,customfield_15009,customfield_15044,customfield_15045,customfield_15046,customfield_15100,customfield_15101,customfield_15200,customfield_15201,customfield_15300
```

认证方式：在 `tcs.telechips.com` 的浏览器 Console 执行、使用 PAT（如果支持）、或复制 `JSESSIONID` 作为 Cookie header。

Reporter/Assignee TITAN 关联负责人审查候选票获取：

```text
summary,reporter,assignee,created,issuelinks
```

## 必做审查流程

### 字段完整性审查（第一步必做）
- Claude 只读取，不修改票。
- 同时检查 Field 标签页和 FAE 标签页的所有字段。
- 忽略 Field 标签页 Labels。
- 返回结构化的审计报告。
- 大批量审计时，优先使用 REST API。
- 不属于客户范围或没有 FAE 标签页的票直接跳过。
- 记录票号和全部缺失字段。

### Reporter/Assignee TITAN 关联负责人审查
- 每次字段完整性审查后都要作为第二步执行。
- 候选 TCS 票必须 Reporter=TITAN 且 Assignee=TITAN。
- 检查 TCS 页面是否存在 `Issue Links -> links to -> TITAN Issue: <key>`。
- 打开每个关联 TITAN Issue 并读取 Assignee。
- 只报告关联 TITAN Issue Assignee 是 `Unassigned` 或中国 FAE 9 人之一的票。
- 关联 TITAN Issue 分配给总部 AE/R&D 或其他非中国 FAE 时跳过。
- REST/XML/export 被拦截时允许使用浏览器页面/DOM 方法；不要读取或导出 Cookie。

### 修改更新模式（只用于自己的票或有明确权限时）
- Claude 只修改 FAE 标签页字段。
- Field 标签页字段默认只检查，不自动修改，除非用户明确要求。
- 进入编辑前必须先点击 FAE 标签页。
- `FAE_Label` 输入后必须点真实候选项。
- 点候选后稍等一下再 Update。
- Comment 必须和该票内容匹配，不能所有票都写同一句。

## 输出格式

审计报告包含：
- 汇总数量（总票数、有缺失的票数、总缺失字段数）
- 每张票的缺失字段明细（区分 Field 标签页和 FAE 标签页）
- 每次都包含 Reporter/Assignee TITAN 关联负责人审查章节，即使结果为 0

给领导看的管理报告，先放简洁汇总表：

| 담당자 | Missing 개수 | Missing Ticket List | In-scope |
|---|---:|---|---:|

如果要求 “Only Titan Issue”，仍然按标准客户范围汇总 `TANCS*` 和 `TMRCR-*`。只有用户明确要求一次性的 TANCS-only 报告时，才排除 `TMRCR`。

## 注意事项

- 大批量审计（50 张以上）时，优先使用 REST API 方式，比浏览器点击快得多。
- 使用 `SKILL.md` 中的固定 customfield ID；只有怀疑 metadata 变化时才调用 `GET /rest/api/2/field` 复核。
- Software Issue Pattern：如果存在多个同名字段，只检查**第一个**。
- `Cause (Customer): None` 是缺失值；JQL 过滤必须包含 `cf[15044] = None`。
- FAE Tab 的 `Comment` 是 `customfield_15201`，不是 Jira 系统评论。
- Labels 是 Jira 内置字段，但字段完整性审查中忽略。
- `git/repo command` 使用 `customfield_15101`，不要使用 metadata 中同名的 `customfield_15008`。
- 中国 FAE 作为 Assignee 是合法的；总部 AE/R&D 作为 Assignee 也是合法的。
- Assignee 为空/null 是不合规。
- `safellink`：只用于 TCC5110 和 SDM 相关票。
- `CarPlay`：用于 CarPlay 相关票。
- `notification`：用于通知/公告类票，且需要新建标签时。
