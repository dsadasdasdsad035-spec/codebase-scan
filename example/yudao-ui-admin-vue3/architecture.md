# 商城商品中心前端 - 系统级架构

入口：frontend-mall-product（单入口聚合）
证据：`./entries/frontend-mall-product/{architecture,ui-layout,business-flows,data-model,state-machines,error-handling,database,DOCUMENTATION}.md`

---

## 1. 系统职责

商城商品中心后台管理前端，覆盖品牌、分类、属性/属性值、评价、SPU/SKU 五大子域。

## 2. 顶级入口分布

| 入口 ID | 类型 | 路径 | 文件数 | 节点数 | TypeCard 数 | 枚举完成 | 文档完成 |
|---|---|---|---|---|---|---|---|
| frontend-mall-product | frontend_root | src/views/mall/product | 26 | 490 | 141 | ✓ | ✓ |

## 3. 依赖总览

详见 [entries/frontend-mall-product/architecture.md § 2](entries/frontend-mall-product/architecture.md)

## 4. 集成状态

- 已接入主流程：25 page + 5 form + 4 公共组件 + 1 入口
- 跨入口调用：1 处（分类列表"查看商品" → SPU 列表）
- 跨子域组件复用：4 个（SpuShowcase / SpuTableSelect / SkuTableSelect / ProductCategorySelect）

## 5. 业务能力地图

详见 [entries/frontend-mall-product/business-flows.md](entries/frontend-mall-product/business-flows.md)
- 5 个一级业务能力
- 7 个核心流程

## 6. 数据模型

详见 [entries/frontend-mall-product/data-model.md](entries/frontend-mall-product/data-model.md)
- 9 个前端实体
- 1 个 ER 图

## 7. 状态机

详见 [entries/frontend-mall-product/state-machines.md](entries/frontend-mall-product/state-machines.md)
- 6 类状态机

## 8. 错误处理

详见 [entries/frontend-mall-product/error-handling.md](entries/frontend-mall-product/error-handling.md)
- 9 类错误处理

## 9. 入口与系统文档双向追溯

- 入口 1：frontend-mall-product（`./entries/frontend-mall-product/`）
- 系统级（本目录）：本文件 + business-flows.md + data-model.md + state-machines.md + error-handling.md + database.md + DOCUMENTATION.md + srs.md
- UML：`./uml/business/`、`./uml/technical/`
- Flowcharts：`./flowcharts/`
