---
name: jira-fae-tickets
description: Inspect and update Jira tickets in Telechips TITAN using the browser tool, especially FAE tab handling for FAE_Label, FAE Pattern, and Comment. Use when auditing another reporter's tickets for missing FAE fields, or when editing your own reporter-filtered tickets to fill/update FAE content. Includes bilingual (English/Chinese) workflow notes, browser-login prerequisites, FAE label selection rules, pagination handling, and safe differences between check-only mode and edit/update mode.
---

# Jira FAE Tickets

Handle Telechips TITAN Jira tickets through the browser tool.

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

For large audits across many tickets/pages, prefer creating an independent sub-task/sub-agent to run the inspection continuously, while the main session only tracks progress and reports results.

If the task is check-only:

- Open the filtered issue list.
- For large multi-page audits, prefer spawning a dedicated sub-task/sub-agent.
- Inspect tickets page by page.
- For each ticket:
  - Click **Edit**.
  - If there is **no FAE tab**, click **Cancel** (or **Update** only if the workflow explicitly requires it) and skip that ticket.
  - If there is an **FAE** tab, click **FAE**.
  - Check whether these are empty:
    - `FAE_Label`
    - `FAE Pattern`
    - `Comment`
  - Record the ticket key if any required field is empty.
  - Click **Cancel**.
- Continue through pagination until all pages are checked.
- If using a dedicated sub-task/sub-agent, let that worker perform the end-to-end audit while the main session only relays progress and final results.
- Return only the missing ticket keys and which fields are empty.

当切换到别的 reporter、只想检查时：

- 打开筛选后的票列表。
- 按页检查。
- 对每一张票：
  - 点击 **Edit**。
  - 如果**没有 FAE 标签页**，点击 **Cancel**（除非用户明确要求直接 Update），然后跳过。
  - 如果有 **FAE** 标签页，点击 **FAE**。
  - 检查以下字段是否为空：
    - `FAE_Label`
    - `FAE Pattern`
    - `Comment`
  - 只要有任一项为空，就记下票号。
  - 点击 **Cancel**。
- 翻页继续，直到所有页面检查完成。
- 最终只汇报缺项票号，以及缺的是哪些字段。

### 2) Edit/update mode / 修改更新模式

Use this only when the user explicitly wants tickets updated.

If the task is edit/update mode:

- Open the Jira issue search/list page.
- Before starting, apply this JQL for the current user's own reported tickets:
  `created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`
- If the filter area is not directly editable, click **Advanced** first, then paste/type the JQL directly.
- Only after this filter is applied should bulk processing begin.
- Work ticket by ticket.
- Click **Edit**.
- **Always click the FAE tab** before touching any FAE-related field.
- If any of these already contains content:
  - `FAE_Label`
  - `FAE Pattern`
  - `Comment`
  then skip rewriting and click **Update** only if the user requested this behavior.
- Otherwise, fill the required FAE fields and write a ticket-specific English technical comment.
- Click **Update**.
- Do **not** close the browser after update.

只有在用户明确要求修改票时才进入这个模式。

- 打开筛选后的票列表。
- 逐票处理。
- 点击 **Edit**。
- **必须先点 FAE 标签页**，再操作 FAE 相关字段。
- 如果以下任意字段已经有内容：
  - `FAE_Label`
  - `FAE Pattern`
  - `Comment`
  则按用户规则决定是否跳过重写并直接 **Update**。
- 如果为空，再填写 FAE 字段，并写一条与该票内容对应的英文技术 comment。
- 点击 **Update**。
- Update 后**不要关闭浏览器**。

## Critical FAE handling rules / FAE 关键规则

### Always click FAE first / 必须先点 FAE

Never assume the visible section is already the FAE form. Always click the **FAE** tab before checking or editing FAE fields.

不要假设当前看到的就是 FAE 表单。检查或修改前，必须先点击 **FAE** 标签页。

### FAE_Label is not plain text / FAE_Label 不是普通文本框

`FAE_Label` is a label picker.

Correct method:

1. Type the intended label into the `FAE_Label` input.
2. Wait for the candidate / suggestion list.
3. Click the focused suggestion (or create-new item if needed).
4. Wait briefly.
5. Only then click **Update**.

Do **not** just assign text and immediately update.

`FAE_Label` 是标签选择控件。

正确做法：

1. 在 `FAE_Label` 输入目标标签。
2. 等候选项出现。
3. 点击聚焦的候选项（如果没有现成项，则点创建新标签项）。
4. 稍等一下。
5. 然后再点 **Update**。

不能只填字符串后立刻 Update。

### No spaces in created labels / 新建标签不要有空格

If Jira suggests creating a new label, do not use labels with spaces.

Examples:

- Good: `notification`
- Bad: `Technical Document`

如果 Jira 提示创建新标签，不要使用带空格的标签。

例如：

- 正确：`notification`
- 错误：`Technical Document`

### FAE_Label selection must follow ticket content / FAE_Label 必须跟票内容走

Do not overgeneralize `safellink`.

Use these remembered rules:

- Use `safellink` only for tickets involving **TCC5110** and **SDM**.
- Use `CarPlay` for CarPlay-related tickets when appropriate.
- Use a content-matching label for other ticket types.
- Use `notification` for notice/announcement-type tickets when a new label is needed.

不要把 `safellink` 当默认值。

记住这些规则：

- 只有涉及 **TCC5110** 和 **SDM** 的票，才使用 `safellink`。
- CarPlay 相关票可使用 `CarPlay`。
- 其他票的 `FAE_Label` 必须根据票内容来选。
- 通知/公告类票如果需要新建标签，可使用 `notification`。

### FAE_Label and FAE Pattern are different / FAE_Label 和 FAE Pattern 不是一回事

- `FAE_Label` = concrete label/tag
- `FAE Pattern` = categorized pattern/classification

Never use the pattern text as if it were the label.

- `FAE_Label` = 具体标签
- `FAE Pattern` = 分类/模式

不要把 pattern 文本误当成 label。

## Known examples / 已确认示例

### Correct edit example

A known-good example was:

- Ticket: `TANCS5-29`
- `FAE_Label = carplay`
- `FAE Pattern = SW / General`
- Write an English technical comment matching the ticket content
- Click **Update**
- Stop there, do not close browser

已确认的正确示例：

- 票号：`TANCS5-29`
- `FAE_Label = carplay`
- `FAE Pattern = SW / General`
- 写与票内容匹配的英文技术 comment
- 点击 **Update**
- 完成后停止，不关闭浏览器

## Pagination / 翻页

For bulk operations:

- Finish the current page first.
- Then click the next page number explicitly (for example `2`, then `3`).
- Keep track of checked pages and checked ticket count.

批量处理时：

- 先完成当前页。
- 再明确点击下一页页码（例如 `2`、再 `3`）。
- 记录已经检查过的页数和票数。

## Reporting format / 汇报格式

When returning results from an audit, include:

1. Check time
2. Filter time range
3. Filter/JQL used
4. Total pages / total tickets if known
5. Skipped tickets with no FAE section
6. Tickets missing one or more FAE fields

检查类结果建议包含：

1. 检查时间
2. 检查条件时间范围
3. 使用的筛选条件 / JQL
4. 总页数 / 总票数（如果已知）
5. 没有 FAE 区域而跳过的票
6. 缺少一个或多个 FAE 字段的票

## References / 参考文件

Read these when needed:

- `references/usage-en.md` for English operator guidance
- `references/usage-zh.md` for Chinese operator guidance
