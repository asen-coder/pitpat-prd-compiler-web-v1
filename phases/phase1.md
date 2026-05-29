# Phase 1: Generate prd.md — Requirement Extraction

This file contains the complete Phase 1 execution logic. Load this file when the user wants to compile unstructured PRD materials into a structured `prd.md`.

---

## Phase 1 Principles

1. **Zero-Fabrication:** Never invent requirements. Unstated or ambiguous details → mark as `[TBD]` in "Open Questions".
2. **Change Type Markers:** Every requirement (OR, RS, NR) tagged with `[NEW]`, `[MOD]`, `[DEL]`, `[OPT]`.
3. **Priority Markers:** Every goal tagged with `P0` (must-have), `P1` (should-have), `P2` (nice-to-have).
4. **Three-Perspective Scope Split:** Tag scope with `[Frontend]`, `[Backend]`, `[Testing]`, `[Shared]`.
5. **No Visual Design (but keep behavior):** Do NOT include visual/layout design, component hierarchies, or animation specs (those live in Figma/Modao). BUT you MUST capture, because frontend depends on them: interaction logic, exact copy (文案), UI states (empty/loading/error/缺省), and unit/i18n display rules. "No UI design" ≠ "no interaction/copy/state/unit".
6. **Structured Grouping (H4 Mandate):** Use `#### [Category]` headings in Section 2.1. Flat lists prohibited.
7. **State Machine Notation:** Use `[Current] -> (Trigger) -> [Target]` format.
8. **Requirement-Implementation Decoupling:** prd.md describes business expectations only, no technical implementation details.
9. **Verification-Oriented Acceptance Criteria:** AC must include both happy paths AND negative/boundary test paths.
10. **Zero-Omission:** Do not omit any requirement mentioned in input materials.
11. **Frontend Capture Completeness:** For pitpat-h5 (Vue3 + Vant + vue-i18n, cross-national multi-language mobile H5), always evaluate and fill these when present in the materials: §4.4 文案与多语言 (i18n), §4.5 单位与本地化展示 (公英制/配速/时区/日期), §4.6 UI 状态覆盖 (空/加载/错误/缺省), §4.7 端能力与兼容 (JSBridge 原生能力、iOS/Android 差异、App 版本兼容), §4.8 涉及页面与路由清单, §4.9 埋点. If a dimension is genuinely not involved, state so explicitly rather than leaving it blank.

---

## Phase 1 Workflow

### Step 1: Source Acquisition and Analysis (Silent Pre-computation)

**MANDATORY TOOL: chrome-devtools MCP is the ONLY allowed tool for accessing dynamic web pages. No substitutes.**

When the user provides a URL (e.g., Modao CC, Feishu, Notion, etc.):

1. **MANDATORY FIRST ACTION — Pre-flight availability gate (do this BEFORE any extraction):**
   - Call `mcp__chrome-devtools__list_pages` to verify the server is connected.
   - **If the call succeeds** → the server is available → proceed to step 2.
   - **If the tool does not exist, is not loaded, or the call fails/errors** → the server is unavailable → **STOP IMMEDIATELY**. Go to **Fallback Handling** (step 4), output the exact message, and **WAIT for the user**.
   - **HARD PROHIBITION — when chrome-devtools is unavailable, you MUST NOT, under any circumstance:**
     - use playwright (`mcp__playwright__*`), the `fs` MCP, `WebFetch`, `Bash`+curl, or ANY other tool to open / fetch / scrape the URL;
     - silently pick "a similar tool that can also do the job" and continue;
     - guess or reconstruct page content from the URL alone.
   - There is no acceptable degraded path that fetches the page with another tool. The ONLY acceptable actions when chrome-devtools is unavailable are: (a) stop and ask the user to fix the MCP, or (b) accept user-provided pasted text / exported file. Anything else is a violation.

2. **Access dynamic web pages with chrome-devtools MCP:**
   - Use `mcp__chrome-devtools__navigate_page` to load the URL
   - Wait for page render: `mcp__chrome-devtools__wait_for` (wait for network idle or key element)
   - Extract content: `mcp__chrome-devtools__take_snapshot` (DOM/a11y text). For canvas-rendered pages, capture the underlying API payloads with `mcp__chrome-devtools__list_network_requests` (list) then `mcp__chrome-devtools__get_network_request` (fetch one request's response body)
   - For Modao CC specifically: wait for `.board-container` or similar canvas elements

3. **Extract requirements from the rendered page:**
   - Ask user to log into Modao.cc manually before starting
   - Wait for full page load (3+ second pauses)
   - Parse functional requirements, user stories, acceptance criteria
   - Identify state machines, workflows, business rules
   - Mark ambiguous or missing information as `[TBD]`

4. **Fallback Handling (when chrome-devtools MCP unavailable) — STOP and report, do NOT proceed:**
   If the chrome-devtools MCP is unavailable, output the message below verbatim and then **WAIT**. Do not attempt the task with any other tool.
   ```
   ⚠️ chrome-devtools MCP 未连接，无法访问动态渲染页面（如墨刀、飞书等）。

   本 Skill 规定：访问动态页面只能使用 chrome-devtools，禁止用其他工具（playwright/fs/WebFetch 等）替代。
   因此现在已停止，等待你处理后再继续。

   **请选择其一：**
   1. 连接 chrome-devtools MCP（命令：claude mcp add chrome-devtools -s user -- cmd /c npx -y chrome-devtools-mcp@latest，需重启会话），然后让我重试
   2. 或直接提供可离线读取的内容：
      - 粘贴 PRD 文本
      - 导出为 Markdown/PDF 文件并给出路径
      - 提供可直接访问的静态 HTML 链接
   ```

5. **After content extraction, silently process:**
   1. **Requirement Classification:** Identify OR, RS, NR, CO and assign change types
   2. **Scope Inventory:** List all functional areas
   3. **Boundary Definition:** Define non-goals with explicit boundary conditions
   4. **Domain Modeling:** Identify entities and their state transitions
   5. **Contract Extraction:** Identify all synchronous APIs and asynchronous events
   6. **Impact Assessment:** Determine change scope

### Step 2: Generate prd.md

**Load template on-demand:**
```
[Read templates/prd-phase1-template.md for complete template structure]
```

Follow the template format and write the file to disk.

**Output location & naming — MUST follow `[Read rules/output-structure.md]`:**
- Write each requirement's Phase 1 doc as a separate `.md` file into `APP_v<version>/prd_md/`.
- File name in Simplified Chinese, self-explanatory, no version number (version is the folder).
- Derive the version folder from the materials' version (e.g., `4.19` → `APP_v4.19.0`); if no version is given, ask the user.

### Step 2.1: Append Handoff Envelope

After generating prd.md, append the following handoff envelope after Section 13 (the last section):

```html
<!-- PRD-HANDOFF-ENVELOPE
targetModules: [comma-separated pitpat-h5 业务域/模块名, mapped from PAGE-XXX and the CLAUDE.md knowledge-graph index, e.g., reports, hardware, ai-coach]
affectedPages: [comma-separated page/弹窗 from PAGE-XXX]
nativeCaps: [comma-separated JSBridge/native capabilities from CAP-XXX, e.g., NFC, 蓝牙, 3D]
i18nImpact: [需新增/改动的 i18n 模块或 key 范围概述]
keyConstraints: [top 3-5 business/前端 rules from RULE-XXX 与 §4.5 单位规则, each one line]
scopeSummary: P0=[count] P1=[count] frontendScope=[total FS-XXX count]
/PRD-HANDOFF-ENVELOPE -->
```

**Envelope format notes:**
- Uses HTML comment markers so it does not render in markdown viewers
- Phase 2 locates it by searching for `PRD-HANDOFF-ENVELOPE` string
- `targetModules` should map to actual pitpat-h5 domains using the knowledge-graph index in pitpat-h5 `CLAUDE.md` (e.g., `运动报告/reports`, `硬件/hardware`, `AI 教练/ai-coach`)
- This envelope is a quick-scan index — Phase 2 still reads full prd.md for business context

---

## Phase 1 Execution Constraints

1. **Language:** All generated content MUST be in Simplified Chinese.
2. **No Visual Design (but keep behavior):** Prohibited in all sections: layouts, visual designs, component trees, animation specs, interaction wireframes. REQUIRED in §4.1/§4.4–§4.9: interaction logic, exact copy, UI states, unit/i18n display rules, native-capability/platform/version compatibility. Do not let "no UI" delete copy/states/units.
3. **No Implementation Terms (scope = §7 business rules only):** §7 业务规则 uses business language ONLY (prohibited: Redis, MQ, cache, lock, sharding, TCC, Saga, etc.). This does NOT apply to §4.1/§4.4/§4.7, where frontend interface dependencies and JSBridge native capabilities MAY be named.
4. **Format Lock:** Do not convert lists to tables, do not append after Section 13 (handoff envelope is the only exception), do not modify H1/H2 headings.
5. **H4 Mandate:** Section 2.1 MUST use `####` for categorization.
6. **Sequential Numbering:** All identifiers must be sequential, no gaps.
7. **Change Type Tags:** Every OR/NR/RS item MUST have `[NEW]`, `[MOD]`, `[DEL]`, `[OPT]`.
8. **Priority Tags:** Every goal (G-XXX) MUST have `P0`, `P1`, or `P2`.
9. **chrome-devtools MCP ONLY (no fallback tool):** Accessing URLs for dynamic pages MUST use chrome-devtools MCP, verified via the Step 1.1 pre-flight gate. If it is unavailable, you MUST STOP and ask the user to fix it or provide offline content. NEVER substitute playwright, fs, WebFetch, Bash+curl, or any other tool, and NEVER continue silently.
