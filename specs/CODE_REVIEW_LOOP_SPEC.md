# 代码评审循环 Spec（v1.0 草案）

> 来源：Anthropic Claude Code 团队官方博客《How the Claude Code team does code review》（2026-10），结合本团队 2026-10-15 定稿的风险分级纪律改写。
> 状态：v1.0 草案，待主人拍板 §11 三项事项。

## 1. 目标与非目标

- 目标：agent 写的代码量上来后，评审不靠人逐行读：机器抓问题，人只看机器抓不住的东西；revert 率不升的前提下把人力压到最低。
- 非目标：不决定"做不做"（那是 roundtable / PM 的事）；不替代 QA 的 AC 验收（那是 dev loop 的事）；P0 事故走人工应急。

## 2. 七层总览

```
L1 PR 前自验 → L2 AI review → L3 安全 review → L4 自动检查 → L5 人工闸门（风险分级）→ L6 合并队列/灰度 → L7 快速发现与回滚
```

前四层机器，后两层是合并后的兜底，人只在 L5 按风险出现。

## 3. 各层 Spec

### L1 PR 前自验（Engineer）
- 给 agent 可运行的东西：测试、build、截图/录屏。不说"做完了"，拿输出证明。
- subagent 用 fresh context 看一遍 diff。
- 配套 skill：找 bug、精简 diff、端到端行为检查；UI 改动对照设计规范。
- 约束文件：`CLAUDE.md`（项目是什么、技术栈、约定；reviewer 也读，代码违约定要标出来）、`REVIEW.md`（见 §5）。

### L2 AI review（每个 PR）
- 多个 agent 并行找 bug → 另一组投票 → 达成共识的 finding 才发出来。只谈风格/清理的 finding 直接丢掉。
- finding 分级：Important / Nit / Pre-existing（PR 没引入的老问题不算作者的锅）。
- 每个 finding 必须有回应：修掉，或写理由 close。修了 → 自动进下一轮 re-review。
- 收敛规则（防 agent 互磨几十轮）：3 轮后 reviewer 要求作者停下来重想方案；之后只看 diff 增量；标 bug 的类别让作者修整类问题，不许只修一行；不重复发已发过的 finding。

### L3 安全 review（可阻塞）
- bug review 或安全 review 的 finding 还有没关的，PR 不许合并。修掉或写短理由 close（false positive 就这么清）。

### L4 自动检查（硬门槛）
- lint、类型、测试、build，全绿。PR 模板必填：怎么测的、怎么 undo（回滚方案）。"测过"这个字由人来签。

### L5 人工闸门（风险分级，沿用 10-15 定稿）
- 低风险（文档、文案、内部功能、flag 后的加法变更）：自动 review + 安全绿灯后 1 小时无异议 → 自动合并。
- 中风险（普通产品变更、缓存 / auto-updater / 系统提示词等敏感区）：1 人批。
- 高风险（鉴权、支付、数据迁移、生产配置、CI workflow 文件）：2 人批（单人团队落法见 §11 待拍板）。
- 定级看改动不看作者：分类器/规则定 tier；敏感路径清单常驻高 tier。
- approval 只覆盖当前 commit：新 commit 进来（agent push、手改 merge）→ 重新批。
- agent 的 approval 不算数；谁 prompt 的 agent，谁就对这个 PR 承担 reviewer 责任。

### L6 合并队列 + 灰度
- 合并队列对 main 重跑全部检查，防两个各自全绿的 PR 合起来炸 main。
- 新行为走 flag：先灰度内部/小流量，无回归再全量。

### L7 快速发现与回滚
- flag 可免部署关闭；on-call 轮值；revert 看板。
- 每次 incident 复盘只问一件事：7 层里哪一层本该抓住它 → 沉淀为 eval（§4）。

## 4. Reviewer 改进闭环

- incident → eval：收集历史上出过事的 PR（含 fix-forward 和 revert），reviewer 每天跑一遍；改 reviewer 先过 eval。
- 噪音度量：每个 comment 按人的处置分类——修了 / 说错了 / 故意这么写的 / 无所谓；另跑一批已知干净的 PR，数误报导致 harmful 改动的次数。目标优先级：先抓真 bug，再减轮次。
- 噪音贵在 agent 作者：人 1 秒 dismiss 的 nit，agent 会认认真真修一轮 → 又触发一轮 review。所以 finding 门槛宁高勿低。

## 5. 标准前置：REVIEW.md 与 PR 模板

- 每个仓库一份 `REVIEW.md`（code owner 写）：这块代码"坏"长什么样、nit 上限、跳过路径、finding 必须带文件行号证据、summary 开头先报数、没问题就直说没问题。
- 单独抽查 reviewer 有没有真按 `REVIEW.md` 走。
- PR 模板：改了什么、怎么测的（证据链接）、怎么 undo、风险自评。

## 6. 人留哪五件事

1. 做不做：方向、品味、范围（跨团队变更保留人工"这事该做吗"）。
2. 风险代码：计费、auto-updater、CI workflow。
3. 架构：会被别人抄的模式。
4. 读证据：测试输出、命令回显、截图、reviewer findings——说不清检查会漏什么就别快批。
5. 背锅：每个上线的改动都有人负责，谁 prompt 的谁负责。

## 7. 与其他 loop 的关系

- bug loop 内环"修 → fuzz"之间插入本 loop L1–L5（fuzz 抓 crash，本 loop 抓逻辑和风格）。
- dev loop 内环"PR → QA 验收"：本 loop 先过，QA 再按 AC 验（本 loop 不验 AC）。
- refine loop 是本 loop 的子循环形态（生成 → 评判 → 回炉）；pass^k 是评判加连续性要求的变体。
- 链条顺序：roundtable 定方案 → dev loop 执行 → 本 loop 评审。

## 8. 失败处理

| 场景 | 处理 |
|---|---|
| reviewer 连续 3 轮还在抓老问题 | 停，升级给人看 approach 是否有问题 |
| finding 无人回应超 24h | 升级 |
| L5 审批超时（中/高风险） | 视为过期，不执行，重新请求 |
| 回滚 | flag 先关 → revert → incident 复盘进 eval |
| reviewer 误报导致 harmful 改动 | 记一笔，调高 finding 门槛 |

## 9. 可观测性

- 每 PR 留：各层 finding、投票记录、回应、轮次、审批人/tier。
- 周指标：人均 review 耗时（approval 数/评论数/等人时长）、revert 率、finding 准确率（"修了"占比）、平均收敛轮次。

## 10. 七步框架对齐

| 步骤 | 本 spec 覆盖 |
|---|---|
| 1 职责/数据源/格式/边界 | §2–§6 |
| 2 真实任务跑一次 | ☐ 待：拿一个真实 PR 跑通 7 层 |
| 3 修正到可审查 | §9 |
| 4 存为 Skill | ✅ `skills/code-review-loop/SKILL.md` |
| 5 第二份输入测试 | ☐ 待 |
| 6 重试/失败定义 | §8 |
| 7 对外动作审批 | §3-L5（风险分级审批） |

## 11. 待拍板

1. 高风险"2 人批"在单人团队里怎么落："他 + 固定第二双眼睛（指定人）"还是"他看两遍算两轮"？
2. L2 的 reviewer agents 用谁：QA bot 兼任，还是另起 reviewer agents（成本翻倍）？
3. 灰度 flag 机制现在没有：先只做"合并队列 + 人工观察期"，还是把 flag 机制也立项？
