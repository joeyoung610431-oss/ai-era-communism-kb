# 知识库本体建模 · 设计说明

> **版本**：KB-ONT v1.12（2026-11-26，赛博朋克关联层校准，新增正式机制 CP-44；`concepts.ttl` 已用 rdflib 解析并复算为 44 概念/36 关系，ABox/子图/graph 未重算）
> **性质**：非正式层资产（不占用正式簇命名空间，不计入 `_baseline26_calc.py` 口径）
> **当前副本机读文件**：`kb-ontology.ttl`、`concepts.ttl` 与 `chains/`；历史配套的 `instances.ttl`、`derived.ttl`、`tools/` 当前未随副本提供。下文 ABox 数量是历史构建记录，不是当前可运行结果。
> **定位**：用本体（Ontology）建模方法对**四层正式知识库**做概念级重构描述——不改动正式层任何一字，而是给正式层建立一张“形式化的骨架图”，使检索、对位纪律、入库纪律、一致性检查从“人读的规则”升级为“机器可校验的模型”。

---

## 一、建模目标（为什么建本体）

现有知识库的规则散落在 RULES.md、LEDGER.md、入口 §四扩展规则与 working memory 中，本质是**隐性约束**。本体建模的目标是把四类东西显式化：

1. **实体类型化**：论题簇、对手立场、反例、来源、锚点……目前靠标题前缀（`Txx-X`、`R-XXX`）和行文惯例区分，本体给出唯一类定义。
2. **关系显式化**："案例支撑论题簇""反例反驳论点""来源佐证案例"目前靠对应总表和`【依据：…】`标记表达，本体给出命名关系（object property）。
3. **纪律约束化**：把 RULES 的硬规则（L-E 不得入库、带数字散文行须附来源、态度/存量不可混说等）编码为 SHACL 约束，可被工具自动校验。
4. **检索结构化**：本体 + 实例化后，可回答"哪些簇引用了未经核验的来源""哪些命题尚无实例支撑"这类 SPARQL 级问题。

---

## 二、建模方法

采用**自顶向下 + 自底向上混合法**：

- **自顶向下**：借用轻量顶层分类（受 BFO 与 SKOS 启发，不引入完整 BFO 以免过度工程），划分"持续性实体 / 过程性实体 / 信息实体 / 元实体"四大顶层。
- **自底向上**：从正式层 9 文件的**实际内容单元**归纳类（42 论题簇、8 核心命题、85 组台账条目、14 簇具名对手、17 个待验证问题、5 条理论增量……），保证每个类都有真实实例，不造空类。
- **术语层**：概念（术语、理论观点）用 SKOS 语义建模，天然获得 `broader/narrower/related` 等现成推理能力。
- **约束层**：纪律规则用 SHACL（Shapes Constraint Language）表达，独立于类定义，便于随 RULES 演进而只改 shape 不改本体。

---

## 三、顶层分类与类层次（TBox）

```
kbont:Entity（根）
├── kbont:ContentUnit ············· 信息实体：正式层中可定位的内容单元
│   ├── kbont:TheoryWork ·········· 文献条目（L1 主文件 51 条 + 专题文献）
│   ├── kbont:TheoryClaim ········· 理论观点/依据（L1 内部论述，含【本文推论】标记者）
│   ├── kbont:Term ················ 术语（L1-C，当前 59 条；concepts.ttl 44 个义项级概念，非一一对应）
│   ├── kbont:CoreProposition ····· 核心命题（M1–M8，全库主轴）
│   ├── kbont:TopicAxis ··········· 主题轴（T01–T10）
│   ├── kbont:ThesisCluster ······· 论题簇（Txx-X，42 个）
│   ├── kbont:Argument ············ 论点（簇内"论点"段）
│   ├── kbont:Objection ··········· 反对意见（簇内"反驳预案"段）
│   ├── kbont:Response ············ 回应（对 Objection 的回应）
│   ├── kbont:AdversaryPosition ··· 对手立场（附录 D 具名对手的"外推版本"）
│   ├── kbont:Case ················ 制度实践案例（L3-A 案例 1-1 …）
│   ├── kbont:Counterexample ······ 反例（L3-A §五：反例 A/B/C）
│   ├── kbont:CaseIndicator ······· 观察指标（案例内带数字的表格行）
│   ├── kbont:ProseDataPoint ······ 带数字的散文行（J 段管控对象）
│   ├── kbont:Insight ············· 理论增量（L3-A §七：增量 1–5）
│   ├── kbont:CorpusSlice ········· 素材语料切片（data/corpus/T0x_*.txt）
│   └── kbont:OutputWork ·········· 成稿（reports/article-0x，4/42）
├── kbont:EpistemicUnit ··········· 认识/证据实体
│   ├── kbont:EvidenceSource ······ 外部来源（台账 R-XXX，85 组）
│   ├── kbont:Anchor ·············· 锚点（数据锚点 / 现象锚点 / 推论标记）
│   └── kbont:CrossCheck ·········· 交叉核验记录（LEDGER §八之二）
└── kbont:MetaUnit ················ 元实体：关于知识库自身的对象
    ├── kbont:OpenQuestion ········ 待验证问题（U1–U21）
    ├── kbont:ForkPath ············ 分叉路径（L1-B，3 条）
    ├── kbont:Switch ·············· 五开关（L1-B 总表）
    ├── kbont:MonitoringTarget ···· 监测对象（LEDGER §八）
    └── kbont:DisciplineRule ······ 纪律规则（RULES 各条，SHACL 化的载体）
```

**设计取舍说明**

- `AdversaryPosition` 单列而非并入 `Objection`：对位纪律第 12 条要求记录"对手本人的条件性表述 + 被登记的外推版本"，二者是不同对象——对手原文是 `TheoryClaim`（属 L1），其外推版本才是 `AdversaryPosition`，二者用 `kbont:extrapolatesFrom` 连接。
- `ProseDataPoint` 与 `CaseIndicator` 分开：表格块豁免 J 段规则、散文行不豁免——模型层面必须可区分，否则 SHACL 校验会误报。
- `Counterexample` 单列而非 `Case` 的子类：反例与正向案例在本库中的论证功能相反（"制度设计落地 ≠ 制度生效"），混类会导致"案例支撑簇"的默认语义出错。
- 锚点性质分层（方法论第 9 条）建模为 `Anchor` 的 `kbont:anchorNature` 数据属性（枚举：`factual / phenomenal / inference`），与"事实性→数据锚点、机制性→现象锚点、纯推论→【本文推论】"一一对应。

---

## 四、对象属性（Object Properties）

| 属性 | 定义域 → 值域 | 对应现状 |
|---|---|---|
| `kbont:belongsToAxis` | ThesisCluster / CorpusSlice → TopicAxis | L2 章节归属 |
| `kbont:unfoldsProposition` | ThesisCluster → CoreProposition | 入口 §二"主要展开处"列 |
| `kbont:supportedBy` | Argument / ThesisCluster → TheoryClaim | `【依据：L1-…】` 标记 |
| `kbont:answeredBy` | Objection → Response；Response → Objection（逆向 `respondsTo`） | 簇内反驳预案结构 |
| `kbont:counters` | AdversaryPosition → Argument | 附录 D 对位关系 |
| `kbont:extrapolatesFrom` | AdversaryPosition → TheoryClaim | 对手本人原文 → 外推版本 |
| `kbont:instantiatedBy` | ThesisCluster → Case（案例支撑簇）；逆向 `verifies` | L3-A §六对应总表 |
| `kbont:contradicts` | Counterexample → ThesisCluster / Argument | §五否证案例栏 |
| `kbont:evidencedBy` | Case / ProseDataPoint / CaseIndicator / TheoryClaim → EvidenceSource | `〔来源：…〕` 标记 |
| `kbont:raises` | Case / Counterexample → OpenQuestion | 案例末"待验证问题" |
| `kbont:suppliesConcept` | Term / TheoryClaim → ThesisCluster | L1 → L2 概念供给 |
| `kbont:hasAnchor` | Argument / Case / Insight → Anchor | 锚点登记 |
| `kbont:in tensionWith` | CoreProposition → CoreProposition | 入口 §2.1 四组张力（对称属性） |
| `kbont:produces` | ThesisCluster → OutputWork | 成稿口径 4/42 |
| `kbont:drawnFrom` | CorpusSlice → EvidenceSource / OutputWork | 语料溯源 |
| `kbont:monitors` | MonitoringTarget → EvidenceSource | LEDGER §八监测清单 |
| `kbont:flagsRisk` | Switch / ForkPath → ThesisCluster | L1-B 灾难路径预警 |

## 五、数据属性（Data Properties）与受控词表

| 属性 | 类型 | 受控值 | 说明 |
|---|---|---|---|
| `kbont:code` | xsd:string | — | 全库唯一编号（T01-A、R-065、M1、U14…） |
| `kbont:sourceGrade` | 枚举 | L-A / L-B / L-C / L-D / L-E | RULES §1 五级 |
| `kbont:verifyStatus` | 枚举 | 未处理 / 检索中 / 待核验 / 已核验 / 已入库 / 退回待补 / 作废 | LEDGER 状态机 |
| `kbont:anchorNature` | 枚举 | factual / phenomenal / inference | 锚点性质三分 |
| `kbont:metricKind` | 枚举 | attitude / stock / rate | 态度类 / 存量类 / 比率类（口径纪律） |
| `kbont:caliber` | xsd:string | — | 统计口径描述（分子分母定义，防跨口径相减） |
| `kbont:firstHandVerified` | xsd:boolean | — | 是否核到一手页（≠"已核验"，方法论第 3/15 条） |
| `kbont:inLedger` | xsd:boolean | — | 是否已登记台账（"使用了但没记账"检查） |
| `kbont:isOriginalInference` | xsd:boolean | — | 是否【本文推论】 |

---

## 六、纪律规则 → SHACL 约束（核心 6 条）

SHACL 完整定义见 `kb-ontology.ttl`，此处给出规则—约束对照：

| # | 现行规则（出处） | SHACL 表达 |
|---|---|---|
| C1 | L-E 一般网络**不得入库**（RULES §1） | `EvidenceSourceShape`：`sourceGrade` 值为 L-E 且 `firstHandVerified=false` 的节点，不得作为任何 `evidencedBy` 的宾语（`sh:not` + 属性路径约束） |
| C2 | 带数字散文行须行内附来源（J 段） | `ProseDataPointShape`：每条 `ProseDataPoint` 必须 `evidencedBy` 至少一个 EvidenceSource（`sh:minCount 1`） |
| C3 | 四要素缺一不可（RULES §2） | `EvidenceSourceShape`：作者/篇名/年份/载体四属性须齐备方可 `verifyStatus=已核验` |
| C4 | 跨口径不得相减（RULES §2 附则） | 涉及比率对比的 `CrossCheck` 必须携带 `caliber` 且两侧口径一致，否则仅允许标"量级对照" |
| C5 | 态度类与存量类不可混说（RULES §2.2） | 同一 `CrossCheck` 两侧的 `metricKind` 必须相同 |
| C6 | 对位不简化对手（纪律第 12 条） | `AdversaryPosition` 必须有 `extrapolatesFrom` 指向对手本人 `TheoryClaim`；缺失即为稻草人违规 |

> 工具联动路径：`tools/verify.py` 的 A~J 十段体检是**文本级正则检查**；SHACL 是**模型级检查**。前者不需迁移，后者在未来实例化后可作为第二道网（阶段三，见 §八）。

---

## 七、实例层盘点（ABox 底数）

本体建好后，现有资产可直接映射为实例，底数如下（全部来自正式层实测，未新造）：

| 实例类型 | 数量 | 来源 |
|---|---|---|
| CoreProposition（M1–M8） | 8 | 入口 §二 |
| TopicAxis（T01–T10） | 10 | 入口 §1.3 |
| ThesisCluster（Txx-X） | 42 | L2（1.3 表：4+4+4+4+5+4+4+4+4+5） |
| 核心命题张力对 | 4 组 | 入口 §2.1 |
| AdversaryPosition（具名对手） | 26 条（21 对位 + 5 援军/正方），14 簇全覆盖 | L2 附录 D v2.1 + §六总表（v1.1 结构化为 `instances.ttl`） |
| Case | 当前正式层 6；历史实例层 7（未重抽） | 当前标题：L3-A 案例 1-1、2-1、3-1、4-1、4-2、6-1；历史实例层计数不当作当前实测 |
| Counterexample | 3（A/B/C） | L3-A §五 |
| Insight | 5 | L3-A §七 |
| OpenQuestion | U1–U21 | L3-A 各案例 + §六历史案例栏（U19–U21） |
| EvidenceSource（台账组） | 87（入库 56 / 已核验 30 / 待核验 1） | LEDGER v1.17 |
| TheoryWork（文献条目） | 51 + 专题 15 条 | L1 / L1-A |
| Term | 59（当前 L1-C 人工盘点；历史实例层为 45，未重抽） | L1-C v2.1 |
| OutputWork（成稿） | 6 | 历史 reports/article-01/05/06/07/08/09；当前发布端入口见[叙事长文](../内容产线/01-公众号长文-公司没有消失.md) |

**已知缺口（本体视角新发现）与补全状态（v1.1 全部闭环）**：

1. ~~**M1–M8 与 42 簇的 `unfoldsProposition` 边不完整**~~ → ✅ 已补全，两级挂接：
   - **轴级**：10 主题轴按入口 §二"主要展开处"全量挂接（T01→M1；T02→M1/M3/M4；T03→M2；T05→M7；T06→M3/M8；T07→M2/M5/M6；T08→M5；T09→M2/M4；T04/T10→M8"全库"）；
   - **簇级**：22 簇 26 条 `unfoldsDirect` 直达边（判读规则：簇内明引命题或论点为命题直接推演者方标 direct；如 T02-B→M3、T07-C→M6、T10-D→M3）。判读由本体建模执行，属**分析性判断**，非正式层文本自有标记——若某条 direct 边与你的判读不符，改 `instances.ttl` 即可，不动正式层。
2. ~~**Counterexample 与所反驳簇的 `contradicts` 边未成文**~~ → ✅ 已补全：反例 A→T02-D/T02-A、反例 B→T01-A、反例 C→T01-D；同时把 L3-A §六总表 **17 行**全部结构化（14 机制行 `verifies`/`relatedCluster`/`verdict` + 3 反例行）。
3. ~~**17 个 OpenQuestion 尚无状态属性**~~ → ✅ 已补全：`questionStatus` 全量赋值——**历史 v1.1 快照为 18 问 17 号**（U11 = `partially-answered`，2026-10-12 部分回答"价格权重最高"；其余 open）。
   - ✅ **U14 撞号已订正**：反例 A 的“场内占比能否抬升”已于 L3-A v1.6 / 入口 v3.9 改为 **U18**；当前正式层为 **U1–U21 无重号**。`U14dup` 是历史实例登记，当前副本缺 `instances.ttl`，未重抽当前实例。
   - ⚠️ **新发现口径差异：P0 双口径**——正式层口径"14 个 P0 簇"按附录 D **条目数**计（附D-8 一条覆盖 T05-B/T05-C 两簇）；按**去重簇**计实为 **15 簇**（且小结枚举漏列了附D-3 的 T03-B）。校验器以簇口径 15 校验，两口径均已注明。
4. ~~**"10 大主题轴 vs 14 P0 簇"口径差异**~~ → ✅ 已澄清（v1.0 的"T11–T14"猜测有误）：P0 是簇级属性（`isP0`），非独立主题轴；主题轴仍为 T01–T10 十个。

---

## 八、落地路线（四阶段）

| 阶段 | 内容 | 产物 | 状态 |
|---|---|---|---|
| **一** | TBox + SHACL + 抽样实例 | 本文件 + `kb-ontology.ttl` | ✅ v1.0 完成 |
| **一·五** | 缺口补全：命题边两级挂接 / contradicts 边 / U 问状态 / P0 口径 / 具名对手结构化 | `instances.ttl`（v1.1） | ✅ 完成 |
| **二** | 实例批量抽取：LEDGER 85 组 → EvidenceSource（含等级/状态）、40 条〔来源〕散文行 → ProseDataPoint、303 个【依据/素材】标记 → supportedBy 指针 | `tools/extract_ontology.py` → `generated/derived.ttl` | ✅ 完成 |
| **三** | 机器校验：TTL 解析 + C1–C6 纪律检查 + 结构完整性 + SPARQL 实战查询 | `tools/check_ontology.py` → `verify-report.txt`（当前 0 ERROR / 2 WARN） | ✅ 完成（pyshacl 引擎为可选增强） |
| **四** | 图谱可视化：论题簇—命题—案例—反例—对手—成稿关系图，供选题与写作时人肉导航 | `tools/render_ontology_graph.py` → `graph.html`（自包含交互页：力导向 / 类型过滤 / 悬停看关系 / P0 白环）+ `graph-preview.png` | ✅ 完成（162 节点 / 174 边，无外部依赖） |

**校验环境**：WorkBuddy 自带 Python（`~/.workbuddy/binaries/python/envs/default`，3.13.14）+ rdflib 7.6.0（wheel 离线装于清华源，pip-tmp 留存于 `.workbuddy/pip-tmp/`）。运行方式：

```bash
PYV="/c/Users/Administrator/.workbuddy/binaries/python/envs/default/Scripts/python.exe"
"$PYV" tools/extract_ontology.py   # 重抽 derived.ttl（正式层变更后重跑）
"$PYV" tools/check_ontology.py     # 校验，ERROR 时 exit 1（可挂 refresh 流程）
"$PYV" tools/link_prose_sources.py   # 〔来源〕散文行 → R-XXX 接线（机器候选 + manual.tsv 人工裁定）
"$PYV" tools/render_ontology_graph.py  # 生成 graph.html（instances.ttl 变更后重跑）
"$PYV" tools/discovery_chain.py --all  # 发现链：10 轴预生成（concepts.ttl/instances.ttl 变更后重跑）
"$PYV" tools/discovery_chain.py T02-B  # 任意簇/命题/概念代码即时生成
```

**WARN 归零（v1.4）**：~~29 条已核验来源四要素在自由文本中~~ → **已字段化**：`extract_ontology.py` 新增 `_extract_fields` 启发式（前缀码剥离 / 《》与 *斜体* 篇名 / 年份守卫正则 / 载《…》、载"…"、英文后置机构、arXiv/出版社兜底 / 机构作者兑底）+ `ledger-fields.manual.tsv` 人工裁定表（5 行：R-010/R-059 载体、R-060/R-061/R-062 多源 descriptor），29/29 全覆盖落 `kbont:author/title/year/venue`。~~40 条〔来源〕散文行未接台账码~~ → **已消（v1.3）**：35 条接 R-XXX（17 机器 auto + 16 人工裁定 + 2 同源群 weak），5 条无 R 行官方直引入另册（`linkConfidence: no-R`，多为案例 3-1 地方政府直引 / 监测层来源）；接线产物 `prose-links.tsv`（机器候选）+ `prose-links.manual.tsv`（人工裁定，优先级最高），C2 从 WARN 降为 INFO。**本体校验现为 0 ERROR / 0 WARN。**

**历史红线（本体构建初版）**：v1.0 构建时正式层 9 文件零改动；本目录不计入正式层口径，无需改 `verify.py` 的 `FORMAL_FILES`；未占用 `### Txx-X` 簇命名空间；思想来源均标 L1/RULES/LEDGER 出处或`【本文推论】`。后续正式层更新由本体索引承接，不倒写正式文本。

### v1.8 人工同步说明（不改 TTL）

- **范围**：仅更新 `chains/` 的检索指针，承接 L1 v1.6 与 L2 v2.4 的利己主义、通胀分配双专题；T03-A/B 与 T10-C/D 同步标题漂移。
- **新增作业指针**：T01/T04 补利己主义与制度性对立；T02/T03/T08/T09 补名义/实际价值、家庭净财富头寸、指数化与通胀成因边界；T10 补可信制度与可迁移性。
- **计数边界**：§一概念子图与既有生成边计数均未重算；补注明确标为“人工索引，未改 TTL / 图”，不得解释为本体新增概念或关系。
- **验证缺口**：当前目录仍缺 `instances.ttl`、`derived.ttl` 及 `tools/` 生成与校验脚本，**本轮未运行 TTL 校验、SHACL 检查或图渲染**；不继承历史“0 ERROR / 0 WARN”作为本轮结果。

---

### v1.9 人工同步说明（M4 与 L4/L4-B，不改 TTL）

- **范围**：正式层已形成 L4 综合推演层与 L4-B 可行性专卷；人工发现链补入 M4 分层、制度成熟度、财政/法源、生态预算、OPC 证据协议与双轨分配指针。
- **计数边界**：L1-C 当前 56 条术语；`concepts.ttl` 仍是历史 40 义项级概念，**未新增“共产主义因素”“制度成熟度”的 TTL 节点**。
- **实例边界**：当前副本缺 `instances.ttl`、`derived.ttl` 与 `tools/`，未重抽 ABox、未运行 SHACL、未重渲染 graph。
- **M4 校准**：人工链不再使用“天然非排他→逆技术本性→必然公有”的强推论；历史概念子图仅作检索入口，判断须回 L4-B §1。

### v1.12 赛博朋克关联层校准（新增正式机制，不重跑实例/图）

- **新增概念**：CP-41 赛博朋克式技术统治、CP-42 技术垄断、CP-43 赛博朋克、CP-44 平台依附固化；`concepts.ttl` 由 40 → **44 个义项级概念**。
- **关系校准**：技术垄断以“制约”连接平台依附固化，平台依附以“再生产”连接平台依附固化；赛博朋克式技术统治分别辨析技术封建主义、赛博共产主义与平台依附固化；赛博朋克辨析赛博朋克式技术统治与赛博共产主义。删除文化—分析总称作为技术垄断下位概念的旧边；当前为 **36 条受控概念—概念关系、9 条 `skos:broader`**，未使用 `is-related-to`。
- **同步校准**：公共池塘资源定义保留 M4 分层，明确数字副本倾向非竞争，但算力、能源、芯片与优质数据仍可稀缺；不改变概念或边数。
- **语义边界**：赛博朋克只作文化表征；赛博朋克式技术统治只作文化—分析总称；**平台依附固化是正式机制概念**；技术封建主义仍拒绝为正式框架。
- **已校验**：用 rdflib 只读解析 `concepts.ttl` 与 `kb-ontology.ttl`；实测 44 个概念、44 个唯一 CP 编号、36 条去重受控关系、9 条 `skos:broader`、42 条分期与 2 条 `ahistorical`；按 C7 白名单检查概念谓词错误为 0，按 C8 检查定义/分期/锚点存在性错误为 0。
- **未做**：`instances.ttl`、`derived.ttl` 与 `tools/` 仍缺失；未重抽 ABox、未重算发现链子图、未运行完整 SHACL 或 graph 渲染，锚点引用解析仍待实例层恢复。历史 40/29、43/34、213/277 等计数仍是旧批次结果。
- **术语口径**：L1-C 59 条术语，与 concepts.ttl 44 个义项级概念不是一一对应，不能把二者差值当作缺失节点清单。

### v1.10 文本校准说明（不改结构、不重跑校验）

- **范围**：只改 `kb-ontology.ttl` 的 M4、算力张力与 T02-A 标签，以及 `concepts.ttl` 中新型生产资料、非排他性、排他性占有的定义和分期文本；消除“逆技术本性”“四属性=天然非排他”的旧口径。人工发现链索引同步至 v2.1，当前正式层案例标题数校正为 6，历史实例层 7 未重抽。
- **未做**：未新增或删除概念、节点、边、实例；未重抽 ABox；未运行 TTL 解析、SHACL、check_ontology 或 graph 渲染。40 个历史义项级概念与 29 条历史辩证边仍是**旧结构计数**，不得当作 v1.10 重算结果。
- **新术语边界**：L1-C 当前 56 条，但 `concepts.ttl` 仍只含历史 40 个义项级概念；“共产主义因素”“制度成熟度”尚未建 TTL 节点。
- **历史报告**：`verify-report.txt` 是旧批次产物，已加历史标识；其中的“0 ERROR / 0 WARN”不适用于本次文本修订。

---

## 九、RedMarx 范式对照（v1.6 核心升级）

**范式出处**：三大洲社会研究所（Tricontinental: Institute for Social Research）旗下 Global South Insights（GSI）2026 年 8 月发布的马克思主义研究 AI 系统「红色马克思」（RedMarx），及其底层 MetaRAG 架构。其公开论文《RedMarx: Teaching AI to Read Marx Dialectically》给出了马克思主义语料上知识库/研究系统的方法论基准。

**核心论点**：通用 AI 处理马克思主义文本的四大缺陷——把辩证运动压扁成定义、把后世解读混入原作者本意、抹平政治与翻译争议、把检索当理解。解法不是更大的语料，而是**概念图谱先行**：先在受控谓词概念图上推出分析骨架，再回原文做带定位的段落检索。

五要素逐条落点：

| # | RedMarx 范式要素 | 本库 v1.6 落点 | 状态 |
|---|---|---|---|
| 1 | **分层概念知识图谱**（价值/劳动/资本/阶级… 编码为层级关系结构而非扁平标签；原型 35 概念 / 32 关系） | `concepts.ttl`：**44 义项级概念**（`kbont:MarxistConcept`，v1.12 文本复算），SKOS 层级（`broader`）＋辩证关系双层结构 | ✅ |
| 2 | **受控谓词词表**（contradicts / mediates / reproduces / commodifies / alienates / exploits / determines / transforms-into…，禁用 is-related-to 泛化边） | 11 个受控辩证谓词，全部 `subPropertyOf kbont:dialecticalRelation`；SHACL C7（sh:closed + 校验器谓词白名单）双实现；`contradictsConcept` 与文档级 `contradicts`（反例否证簇）严格分名 | ✅ |
| 3 | **历史歧义消解**（概念带 1844→1857→1867 式分期标注，区分早期与成熟表述） | `kbont:periodTrace`（42 概念带分期）＋ `kbont:ahistorical`（CAP 定理、古德哈特定律 2 条显式豁免）；C8 强制二者居其一 | ✅ |
| 4 | **词义拆分**（同术语异语境拆义项，不强行合并） | `kbont:lemma`（分组键）＋ `kbont:senseContext`（语境）＋ `kbont:distinguishedFrom`（辨析边）；3 组示范：**占有关系**（所有制/所有权/占有形式，对应 T02-D）、**价格信号**（米塞斯内生价格/兰格参数价格）、**异化**（1844 人类学语境/1867 拜物教结构语境） | ✅ |
| 5 | **发现链（discovery chains）**：收到问题先在图谱上推理出分析骨架，再回语料定位带章节标识的原文；检索片段只作指针，写作前读整章 | `tools/discovery_chain.py`：任意轴/簇/命题/概念代码 → 概念种子 → 受控谓词 BFS ≤2 跳 → 概念子图 + 检索与对位作业清单（簇/命题/案例/反例/对手/问/成稿/跨簇联动）+ 检索三纪律；`chains/` 预生成 10 轴 | ✅ |

**与 RedMarx 的结构差异（如实注明）**：
- RedMarx 的概念图独立于语料存在；本库概念层是**正式层之上的索引骨架**——每个概念经 `kbont:conceptAnchor` 锚回可定位文本（C8 强制），保证"分析预设可见"且不产生脱离正式层的悬空概念。
- RedMarx 由领域专家人工维护图谱；本库同此（periodTrace 与辩证边均为**分析性判读**，标注于 concepts.ttl 头注，判读分歧改 `concepts.ttl` 即可，不动正式层一字）。
- RedMarx 运行时动态生成子图；本库预生成 10 轴链 + 支持任意单元代码即时生成。
- 概念层已并入 `graph.html`（v1.6：213 节点 / 277 边；绿色边=受控辩证关系，暗边=概念锚定；`graph-preview.png` 同步更新）。

**C8 新增校验**（概念落地三要件）：每概念须有 `periodTrace` 或 `ahistorical=true`、须有 `skos:definition`、须有 ≥1 条 `conceptAnchor` 且锚点目标真实存在（防悬空锚点/黑箱概念）。

---

## 版本记录

| 版本 | 日期 | 变更 |
|---|---|---|
| v1.0 | 2026-10-24 | 初版：21 类 + 17 对象属性 + 9 数据属性 + 6 条 SHACL 约束 + ABox 底数盘点 + 四阶段路线 |
| v1.1 | 2026-10-24 | 缺口补全：新增 `instances.ttl` 完整实例层（42 簇两级命题挂接、17 总表行结构化、18 问含 U14 撞号登记、26 具名对手、4 成稿）；TBox 增补 TensionChallenge 类与 countersCluster/reinforcesCluster/partOf/relatedCluster/concernsCluster/illustratedBy/verdict/isP0/unfoldsDirect 等属性；订正 v1.0 两处：张力模型（命题×挑战，非命题互张力）、对手计数 24→26 |
| v1.2 | 2026-10-24 | 阶段四完成：新增 `tools/render_ontology_graph.py` → `graph.html`（162 节点 / 174 边，自包含无依赖）；成稿 4→5（article-08 跨簇重组成稿，T02-A/T02-B `produces` 边挂接，入口 v3.8） |
| v1.3 | 2026-10-24 | 散文行接台账码：新增 `tools/link_prose_sources.py`（双通道匹配：附D 节级映射 + 词元打分）+ `prose-links.manual.tsv` 人工裁定；40 行全部处置（35 接 R-XXX / 5 官方直引另册），extract 新增 `kbont:ledgerRef`/`linkConfidence` 属性，C2 降为 INFO；**余 1 WARN**（C3 四要素字段化） |
| v1.4 | 2026-10-24 | C3 四要素字段化：`_extract_fields` 启发式（前缀码/篇名/年份/载体）+ `ledger-fields.manual.tsv` 人工裁定，29 条已核验来源 author/title/year/venue 全部落字段；**0 ERROR / 0 WARN（首次全绿）** |
| v1.5 | 2026-10-24 | 历史案例栏同步（入口 v4.1 / L3-A v1.7）：`instances.ttl` 新增 Case-6-1（L3-A §六 1956—1976）+ 机制行 6 条（历史行 verdict 以 comment 记「历史参照」，不占验证值域）+ U19–U21（21 问 21 号无重号）；extract 重跑 derived.ttl（散文行 40 不变）；check_ontology **0 ERROR / 0 WARN**；graph.html 重渲染（185 边） |
| v1.6 | 2026-11-04 | **RedMarx 范式升级**：新增 `concepts.ttl` 概念层（40 义项级概念 / 29 条受控辩证边 / 11 受控谓词 / 3 组词义拆分 / 38 条 periodTrace 历史分期）；TBox 新增 MarxistConcept 类、dialecticalRelation 谓词族、lemma/senseContext/periodTrace/ahistorical/conceptAnchor 属性、SHACL C7/C8；`check_ontology.py` 扩展 C7（概念谓词封闭白名单）/C8（分期+定义+锚点不悬空）；新增 `tools/discovery_chain.py` 发现链生成器（`chains/` 10 轴预生成 + 任意单元即时生成）；校验 **0 ERROR / 0 WARN**（概念 40 / 辩证边 29 / 拆分边 5）；graph.html 重渲染并入概念层（213 节点 / 277 边，辩证边绿色 / 锚定边暗色）+ graph-preview.png 更新 |
| v1.7 | 2026-11-25 | **发现链索引同步（不改 TTL）**：按 L2 v2.3 更新 `chain-T03/T07/T09/T10` 中 T03-A、T03-B、T10-C 的标题与跨簇联动，补 T03-B「企业 OPC 化」↔ T10-A/C/D 指针。概念、实例、边与 graph 计数不变；当前目录缺 `instances.ttl` / `derived.ttl` 与生成脚本，未重跑本体校验或图渲染 |
| v1.8 | 2026-11-25 | **利己主义与通胀专题发现链人工同步（不改 TTL）**：承接 L1 v1.6 / L2 v2.4，`chain-T01/T04` 补个人利益—排他利己—「精致利己」、制度性对立三判据与条件分析；`chain-T02/T03/T08/T09` 补名义/实际价值、家庭净财富头寸、工资指数化、灵活就业暴露与通胀成因边界；`chain-T10` 同步产权可迁移性标题。补注与生成清单分节，概念、实例、边及 graph 计数均不变。**当前缺 `instances.ttl` / `derived.ttl` 与 `tools/`，本轮未运行 TTL / SHACL 校验或图渲染，不宣称“全绿”** |
| **v1.9** | **2026-11-26** | **M4 分层与 L4/L4-B 人工索引**：更新 T02/T10 发现链的 M4、T02-A 标题与可行性指针；README 改“四层正式知识库”，L1-C 当前术语 56；明确 `concepts.ttl` 仍为历史 40 义项级概念，当前缺 `instances.ttl` / `derived.ttl` / `tools/`，**未改 TTL、未重跑 SHACL、未重渲染图，不宣称全绿** |
| **v1.10** | **2026-11-26** | **M4 相关 TTL 文本标签校准（不改结构）**：修正 `kb-ontology.ttl` 的 M4、算力张力与 T02-A 标签，以及 `concepts.ttl` 的新型生产资料、非排他性、排他性占有定义和分期文本；发现链索引 v2.0 增 L4-B/内容产线入口，历史校验报告加失效标识。**节点、边、实例与图计数未重算；当前缺实例/派生文件与工具，未运行 TTL/SHACL/graph，不继承历史全绿** |
| **v1.11** | **2026-11-26** | **赛博朋克关联概念入 TTL**：新增 CP-41～CP-43 与 6 条受控关系，concepts.ttl 当批为 **43 概念/34 条去重受控关系**（新增 6 条、删除 2 条重复三元组）；算法治理定义补“算法监狱”边界。L1-C 术语更新为 59；发现链只加人工指针。rdflib 解析通过，C7 白名单与 C8 定义/分期/锚点存在性检查均为 0 错误。**当批缺 instances/derived/tools，未重抽 ABox、未重算子图、未运行完整 SHACL/graph，不继承历史全绿** |
| **v1.12** | **2026-11-26** | **赛博朋克关系强度与正式机制校准**：新增 CP-44“平台依附固化”，文化—分析总称不再作为技术垄断下位概念；技术垄断—正式机制改为条件关系，平台依附—正式机制改为再生产关系。当前 rdflib 复算 **44 概念/36 条去重受控关系/9 条 broader/42 条分期/2 条 ahistorical**，C7 与 C8 存在性检查 0 错误。同步清除 U14 待决旧说明，链接内容产线入口。**未重抽 ABox/子图，未运行完整 SHACL/graph** |
