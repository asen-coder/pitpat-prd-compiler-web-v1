# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库性质

本仓库是一个 Claude Code Skill 定义（纯 Markdown），无可执行代码，无构建/测试命令。所有文件均为工作流指令和输出模板。

## Skill 触发条件

当用户提及 PRD、需求编译、prd-to-backend、需求提取、后端架构文档，或提供会议记录/原始 PRD 链接/非结构化需求输入时，调用此 Skill。

## 两阶段架构

### Phase 1 → 生成 `prd.md`

将非结构化 PRD 素材提炼为结构化业务需求文档（描述 What，不描述 How）。

- 工作流详情：`phases/phase1.md`
- 输出模板：`templates/prd-phase1-template.md`
- 输出文件命名：`prd-[english-kebab-case-name]-[version].md`
- 文件顶部声明：`📁 **File: prd-[english-kebab-case-name]-[version].md**`

Phase 1 结束时，必须在 Section 13 之后追加 Handoff Envelope（HTML 注释块），供 Phase 2 快速定位目标域。

### Phase 2 → 生成 `prd-to-backend.md`

从 `prd.md` 生成后端实现映射文档（描述 How，映射到 pitpat 微服务架构）。

- 工作流详情：`phases/phase2.md`
- 输出模板：`templates/prd-phase2-template.md`
- 输出文件命名：`prd-to-backend-[english-kebab-case-name]-[version].md`
- 文件顶部声明：`📁 **File: prd-to-backend-[english-kebab-case-name]-[version].md**`

Phase 2 必须先读取 CLAUDE.md（pitpat 工程约定来源），再扫描代码库，最后生成文档并追加架构风险清单（Section 10）。

## 关键设计决策

### 渐进式加载（Progressive Disclosure）

`SKILL.md` 只存放入口说明和阶段索引表。两个阶段的完整执行逻辑分别在 `phases/phase1.md` 和 `phases/phase2.md` 中，**按需读取，不预加载**，以控制上下文消耗。

### Handoff Envelope

Phase 1 输出的 `prd.md` 末尾嵌入一个 HTML 注释块（`PRD-HANDOFF-ENVELOPE`），字段包括：
- `targetDomains`：映射到 `pitpat-data/` 子目录名的目标域列表
- `domainObjects`：DO-XXX 标识符列表
- `keyConstraints`：RULE-XXX 摘要（3-5 条）
- `scopeSummary`：P0/P1 数量和 BS-XXX 总数

Phase 2 读取此 Envelope 加速目标域定位，但不替代对完整 `prd.md` 的阅读。

### 扫描缓存

Phase 2 在代码库扫描后将结果写入 `{prd目录}/.prd-scan-cache.json`，以 `gitSha` 字段判断缓存是否有效，命中时跳过扫描步骤（Step 3.2–3.3）。

## 全局约束

- 所有生成内容必须使用**简体中文**
- Phase 1 输出禁止包含 UI 规格（布局、视觉设计、组件层级、动画）
- Phase 1 业务规则禁止使用技术实现术语（Redis、MQ、缓存、分布式锁、TCC 等）
- 所有标识符（OR-XXX、BS-XXX 等）必须连续编号，无间隙
- 每个 OR/NR/RS 条目必须标注变更类型（`[NEW]`/`[MOD]`/`[DEL]`/`[OPT]`）
- 每个目标（G-XXX）必须标注优先级（`P0`/`P1`/`P2`）
- 未明确说明的细节标注为 `[TBD]`，禁止自行推断填充

## 文件结构

```
SKILL.md                        # Skill 入口：触发条件、阶段索引、文件命名规则
phases/
  phase1.md                     # Phase 1 完整执行逻辑（含 URL 抓取、分析、生成步骤）
  phase2.md                     # Phase 2 完整执行逻辑（含扫描、生成、架构风险检查）
templates/
  prd-phase1-template.md        # prd.md 输出结构和编号规范
  prd-phase2-template.md        # prd-to-backend.md 输出结构（工程约定引用 CLAUDE.md）
```
