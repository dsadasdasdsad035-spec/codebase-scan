# codebase-scan

> 通过 IIDP `iidp-codebase-scan` skill 对存量代码库做逆向工程，自动产出系统架构、业务流程、状态机、UML、SRS 等可点开的 Markdown 文档。本仓库包含两个完整的扫描示例（`example/yudao-ui-admin-vue3` 与 `example/ruoyi-vue-pro`）以及执行 skill 所需的全部默认配置。

实现思路：
1. README.md 以目录树形式列出 `example/` 中所有可点开的文档，每篇附一句话简介 + 相对路径链接。
2. 单独提供"skills 使用教程"章节，把 SKILL.md 中最常用的 4 种调用范式浓缩成可复制的命令。

---

## 目录

- [仓库结构](#仓库结构)
- [example 索引](#example-索引)
  - [yudao-ui-admin-vue3：商城商品中心前端](#yudao-ui-admin-vue3商城商品中心前端)
  - [ruoyi-vue-pro：商城商品中心后端](#ruoyi-vue-pro商城商品中心后端)
- [skills 使用教程](#skills-使用教程)
  - [1. 最快体验：单模组仓库](#1-最快体验单模组仓库)
  - [2. 显式指定一个后端包入口](#2-显式指定一个后端包入口)
  - [3. 显式指定一个前端视图入口](#3-显式指定一个前端视图入口)
  - [4. 启用受控关联文件扩展](#4-启用受控关联文件扩展)
  - [常用参数速查](#常用参数速查)
  - [默认配置与产物路径](#默认配置与产物路径)
  - [质量门禁关键字段](#质量门禁关键字段)

---

## 仓库结构

```text
codebase-scan/
├── README.md                            ← 你正在阅读
├── example/                             ← 两个完整的代码扫描产物示例
│   ├── yudao-ui-admin-vue3/              ← 前端示例（Vue3 + Element Plus）
│   │   ├── DOCUMENTATION.md
│   │   ├── architecture.md / business-flows.md / data-model.md
│   │   ├── state-machines.md / error-handling.md / database.md
│   │   ├── srs.md / quality.json
│   │   ├── entries/frontend-mall-product/
│   │   ├── uml/{business,technical}/
│   │   ├── flowcharts/
│   │   └── .codebook/                   ← 私有索引（collect/evidence/manifest 等）
│   └── ruoyi-vue-pro/                   ← 后端示例（Spring Boot + MyBatis-Plus）
│       ├── DOCUMENTATION.md
│       ├── architecture.md / business-flows.md / data-model.md
│       ├── state-machines.md / error-handling.md / database.md
│       ├── srs.md / quality.json
│       ├── entries/backend-package-yudao-module-product/
│       ├── uml/{business,technical}/
│       ├── flowcharts/
│       └── .codebook/                   ← 私有索引
└── skills/
    └── codebase-scan/                   ← iidp-codebase-scan skill 定义
        ├── SKILL.md                     ← skill 主文件（12 步执行流程）
        └── resources/codebook/defaults/
            ├── manifest.json
            ├── entries.json
            ├── codegraph-tools.json
            ├── semantic-kinds.json
            ├── typecard-policy.json
            └── prompts/
                ├── temp_typecard.json
                ├── temp_architecture.json
                ├── temp_business_flows.json
                ├── temp_data_model.json
                ├── temp_srs.json
                └── temp_ui.json
```

---

## example 索引

`example/` 收录了两套完整、互相对应的代码扫描产物：前者是商城的**前端**实现，后者是商城的**后端**实现。两者共享同一组业务概念（品牌 / 分类 / 属性 / 评价 / SPU-SKU），可作为对照阅读。

### yudao-ui-admin-vue3：商城商品中心前端

入口 ID：`frontend-mall-product`
源码范围：`yudao-ui-admin-vue3/src/views/mall/product/**`
规模：26 个 .vue/.ts 文件，490 个原生节点，141 个可分析符号
配套后端：[ruoyi-vue-pro](#ruoyi-vue-pro商城商品中心后端)

#### 系统级文档

| 文档 | 简介 |
|---|---|
| [DOCUMENTATION.md](example/yudao-ui-admin-vue3/DOCUMENTATION.md) | 入口文档总览，串联所有系统级文档与 UML/flowchart 索引 |
| [architecture.md](example/yudao-ui-admin-vue3/architecture.md) | 系统级架构：模块依赖、组件复用、跨入口调用 |
| [business-flows.md](example/yudao-ui-admin-vue3/business-flows.md) | 7 个核心业务流程汇总（品牌 / 分类 / 属性 / 评价 / SPU 等） |
| [data-model.md](example/yudao-ui-admin-vue3/data-model.md) | 9 个前端实体 + ER 关系 + 关键约束 |
| [state-machines.md](example/yudao-ui-admin-vue3/state-machines.md) | 6 类状态机汇总（SPU 状态、表单弹窗、评价可见性等） |
| [error-handling.md](example/yudao-ui-admin-vue3/error-handling.md) | 9 类错误处理模式（表单校验、Switch 回滚、价格校验等） |
| [database.md](example/yudao-ui-admin-vue3/database.md) | ER 图 + 写入/读取路径（前端视角） |
| [srs.md](example/yudao-ui-admin-vue3/srs.md) | 技术无关 SRS：FR/NFR + 追溯表 |
| [quality.json](example/yudao-ui-admin-vue3/quality.json) | 质量评估结果（tech-agnostic / completeness / uml coverage） |

#### 入口级文档（`entries/frontend-mall-product/`）

| 文档 | 简介 |
|---|---|
| [DOCUMENTATION.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/DOCUMENTATION.md) | 入口详细文档：子域、文件统计、权限点、source_nodes |
| [architecture.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/architecture.md) | 入口依赖：内部 API、Element Plus、项目工具 |
| [business-flows.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/business-flows.md) | 7 个核心流程详解 |
| [data-model.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/data-model.md) | 9 个 VO 实体的字段与来源 |
| [state-machines.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/state-machines.md) | 6 类状态机详解 |
| [error-handling.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/error-handling.md) | 错误处理详解 |
| [database.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/database.md) | 实体 ER 图 + 写入/读取路径 |
| [ui-layout.md](example/yudao-ui-admin-vue3/entries/frontend-mall-product/ui-layout.md) | 每个 page 的"搜索栏 + 表格 + 弹窗"布局图（Mermaid） |

#### UML 图（`uml/`）

业务状态机：

- [state-spu.md](example/yudao-ui-admin-vue3/uml/business/state-spu.md) — SPU 销售状态（ENABLE / DISABLE / RECYCLE）
- [state-comment-visible.md](example/yudao-ui-admin-vue3/uml/business/state-comment-visible.md) — 评价可见性 Switch 切换 + 异常回滚
- [state-form-dialog.md](example/yudao-ui-admin-vue3/uml/business/state-form-dialog.md) — 所有 *Form.vue 的 dialogVisible 显隐
- [class-entity.md](example/yudao-ui-admin-vue3/uml/business/class-entity.md) — 前端 TS 实体的 classDiagram

技术视图：

- [component-diagram.md](example/yudao-ui-admin-vue3/uml/technical/component-diagram.md) — 外部 UI 库 + 公共组件 + 项目工具
- [hotspot-callers.md](example/yudao-ui-admin-vue3/uml/technical/hotspot-callers.md) — handleDelete / openForm 等跨页同名函数

#### 流程图（`flowcharts/`）

- [flowchart.md](example/yudao-ui-admin-vue3/flowcharts/flowchart.md) — 整体导航关系
- [sequence-brand-crud.md](example/yudao-ui-admin-vue3/flowcharts/sequence-brand-crud.md) — 品牌 CRUD 序列图
- [sequence-category-crud.md](example/yudao-ui-admin-vue3/flowcharts/sequence-category-crud.md) — 分类树 CRUD（含跨入口跳转 SPU 列表）
- [sequence-property-flow.md](example/yudao-ui-admin-vue3/flowcharts/sequence-property-flow.md) — 属性 / 属性值两级管理
- [sequence-spu-status.md](example/yudao-ui-admin-vue3/flowcharts/sequence-spu-status.md) — SPU 5-Tab 状态流转
- [sequence-spu-form.md](example/yudao-ui-admin-vue3/flowcharts/sequence-spu-form.md) — SPU 5-Tab 复合表单提交
- [sequence-comment-flow.md](example/yudao-ui-admin-vue3/flowcharts/sequence-comment-flow.md) — 评价多条件管理
- [sequence-component-reuse.md](example/yudao-ui-admin-vue3/flowcharts/sequence-component-reuse.md) — 跨子域 SPU/SKU 组件复用

---

### ruoyi-vue-pro：商城商品中心后端

入口 ID：`backend-package-yudao-module-product`
源码范围：`ruoyi-vue-pro/yudao-module-mall/yudao-module-product/**`
规模：128 个 Java 源文件，2132 个原生节点，488 个可分析符号
配套前端：[yudao-ui-admin-vue3](#yudao-ui-admin-vue3商城商品中心前端)

#### 系统级文档

| 文档 | 简介 |
|---|---|
| [DOCUMENTATION.md](example/ruoyi-vue-pro/DOCUMENTATION.md) | 入口文档总览：7 大子域 + 53 admin + 12 app endpoint |
| [architecture.md](example/ruoyi-vue-pro/architecture.md) | 系统级架构：四层分层 + 跨模块 RPC |
| [business-flows.md](example/ruoyi-vue-pro/business-flows.md) | 9 个核心业务流程（CRUD + 复合表单 + RPC） |
| [data-model.md](example/ruoyi-vue-pro/data-model.md) | 9 个 DO 实体 + 主键策略 + 字段类型 |
| [state-machines.md](example/ruoyi-vue-pro/state-machines.md) | 5 类状态机（SPU / 分类层级 / 评价可见性 / 启停 / SKU 规格） |
| [error-handling.md](example/ruoyi-vue-pro/error-handling.md) | 分层错误处理 + 8 组错误码（24 条） |
| [database.md](example/ruoyi-vue-pro/database.md) | 9 张核心表 + 关系 + 索引 + 缓存策略 |
| [srs.md](example/ruoyi-vue-pro/srs.md) | 技术无关 SRS：8 项业务能力 + FR/NFR |
| [quality.json](example/ruoyi-vue-pro/quality.json) | 质量评估结果 |

#### 入口级文档（`entries/backend-package-yudao-module-product/`）

| 文档 | 简介 |
|---|---|
| [DOCUMENTATION.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/DOCUMENTATION.md) | 入口详细文档：7 大子域、文件统计、权限点、异常码 |
| [architecture.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/architecture.md) | 入口依赖：Spring Boot + MyBatis-Plus + yudao-framework |
| [technical-architecture.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/technical-architecture.md) | 技术栈、并发模型、可观测性 |
| [business-flows.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/business-flows.md) | 9 个核心流程详解（主路径 + 失败路径） |
| [data-model.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/data-model.md) | 9 个 DO 实体的字段与业务约束 |
| [state-machines.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/state-machines.md) | 5 类状态机详解 |
| [error-handling.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/error-handling.md) | Controller / Service / Repository 分层错误处理 |
| [database.md](example/ruoyi-vue-pro/entries/backend-package-yudao-module-product/database.md) | 9 张表的 DDL + 索引 + 缓存策略 |

#### UML 图（`uml/`）

业务状态机：

- [state-spu.md](example/ruoyi-vue-pro/uml/business/state-spu.md) — 商品 SPU 销售状态机
- [state-category.md](example/ruoyi-vue-pro/uml/business/state-category.md) — 商品分类层级（顶级 / 一级 / 二级）
- [state-comment.md](example/ruoyi-vue-pro/uml/business/state-comment.md) — 评价可见性 × 回复状态
- [class-entity.md](example/ruoyi-vue-pro/uml/business/class-entity.md) — 9 个 DO 的继承关系（BaseDO 派生）

技术视图：

- [component-diagram.md](example/ruoyi-vue-pro/uml/technical/component-diagram.md) — 外部调用方 + 模块拓扑（Admin / App / RPC）
- [hotspot-callers.md](example/ruoyi-vue-pro/uml/technical/hotspot-callers.md) — 高频被调用方法 Top 20

#### 流程图（`flowcharts/`）

- [flowchart.md](example/ruoyi-vue-pro/flowcharts/flowchart.md) — 整体业务架构（Admin + App + RPC）
- [sequence-spu-form.md](example/ruoyi-vue-pro/flowcharts/sequence-spu-form.md) — SPU 复合表单保存
- [sequence-spu-status.md](example/ruoyi-vue-pro/flowcharts/sequence-spu-status.md) — SPU 状态流转
- [sequence-category-flow.md](example/ruoyi-vue-pro/flowcharts/sequence-category-flow.md) — 分类树 CRUD + 跨模块校验
- [sequence-comment-flow.md](example/ruoyi-vue-pro/flowcharts/sequence-comment-flow.md) — 评价创建与回复
- [sequence-favorite-history.md](example/ruoyi-vue-pro/flowcharts/sequence-favorite-history.md) — 用户收藏与浏览历史
- [sequence-rpc-flow.md](example/ruoyi-vue-pro/flowcharts/sequence-rpc-flow.md) — 跨模块 RPC 调用拓扑

---

## skills 使用教程

`skills/codebase-scan/SKILL.md` 定义了一个由用户直接调用的 `iidp-codebase-scan` skill，用于对存量代码库执行**全量**逆向工程：Codegraph CLI 采集 → Codebook 证据落盘 → TypeCard 生成 → 文档/状态机/UML/SRS 落盘 → 质量门禁与 refine loop。

完整规则请阅读 [SKILL.md](skills/codebase-scan/SKILL.md)；下面是**最常用的 4 种调用范式**。

### 1. 最快体验：单模组仓库

```bash
/iidp-codebase-scan --source /path/to/your-repo
```

默认行为：
- 输出目录：`./output/<项目名>/`
- 自动识别多模组 / 后端公共根包 / 前端工程
- 质量门禁阈值：`0.8`

### 2. 显式指定一个后端包入口

```bash
/iidp-codebase-scan \
  --source /path/to/ruoyi-vue-pro \
  --package cn.iocoder.yudao.module.product
```

`--package` 可重复多次；每个值生成一个 `backend_package` 入口。

### 3. 显式指定一个前端视图入口

```bash
/iidp-codebase-scan \
  --source /path/to/yudao-ui-admin-vue3 \
  --frontend-view src/views/mall/product
```

业务 key 默认从路径最后一段（`product`）推导；可用 `--business-key` 覆盖。

### 4. 启用受控关联文件扩展

```bash
/iidp-codebase-scan \
  --source /path/to/ruoyi-vue-pro \
  --package cn.iocoder.yudao.module.product \
  --backend-business product \
  --expand-related
```

`--expand-related` 会基于基础入口文件做一层受控扩展，把 `service.product / dal.dataobject.product / api.dto.product` 等命中业务 key 的相邻包纳入 inventory（深度固定为 1，不递归）。

### 常用参数速查

| 参数 | 说明 |
|---|---|
| `--source` | 必填，源码目录绝对路径 |
| `--output` | 输出目录，默认 `./output/<项目名>` |
| `--module` | 显式顶级模组入口（可多次） |
| `--package` | 显式后端包入口（可多次） |
| `--backend-business` | 后端业务切片，按 package 段精确匹配（可多次） |
| `--frontend-view` | 前端视图入口（可多次） |
| `--business-key` | 显式业务 key |
| `--expand-related` | 启用一层受控关联文件扩展 |
| `--init-only` | 仅初始化或校验 `.codebook/`，不进入采集与文档生成 |
| `--refresh-entries` | 只重新生成 `entries.json` |
| `--doc-quality-threshold` | 文档质量门禁阈值（默认 `0.8`） |

### 默认配置与产物路径

执行一次完整扫描后，会得到如下结构（以 `--source /repo --output /repo/out` 为例）：

```text
/out/
├── .codebook/
│   ├── manifest.json
│   ├── entries.json
│   ├── codegraph-tools.json
│   ├── semantic-kinds.json
│   ├── typecard-policy.json
│   ├── prompts/                      ← temp_typecard / temp_architecture / temp_business_flows / temp_data_model / temp_srs / temp_ui
│   ├── evidence/<entry-id>/          ← inventory.json / nodes.json / typecards.json
│   ├── checkpoints/
│   ├── proposals/                    ← 3 轮 refine 仍不达标时写入 iidp-codebase-scan-<date>.md
│   └── skill-evolution.log           ← 已应用的 skill 进化提案
├── entries/<entry-id>/               ← DOCUMENTATION / architecture / business-flows / data-model / state-machines / error-handling / database / ui-layout
├── uml/
│   ├── business/                     ← state-<主题> / class-<主题>
│   └── technical/                    ← component-diagram / hotspot-callers
├── flowcharts/                       ← flowchart / sequence-<流程>
├── srs.md                            ← 技术无关 SRS
└── quality.json                      ← generation_mode / codegraph_cli_version / entry_coverage / typecard_planning / refine_count
```

### 质量门禁关键字段

`quality.json` 中以下字段决定本次扫描是否算"正式完成"：

- `generation_mode == "llm_prompted"`：必须由大模型生成 TypeCard / 文档 / SRS；脚本仅做采集与组装，不得用规则脚本伪造正式产物。
- `entry_coverage[*].symbol_enumeration_complete == true`：逐文件、逐 kind 枚举与 `nodeCount` 完整对账。
- `entry_coverage[*].document_generation_complete == true`：每个入口的全部 `required_documents` 已通过完整性校验并原子写入。
- `doc_completeness_score`、`tech_agnostic_score`、`completeness_score`、`uml_coverage_score`：四项分数均需 ≥ `--doc-quality-threshold`。

3 轮 refine 后仍不达标时，skill 会把改进建议写入 `.codebook/proposals/iidp-codebase-scan-<date>.md`，下次运行会自动应用状态为 `approved` 的提案（并写入 `skill-evolution.log`）。