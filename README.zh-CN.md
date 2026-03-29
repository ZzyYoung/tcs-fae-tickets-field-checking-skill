# Titan FAE Tickets Field Checking Skill

这是一个用于处理 Telechips TITAN Jira 中 FAE 工作流的 skill。

## 这个 skill 做什么

它主要支持两类实际场景：

1. **FAE 字段修改模式**
   - 从 Jira 的筛选条件 / JQL 结果中打开票
   - 进入 **FAE** 标签页
   - 填写或修改：
     - `FAE_Label`
     - `FAE Pattern`
     - `Comment`
   - 安全地更新票内容

2. **FAE 审计检查模式**
   - 检查别人 reporter 名下的票
   - 验证以下字段是否为空：
     - `FAE_Label`
     - `FAE Pattern`
     - `Comment`
   - 没有 FAE 标签页的票直接跳过
   - 最终只返回缺项票号和缺失字段

## 使用前提

使用前必须满足：

- 你已经登录了 Telechips TITAN Jira
- 登录态存在于 **OpenClaw 可以控制的同一个 Chrome 浏览器** 中
- Jira 的筛选页面或票列表能在该浏览器中打开
- 批量处理时，请保持浏览器窗口打开并稳定

## 修改本人票前的启动规则

如果是批量处理用户本人 reporter 的票，开始前先在 Jira 中应用这条 JQL：

`created >= 2025-01-01 AND created <= 2026-03-27 AND reporter in (currentUser()) order by created DESC`

如果当前模式下筛选框不能直接编辑，就先点击 **Advanced**，再进入直接输入模式填写这条 JQL。

## FAE 关键规则

- 检查或修改 FAE 前，必须先点 **FAE** 标签页
- `FAE_Label` 是标签选择器，不是普通文本框
- 输入标签后，必须：
  1. 等待候选项出现
  2. 点击聚焦的候选项或创建项
  3. 稍等一下
  4. 再点击 **Update**
- 如果需要创建新标签，不要使用带空格的标签
  - 正确：`notification`
  - 错误：`Technical Document`

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
- 没有 FAE 标签页而跳过的票
- 缺少 FAE 字段的票

## 仓库包含文件

- `SKILL.md` — 主 skill 说明
- `references/usage-en.md` — 英文使用说明
- `references/usage-zh.md` — 中文使用说明
- `jira-fae-tickets.skill` — 打包后的 skill 文件

## English version

See: [README.md](README.md)
