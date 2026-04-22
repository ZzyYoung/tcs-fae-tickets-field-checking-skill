# Titan FAE Tickets Field Checking Skill

这是一个用于 Telechips TITAN Jira 票据审计与更新的 OpenClaw skill，覆盖 **FAE** 标签页和 **Field** 标签页。

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
- `SDK Version (TITAN)`
- `Ref. H/W version`

在审计模式下，`SDK Version (TITAN): None` 和 `Ref. H/W version: None` 也视为缺失。

### Field 标签页
- `O/S`
- `Self Resolution`
- `Cause(Customer)`
- `Hardware Issue Pattern`
- `Software Issue Pattern`（只检查第一个）
- `Labels`
- `FAE Person`
- `git/repo command`

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
- `FAE_Label` 是标签选择器，不是普通文本输入框
- 创建或选择标签时，要等待候选项出现并选中目标项，再点击更新
- 大批量只读检查时，优先使用更快的已认证检查方式

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
- 任一标签页存在缺失字段的票
- 明确标出 `SDK Version (TITAN)` 和 `Ref. H/W version` 为空或 `None` 的情况

## 最近更新

- FAE 标签页审计范围从 3 个字段扩展为 5 个字段
- 新增 `SDK Version (TITAN)` 和 `Ref. H/W version` 两个检查项
- 明确这两个 FAE 字段在值为 `None` 时也视为缺失
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
