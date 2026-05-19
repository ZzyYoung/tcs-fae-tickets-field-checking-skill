---
name: jira-fae-tickets
description: Inspect and update Telechips TITAN Jira tickets on tcs.telechips.com, especially FAE tab handling for FAE_Label, FAE Pattern, and Comment, Field tab handling for O/S, Self Resolution, Cause (Customer), Hardware Issue Pattern, Software Issue Pattern, FAE Person, and git/repo command, and Reporter/Assignee correction audits for tickets still assigned to the TITAN system account. Use when auditing TITAN/TMRCR customer-scope tickets only (any TANCS* prefix plus TMRCR) for missing FAE or Field tab fields, incorrect TITAN system Reporter/Assignee values, or when editing your own reporter-filtered tickets to fill/update FAE content. Includes bilingual workflow notes, China FAE team workflow context, project-scope guardrails, fixed customfield IDs, Jira REST API audit guidance, authentication options, browser-login prerequisites, FAE label selection rules, and safe differences between check-only mode and edit/update mode.
---

# Jira FAE Tickets

Handle Telechips TITAN Jira tickets through the browser tool or Jira REST API.

## System & Scope (READ THIS FIRST) / 系统与范围（请先读）

**TCS and TITAN are not two independent systems.** `tcs.telechips.com` is the Telechips Collaboration System domain where Jira is deployed. **TITAN** is the Jira instance/project context used inside that same system.

**TCS 和 TITAN 不是两个独立系统。** `tcs.telechips.com` 是部署 Jira 的 Telechips Collaboration System 域名，**TITAN** 是其中的 Jira 实例/项目上下文。

Audit scope is restricted to customer-scope tickets only:

- Include all `TANCS*` ticket prefixes, such as `TANCS-`, `TANCS1-`, `TANCS2-`, `TANCS3-`, `TANCS4-`, `TANCS5-`, `TANCS6-`, `TANCS7-`, etc.
- Include `TMRCR-*` tickets.
- Skip non-scope projects/prefixes such as `TMCF-*`, `TPCP-*`, `IM*`, `IS*`, `IG*`, etc.
- Reason: those other projects do not have the target **FAE** tab and are outside this skill's audit scope.

审计范围只限客户相关范围票：

- 包含所有 `TANCS*` 票号前缀，例如 `TANCS-`、`TANCS1-`、`TANCS2-`、`TANCS3-`、`TANCS4-`、`TANCS5-`、`TANCS6-`、`TANCS7-` 等
- 包含 `TMRCR-*` 票
- 跳过非范围项目/前缀，例如 `TMCF-*`、`TPCP-*`、`IM*`、`IS*`、`IG*` 等
- 原因：这些项目没有目标 **FAE** 标签页，不属于本 skill 审计范围

### Common pitfalls / 常见错误

1. Do **not** treat TCS and TITAN as separate systems; use `https://tcs.telechips.com/`.
2. Do **not** audit outside-scope tickets; include only `TANCS*` and `TMRCR`, and skip `TMCF/TPCP/IM/IS/IG`.
3. Do **not** audit fields outside the defined FAE/Field tab scope. In particular, do not check `SDK Version (TITAN)` or `Ref. H/W version`.
4. Do **not** audit the Field Tab `Labels` field for completeness. It can be ignored.
5. Do **not** click through tickets one by one for large audits. Use Jira REST API search in 50-ticket pages.
6. Do **not** treat a China FAE assignee as invalid; it is valid when the FAE team handles the issue directly.
7. Do **not** treat headquarters AE/R&D assignees as invalid; this is the common valid state for technical investigation.
8. Reporter is expected to be one of the 9 China FAE members. Other reporters need confirmation.
9. Assignee `null` / empty is non-compliant and must be reported.
10. Match Jira usernames case-insensitively, because Jira may return variants such as `williamTang` vs `williamtang`.
11. In Mode B, do **not** report every TITAN-placeholder TANCS candidate directly; first confirm China FAE scope through linked issue Assignee.
12. In Mode B, do **not** filter linked issue keys by prefix. Check all Issue Links, regardless of project/key format.

## Telechips Jira workflow context / Telechips Jira 工作流背景

1. Customers create issues in TITAN Jira at `https://tcs.telechips.com`.
2. The system creates corresponding tickets in the `TITAN_Customer` project, usually with keys such as `TANCS-xxxx`.
3. Newly auto-created tickets default both **Reporter** and **Assignee** to the TITAN system account (`username: titan`, `email: titan@telechips.com`). This is a placeholder, not a real person.
4. The China FAE team must manually correct Reporter and Assignee after receiving the ticket.
5. Reporter should be a China FAE team member in about 99% of cases, because the FAE is the real customer-facing reporter/follower.
6. Assignee is usually a headquarters AE/R&D engineer who owns the technical investigation. In fewer cases, Assignee may be the China FAE member when the FAE team can handle the issue directly.
7. Assignee should never remain the TITAN system account or be unassigned.

客户在 `https://tcs.telechips.com` 创建问题后，系统会在 `TITAN_Customer` 项目下自动生成 `TANCS-xxxx` 票。自动票的 Reporter 和 Assignee 默认都是 TITAN 系统账号（`username: titan`，`email: titan@telechips.com`），只是占位符，不是真实负责人。中国 FAE 团队收到票后必须手动修正 Reporter 和 Assignee；总部 FAE Team Leader 要求定期审计，不能留下 TITAN 系统账号或空 Assignee。

## Startup URL and opening flow / 固定入口与打开流程

Use this fixed TITAN URL:

`https://tcs.telechips.com/secure/Dashboard.jspa`

At the start of a new run:

1. Automatically open this URL in **Chrome** with the browser tool.
2. If TITAN is not logged in yet, stop and let the user log in manually in that same controllable Chrome window.
3. After the user says login is complete, continue from the Jira UI.

固定使用这个 TITAN 入口：

`https://tcs.telechips.com/secure/Dashboard.jspa`

每次新任务开始时：

1. 用 browser tool 自动在 **Chrome** 打开这个 URL。
2. 如果还没登录 TITAN，就暂停，等待用户在同一个可控 Chrome 窗口里手动登录。
3. 用户说登录完成后，再继续后续 Jira 操作。

## Preconditions / 前提条件

Before doing anything, ensure all of the following are true:

1. A Chrome browser controllable by OpenClaw is available.
2. The user is already logged in to the target Jira/TITAN site **inside that same controllable Chrome browser**.
3. The relevant Jira filter or JQL can be accessed from that browser session.
4. For bulk work, prefer the issue navigator / split view and preserve the logged-in session.

在开始前，必须满足以下条件：

1. OpenClaw 可以控制一个 Chrome 浏览器。
2. 用户已经在**这个可控制的 Chrome 浏览器里**登录了 Jira/TITAN。
3. 需要处理的筛选条件或 JQL 能在该浏览器会话中正常打开。
4. 批量处理时，优先使用 issue navigator / split view，并保持当前登录态。

## Supported audit modes / 支持的两种审查模式

### Mode A: Field completeness audit / 模式 A：字段完整性审查

- Purpose: check whether required **FAE Tab** and **Field Tab** fields are filled.
- Grouping: usually grouped by reporter / China FAE member.
- JQL example: `reporter in ("user@telechips.com") AND created >= 2025-01-01 ORDER BY created DESC`
- Scope: include all `TANCS*` prefixes and `TMRCR`; skip non-scope projects.
- Field Tab `Labels` is ignored and should not be reported as missing.

### Mode B: Reporter/Assignee correction audit / 模式 B：Reporter/Assignee 修正审查

- Purpose: find TANCS tickets whose Reporter or Assignee still uses the TITAN system account, or whose Assignee is empty.
- Grouping: do **not** group by person, because problem tickets may still have `reporter = titan`.
- JQL example: `project = TITAN_Customer AND (reporter in ("titan") OR assignee in ("titan") OR assignee is EMPTY) AND created >= 2025-01-01 ORDER BY created DESC`
- Scope: first scan TANCS candidates, then decide whether each candidate belongs to China FAE only by checking linked issue Assignee through Issue Links.

---

## Operating modes / 工作模式

### 1) Check-only audit mode / 只检查模式

Use this when the JQL reporter is changed to another person and the user only wants an audit.

For large audits across many tickets/pages, prefer a high-efficiency read-only method. Do not use slow human-like ticket-by-ticket visible clicking unless the REST API is unavailable and the user explicitly accepts the slowdown.

**Preferred method: Jira REST API (when Chrome extension session is active)**

When the user's Chrome session is already authenticated to TITAN on `tcs.telechips.com`, prefer calling the Jira REST API directly for bulk read-only audits instead of browser-clicking through tickets. This is dramatically faster and more reliable: roughly 200 tickets need only 4 API calls (`maxResults=50`) and usually finish in seconds. See the "API-based audit method" section below.

If the task is check-only:

- Open the filtered issue list.
- For large multi-page audits, use the REST API method.
- When possible, use a high-efficiency authenticated audit method for read-only inspection instead of slow human-like browser clicking.
- Restrict the result set to `TANCS*` and `TMRCR` tickets or skip non-matching issue keys after fetch.
- For each ticket, check **both** the Field tab AND the FAE tab fields (see sections below).
- Record the ticket key if any required field is empty.
- Continue REST pagination until all results are checked.
- Return only the missing ticket keys and which fields are empty or set to `None`.

当切换到别的 reporter、只想检查时：

- 打开筛选后的票列表。
- 使用 REST API 批量审计。
- 只审计 `TANCS*` 和 `TMRCR` 票，或在获取后跳过不符合前缀的票号。
- 对每一张票，同时检查 FAE 标签页和 Field 标签页字段。
- 只要有任一项为空，就记下票号和缺失字段名。
- REST 分页直到所有结果检查完成。
- 最终只汇报缺项票号，以及缺的是哪些字段。

### 2) Edit/update mode / 修改更新模式

Use this only when the user explicitly wants tickets updated.

If the task is edit/update mode:

- Start from the fixed TITAN URL: `https://tcs.telechips.com/secure/Dashboard.jspa`
- If login is required, wait for the user to complete login in the same Chrome window.
- Do **not** use the top global header search box as plain text JQL input.
- To begin correctly:
  1. Open the TITAN dashboard in Chrome.
  2. Enter the real issue search page.
  3. Use the **Advanced Query** field there for JQL.
- Before starting, apply this JQL for the current user's own reported tickets:
  `created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`
- If the filter area is not directly editable, click **Advanced** first, then paste/type the JQL directly.
- Only after this filter is applied should bulk processing begin.
- Work ticket by ticket.
- Click **Edit**.
- **Always click the FAE tab** before touching any FAE-related field.
- If any of these FAE fields already contains content:
  - `FAE_Label`
  - `FAE Pattern`
  - `Comment`
  then skip rewriting and click **Update** only if the user requested this behavior.
- Otherwise, fill the required FAE fields and write a ticket-specific English technical comment.
- Click **Update**.
- Do **not** close the browser after update.

- 打开筛选后的票列表。
- 逐票处理。
- 点击 **Edit**。
- **必须先点 FAE 标签页**，再操作 FAE 相关字段。
- 如果以下任意 FAE 字段已经有内容：
  - `FAE_Label`
  - `FAE Pattern`
  - `Comment`
  则按用户规则决定是否跳过重写并直接 **Update**。
- 如果为空，再填写 FAE 字段，并写一条与该票内容对应的英文技术 comment。
- 点击 **Update**。
- Update 后**不要关闭浏览器**。

---

## Field Tab audit / Field 标签页审计

### Overview / 概述

The **Field** tab (also called the "Field Tab" or general fields area) contains operational metadata that must be filled for every ticket. During check-only audits, **both** the FAE tab and the Field tab must be inspected. The Field Tab `Labels` field is intentionally ignored and must not be reported as missing.

**Field** 标签页包含每张票必须填写的运营元数据。只检查模式下，**FAE 标签页和 Field 标签页都要检查**。Field 标签页的 `Labels` 字段当前忽略，不作为缺失项汇报。

### Fields to check in Field Tab / 需检查的字段

Check these 7 Field Tab fields for empty/None value:

| Field Name | Description / 说明 | Empty condition |
|---|---|---|
| `O/S` | Operating System / 操作系统 | null, empty string, or "None" |
| `Self Resolution` | Whether FAE resolved autonomously / 自主解决情况 | null, empty string, or "None" |
| `Cause (Customer)` | Customer-side root cause category / 客户原因分类 | null, empty string, or "None" |
| `Hardware Issue Pattern` | Hardware failure pattern / 硬件问题模式 | null, empty string, or "None" |
| `Software Issue Pattern` | Software failure pattern (first field only) / 软件问题模式（只检查第一个） | null, empty string, or "None" |
| `FAE Person` | Responsible FAE engineer / 负责 FAE | null, empty string, or "None" |
| `git/repo command` | Git/Repo command reference / Git/Repo 命令 | null, empty string, or "None" |

**Important for Software Issue Pattern:** If there are multiple Software Issue Pattern fields in the Field tab, only check the **first** one. Ignore subsequent instances.

**Software Issue Pattern 注意：** 如果 Field 标签页中有多个 Software Issue Pattern 字段，只检查**第一个**，忽略其余的。

Do **not** audit fields outside this scope, including Field Tab `Labels`, `SDK Version (TITAN)`, and `Ref. H/W version`.

不要审计范围外字段，包括 Field 标签页 `Labels`、`SDK Version (TITAN)` 和 `Ref. H/W version`。

### Browser-click method for Field Tab / 浏览器点击检查方法

When inspecting via browser:

1. Open the ticket in edit mode (click **Edit**).
2. Click the **Field** tab (not the FAE tab).
3. Check each of the 7 Field tab fields listed above.
4. Record any that are empty/None.
5. Then switch to the **FAE** tab and check FAE fields (see FAE section below).
6. Click **Cancel** when done.

浏览器点击检查步骤：

1. 打开票，点击 **Edit** 进入编辑模式。
2. 点击 **Field** 标签页（不是 FAE 标签页）。
3. 逐一检查上述 7 个字段。
4. 记录空值字段。
5. 再切换到 **FAE** 标签页检查 FAE 字段。
6. 完成后点击 **Cancel**。

### API-based audit method for Field Tab / REST API 批量检查方法

When using the Jira REST API for bulk auditing, fetch all required fields in a single call per page. This is the default method for check-only audits:

```
GET https://tcs.telechips.com/rest/api/2/search?jql=<JQL>&fields=<field_ids>&maxResults=50&startAt=<offset>
```

### JQL pre-filter for missing values / 用 JQL 预筛选缺失值

For faster field-completeness audits, first use Jira JQL to export only tickets with missing or `None` values. Do not rely only on browser-visible inspection.

For option fields, treat both `IS EMPTY` and `= None` as missing. For text fields, use `IS EMPTY`. Always keep the normal reporter/date/scope filters.

Example for FAE Tab:

```jql
(FAE_Label = empty OR cf[15200] = empty OR cf[15201] IS EMPTY)
AND created >= 2025-01-01
AND created <= 2026-03-27
AND reporter in ("shzhzeng@telechips.com")
ORDER BY created DESC
```

Example for Field Tab:

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

`cf[15044] = None` is important: a ticket such as `TANCS-4418` with `Cause (Customer): None` must be reported as missing.

### Fixed field ID mapping / 固定字段 ID 映射

Use this mapping for customer-scope audits (`TANCS*` plus `TMRCR`):

#### FAE Tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `FAE_Label` | `customfield_15300` | array | null or `[]` |
| `FAE Pattern` | `customfield_15200` | option | null |
| `Comment` | `customfield_15201` | textarea/string | null or `''` |

#### Field Tab

| Field | customfield_id | Type | Empty condition |
|---|---|---|---|
| `O/S` | `customfield_10684` | option | null or `value === 'None'` |
| `Self Resolution` | `customfield_15009` | option | null or `value === 'None'` |
| `Cause (Customer)` | `customfield_15044` | option | null or `value === 'None'` |
| `Hardware Issue Pattern` | `customfield_15045` | option | null or `value === 'None'` |
| `Software Issue Pattern` | `customfield_15046` | option (first dropdown only) | null or `value === 'None'` |
| `Labels` | `labels` | array | Ignored; do not audit |
| `FAE Person` | `customfield_15100` | user/string | null or empty string |
| `git/repo command` | `customfield_15101` | string | null or empty string |

`Labels` is shown here for mapping completeness only. Do not fetch it or report it as missing in Field Tab completeness audits.

Note: Jira metadata contains two fields named `git/repo command` (`customfield_15008` and `customfield_15101`). The Field tab uses `customfield_15101`.

注意：Jira metadata 中有两个字段都叫 `git/repo command`（`customfield_15008` 和 `customfield_15101`），Field 标签页实际显示的是 `customfield_15101`。

### Authentication / 认证方式

The Jira REST API requires a logged-in session. Use one of these methods:

1. **Browser Console (simplest):** open DevTools on `https://tcs.telechips.com`, paste the script below, and the browser automatically sends cookies.
2. **Personal Access Token (PAT):** use it if TITAN supports PAT authentication.
3. **JSESSIONID cookie:** copy from DevTools and send as a Cookie header, for example `export JIRA_COOKIE='JSESSIONID=...'`.

**Empty detection logic:**

```javascript
function isBlank(val) {
  if (val === null || val === undefined) return true;
  if (Array.isArray(val)) return val.length === 0;
  if (typeof val === 'string') return val.trim() === '' || val.trim().toLowerCase() === 'none';
  if (typeof val === 'object') {
    if (val.value !== undefined) return isBlank(val.value);
    if (val.name !== undefined) return isBlank(val.name);
  }
  return false;
}
```

**Pagination:**

```
startAt=0, maxResults=50  → page 1
startAt=50, maxResults=50 → page 2
...continue until startAt >= total
```

### Browser Console audit script / 浏览器 Console 审计脚本

Run this from the DevTools Console on `https://tcs.telechips.com`:

```javascript
(async () => {
  const jql = encodeURIComponent('created >= 2025-01-01 AND reporter in ("user@telechips.com") order by created DESC');
  const fields = [
    'summary',
    'customfield_10684', 'customfield_15009', 'customfield_15044',
    'customfield_15045', 'customfield_15046',
    'customfield_15100', 'customfield_15101',
    'customfield_15200', 'customfield_15201', 'customfield_15300'
  ].join(',');

  const isIncludedIssue = key => {
    const prefix = key.split('-')[0];
    return prefix.startsWith('TANCS') || prefix === 'TMRCR';
  };
  let allIssues = [], startAt = 0;
  while (true) {
    const resp = await fetch(`/rest/api/2/search?jql=${jql}&fields=${fields}&maxResults=50&startAt=${startAt}`);
    const data = await resp.json();
    allIssues = allIssues.concat(data.issues);
    if (allIssues.length >= data.total) break;
    startAt += 50;
  }

  const missing = [];
  for (const issue of allIssues) {
    if (!isIncludedIssue(issue.key)) continue;
    const f = issue.fields;
    const m = [];
    if (!f.customfield_10684 || f.customfield_10684.value === 'None') m.push('O/S');
    if (!f.customfield_15009 || f.customfield_15009.value === 'None') m.push('Self Resolution');
    if (!f.customfield_15044 || f.customfield_15044.value === 'None') m.push('Cause (Customer)');
    if (!f.customfield_15045 || f.customfield_15045.value === 'None') m.push('Hardware Issue Pattern');
    if (!f.customfield_15046 || f.customfield_15046.value === 'None') m.push('Software Issue Pattern');
    if (!f.customfield_15100) m.push('FAE Person');
    if (!f.customfield_15101) m.push('git/repo command');
    if (!f.customfield_15300 || f.customfield_15300.length === 0) m.push('FAE_Label');
    if (!f.customfield_15200) m.push('FAE Pattern');
    if (!f.customfield_15201) m.push('Comment (FAE Tab)');
    if (m.length > 0) missing.push({ key: issue.key, summary: f.summary, missing: m });
  }
  console.log(`Total: ${allIssues.length}, Missing: ${missing.length}`);
  console.table(missing);
  return missing;
})();
```

---

## Reporter/Assignee correction audit / Reporter/Assignee 修正审查

Use this mode to find candidate `TANCS*` tickets whose Reporter or Assignee was not corrected after automatic creation, then refine the result through Issue Links so non-China-FAE tickets are not falsely reported.

### Linked issue ownership rule / 关联工单归属判定

The only rule for deciding whether a TITAN-placeholder `TANCS*` ticket belongs to China FAE is:

1. Read the candidate ticket's `issuelinks`.
2. Extract every linked issue key from both `outwardIssue` and `inwardIssue`.
3. Do **not** filter linked issue keys by prefix. The linked issue key prefix is irrelevant.
4. Fetch every linked issue with `GET /rest/api/2/issue/<linked-key>?fields=assignee,summary`.
5. If any linked issue Assignee username, lowercased, is one of the 9 China FAE usernames, report the candidate `TANCS*` ticket.
6. If no linked issue Assignee is China FAE, skip the candidate ticket as out of China FAE scope.
7. If the candidate ticket has no Issue Links, put it in an `UNCERTAIN - no issue links` section for manual review.

判断一张 reporter/assignee 仍是 TITAN 的 `TANCS*` 票是否归中国 FAE 处理，唯一标准是：通过 Issue Links 找到关联工单（不限制 key 前缀），读取关联工单 Assignee；只要任一关联工单 Assignee 是中国 FAE 9 人之一，就报告该 TANCS 票，否则跳过。没有 Issue Links 的票放入“待人工确认”。

### Reverse-filter JQL / 反向过滤 JQL

Scan candidate `TANCS*` tickets first:

```jql
project = TITAN_Customer
AND (reporter in ("titan") OR assignee in ("titan") OR assignee is EMPTY)
AND created >= 2025-01-01
ORDER BY created DESC
```

Fields to fetch for candidates:

```text
summary,reporter,assignee,created,issuelinks
```

For each linked issue, fetch:

```text
GET /rest/api/2/issue/<linked-issue-key>?fields=assignee,summary
```

### Ticket link structure / 工单关联结构

Telechips Jira automatically creates a `TANCS*` customer-service/internal tracking ticket after a customer submits an issue. The `TANCS*` ticket is linked to the original/source issue through Jira Issue Links.

- Source/linked issue: the original customer issue; its Assignee is the actual FAE owner.
- `TANCS*` ticket: internal coordination ticket; initial Reporter/Assignee may both be the TITAN system account and may require manual China FAE correction.
- Linked issue key prefix is not meaningful for this audit. Check all Issue Links.

Telechips Jira 中，客户提交工单后系统会自动创建一张 `TANCS*` 客服/内部跟进票。这张 `TANCS*` 票通过 Issue Links 与原工单关联。原工单的 Assignee 才是真正负责跟进的 FAE。不要根据关联工单 key 前缀判断范围；必须检查所有 Issue Links。

### Expected values and severity / 期望值与严重程度

| Field | Expected value | Non-compliant case | Severity |
|---|---|---|---|
| Reporter | China FAE identified via linked issue Assignee | Still TITAN system account on in-scope TANCS ticket | HIGH |
| Assignee | China FAE identified via linked issue Assignee, or valid owner after correction | Still TITAN system account on in-scope TANCS ticket | HIGH |
| Assignee | Valid owner after correction | Unassigned / null on in-scope TANCS ticket | MEDIUM |
| Scope | Linked issue Assignee is China FAE | No Issue Links | UNCERTAIN |

Important:

- China FAE as Assignee is valid in the minority of cases where the FAE team can resolve the issue directly.
- Headquarters AE/R&D as Assignee can be valid after correction, but Mode B's China FAE ownership decision for TITAN-placeholder `TANCS*` tickets must come only from linked issue Assignee.
- Only report TITAN-placeholder candidate tickets if at least one linked issue Assignee is China FAE.
- Do not report candidate tickets linked only to non-China-FAE assignees; summarize them as skipped/out of scope.

### China FAE team members / 中国 FAE 团队成员

Use username first when auditing. Jira REST API returns `reporter.name` and `assignee.name` as username strings. Compare usernames case-insensitively.

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

TITAN system account:

- Username: `titan`
- Email: `titan@telechips.com`
- Jira XML/API may also show `system.titan@telechips.com` or display name `TITAN`.
- Meaning: placeholder account used by automatic ticket creation.

### Report format / 报告格式

Mode B reports must contain three sections:

1. **Tickets China FAE needs to fix**
   - TANCS ticket key
   - Summary
   - Current Reporter
   - Current Assignee
   - Matched linked issue key
   - Recommended Reporter/Assignee: linked issue Assignee, one of China FAE 9
   - Created date
2. **Uncertain - no issue links**
   - Tickets with no Issue Links; manual review required.
3. **Skipped - out of China FAE scope**
   - Summary count and first 3 examples only to avoid noisy reports.

### Browser Console script / 浏览器 Console 脚本

Run this from the DevTools Console on `https://tcs.telechips.com`:

```javascript
(async () => {
  const CHINA_FAE_USERNAMES = [
    'williamtang', 'hmyang', 'shzhzeng', 'zyzhong', 'jingoust',
    'chris.hsieh', 'richard.li', 'simon.sun', 'junkai.he'
  ];
  const TITAN_ACCOUNT = 'titan';

  const jql = encodeURIComponent(
    'project = TITAN_Customer ' +
    'AND (reporter in ("' + TITAN_ACCOUNT + '") OR assignee in ("' + TITAN_ACCOUNT + '") OR assignee is EMPTY) ' +
    'AND created >= 2025-01-01 ' +
    'ORDER BY created DESC'
  );
  const fields = 'summary,reporter,assignee,created,issuelinks';

  let allIssues = [], startAt = 0;
  while (true) {
    const resp = await fetch(`/rest/api/2/search?jql=${jql}&fields=${fields}&maxResults=50&startAt=${startAt}`);
    const data = await resp.json();
    allIssues = allIssues.concat(data.issues);
    if (allIssues.length >= data.total) break;
    startAt += 50;
  }

  console.log(`Step 1: ${allIssues.length} candidate TANCS tickets`);

  const inChinaScope = [];
  const outOfScope = [];
  const uncertain = [];

  for (let i = 0; i < allIssues.length; i++) {
    const issue = allIssues[i];
    const links = issue.fields.issuelinks || [];

    const linkedKeys = [];
    for (const link of links) {
      if (link.outwardIssue) linkedKeys.push(link.outwardIssue.key);
      if (link.inwardIssue) linkedKeys.push(link.inwardIssue.key);
    }

    if (linkedKeys.length === 0) {
      uncertain.push({
        key: issue.key,
        summary: (issue.fields.summary || '').substring(0, 60),
        reason: 'No issue links found',
        created: issue.fields.created.substring(0, 10)
      });
      continue;
    }

    let belongsToChinaFAE = false;
    let matched = null;
    for (const linkedKey of linkedKeys) {
      try {
        const linkedResp = await fetch(`/rest/api/2/issue/${linkedKey}?fields=assignee,summary`);
        if (!linkedResp.ok) {
          console.warn(`Cannot fetch ${linkedKey}: ${linkedResp.status}`);
          continue;
        }
        const linkedData = await linkedResp.json();
        const linkedAssignee = linkedData.fields.assignee;
        if (linkedAssignee && CHINA_FAE_USERNAMES.includes(linkedAssignee.name.toLowerCase())) {
          belongsToChinaFAE = true;
          matched = {
            linkedKey,
            linkedAssignee: linkedAssignee.displayName,
            linkedAssigneeUsername: linkedAssignee.name
          };
          break;
        }
      } catch (e) {
        console.warn(`Error fetching ${linkedKey}:`, e);
      }
    }

    const result = {
      key: issue.key,
      summary: (issue.fields.summary || '').substring(0, 60),
      reporter: issue.fields.reporter ? issue.fields.reporter.displayName : '(none)',
      assignee: issue.fields.assignee ? issue.fields.assignee.displayName : '(unassigned)',
      created: issue.fields.created.substring(0, 10),
      linkedKeys: linkedKeys.join(', ')
    };

    if (belongsToChinaFAE) {
      result.matchedLinkedKey = matched.linkedKey;
      result.suggestedReporter = matched.linkedAssignee;
      result.suggestedReporterUsername = matched.linkedAssigneeUsername;
      inChinaScope.push(result);
    } else {
      outOfScope.push(result);
    }

    if ((i + 1) % 10 === 0) console.log(`  Processed ${i + 1}/${allIssues.length}`);
  }

  console.log(`\n===== Summary =====`);
  console.log(`Total candidates: ${allIssues.length}`);
  console.log(`In China FAE scope: ${inChinaScope.length}`);
  console.log(`Out of scope: ${outOfScope.length}`);
  console.log(`Uncertain (no links): ${uncertain.length}`);

  console.log(`\n===== Tickets China FAE needs to fix =====`);
  console.table(inChinaScope);

  if (uncertain.length > 0) {
    console.log(`\n===== Uncertain (manual review) =====`);
    console.table(uncertain);
  }

  return { inChinaScope, outOfScope, uncertain };
})();
```

---

## FAE Tab audit / FAE 标签页审计

### Fields to check in FAE Tab / 需检查的字段

Check these 3 fields:

| Field | Empty condition |
|---|---|
| `FAE_Label` | no labels selected |
| `FAE Pattern` | null or empty |
| `Comment` | null or empty |

### Critical FAE handling rules / FAE 关键规则

#### Always click FAE first / 必须先点 FAE

Never assume the visible section is already the FAE form. Always click the **FAE** tab before checking or editing FAE fields.

不要假设当前看到的就是 FAE 表单。检查或修改前，必须先点击 **FAE** 标签页。

#### FAE_Label is not plain text / FAE_Label 不是普通文本框

`FAE_Label` is a label picker.

Correct method:

1. Type the intended label into the `FAE_Label` input.
2. Wait for the candidate / suggestion list.
3. Click the focused suggestion (or create-new item if needed).
4. Wait briefly.
5. Only then click **Update**.

#### No spaces in created labels / 新建标签不要有空格

Examples:
- Good: `notification`
- Bad: `Technical Document`

#### FAE_Label selection must follow ticket content / FAE_Label 必须跟票内容走

- Use `safellink` only for tickets involving **TCC5110** and **SDM**.
- Use `CarPlay` for CarPlay-related tickets when appropriate.
- Use a content-matching label for other ticket types.
- Use `notification` for notice/announcement-type tickets when a new label is needed.

#### FAE_Label and FAE Pattern are different / FAE_Label 和 FAE Pattern 不是一回事

- `FAE_Label` = concrete label/tag
- `FAE Pattern` = categorized pattern/classification

Never use the pattern text as if it were the label.

---

## Combined audit workflow / 综合审计流程

When performing a full audit (both FAE tab + Field tab), follow this sequence per ticket:

```
For each ticket:
  1. Open Edit mode
  2. → Click "Field" tab
     Check: O/S, Self Resolution, Cause (Customer),
            Hardware Issue Pattern, Software Issue Pattern (1st only),
            FAE Person, git/repo command
     Record any empty fields or fields set to None
  3. → Click "FAE" tab
     Check: FAE_Label, FAE Pattern, Comment
     Record any empty fields
  4. Click Cancel
  5. Log: ticket key + list of all missing fields (Field tab + FAE tab)
```

综合审计顺序（每张票）：

```
对每张票：
  1. 进入 Edit 模式
  2. → 点击 "Field" 标签页
     检查：O/S、Self Resolution、Cause (Customer)、
           Hardware Issue Pattern、Software Issue Pattern（只检查第一个）、
          FAE Person、git/repo command
     记录空值字段，以及值为 None 的字段
  3. → 点击 "FAE" 标签页
     检查：FAE_Label、FAE Pattern、Comment
     记录空值字段
  4. 点击 Cancel
  5. 记录：票号 + 所有缺失字段名（Field 标签页 + FAE 标签页）
```

---

## Pagination / 翻页

For bulk operations:

- Finish the current page first.
- Then click the next page number explicitly (for example `2`, then `3`).
- Keep track of checked pages and checked ticket count.

批量处理时：

- 先完成当前页。
- 再明确点击下一页页码（例如 `2`、再 `3`）。
- 记录已经检查过的页数和票数。

---

## Reporting format / 汇报格式

When returning results from an audit, include:

1. Check time
2. Filter time range
3. Filter/JQL used
4. Total pages / total tickets if known
5. Skipped tickets with no FAE section
6. **Field tab missing fields** — per ticket
7. **FAE tab missing fields** — per ticket
8. Summary table: ticket key | missing Field fields | missing FAE fields | total missing count

检查类结果建议包含：

1. 检查时间
2. 检查条件时间范围
3. 使用的筛选条件 / JQL
4. 总页数 / 总票数（如果已知）
5. 没有 FAE 区域而跳过的票
6. **Field 标签页缺失字段**（按票列出）
7. **FAE 标签页缺失字段**（按票列出）
8. 汇总表：票号 | Field 缺失字段 | FAE 缺失字段 | 总缺失数

### Example report output / 示例汇报格式

```
=== TITAN Jira Field Audit Report ===
Reporter: zyzhong@telechips.com
Date Range: 2025-01-01 ~ 2026-04-17
JQL: created >= 2025-01-01 AND created <= 2026-04-17 AND reporter in ("zyzhong@telechips.com") order by created DESC
Total Tickets: 42
Tickets with Missing Fields: 15
Total Missing Field Entries: 38

--- Non-Compliant Tickets ---
TANCS5-101 | Field: O/S, git/repo command | FAE: FAE_Label | Total: 3
TANCS5-88  | Field: Self Resolution, FAE Person | FAE: Comment | Total: 3
...

--- Skipped (no FAE tab): TANCS5-77, TANCS5-55
```

---

## Known examples / 已确认示例

### Correct edit example

- Ticket: `TANCS5-29`
- `FAE_Label = carplay`
- `FAE Pattern = SW / General`
- Write an English technical comment matching the ticket content
- Click **Update**
- Stop there, do not close browser

---

## References / 参考文件

Read these when needed:

- `references/usage-en.md` for English operator guidance
- `references/usage-zh.md` for Chinese operator guidance
- `references/field-tab-fields.md` for Field tab custom field ID lookup guide
