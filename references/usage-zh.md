# 使用指南 — 中文

## 快速开始

1. 确认 Chrome 已打开，TITAN 已登录。
2. 提供审计用的 JQL（reporter、日期范围）。
3. 告知 Claude 当前是**只检查模式**还是**修改更新模式**。
4. 只检查模式：Claude 会同时检查 Field 标签页和 FAE 标签页。
5. 修改更新模式：Claude 只修改 FAE 标签页字段（需明确指示）。

## 系统与项目范围

- TCS（`tcs.telechips.com`）是 Jira 部署域名；TITAN 是 Jira 实例/上下文。对这个 skill 来说二者是同一个系统。
- 只审计 TITAN_Customer 票号前缀：`TANCS`、`TANCS4`、`TANCS5`、`TANCS6`、`TANCS7`。
- 跳过 `TMRCR-*`、`TMCF-*`、`TPCP-*`、`IM*`、`IS*`、`IG*` 以及其他非 TITAN_Customer 项目。
- 大批量审计时，使用 Jira REST API search 分页（`maxResults=50`），不要逐张点击工单。

## 审计范围

### Field 标签页（8 个字段）
- O/S（操作系统）
- Self Resolution（自主解决情况）
- Cause (Customer)（客户原因分类）
- Hardware Issue Pattern（硬件问题模式）
- Software Issue Pattern（软件问题模式，只检查第一个）
- Labels（标签）
- FAE Person（负责 FAE）
- git/repo command（Git/Repo 命令）

不要审计 SDK Version (TITAN)、Ref. H/W version 或其他范围外字段。

### FAE 标签页（3 个字段）
- FAE_Label
- FAE Pattern
- Comment

## 常用 JQL

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

如果 JQL 范围较宽，获取结果后仍然必须按 TITAN_Customer 前缀过滤，再生成报告。

## REST API 快速审计

使用：

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

必取字段：

```text
summary,customfield_10684,customfield_15009,customfield_15044,customfield_15045,customfield_15046,labels,customfield_15100,customfield_15101,customfield_15200,customfield_15300,comment
```

认证方式：在 `tcs.telechips.com` 的浏览器 Console 执行、使用 PAT（如果支持）、或复制 `JSESSIONID` 作为 Cookie header。

## 工作模式说明

### 只检查模式（推荐用于审计他人的票）
- Claude 只读取，不修改票。
- 同时检查 Field 标签页和 FAE 标签页的所有字段。
- 返回结构化的审计报告。
- 大批量审计时，优先使用 REST API。
- 不属于 TITAN_Customer 范围或没有 FAE 标签页的票直接跳过。
- 记录票号和全部缺失字段。

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
- 跳过的票列表（范围外或没有 FAE 标签页）

## 注意事项

- 大批量审计（50 张以上）时，优先使用 REST API 方式，比浏览器点击快得多。
- 使用 `SKILL.md` 中的固定 customfield ID；只有怀疑 metadata 变化时才调用 `GET /rest/api/2/field` 复核。
- Software Issue Pattern：如果存在多个同名字段，只检查**第一个**。
- Labels 是 Jira 内置字段，直接使用 `labels`，无需 customfield ID。
- `git/repo command` 使用 `customfield_15101`，不要使用 metadata 中同名的 `customfield_15008`。
- `safellink`：只用于 TCC5110 和 SDM 相关票。
- `CarPlay`：用于 CarPlay 相关票。
- `notification`：用于通知/公告类票，且需要新建标签时。
