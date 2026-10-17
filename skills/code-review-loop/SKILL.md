# Code Review 循环

## Purpose
Agent 写代码量上来后的评审循环：机器抓问题，人只看机器抓不住的东西。七层：PR 前自验 → AI review → 安全 review → 自动检查 → 人工闸门（风险分级）→ 合并队列/灰度 → 快速发现与回滚。
当用户说"过一遍 code review"、"这个 PR 走评审循环"时，用本 skill。
完整版 spec：`../specs/CODE_REVIEW_LOOP_SPEC.md`（v1.0 草案）。

## 七层速查
1. **自验**：Engineer 跑测试/build/截图拿输出证明；subagent fresh eyes 看 diff；读 `CLAUDE.md` + `REVIEW.md`。
2. **AI review**：多 agent 并行找 bug → 投票共识才发；分级 Important/Nit/Pre-existing；每个 finding 必须回应；3 轮不收敛叫停。
3. **安全**：finding 不清零不许合并。
4. **自动检查**：lint/类型/测试/build 全绿；PR 模板填"怎么测的、怎么 undo"。
5. **人工闸门**：低风险 1 小时无异议自动合并；中风险 1 人批；高风险 2 人批；approval 只覆盖当前 commit；agent 的 approval 不算数。
6. **合并队列/灰度**：对 main 重跑检查；新行为走 flag 灰度。
7. **回滚**：flag 免部署关；incident 复盘沉淀为 reviewer 每日 eval。

## 关键纪律
- 定级看改动不看作者；敏感路径常驻高 tier。
- 谁 prompt 的 agent，谁对 PR 负责。
- finding 门槛宁高勿低：agent 会把每个 nit 都认认真真修一轮。
- 每次 incident 只问：7 层里哪一层本该抓住它。

## 与其他 loop 的衔接
- bug loop：插在"修 → fuzz"之间（fuzz 抓 crash，本 loop 抓逻辑）。
- dev loop：插在"PR → QA 验收"之前（本 loop 不验 AC）。
- refine loop 是本 loop 的子循环形态。
