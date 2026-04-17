---
name: jira-fae-tickets
description: Inspect and update Jira tickets in Telechips TITAN using the browser tool, especially FAE tab handling for FAE_Label, FAE Pattern, and Comment, AND Field tab handling for O/S, Self Resolution, Cause(Customer), Hardware Issue Pattern, Software Issue Pattern, Labels, FAE Person, and git/repo command. Use when auditing another reporter's tickets for missing FAE or Field tab fields, or when editing your own reporter-filtered tickets to fill/update FAE content. Includes bilingual (English/Chinese) workflow notes, browser-login prerequisites, FAE label selection rules, pagination handling, and safe differences between check-only mode and edit/update mode.
---

# Jira FAE Tickets

Handle Telechips TITAN Jira tickets through the browser tool.

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

## Two operating modes / 两种工作模式

### 1) Check-only audit mode / 只检查模式

Use this when the JQL reporter is changed to another person and the user only wants an audit.

For large audits across many tickets/pages, prefer creating an independent sub-task/sub-agent to run the inspection continuously, while the main session only tracks progress and reports results. For audit-only work, prefer a high-efficiency read-only method when available instead of slow human-like ticket-by-ticket visible clicking.

**Preferred method: Jira REST API (when Chrome extension session is active)**

When the user's Chrome session is already authenticated to TITAN, prefer calling the Jira REST API directly for bulk read-only audits instead of browser-clicking through tickets. This is dramatically faster and more reliable. See the "API-based audit method" section below.

If the task is check-only:

* Open the filtered issue list.
* For large multi-page audits, prefer spawning a dedicated sub-task/sub-agent, or using the REST API method.
* When possible, use a high-efficiency authenticated audit method for read-only inspection instead of slow human-like browser clicking.
* Inspect tickets page by page.
* For each ticket, check **both** the FAE tab AND the Field tab fields (see sections below).
* Record the ticket key if any required field is empty.
* Click **Cancel** after inspecting (browser mode only).
* Continue through pagination until all pages are checked.
* Return only the missing ticket keys and which fields are empty.

当切换到别的 reporter、只想检查时：

* 打开筛选后的票列表。
* 按页检查（或优先使用 REST API 批量审计）。
* 对每一张票，同时检查 FAE 标签页和 Field 标签页字段。
* 只要有任一项为空，就记下票号和缺失字段名。
* 点击 **Cancel**（浏览器模式）。
* 翻页继续，直到所有页面检查完成。
* 最终只汇报缺项票号，以及缺的是哪些字段。

### 2) Edit/update mode / 修改更新模式

Use this only when the user explicitly wants tickets updated.

If the task is edit/update mode:

* Start from the fixed TITAN URL: `https://tcs.telechips.com/secure/Dashboard.jspa`
* If login is required, wait for the user to complete login in the same Chrome window.
* Do **not** use the top global header search box as plain text JQL input.
* To begin correctly:
  1. Open the TITAN dashboard in Chrome.
  2. Enter the real issue search page.
  3. Use the **Advanced Query** field there for JQL.
* Before starting, apply this JQL for the current user's own reported tickets:
  `created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`
* If the filter area is not directly editable, click **Advanced** first, then paste/type the JQL directly.
* Only after this filter is applied should bulk processing begin.
* Work ticket by ticket.
* Click **Edit**.
* **Always click the FAE tab** before touching any FAE-related field.
* Fill required fields and write a ticket-specific English technical comment.
* Click **Update**.
* Do **not** close the browser after update.

---

## Field Tab audit / Field 标签页审计

### Overview / 概述

The **Field** tab (also called the "Field Tab" or general fields area) contains operational metadata that must be filled for every ticket. During check-only audits, **both** the FAE tab and the Field tab must be inspected.

**Field** 标签页包含每张票必须填写的运营元数据。只检查模式下，**FAE 标签页和 Field 标签页都要检查**。

### Fields to check in Field Tab / 需检查的字段

Check all 8 of the following fields for empty/None value:

| Field Name | Description / 说明 | Empty condition |
|---|---|---|
| `O/S` | Operating System / 操作系统 | null, empty string, or "None" |
| `Self Resolution` | Whether FAE resolved autonomously / 自主解决情况 | null, empty string, or "None" |
| `Cause(Customer)` | Customer-side root cause category / 客户原因分类 | null, empty string, or "None" |
| `Hardware Issue Pattern` | Hardware failure pattern / 硬件问题模式 | null, empty string, or "None" |
| `Software Issue Pattern` | Software failure pattern (first field only) / 软件问题模式（只检查第一个） | null, empty string, or "None" |
| `Labels` | Jira labels / 标签 | empty array or no labels |
| `FAE Person` | Responsible FAE engineer / 负责 FAE | null, empty string, or "None" |
| `git/repo command` | Git/Repo command reference / Git/Repo 命令 | null, empty string, or "None" |

**Important for Software Issue Pattern:** If there are multiple Software Issue Pattern fields in the Field tab, only check the **first** one. Ignore subsequent instances.

**Software Issue Pattern 注意：** 如果 Field 标签页中有多个 Software Issue Pattern 字段，只检查**第一个**，忽略其余的。

### Browser-click method for Field Tab / 浏览器点击检查方法

When inspecting via browser:

1. Open the ticket in edit mode (click **Edit**).
2. Click the **Field** tab (not the FAE tab).
3. Check each of the 8 fields listed above.
4. Record any that are empty/None.
5. Then switch to the **FAE** tab and check FAE fields (see FAE section below).
6. Click **Cancel** when done.

浏览器点击检查步骤：

1. 打开票，点击 **Edit** 进入编辑模式。
2. 点击 **Field** 标签页（不是 FAE 标签页）。
3. 逐一检查上述 8 个字段。
4. 记录空值字段。
5. 再切换到 **FAE** 标签页检查 FAE 字段。
6. 完成后点击 **Cancel**。

### API-based audit method for Field Tab / REST API 批量检查方法

When using the Jira REST API for bulk auditing, fetch all required fields in a single call per page:

```
GET /rest/api/2/search?jql=<JQL>&fields=customfield_XXXXX,labels,...&maxResults=50&startAt=0
```

**Field-to-customfield mapping (verify in TITAN before use):**

The custom field IDs for TITAN must be confirmed by first calling:

```
GET /rest/api/2/field
```

Look for fields matching these names (case-insensitive):
- `O/S` → find the matching `customfield_XXXXX`
- `Self Resolution` → find the matching `customfield_XXXXX`
- `Cause(Customer)` → find the matching `customfield_XXXXX`
- `Hardware Issue Pattern` → find the matching `customfield_XXXXX`
- `Software Issue Pattern` → find the matching `customfield_XXXXX` (take the first one if multiple)
- `Labels` → standard field, use `labels`
- `FAE Person` → find the matching `customfield_XXXXX`
- `git/repo command` → find the matching `customfield_XXXXX`

**Empty detection logic:**

```javascript
function isBlank(val) {
  if (val === null || val === undefined) return true;
  if (Array.isArray(val)) return val.length === 0;
  if (typeof val === 'string') return val.trim() === '' || val.trim().toLowerCase() === 'none';
  if (typeof val === 'object') {
    // Jira option fields return { id, value } or { id, name }
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
* Good: `notification`
* Bad: `Technical Document`

#### FAE_Label selection must follow ticket content / FAE_Label 必须跟票内容走

* Use `safellink` only for tickets involving **TCC5110** and **SDM**.
* Use `CarPlay` for CarPlay-related tickets when appropriate.
* Use a content-matching label for other ticket types.
* Use `notification` for notice/announcement-type tickets when a new label is needed.

#### FAE_Label and FAE Pattern are different / FAE_Label 和 FAE Pattern 不是一回事

* `FAE_Label` = concrete label/tag
* `FAE Pattern` = categorized pattern/classification

Never use the pattern text as if it were the label.

---

## Combined audit workflow / 综合审计流程

When performing a full audit (both FAE tab + Field tab), follow this sequence per ticket:

```
For each ticket:
  1. Open Edit mode
  2. → Click "Field" tab
     Check: O/S, Self Resolution, Cause(Customer),
            Hardware Issue Pattern, Software Issue Pattern (1st only),
            Labels, FAE Person, git/repo command
     Record any empty fields
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
     检查：O/S、Self Resolution、Cause(Customer)、
           Hardware Issue Pattern、Software Issue Pattern（只检查第一个）、
           Labels、FAE Person、git/repo command
     记录空值字段
  3. → 点击 "FAE" 标签页
     检查：FAE_Label、FAE Pattern、Comment
     记录空值字段
  4. 点击 Cancel
  5. 记录：票号 + 所有缺失字段名（Field 标签页 + FAE 标签页）
```

---

## Pagination / 翻页

For bulk operations:

* Finish the current page first.
* Then click the next page number explicitly (for example `2`, then `3`).
* Keep track of checked pages and checked ticket count.

批量处理时：

* 先完成当前页。
* 再明确点击下一页页码（例如 `2`、再 `3`）。
* 记录已经检查过的页数和票数。

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
TANCS5-101 | Field: O/S, Labels | FAE: FAE_Label | Total: 3
TANCS5-88  | Field: Self Resolution, FAE Person | FAE: — | Total: 2
...

--- Skipped (no FAE tab): TANCS5-77, TANCS5-55
```

---

## Known examples / 已确认示例

### Correct edit example

* Ticket: `TANCS5-29`
* `FAE_Label = carplay`
* `FAE Pattern = SW / General`
* Write an English technical comment matching the ticket content
* Click **Update**
* Stop there, do not close browser

---

## References / 参考文件

Read these when needed:

* `references/usage-en.md` for English operator guidance
* `references/usage-zh.md` for Chinese operator guidance
* `references/field-tab-fields.md` for Field tab custom field ID lookup guide
