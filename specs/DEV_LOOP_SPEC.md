# 研发循环 Spec（v1.0）

> 来源：2026-10-15 用户确认的研发 loop 描述，对标 bug-fix loop 的三层结构。
> 状态：v1.0。

## 1. 目标与非目标

- 目标：需求 → 上线，全程自动化，人只在两个闸门拍板（cut 确认、发布）。
- 非目标：bug 修复（走 bug-fix loop）、线上 P0 事故应急（走人工通道）。

## 2. 三层总览

```
外环（分诊立项）：需求输入 → 幕僚长分诊 → PM 写 binding dated cut → 人确认 cut
内环（执行验收）：Engineer 执行 → PR → QA 按 AC 验收 → 不通过回修
闸门（发布）：QA PASS → 人拍板 → 发布 → 接下一条
```

## 3. Cut Schema（PM 输出固定格式）

binding dated cut 包含：
- dated in/out：起止日期，有约束力；
- 编号 AC：验收标准，每条带证明类型（演示 / 测试 / 文档 / 数据）；
- 范围边界：明确不做什么；
- 排期里程碑。

## 4. 外环 Spec（分诊立项）

执行者：幕僚长（分诊）+ PM bot。

1. 输入：需求（一句话 / 文档 / Linear issue），来源：他 / Slack / Linear。
2. 分诊：去重（已有类似需求则关联原单）、分级（业务优先级 P0–P3）、分流（bug → bug loop，提问 → 直接回，需求 → 立项）。
3. PM 按 §3 写 cut。需求含糊 → 打回需求方澄清（对标 bug loop 的 needs-info），48h 无回应 → 升级。
4. 人确认 cut = 第一闸门：10 分钟未回复=过期，不开工，重新请求。确认后状态 → `进行中`，Engineer 开工。

## 5. 内环 Spec（执行验收）

执行者：Engineer Bot + QA bot。

### 执行
- Engineer 按 cut hands-off 执行，开 PR；done = merged。
- 无 binding cut 不开工（QA 判 BLOCKED）。

### 验收
- QA 按 cut 里 dated AC 逐项验收，verdict 只用 `PASS / FAIL / BLOCKED`。
- PASS → `待发布`；FAIL → 回 Engineer（保持 `待验收`）；BLOCKED → 备注卡点。
- FAIL 回修计数，3 次不过 → 升级给人。
- 中/高风险 PR 的 diff 走人工 review。

### 看板
`待分诊 → 进行中 → 待验收 → 待发布 → 已完成`；分支 `阻塞`。

## 6. 发布闸门 Spec

- 前置：QA PASS。
- 人拍板：发布 = 对外动作，一律审批，无自动合并。10 分钟未回复=过期，不得执行，重新请求。
- 发布后：Linear 关单（`已完成`），通知相关方，按优先级接下一条。
- tinkabot 按需做发布后的插件化沉淀（API → MCP + skills）。

## 7. 失败处理与升级矩阵（稳定 Bot 第 6 步）

| 场景 | 处理 |
|---|---|
| 需求含糊 | 打回澄清；48h 无回应 → 升级 |
| QA FAIL | 回修；3 次不过 → 升级给人 |
| BLOCKED 超 24h | 幕僚长升级上报 |
| cut 确认超时 | 不开工，重新请求 |
| 发布失败 | 按回滚预案执行 → 升级 |

## 8. 队列纪律

- 按业务优先级排序，同级按创建时间 FIFO；一次只做一条。
- bug 与需求互相踢单时，分诊阶段定死归属，不许踢皮球。

## 9. 可观测性（可审查）

- 每单留：cut、PR、AC 验收记录、发布记录、失败原因。
- 周指标：需求 lead time、一次 PASS 率、回修次数分布、发布成功率。

## 10. 与 bug loop 的关系

同一哲学，四处对应：杀歧义（复现 / binding cut）、执行（PR）、对抗验收（fuzz / AC 验收）、打回（needs-info+迭代上限 / 澄清+FAIL 计数）。
四处差异：闸门 1 个 vs 2 个；终点合并 vs 发布；验收依据 fuzz 干净 vs AC 全过；bug 按 severity FIFO，研发按业务优先级。

## 11. 七步框架对齐

| 步骤 | 本 spec 覆盖 |
|---|---|
| 1 职责/数据源/格式/边界 | §2–§6 |
| 2 真实任务跑一次 | ☐ 待：等第一个真实需求 |
| 3 修正到可审查 | §9 |
| 4 存为 Skill | ✅ `~/workspace/custom-skills/dev-loop/SKILL.md` |
| 5 第二份输入测试 | ☐ 待 |
| 6 重试/失败定义 | §7 |
| 7 对外动作审批 | §4 cut 确认闸门 + §6 发布闸门 |
