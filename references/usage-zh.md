# 使用指南 — 中文

## 快速开始

1. 确认 Chrome 已打开，TITAN 已登录。
2. 提供审计用的 JQL（reporter、日期范围）。
3. 告知 Claude 当前是**只检查模式**还是**修改更新模式**。
4. 只检查模式：Claude 会同时检查 Field 标签页和 FAE 标签页。
5. 修改更新模式：Claude 只修改 FAE 标签页字段（需明确指示）。

## 审计范围

### Field 标签页（8 个字段）
- O/S（操作系统）
- Self Resolution（自主解决情况）
- Cause(Customer)（客户原因分类）
- Hardware Issue Pattern（硬件问题模式）
- Software Issue Pattern（软件问题模式，只检查第一个）
- Labels（标签）
- FAE Person（负责 FAE）
- git/repo command（Git/Repo 命令）

### FAE 标签页（5 个字段）
- FAE_Label
- FAE Pattern
- Comment
- SDK Version (TITAN)
- Ref. H/W version

在审计模式下，`SDK Version (TITAN): None` 和 `Ref. H/W version: None` 也要判定为缺失。

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

## 工作模式说明

### 只检查模式（推荐用于审计他人的票）
- Claude 只读取，不修改票。
- 同时检查 Field 标签页和 FAE 标签页的所有字段。
- 返回结构化的审计报告。
- 大批量审计时，优先使用 REST API 或其他高效率认证检查方式。
- 没有 FAE 标签页的票直接跳过。
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
- 跳过的票列表（没有 FAE 标签页的票）
- 明确标出 `SDK Version (TITAN)` 和 `Ref. H/W version` 为空或 `None` 的情况

## 注意事项

- 大批量审计（50 张以上）时，优先使用 REST API 方式，比浏览器点击快得多。
- 首次使用前，先调用 `GET /rest/api/2/field` 确认各字段的 customfield ID。
- Software Issue Pattern：如果存在多个同名字段，只检查**第一个**。
- Labels 是 Jira 内置字段，直接使用 `labels`，无需 customfield ID。
- `safellink`：只用于 TCC5110 和 SDM 相关票。
- `CarPlay`：用于 CarPlay 相关票。
- `notification`：用于通知/公告类票，且需要新建标签时。
