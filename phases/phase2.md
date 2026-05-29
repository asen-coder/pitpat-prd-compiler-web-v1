# Phase 2: Generate frontend technical doc — Implementation Mapping (pitpat-h5)

This file contains the complete Phase 2 execution logic. Load this file when the user wants to generate the **frontend technical doc** from an existing `prd.md`, mapped to the **pitpat-h5** frontend project.

> Target project: **pitpat-h5** (Vue3 + TypeScript + Vite + Vant + Vuex + Vue Router + vue-i18n + Axios + Stylus). If the current working directory is not the pitpat-h5 repo, ask the user for its path before scanning.

---

## Phase 2 Principles

1. **Project Context First:** MUST read pitpat-h5's `CLAUDE.md` before generating (tech stack, coding conventions, i18n workflow, 公英制 rules, knowledge-graph index, forbidden practices).
2. **Module Mapping:** Map every frontend requirement to actual pitpat-h5 modules (`src/views`, `src/router`, `src/api`, `src/store`, `src/components`, `src/i18n`, `src/hooks`, `src/utils`).
3. **Convention Compliance:** Apply pitpat-h5 conventions — i18n (`$t`, no hardcoded text, en/fr/de sync), API layer only (no axios in components), kebab-case files, `<script lang="ts" setup>` Composition API, Stylus styles, 公英制 via `metricType`.
4. **Task Dependency Ordering:** Decompose into phases (e.g., Phase 0: 路由与页面骨架/i18n key, Phase 1: 接口对接与核心交互, Phase 2: 端能力/兼容/埋点).
5. **Reuse First:** Prefer reusing existing 公共组件 / hooks / utils / i18n keys over creating new ones.
6. **Source Code Is the Authoritative Truth:** The pitpat-h5 **source code (via CodeGraph)** is the single authoritative source. Knowledge docs (`.claude/Document/Code/*.md`) and the knowledge-graph index are **optional accelerators only** — they may be stale and MUST NOT be treated as ground truth. MUST scan the actual codebase before generating; never generate a design from docs alone. If a module has no doc or index entry, locate it directly via `codegraph_search`/`codegraph_context`. Use discovered patterns (existing api method signatures, store fields, util/hook names, i18n key style, component conventions) so the new design aligns with the real code. A technical doc not grounded in source code has no real value.
7. **Architecture Risk Awareness:** MUST check the generated design against known frontend risks (巨型组件、组件内直接 axios、硬编码文案、非 Stylus 样式、重复造轮子、i18n 未同步、公英制未走统一换算) and report with severity (BLOCK/WARN).

---

## Handoff Envelope Reading

Before scanning, read the handoff envelope from prd.md:

1. Search for `PRD-HANDOFF-ENVELOPE` marker in the generated prd.md.
2. Extract `targetModules` for targeted scan scope (Step 3.3) — map to pitpat-h5 domains via the knowledge-graph index in pitpat-h5 `CLAUDE.md`.
3. Extract `affectedPages` for `src/views`/`src/router` focus, `nativeCaps` for JSBridge focus, `i18nImpact` for i18n module focus.
4. Also read full prd.md for business context validation and the §4.4–§4.9 frontend dimensions.

The envelope accelerates module identification but does NOT replace full prd.md reading. If the envelope is missing (older prd.md), parse the prd.md `§4.1/§4.8` and PAGE-XXX/FS-XXX to derive scope.

---

## Phase 2 Workflow

### Step 3: Collect Project Context

#### Step 3.1: Read pitpat-h5 CLAUDE.md

Read pitpat-h5's `CLAUDE.md` to extract:
- Tech stack and version constraints
- Directory structure and module responsibilities
- Coding conventions (component style, file naming, API layer, Stylus)
- i18n rules and workflow (`.claude/i18n/i18n-workflow.md`)
- 公英制换算 rules (`metricType`, km×1.6, 配速格式)
- The **knowledge-graph index** (业务域 → 文档路径) used to locate target modules
- Forbidden practices (no axios in components, no hardcoded text, no non-Stylus syntax)

#### Step 3.1b: Scan Cache Check

Before executing Steps 3.2-3.3, check for a valid scan cache:

1. Determine the prd directory (same directory as the input prd.md).
2. Check if `{prd-directory}/.prd-scan-cache.json` exists.
3. If exists: read it, compare `gitSha` with `git rev-parse HEAD` of the **pitpat-h5 repo**.
   - SHA matches → **cache hit**: skip Steps 3.2-3.3, use cached data for Step 4.
   - SHA mismatches → **cache stale**: proceed to full scan.
   - If pitpat-h5 is not a git repo or HEAD is unavailable → skip caching, always full scan.
4. If not exists → proceed to full scan.

#### Step 3.2: Lightweight Scan (CodeGraph first)

> pitpat-h5 `CLAUDE.md` mandates CodeGraph for code lookup. Use `mcp__codegraph__*` first; fall back to Glob/Grep/Read only when CodeGraph cannot locate.

1. **Module location:** From `targetModules` (or prd.md analysis), use the knowledge-graph index to locate the module docs and code paths (`src/views/...`, `src/api/...`).
2. **Page/route inventory:** Locate affected pages under `src/views` and their route entries in `src/router`.
3. **API layer inventory:** Find the matching `src/api/*.ts` file(s) for the domain and list existing exported methods.
4. **i18n inventory:** Find the domain's i18n module under `src/i18n/lang/en/` and whether it is registered in `en/index.ts`.
5. **Giant component detection:** Use `wc -l` on the target `.vue` files; flag large single-file components: WARN >600 lines, BLOCK >1500 lines (adjust to repo norms if CLAUDE.md specifies).

#### Step 3.3: Deep Targeted Scan

For each target module, scan:

1. **Existing API methods:** Read the domain's `src/api/*.ts` to extract method naming style and request/response shapes; reuse when the PRD needs the same data.
2. **Reusable components/hooks/utils:** Search `src/components`, `src/hooks`, `src/utils` (and the 可复用能力 index) for existing capabilities (上传/海报/倒计时/复制/JSBridge/公英制换算/埋点) to reuse instead of rebuilding.
3. **i18n key style:** Inspect the domain's existing keys to match naming and structure; identify which new keys are needed.
4. **Store usage:** Read relevant `src/store` modules/fields the feature reads or must extend.
5. **公英制 / 本地化 utils:** Locate the existing unit-conversion and date/timezone helpers in `src/utils`; new display must reuse them.
6. **JSBridge / native:** Locate the existing JSBridge wrapper in `src/utils` and the native methods it exposes (NFC/蓝牙/3D/跳转设置), for §8 design.
7. **Component conventions:** Confirm the domain follows `<script lang="ts" setup>` + Stylus + kebab-case so new components match.

#### Step 3.4: Write Scan Cache

After a full scan, write results to `{prd-directory}/.prd-scan-cache.json`:

```json
{
  "version": 2,
  "project": "pitpat-h5",
  "gitSha": "<output of git rev-parse HEAD in pitpat-h5>",
  "timestamp": "<ISO 8601 datetime>",
  "lightweightScan": {
    "modulePaths": {},
    "pageRouteInventory": {},
    "apiInventory": {},
    "i18nInventory": {},
    "giantComponents": []
  },
  "targetedScans": {
    "<moduleName>": {
      "apiMethods": [],
      "reusableComponents": [],
      "reusableHooksUtils": [],
      "i18nKeyStyle": [],
      "storeFields": [],
      "unitLocaleUtils": [],
      "jsBridgeMethods": []
    }
  }
}
```

### Step 4: Generate the frontend technical doc

**Load template on-demand:**
```
[Read templates/prd-phase2-template.md for complete template structure]
```

Follow the template format. Per `rules/output-structure.md`, the final artifact is a `.md` file in `APP_v<version>/document/`, file name = prd_md counterpart + `方案设计` suffix.

**Silent quality boost — apply scan findings (or cached data) to template sections:**

| Template Section | How Scan Findings Improve Quality |
|----------------|-----------------------------------|
| §2 涉及页面与路由 | Real `src/views` paths and `src/router` entries from Step 3.2.1-3.2.2 |
| §4 接口对接 | Match existing `src/api` method names/shapes from Step 3.3.1; mark truly-new APIs that need backend |
| §5 状态管理 | Use actual `src/store` modules/fields from Step 3.3.4 |
| §6 国际化 | Match existing i18n module/key style from Step 3.2.4/3.3.3; only flag genuinely new keys |
| §7 单位与本地化 | Reuse existing 公英制/date utils from Step 3.3.5 (not ad-hoc conversion) |
| §8 端能力与兼容 | Reference the existing JSBridge wrapper/methods from Step 3.3.6 |
| §9 关键技术与复用 | Recommend reusing components/hooks/utils found in Step 3.3.2 |

### Step 5: Architecture Guardrail Check (frontend)

After generating the doc, run these checks against the output and the scan data:

**BLOCK-level checks (must resolve):**

| # | Check | Threshold | Action |
|---|-------|-----------|--------|
| B1 | Giant component | New code lands in a `.vue` >1500 lines | Recommend splitting into sub-components under `components/` |
| B2 | axios in component | Design calls axios directly in a component | Recommend moving to `src/api/` layer |
| B3 | Hardcoded display text | Any user-facing text not via `$t` | Recommend i18n keys + en/fr/de sync |
| B4 | Non-Stylus style | Design uses SCSS/LESS syntax in `<style>` | Recommend Stylus |

**WARN-level checks (should address):**

| # | Check | Threshold | Action |
|---|-------|-----------|--------|
| W1 | Giant component proximity | Target `.vue` >600 lines | Suggest evaluating a split |
| W2 | Reinventing | New util/component duplicates an existing one found in Step 3.3.2 | Suggest reuse |
| W3 | i18n not synced | New key added to `en/` only | Suggest fr/de sync via i18n workflow |
| W4 | Ad-hoc unit conversion | Distance/pace converted without the shared util | Suggest the existing 公英制 helper |
| W5 | Missing UI state | A data page lacks empty/loading/error handling (prd.md §4.6) | Suggest covering all states |

**Output format — append as Section 12 of the frontend technical doc:**

```markdown
## 12. 架构风险清单（前端）

### BLOCK 级风险
- **RISK-001**: [BLOCK] 在 `xxx.vue`（X 行）继续堆叠新逻辑
  - 建议：拆分为 `components/` 下的子组件

### WARN 级风险
- **RISK-002**: [WARN] 新增换算逻辑与 `src/utils/xxx` 重复
  - 建议：复用现有公英制换算工具
```

If no risks are found:
```markdown
## 12. 架构风险清单（前端）

> 本次技术方案未检测到架构风险。
```

---

## Phase 2 Execution Constraints

1. **Language:** All generated content MUST be in Simplified Chinese.
2. **Project Context:** MUST read pitpat-h5's `CLAUDE.md` before generating.
3. **Module Accuracy:** Map to actual pitpat-h5 modules/paths only (verified via CodeGraph/scan), never invented paths.
4. **Convention Compliance:** All designs MUST follow pitpat-h5 conventions (i18n, API layer, Composition API, Stylus, kebab-case, 公英制).
5. **Codebase Scan:** MUST perform Step 3.2 (lightweight) and Step 3.3 (deep targeted) before generating, unless a valid scan cache exists. Use CodeGraph first.
6. **Guardrail Output:** MUST append the frontend architecture risk checklist (Section 12) after Step 5.
7. **Reuse & Pattern Alignment:** All designs MUST align with patterns discovered in Step 3.3 (existing api methods, store fields, util/hook names, i18n key style, component style), and prefer reuse over new code.
8. **Output:** Final artifact per `rules/output-structure.md` — `.md` in `APP_v<version>/document/`, one per requirement, file name = prd_md counterpart + `方案设计` suffix.
