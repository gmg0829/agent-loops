# Agent Loops

一套给 AI 工程团队用的循环工作法：外环分诊、内环执行、人工闸门。五个 loop，同一套哲学。

## 五个 loop

| Loop | 管什么 | 一句话 |
|---|---|---|
| [BUG](specs/BUG_LOOP_SPEC.md) | 修 bug | Slack 反馈 → 复现 → 修 → fuzz → 人工闸门 → 合并 |
| [DEV](specs/DEV_LOOP_SPEC.md) | 做需求 | 需求 → binding cut → 执行 → QA 验收 → 人拍板发布 |
| [PATROL](specs/PATROL_LOOP_SPEC.md) | 巡逻 | 每 15 分钟扫一遍责任面，异常轻处置或升级，正常静默 |
| [ROUNDTABLE](specs/ROUNDTABLE_LOOP_SPEC.md) | 议事 | 多 agent 背对背提议、交叉批评、收敛，僵持升级 |
| [REFINE](specs/REFINE_LOOP_SPEC.md) | 打磨 | 生成 → 按标准评判 → 回炉，超 3 次升级 |

每个 loop 目录下有完整 spec（含状态机、失败处理矩阵、决策记录），`skills/` 下是对应的 skill 精简版（执行手册）。

## 共同哲学

1. **杀歧义前置**：bug loop 用"复现"杀歧义，dev loop 用 binding cut（起止日期 + 编号验收标准 + 明确不做什么）杀歧义，roundtable 要求议题写成"选 A 还是 B"。
2. **对抗验收**：fuzz 干净（10 分钟 / 1 万次零 crash）、AC 逐项验收、交叉批评专职挑刺，执行者不验收自己。
3. **迭代有上限**：fuzz 回修 3 次、QA FAIL 回修 3 次、打磨 3 次、roundtable 3 轮，超限一律升级给人，不许无限打转。
4. **人只出现在闸门**：bug loop 1 个闸门（合并审批），dev loop 2 个闸门（cut 确认、发布），roundtable 2 处介入（僵持仲裁、最终拍板），patrol 不碰对外动作。
5. **默认项分级**：低风险单默认执行（1 小时无异议自动合并），中高风险默认不执行（10 分钟未回复=过期）。

## 基础：稳定 Bot 七步

所有 bot 和 routine 上线前走完 [BOT_STABILITY.md](BOT_STABILITY.md)：职责写进 Description → 真实任务跑一次 → 修正到可审查 → 存为 Skill → 第二份输入测试 → 定义重试失败 → 对外动作一律审批。

## 状态

v1.0（2026-10-15 定稿）。五个 spec 均为待真实任务验证：bug loop 待一条真实反馈跑通，dev loop 待第一个真实需求，patrol 待跑一周看误报率，roundtable 待一个真实议题，refine 待一篇真实产出。

## 目录

```
specs/    五个 loop 的完整 spec
skills/   五个 loop 的执行手册（skill 格式）
BOT_STABILITY.md  稳定 Bot 七步（团队标准）
```
