# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库性质

本仓库是一个 Claude Code Skill 定义（纯 Markdown），无可执行代码，无构建/测试命令。所有文件均为工作流指令和输出模板。

## Skill 触发条件

当用户提及 PRD、需求编译、需求提取、前端技术/架构文档，或提供会议记录/原始 PRD 链接（墨刀/Figma 等）/非结构化需求输入时，调用此 Skill。本 Skill 面向 **pitpat-h5 前端**。

## 两阶段架构

### Phase 1 → 生成业务需求文档（`.md`）

将非结构化 PRD 素材提炼为结构化业务需求文档（描述 What，不描述 How），并面向前端补齐 i18n / 公英制 / UI 状态 / 端能力等维度（见 §4.4–§4.9）。

- 工作流详情：`phases/phase1.md`
- 输出模板：`templates/prd-phase1-template.md`
- 输出位置与命名：遵循 `rules/output-structure.md`（版本目录 `APP_v<版本>/md/`，中文文件名，每需求一份）

Phase 1 结束时，必须在 Section 13 之后追加 Handoff Envelope（HTML 注释块），供 Phase 2 快速定位目标模块。

### Phase 2 → 生成前端技术文档（`.docx`）

从 `prd.md` 生成**前端实现映射文档**（描述 How，映射到 pitpat-h5 架构：Vue3 + Vant + Vuex + vue-i18n + Axios + Stylus）。

- 工作流详情：`phases/phase2.md`
- 输出模板：`templates/prd-phase2-template.md`
- 输出位置与命名：遵循 `rules/output-structure.md`（`APP_v<版本>/docx/`，与 `md/` 同名，每需求一份）

Phase 2 必须先读取 **pitpat-h5 的 `CLAUDE.md`**（前端工程约定来源），再用 CodeGraph 扫描 pitpat-h5 代码库，最后生成文档并追加前端架构风险清单（Section 12）。

## 关键设计决策

### 渐进式加载（Progressive Disclosure）

`SKILL.md` 只存放入口说明和阶段索引表。两个阶段的完整执行逻辑分别在 `phases/phase1.md` 和 `phases/phase2.md` 中，**按需读取，不预加载**，以控制上下文消耗。

### Handoff Envelope

Phase 1 输出的 `prd.md` 末尾嵌入一个 HTML 注释块（`PRD-HANDOFF-ENVELOPE`），字段包括：
- `targetModules`：映射到 pitpat-h5 业务域/模块（依 pitpat-h5 CLAUDE.md 知识图谱索引）
- `affectedPages`：受影响页面/弹窗（PAGE-XXX）
- `nativeCaps`：JSBridge 原生能力（NFC/蓝牙/3D 等）
- `i18nImpact`：需新增/改动的 i18n 模块或 key 范围
- `keyConstraints`：RULE-XXX 与单位规则摘要（3-5 条）
- `scopeSummary`：P0/P1 数量和 FS-XXX 总数

Phase 2 读取此 Envelope 加速目标模块定位，但不替代对完整 `prd.md` 的阅读。

### 扫描缓存

Phase 2 在扫描 pitpat-h5 代码库后将结果写入 `{prd目录}/.prd-scan-cache.json`，以 `gitSha`（pitpat-h5 仓库 HEAD）字段判断缓存是否有效，命中时跳过扫描步骤（Step 3.2–3.3）。

## 全局约束

- 所有生成内容必须使用**简体中文**
- Phase 1 禁止包含**视觉设计**（布局、视觉规格、组件层级、动画）；但**必须保留**交互逻辑、精确文案、UI 状态、单位/i18n 展示规则（§4.4–§4.9）
- Phase 1 **§7 业务规则**禁止使用技术实现术语（Redis、MQ、缓存、分布式锁、TCC 等）；§4.1/§4.4/§4.7 描述前端接口依赖、JSBridge 原生能力时不受此限
- 所有标识符（OR-XXX、FS-XXX、COPY-XXX 等）必须连续编号，无间隙
- 每个 OR/NR/RS 条目必须标注变更类型（`[NEW]`/`[MOD]`/`[DEL]`/`[OPT]`）
- 每个目标（G-XXX）必须标注优先级（`P0`/`P1`/`P2`）
- 未明确说明的细节标注为 `[TBD]`，禁止自行推断填充

## 文件结构

```
SKILL.md                        # Skill 入口：触发条件、阶段索引、输出结构摘要
phases/
  phase1.md                     # Phase 1 完整执行逻辑（含 URL 抓取、分析、生成步骤、前端捕获维度）
  phase2.md                     # Phase 2 完整执行逻辑（读 pitpat-h5 CLAUDE.md、CodeGraph 扫描、生成、前端架构风险检查）
templates/
  prd-phase1-template.md        # 业务需求文档输出结构和编号规范（含 §4.4–§4.9 前端维度）
  prd-phase2-template.md        # 前端技术方案输出结构（工程约定引用 pitpat-h5 CLAUDE.md）
rules/
  output-structure.md           # 输出目录与文件命名唯一事实源（APP_v<版本>/md + docx，中文名）
```
