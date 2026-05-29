# 更新日志

## [v1.0.0] - 2026-05-29

### 🎉 首次发布

**PRD Compiler v5** 是一个两阶段 PRD 编译器，专为 pitpat-h5 前端团队设计，将非结构化 PRD 素材转换为结构化前端工程文档。

#### ✨ 核心功能

- **Phase 1**：将墨刀/Figma/会议记录等非结构化素材提炼为业务需求文档（`.md`）
  - 自动分类需求（OR/RS/NR/CO）
  - 生成领域模型、状态机、业务规则
  - 输出前端维度的 i18n / 公英制 / UI 状态 / 端能力清单

- **Phase 2**：基于 Phase 1 输出，生成前端技术实现文档（`.md`）
  - 结合 pitpat-h5 工程规范（Vue3 + Vant + Vuex + vue-i18n）
  - 源码扫描映射到具体组件/路由/状态管理
  - 输出架构风险清单和技术决策

#### 📝 输出结构规范

- 每个版本独立目录：`APP_v<MAJOR>.<MINOR>.<PATCH>/`
- `prd_md/` — 业务需求文档（Phase 1 输出）
- `document/` — 技术实现文档（Phase 2 输出）
- 文件命名：简体中文、见名知义、无需版本号

示例：
```
APP_v4.19.0/
├── document/
│   ├── 积分签到（每日签到与补签卡）方案设计.md
│   └── 商城入口迁移方案设计.md
└── prd_md/
    ├── 积分签到（每日签到与补签卡）.md
    └── 商城入口迁移.md
```

#### 🔧 工程规范

- 新增 `CLAUDE.md`：项目性质、触发条件、两阶段架构说明
- 新增 `SKILL.md`：Skill 入口、输出结构引用、阶段索引表
- 新增 `rules/output-structure.md`：输出目录与文件命名唯一权威来源
- 新增 `.gitignore`：忽略运行时缓存（`output/`、`.playwright-mcp/`）

#### 📚 模板与工作流

**Phase 1 模板更新**（`templates/prd-phase1-template.md`）：
- 强化前端完整性捕获（§4.4–§4.9）
  - §4.4 文案与多语言（i18n）
  - §4.5 单位与本地化（公英制/配速/时区）
  - §4.6 UI 状态覆盖（空/加载/错误/缺省）
  - §4.7 端能力与兼容（JSBridge、iOS/Android 差异）
  - §4.8 涉及页面与路由清单
  - §4.9 埋点
- 统一编号规范（OR/RS/NR/CO、G、FS/BS/TS、DO、STATE、RULE、PERM、ERR、DATA、OBS、AC、OQ）
- 强化验收标准（AC 必须包含正向 + 边界/异常）

**Phase 2 模板更新**（`templates/prd-phase2-template.md`）：
- 新增扫描缓存机制（`.prd-scan-cache.json`）
- 新增架构风险清单（§10）
- 强调前端工程约束（组件复用、路由规范、i18n 下发、埋点规范）

#### 🚀 工作流增强

**Phase 1 工作流更新**（`phases/phase1.md`）：
- 优先使用 chrome-devtools MCP 访问动态页面（墨刀/Figma）
- 明确"禁止 UI 设计"≠"禁止交互/文案/状态"：必须捕获交互逻辑、文案、UI 状态、单位规则
- 新增输出位置与命名强制引用 `rules/output-structure.md`
- 改进 Handoff Envelope 字段：
  - `targetModules`：业务域/模块名
  - `affectedPages`：涉及页面/弹窗
  - `nativeCaps`：JSBridge 原生能力
  - `i18nImpact`：i18n 影响范围

**Phase 2 工作流更新**（`phases/phase2.md`）：
- 新增扫描缓存机制（避免重复扫描，提升效率）
- 新增架构风险检查（组件复用风险、路由命名冲突、i18n 下发遗漏）
- 强化源码扫描 → 前端映射 → 风险清单的完整链路

#### 📊 版本对比

| 指标 | 初始版本 | 当前版本 | 变化 |
|------|---------|---------|------|
| 文件数 | 4 | 8 | +4 |
| 代码行数 | 579 | 795 | +216 |
| 核心文档 | 4 | 8 | +4 |
| 规范文件 | 0 | 3 | +3 |

#### 🔍 详细变更

**新增文件**：
- `CLAUDE.md` — 项目说明与工程规范
- `SKILL.md` — Skill 入口与引用索引
- `rules/output-structure.md` — 输出结构唯一规范
- `.gitignore` — 忽略运行时缓存

**重大更新**：
- `phases/phase1.md`：前端完整性捕获 + chrome-devtools MCP 优先 + Handoff Envelope 改进
- `phases/phase2.md`：扫描缓存 + 架构风险检查 + 前端工程约束
- `templates/prd-phase1-template.md`：新增 §4.4–§4.9 前端维度 + 统一编号规范
- `templates/prd-phase2-template.md`：新增扫描缓存机制 + 架构风险清单

#### 🎯 适用场景

- 产品经理：提供墨刀/Figma 原型，自动生成结构化 PRD
- 前端开发：从 PRD 直接生成技术实现文档，含组件映射、路由规划、i18n 清单
- 技术评审：自动生成架构风险清单，提前识别潜在问题

#### 📖 使用示例

```bash
# Phase 1：墨刀原型 → 业务需求文档
用户提供墨刀链接 → CLAUDE 自动提取需求 → 生成 prd_md/XXX.md

# Phase 2：业务需求 → 技术实现文档
读取 prd_md/XXX.md → 扫描 pitpat-h5 源码 → 生成 document/XXX方案设计.md
```

---

## [未发布]

### 未来规划

- [ ] 支持更多 PRD 来源（飞书文档、Notion、Confluence）
- [ ] 增加组件自动推荐功能（基于需求描述匹配现有组件）
- [ ] 增加埋点代码自动生成
- [ ] 增加测试用例自动生成
- [ ] 支持导出为 PDF/Word 格式

---

**Full Changelog**: https://github.com/asen-coder/pitpat-prd-compiler-web-v1/commits/master
