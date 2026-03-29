# Jira FAE Tickets Skill - 中文使用说明

## 目的

这个 skill 用于 Telechips TITAN Jira 的两类工作：

1. **检查模式**：检查别人筛选后的票，找出哪些票缺少 FAE 内容。
2. **修改模式**：处理自己筛选后的票，填写 FAE_Label、FAE Pattern、Comment。

## 使用前提

- 用户必须已经登录 TITAN Jira。
- 登录态必须存在于 **OpenClaw 可控制的同一个 Chrome 浏览器** 中。
- 需要处理的票列表必须能在该浏览器标签页中打开。
- 如果是大批量检查，要预留足够时间给多页扫描。

## 典型请求示例

### 检查模式示例

- “帮我检查这个 reporter 的票，看看哪些票缺 FAE 字段。”
- “用这个 JQL，把所有页都检查一遍，只告诉我 FAE_Label、FAE Pattern、Comment 为空的票。”
- “不要改票，只做审计检查。”

### 修改模式示例

- “继续把剩下的票 FAE 填完。”
- “打开每张票，进入 FAE，填 label / pattern / comment，然后 update。”
- “如果 FAE 区域已经有内容，就不要重写，直接 update。”

## 操作关键点

### 检查模式

- 进入 Edit 只是为了检查，检查完就 Cancel。
- 不要填写，不要改内容。
- 没有 FAE 标签页的票直接跳过。
- 记录票号和缺失字段。

### 修改模式

- 必须先点击 FAE 标签页。
- `FAE_Label` 输入后必须点真实候选项。
- 点候选后稍等一下再 Update。
- Comment 必须和该票内容匹配，不能所有票都写同一句。

## 标签规则

- `safellink`：只用于 TCC5110 和 SDM 相关票。
- `CarPlay`：用于 CarPlay 相关票。
- `notification`：用于通知/公告类票，且需要新建标签时。
- 其他标签必须根据票内容决定，不能乱套。

## 建议汇报格式

建议检查结果至少包含：

- 检查时间
- 检查条件时间范围：2025-01-01 到 2026-03-27
- 使用的 JQL
- 已检查页数 / 总票数
- 没有 FAE 标签页而跳过的票
- 缺少 FAE 字段的票
- 每张缺失票明确写出缺的是：`FAE_Label`、`FAE Pattern`、`Comment`
