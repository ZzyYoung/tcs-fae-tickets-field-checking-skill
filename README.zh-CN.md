# Titan FAE Tickets Field Checking Skill

这是一个用于 Telechips TITAN Jira 票据审计与更新的 OpenClaw skill，目标系统是 `tcs.telechips.com`，覆盖 **FAE** 标签页和 **Field** 标签页。

## 系统与范围

- TCS（`tcs.telechips.com`）是 Jira 部署域名；TITAN 是 Jira 实例/上下文。对这个 skill 来说，它们是同一个系统。
- 只审计 TITAN_Customer 票：`TANCS-*`、`TANCS4-*`、`TANCS5-*`、`TANCS6-*`、`TANCS7-*`。
- 跳过其他项目，例如 `TMRCR-*`、`TMCF-*`、`TPCP-*`、`IM*`、`IS*`、`IG*`；这些不属于本 skill 的 FAE/Field 审计范围。
- 大批量只读检查时，使用 Jira REST API 分页获取，不要逐张打开工单。

## 这个 skill 做什么

它主要支持两类场景：

1. **只检查审计模式**
   - 检查别人 reporter 名下的 Jira 票
   - 同时检查 **FAE** 标签页和 **Field** 标签页字段
   - 最终只返回票号和缺失字段
   - 大批量检查时优先使用高效率只读检查方式

2. **修改更新模式**
   - 从 JQL 过滤结果中打开你自己的 Jira 票
   - 修改 FAE 相关内容前必须先进入 **FAE** 标签页
   - 安全填写或更新相关字段
   - 更新后保持浏览器会话不被破坏

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
- `Labels`
- `FAE Person`
- `git/repo command`

不要审计 `SDK Version (TITAN)`、`Ref. H/W version` 或上述列表之外的其他字段。

## 固定字段 ID 映射

### FAE 标签页

| 字段 | customfield_id | 类型 | "空" 判定 |
|---|---|---|---|
| `FAE_Label` | `customfield_15300` | array | null 或 `[]` |
| `FAE Pattern` | `customfield_15200` | option | null |
| `Comment` | `comment` | array | `comment.comments.length === 0` |

### Field 标签页

| 字段 | customfield_id | 类型 | "空" 判定 |
|---|---|---|---|
| `O/S` | `customfield_10684` | option | null 或 `value === 'None'` |
| `Self Resolution` | `customfield_15009` | option | null 或 `value === 'None'` |
| `Cause (Customer)` | `customfield_15044` | option | null 或 `value === 'None'` |
| `Hardware Issue Pattern` | `customfield_15045` | option | null 或 `value === 'None'` |
| `Software Issue Pattern` | `customfield_15046` | option，仅第一个下拉框 | null 或 `value === 'None'` |
| `Labels` | `labels` | array | `[]` |
| `FAE Person` | `customfield_15100` | user/string | null 或 `''` |
| `git/repo command` | `customfield_15101` | string | null 或 `''` |

注意：Jira metadata 中有两个字段都叫 `git/repo command`（`customfield_15008` 和 `customfield_15101`），Field 标签页实际显示的是 `customfield_15101`。

## REST API 审计

批量只读审计使用 Jira search API：

```text
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

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
- 审计结果必须限制在上面列出的 TITAN_Customer 票号前缀内
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
- 因不属于 TITAN_Customer 范围或没有 FAE 标签页而跳过的票
- 任一标签页存在缺失字段的票

## 最近更新

- FAE 标签页维持 3 个字段：`FAE_Label`、`FAE Pattern`、`Comment`
- Field 标签页范围限定为 8 个字段，并明确排除 `SDK Version (TITAN)` 和 `Ref. H/W version`
- 明确 TCS 和 TITAN 在本 skill 中指同一个 Jira 系统
- 增加固定 customfield ID 映射和 REST API 快速审计说明
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
