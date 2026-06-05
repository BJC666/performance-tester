# Changelog

## v1.9.0 (2026-06-05)

### Added — 元认知分析层（100 创新点精选 5）
- **🧠 Likely Thought Process** — 重构开发者思维链；标注偏离点；从"找错误"升级为"理解错误如何产生"
- **🪤 Self-Praise Density Scan** — 扫描自夸关键词密度；统计发现自夸密度与漏洞数正相关；代码越说"安全"越不安全
- **💎 Surface-Deep Decoupling** — 评分表面质量 vs 深层质量；脱钩指数 > 50 = "装修精美的危房"——审查者 30 秒 LGTM，实际需要 30 分钟深挖
- **🦠 Complexity Contagion** — 追踪一个函数的复杂度如何"传染"其调用方；修复感染源 → 调用方自动简化
- **🧟 Zombie Code Paths** — 检测结构上不可达但保留了开发者意图的代码路径（被 Exception 吞噬的 except、永远不触发的 callback）

### Changed
- Full Report Template 加入 🧠 Meta-Cognitive Analysis 区（Phase 3 之后，Advanced Dimensions 之前）
- Knowledge base 追加 META-001, META-002, ZOMBIE-001

## v1.8.0 (2026-06-05)

### Added — 陷阱模式识别
- **Comment-Code Contradiction Scanner** — Phase 2 新增注释真值校验；5 种矛盾检测规则；输出 🪤 标记
- **Wrong-Tool-for-Job Patterns** — 4 种"用重型机制做简单事"反模式 (Generator-as-Cache / WeakKeyDictionary-as-thread_local / id()-as-Cache-Key / Comment-vs-Code)
- **Vulnerability Library** 追加 VUL-008 (`id()` as key), VUL-009 (Generator as cache), BUG-004 (Comment contradiction)

## v1.7.0 (2026-06-05)

### Added — 持续学习系统
- **Continuous Learning Protocol** — 每次诊断前查 `references/vulnerability-library.md`，诊断后追加新模式
- **Vulnerability & Bug Pattern Library** — 初始化 12 个模式 (VUL-001~007, BUG-001~003, PERF-001~002)
- 库格式精简为每模式 3 行 (Signature + Fix + Severity)

## v1.6.1 (2026-06-05)

### Added — 强制总结区
- **Final Summary (📋)** — 每份报告末尾输出 🔒 Vulnerabilities + ⚠️ Drawbacks + 🔗 Root Cause Convergence

## v1.6.0 (2026-06-05)

### Added — 漏洞检测
- **8-type Vulnerability Detection** — Phase 2 节点发现新增 Race/SQL注入/连接泄漏/无界输入/缺超时/ReDoS/死锁/TOCTOU
- **RACE 因果边类型** — 共享可变状态无原子性 → 数据损坏 → 重试放大
- Full Report Template 加入 🔒 Vulnerability Detection 区

## v1.5.0 (2026-06-05)

### Changed — 输出结构去重
- 🔨 One Fix + ⚡ Quick Wins → 🔨 Priority Actions（合并）
- 📉 Failure Story → 并入 ⏳ Debt Interest 维度
- 🔗 Fix Order + ⏱️ Time-to-Value → ⏱️ Execution Plan（合并）
- 输出从 14 节减至 10 节

## v1.4.0 (2026-06-05)

### Added — 可执行行动计划 + 自批判 + 模式传播
- **Fix Propagation (Pattern Similarity Matching)** — 同模式全代码库检测；提取结构签名→扫描→报告传播目标；≥2 目标时触发 🔁 PATTERN PROPAGATION 报告
- **Adversarial Self-Challenge (Rule 6)** — Agent 必须尝试反驳自己的 top 2 建议；"此修复在 [场景] 下可能恶化"；仅 4 种修复类型可声明"无失败模式"
- **Time-to-Value Action Plan (Rule 7)** — 按 (impact×confidence)/effort_minutes 排序；5分钟/1小时/1天三级行动计划 + 每级累积健康分数
- **Quick Wins section** — ≤5 分钟零风险的修复列表
- **Fix Dependency Ordering** — 修复之间的先后依赖关系图
- **Monitoring Recommendations** — 每项修复后的告警阈值 + 仪表盘建议

## v1.3.0 (2026-06-05)

### Added — 自进化 + 漏洞修补 + 创新
- **Self-Calibration Protocol** — 三步校准门（Pattern Recall / Null-Case Smoke Test / Blind Spot Check），每次调用前自我诊断敏感度和盲点
- **Clean Bill of Health Protocol** — 空假设规则 + 模式分级阈值 + ≥2 边界条件强制输出 + 反造假规则
- **Telemetry Injection Protocol** — 运行时数据集成：因果图 [C]/[H]/[I] 节点标注 + 预算表 [measured]/[estimated] + Data-Poor Warning
- **Over-Engineering Traps** — 5 个过度工程陷阱（Cache Everything / Index Proliferation / Micro-Optimization / Over-Parallelization / Microservices）+ 自审计规则
- **Regression Risk column** — 修复表新增回归风险列 🟢🟡🔴 + 缓解措施
- **Fixability Gate** — P2 降级防躲：2x 规模测试 + TTI 交叉验证

### Fixed — 漏洞修补
- Iron Law 与 Quick Mode 矛盾：替换为模式特定需求表（6 列 × 3 模式）
- "If genuinely stuck" 逃逸口：替换为 MVD 最小可行维度表（5 维度 × 1 行最低分析）
- P2 降级躲代码：Fixability Gate 交叉验证 TTI 预测

## v1.2.0 (2026-06-04)

### Added
- **Output Quality Rules** — 5 mandatory rules governing report content quality:
  1. Concrete Fixes — every P0/P1 fix must include runnable code, not just descriptions
  2. Confidence-Annotated Estimates — all time estimates labeled HIGH/MEDIUM/LOW confidence
  3. The One Fix — single-line callout for the highest-impact fix
  4. The Failure Story — 2-3 sentence incident prediction for the key bottleneck
  5. Persona Emotional Variety — banned word reuse, required emotional palette rotation

## v1.1.0 (2026-06-04)

### Added
- **Output Mode Selector** — Quick (⚡) / Standard (📋) / Deep (🔬) three-tier output depth, auto-detected from user language intensity
- **Pipeline Checkpoints** — 4 🔴 CHECKPOINT markers between phases, each with explicit verification criteria and STOP conditions
- **Absolute Prohibitions** — 10-item "NEVER Do These" blacklist with catch-and-correct instructions
- Mode detection rules: intensity signal keywords for automatic mode selection

### Changed
- Core Pattern pipeline diagram: added output modes and checkpoint markers
- Version bumped to 1.1.0

## v1.0.0 (2026-06-04)

### Added
- **Phase 1: Budget Planning Engine** — 时间预算分解，弹性/固定层标注，健康评分公式
- **Phase 2: Causal Reasoning Engine** — 性能症状因果有向图，杠杆分排名算法，修复模拟
- **Phase 3: Persona Narrative Engine** — 请求人格创建，第一人称 Before/After 旅程叙事
- **Dimension 1: ⏳ Performance Debt Interest Rate** — 性能债务复利计算，TTI 预测
- **Dimension 2: 👻 Ghost Load Detection** — 幽灵载荷（死数据流）检测，6 种浪费模式
- **Dimension 3: 💣 Performance Blast Radius** — 爆炸半径映射，BR 评分公式
- **Dimension 4: 🕳️ Time Leak Detection** — 时间泄漏检测，6 种渐变退化模式
- **Dimension 5: 📐 Performance Entropy** — 香农熵应用于响应时间分布，Entropy-Latency 矩阵
- Iron Law enforcement — 三阶段 + 至少一维度强制运行
- Rationalization counter-table — 8 条防跳过话术
- Default performance targets table — 9 种上下文的默认基准
- Dimension selection guide — 5 维度的自动选择决策表
- Dimension–Causal Graph integration rules
- Handling User Context — 用户上下文合并机制
- Multi-entry-point file handling
- 10 Red Flags auto-detection
