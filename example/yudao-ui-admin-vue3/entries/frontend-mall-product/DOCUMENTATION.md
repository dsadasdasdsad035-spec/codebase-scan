# 商城商品中心前端文档 (frontend-mall-product)

入口 ID：frontend-mall-product
类型：frontend_root
源码范围：yudao-ui-admin-vue3/src/views/mall/product/**
证据：evidence/frontend-mall-product/{inventory,nodes,typecards}.json
配套后端：ruoyi-vue-pro / yudao-module-mall/yudao-module-product

---

## 1. 入口总览

本入口是"商城"模块下"商品中心"的后台管理前端，覆盖 5 大子域：

| 子域 | 一级路径 | 主要 Page |
|---|---|---|
| 商品品牌 | `brand/` | 品牌列表、品牌表单 |
| 商品分类 | `category/` | 分类列表、分类表单、分类选择器 |
| 商品属性 | `property/` 与 `property/value/` | 属性项列表、属性项表单、属性值列表、属性值表单 |
| 商品评价 | `comment/` | 评价列表、添加评论表单、回复表单 |
| SPU/SKU | `spu/` 与 `spu/form/` 与 `spu/components/` | SPU 列表、SPU 复合表单、5 子表单、4 公共组件、components 入口 |

文件统计：26 个 .vue/.ts 文件，490 个原生节点，141 个可分析符号（25 page + 116 function/interface/type_alias）。

## 2. 模块依赖

详见 [architecture.md](architecture.md)。核心依赖：
- 后端 API：`@/api/mall/product/{brand,category,property,comment,spu}` 共 9 类
- Element Plus 组件库
- 项目内工具：`@/utils/{dict,formatTime,tree,constants,download,propTypes,is}`
- 公共组件：`ContentWrap` / `Pagination` / `Dialog` / `UploadImg` / `UploadImgs` / `Icon` / `ImageViewer`
- 路由：`ProductBrand` / `ProductCategory` / `ProductProperty` / `ProductPropertyValue` / `ProductComment` / `ProductSpu` / `ProductSpuAdd` / `ProductSpuEdit` / `ProductSpuDetail`

## 3. 业务能力

详见 [business-flows.md](business-flows.md)。核心 7 个流程：
1. 品牌 CRUD
2. 商品分类树 CRUD（含跨入口跳转查看商品）
3. 商品属性与属性值两级管理（路由 params 传递）
4. 商品评价管理（多条件搜索 + 回复 + 显隐）
5. SPU 5-Tab 状态流转（5 Tab + 上架/下架/回收/恢复/删除/导出）
6. SPU 5-Tab 复合表单提交（5 子表单校验 + 价格分↔元换算 + 轮播图 URL 化）
7. 跨子域 SPU/SKU 组件复用（SpuShowcase / SpuTableSelect / SkuTableSelect / ProductCategorySelect）

## 4. 数据模型

详见 [data-model.md](data-model.md)。9 类前端实体：BrandVO / CategoryVO / PropertyVO / PropertyValueVO / CommentVO / Spu / Sku / PropertyAndValues / RuleConfig。

## 5. 状态机

详见 [state-machines.md](state-machines.md)。6 类状态机：
1. SPU 销售状态（ENABLE/DISABLE/RECYCLE）
2. 表单弹窗显隐
3. 评论可见性
4. SPU 规格类型（单/多）
5. SPU 分销类型（默认/单独）
6. 加载状态（loading/formLoading）

## 6. 错误处理

详见 [error-handling.md](error-handling.md)。9 类处理：
1. 表单校验（Element Plus 内联 + 复合表单 Tab 自动跳转）
2. 二次确认取消（删除/状态变更/导出）
3. 加载状态保护（try-finally + Switch 异常回滚）
4. SKU 价格校验（ruleConfig 4 字段）
5. 路由参数处理（propertyId / categoryId / id）
6. 分↔元换算与精度
7. 轮播图 URL 化
8. API 错误（axios 拦截器统一处理）
9. 越界与回滚（删除仅回收站可见、详情模式只读）

## 7. UI 布局

详见 [ui-layout.md](ui-layout.md)。25 个 page + 5 个 spu 子表单 + 4 个 spu 公共组件的完整布局 + 交互链路。

## 8. 数据库视图

详见 [database.md](database.md)。9 类实体的 ER 图、表约束、写入/读取路径、缓存策略。

## 9. 权限点

| 资源 | create | update | delete | export | query |
|---|---|---|---|---|---|
| brand | product:brand:create | product:brand:update | product:brand:delete | - | - |
| category | product:category:create | product:category:update | product:category:delete | - | product:spu:query（查看商品） |
| property | product:property:create | product:property:update | product:property:delete | - | - |
| comment | product:comment:create | product:comment:update | - | - | - |
| spu | product:spu:create | product:spu:update | product:spu:delete | product:spu:export | - |

## 10. source_nodes 完整索引

| Node ID | 类型 | 文件 | 名称 |
|---|---|---|---|
| component:fbb82fd3451fdc7dcf941a9a95fdcb8b | page | brand/index.vue | 品牌列表 |
| component:2d7a9b88d8c6b9d362467ec2d6beba0e | page | brand/BrandForm.vue | 品牌表单 |
| component:f69675a209b3865375b7e42f60c85466 | page | category/index.vue | 分类列表 |
| component:415071f3d41649815096ab65c9a139f3 | page | category/CategoryForm.vue | 分类表单 |
| component:90f10902441d322000edf9da69f88370 | page | category/components/ProductCategorySelect.vue | 分类选择器 |
| component:30b1c90977ec826a83ee2f1777895b5d | page | property/index.vue | 属性项列表 |
| component:43ccda97fa2906aab36871cb7fca6606 | page | property/PropertyForm.vue | 属性项表单 |
| component:3c079e65e1a015eb488ef8ed4aebf6b6 | page | property/value/index.vue | 属性值列表 |
| component:1ba7f951ac58dcb25c3f4a691ecd155e | page | property/value/ValueForm.vue | 属性值表单 |
| component:8b462134c11b251f030d72d983c2b803 | page | comment/index.vue | 评价列表 |
| component:c60546efcc9ed0b1d4527bb2ee73663d | page | comment/CommentForm.vue | 添加虚拟评论 |
| component:e3823f838eb365c67f0bd5c29489bd91 | page | comment/ReplyForm.vue | 回复评论 |
| component:766a92ffa67135de01a5219da9b57bf5 | page | spu/index.vue | SPU 列表 |
| component:be487fd29883255ef6a9971a99c1c589 | page | spu/form/index.vue | SPU 复合表单聚合 |
| component:68c9a279711c588e6c9f52ac19abc2be | page | spu/form/InfoForm.vue | SPU 基础设置 |
| component:18a9cdf6cda3537530eea5e2dc6b6492 | page | spu/form/SkuForm.vue | SPU 价格库存 |
| component:13391245e64d2417c7bc0b6da4718a78 | page | spu/form/DeliveryForm.vue | SPU 物流设置 |
| component:7a6ff01634b3ee0a30a8a774844408c9 | page | spu/form/DescriptionForm.vue | SPU 商品详情 |
| component:70c3384cad029eedab3d527723aaebb4 | page | spu/form/OtherForm.vue | SPU 其它设置 |
| component:61fc70313ceef9ac0c4570dba9d9ac51 | page | spu/form/ProductAttributes.vue | SPU 多规格属性 |
| component:58b810df1c6c35195b854d4b8e616178 | page | spu/form/ProductPropertyAddForm.vue | SPU 添加属性表单 |
| component:7e42462b163ba6dd1fd60611ebe27fc6 | page | spu/components/SkuList.vue | SKU 列表 |
| component:71096ed3227a58af4005a43b8f2b0711 | page | spu/components/SkuTableSelect.vue | SKU 表格选择器 |
| component:e66cfb70aadcb8c836cba8d713911f31 | page | spu/components/SpuShowcase.vue | SPU 展示选择器 |
| component:8aa636172dabd1f67e1b4ded472f0715 | page | spu/components/SpuTableSelect.vue | SPU 表格选择器 |
| (其他 116 个 function/interface/type_alias 节点) | function | 全部 26 文件 | 见 nodes.json |
