---
name: dev-autoflow
description: 开发任务自动流编排。当用户给出开发类需求（做网页/工具/脚本/应用/数据处理/报告/文档等），需要"需求重构 → 通道路由 → 按验收标准交付"一条龙处理时使用。与默认行为的差异：①强制用 scripts/spec_check.py 做规格硬校验（缺字段/验收不可测直接拦截）；②先查 references/case-library.md 复用历史任务模式，交付后沉淀新案例；③按 references/channel-map.md 的通道红线路由（本环境特有约束）。适用于需求模糊需拆解、多环节开发需先梳理、或需要在多个执行能力间做分发决策的任务。
---

# dev-autoflow 开发自动流

## 本 skill 的差异点（没有它就不会发生的事）

1. **规格硬校验**：规格必须跑 `scripts/spec_check.py` 通过后才允许开工——不是"检查过"，是脚本拦截。
2. **经验复用与沉淀**：开工前先查 [`references/case-library.md`](references/case-library.md)，有相似历史案例就复用其模式；交付后追加新案例。skill 越用越准。
3. **通道红线路由**：按 [`references/channel-map.md`](references/channel-map.md) 判定该不该用某通道——本环境的真实约束（pwsh 非 bash、html/app-builder 分流、CDN 红线、sheet 首调规则），不查就会用错工具。

## 流水线

```
查案例库 → 写规格 → 跑硬校验 → 定通道 → 执行 → 验证交付 → 沉淀案例
```

## Phase 1：需求重构

1. **先读** [`references/case-library.md`](references/case-library.md)——有相似案例直接复用其规格与通道，不重复推理。
2. **再读** [`references/spec-template.md`](references/spec-template.md)，按契约字段写规格。
3. 关键信息缺失且实质影响方案 → `interaction.ask` 一次问清（≤3 问）；低影响项直接假设并写入【待确认项】。
4. **跑硬校验**：
   ```
   python scripts/spec_check.py <规格文件>
   ```
   退出码非 0 → 补齐后重跑。**通过前禁止开工。**

## Phase 2：通道路由

1. **读** [`references/channel-map.md`](references/channel-map.md) 选通道。
2. 判据优先级：**该不该用（边界/红线）> 会不会用**。误路由 = 返工。
3. 委派 OrganizeAgent 的阈值按系统委派规则判定，不在此重复推演。

## Phase 3：执行与验证

1. 按规格执行，逐条对照验收标准。
2. 交付前验证：代码跑测试/真实入口；网页渲染检查；表格重算；文档结构与来源一致。
3. 交付说明写明：验证方式、覆盖范围、已知缺口。
4. 交付后偏差 → 回 Phase 1 更新规格 → 重跑受影响环节，不原地打补丁。
5. **沉淀案例**：把本次任务追加到 [`references/case-library.md`](references/case-library.md)（需求要点/通道/结果/教训）。

## 资源

- [`references/spec-template.md`](references/spec-template.md)：需求规格契约字段。Phase 1 必读。
- [`references/case-library.md`](references/case-library.md)：历史任务经验库。Phase 1 前读、Phase 3 后写。
- [`references/channel-map.md`](references/channel-map.md)：本环境通道边界与红线。Phase 2 必读。
- [`scripts/spec_check.py`](scripts/spec_check.py)：规格硬校验器。Phase 1 必跑。

**成本纪律**：只读当前阶段资产；规格定稿不重复生成；能直接做的不委派；案例库只追加不重写。
