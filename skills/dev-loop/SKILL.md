# 研发循环

## Purpose
需求 → 上线的全自动研发循环：分诊立项 → 执行验收 → 发布。
当用户说"这个需求进研发 loop"、"按 dev loop 跑"或有新需求要立项开发时，用本 skill。
完整版 spec：`../specs/DEV_LOOP_SPEC.md`（v1.0）。
bug 修复走 `bug-fix-loop`，P0 事故走人工应急，不要混用。

## 三层总览
```
外环（分诊立项）：需求输入 → 幕僚长分诊 → PM 写 binding dated cut → 人确认 cut
内环（执行验收）：Engineer 执行 → PR → QA 按 AC 验收 → 不通过回修
闸门（发布）：QA PASS → 人拍板 → 发布 → 接下一条
```

## 外环：分诊立项
1. 输入：需求（一句话/文档/Linear issue）。
2. 幕僚长分诊：去重（关联已有相似需求）、分级（业务优先级）、分流（bug → bug loop，提问 → 直接回）。
3. PM 写 binding dated cut：dated in/out + 编号 AC（带证明类型：演示/测试/文档/数据）+ 范围边界 + 排期。
4. 需求含糊 → 打回澄清，48h 无回应升级。
5. 人确认 cut = 第一闸门：10 分钟未回复=过期，不开工，重新请求。确认后 Engineer 开工。

## 内环：执行验收
- Engineer 按 cut hands-off 执行，开 PR；无 cut 不开工；done = merged。
- QA 按 cut 里 dated AC 逐项验收，verdict 只用 PASS / FAIL / BLOCKED。
- PASS → 待发布；FAIL → 回 Engineer（保持待验收），3 次不过升级；BLOCKED → 备注卡点。
- 中/高风险 PR diff 走人工 review。
- 看板：待分诊 → 进行中 → 待验收 → 待发布 → 已完成；阻塞。

## 发布闸门
- 前置：QA PASS。发布=对外动作，一律审批，无自动合并。
- 10 分钟未回复=过期，不得执行，重新请求。
- 发布后：Linear 关单，通知相关方，按优先级接下一条；tinkabot 按需插件化沉淀。

## 失败升级矩阵
| 场景 | 处理 |
|---|---|
| 需求含糊 | 打回澄清；48h 无回应 → 升级 |
| QA FAIL | 回修；3 次不过 → 升级 |
| BLOCKED 超 24h | 幕僚长升级上报 |
| cut 确认/发布审批超时 | 不执行，重新请求 |
| 发布失败 | 回滚预案 → 升级 |

## 可观测性
每单留：cut、PR、AC 验收记录、发布记录、失败原因。周指标：lead time、一次 PASS 率、回修分布、发布成功率。
