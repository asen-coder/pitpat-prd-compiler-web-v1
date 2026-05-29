# Phase 2 输出模板: 前端技术方案（pitpat-h5）

> 本模板产出**前端技术实现文档**（描述 How），映射到 pitpat-h5（Vue3 + Vant + Vuex + vue-i18n + Axios + Stylus）。
> 所有工程约定（组件写法、文件命名、API 分层、i18n 工作流、公英制换算、CodeGraph 检索）遵循 **pitpat-h5 的 `CLAUDE.md`**，不在模板中重复。

---

## 模板结构

```markdown
# [Title] — 前端技术方案

## 1. 背景
- 版本信息：[version]
- 需求信息：[对应 prd.md 文件名/链接]
- 相关 PRD 地址：[prd 链接或文件路径]
- 一句话技术目标：[本需求前端要做什么]

## 2. 涉及页面与路由
> 来自 prd.md §4.8，结合代码库实际路径。
- 受影响/新增页面：[src/views 下的目录与组件，标注 新增/改动]
- 路由：[src/router 新增/改动的路由配置]
- 组件拆分：[页面级组件 + 局部 components/ 子组件；可复用的公共组件]

## 3. 整体方案
- 页面流程图 / 跳转关系：[Mermaid]
- 关键交互时序：[用户操作 → 组件 → api → store/状态 → 视图更新，Mermaid sequence]

## 4. 接口对接（API）
> 所有请求走 src/api/ 层，禁止组件内直接用 axios。
### 4.1 接口清单
#### API-XXX: [接口用途]
- api 文件与方法：[src/api/xxx.ts 中的方法名（遵循 kebab-case 文件、与业务域对应）]
- 请求参数：[前端需传的字段]
- 响应字段：[前端要用的字段]
- 状态：[复用现有 / 新增（需后端提供，附后端接口文档位）]
- 错误处理：[错误码 → 前端表现（toast/缺省/重试）]

## 5. 状态管理（Vuex）
- 涉及 store 模块/字段：[读取或新增的 state/getter/mutation/action]
- 跨页面共享：[哪些状态需跨页面共享，为什么放 store]

## 6. 国际化（i18n）
> 禁止硬编码展示文本；新增 key 同步 en/fr/de；新增模块在 en/index.ts 注册。详见 pitpat-h5 `.claude/i18n/i18n-workflow.md`。
- 新增 key：[模块/key 列表，对应 prd.md §4.4 COPY-XXX 的源文案]
- 复用 key：[已存在可复用的 key]
- 模块注册：[是否需在 src/i18n/lang/en/index.ts 新增模块]

## 7. 单位与本地化实现
> 对应 prd.md §4.5。依据 store.state.userInfo.metricType（0=公制，1=英制）。
- 公英制换算：[字段 → 换算规则；km→mile 系数 1.6；配速 公制 5'23" / 英制 8'42"]
- 时间/时区/日期：[展示格式与用户时区处理]
- 复用的换算工具：[src/utils 下已有的换算/格式化方法]

## 8. 端能力与兼容
> 对应 prd.md §4.7。
- JSBridge 原生能力：[调用的原生方法：NFC/蓝牙/3D/跳转系统设置等；对应 utils 封装]
- iOS / Android 差异：[差异点与分支处理]
- App 版本兼容：[新老版本表现、降级策略、灰度]

## 9. 关键技术与复用
1. **组件选型**：[使用的 Vant 组件、是否需新增公共组件]
2. **复用优先**：[可复用的现有 公共组件 / hooks / utils（先查 CodeGraph 与知识图谱索引，避免重复造轮子）]
3. **性能**：[长列表、图片、懒加载、首屏等，如涉及]
4. **三方库**：[echarts / lottie / html2canvas / three 等是否引入]

## 10. 埋点
> 对应 prd.md §4.9。
- 埋点事件与参数：[事件名、触发时机、上报参数、对应埋点工具]

## 11. 测试与自测要点
- [关键自测路径、需覆盖的状态与机型/语言/制式组合]

## 12. 架构风险清单（前端）
[见 phase2.md Step 5 的检查项，按 BLOCK/WARN 输出]
```

> 工程约定一律以 pitpat-h5 `CLAUDE.md` 为准，不在文档中复制。
