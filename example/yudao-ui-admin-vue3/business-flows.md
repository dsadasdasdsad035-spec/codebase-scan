# 业务流程汇总：商城商品中心前端

入口：frontend-mall-product（单入口）
证据：`./entries/frontend-mall-product/business-flows.md`

---

## 核心流程（7 个）

| # | 流程 | 入口文件 | source_nodes |
|---|---|---|---|
| 1 | 品牌 CRUD | brand/index.vue + BrandForm.vue | component:fbb..., component:2d7..., function:a7b6..., function:9d41..., function:983b..., function:1b47..., function:fca2... |
| 2 | 商品分类树 CRUD | category/index.vue + CategoryForm.vue | component:f696..., component:4150..., function:9d41...（openForm/handleViewSpu） |
| 3 | 属性项与属性值两级管理 | property/index.vue + value/index.vue | component:30b1..., component:43cc..., component:3c07..., component:1ba7... |
| 4 | 评价管理 | comment/index.vue + CommentForm.vue + ReplyForm.vue | component:8b46..., component:c605..., component:e382..., function:983b...（handleVisibleChange） |
| 5 | SPU 5-Tab 状态流转 | spu/index.vue | component:766a..., function:983b...（handleStatusChange/02Change/handleDelete/handleExport） |
| 6 | SPU 5-Tab 复合表单提交 | spu/form/index.vue + 5 form | component:be48..., component:68c9..., component:18a9..., component:1339..., component:7a6f..., component:70c3..., component:61fc..., component:58b8... |
| 7 | 跨子域 SPU/SKU 组件复用 | spu/components/* + comment/CommentForm.vue | component:e66c..., component:8aa6..., component:7109..., component:90f1... |

详见 [entries/frontend-mall-product/business-flows.md](entries/frontend-mall-product/business-flows.md)
