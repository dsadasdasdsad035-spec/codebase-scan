# 商城商品中心后端文档 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
类型：backend_package
源码范围：ruoyi-vue-pro/yudao-module-mall/yudao-module-product/**
证据：evidence/backend-package-yudao-module-product/{inventory,nodes,typecards}.json
配套前端：yudao-ui-admin-vue3 / src/views/mall/product

---

## 1. 入口总览

本入口是"商城"模块下"商品中心"的后端 Spring Boot 模块，对应包前缀 `cn.iocoder.yudao.module.product`。覆盖 7 大子域：

| 子域 | 主要类 | 职责 |
|---|---|---|
| 商品 SPU | ProductSpuController、ProductSpuService、ProductSpuDO | 商品发布、5-Tab 状态流转、价格汇总 |
| 商品 SKU | ProductSkuService、ProductSkuDO | SKU 编辑、库存管理 |
| 商品分类 | ProductCategoryController、ProductCategoryService、ProductCategoryDO | 树形分类、层级校验 |
| 商品品牌 | ProductBrandController、ProductBrandService、ProductBrandDO | 品牌 CRUD、唯一性 |
| 商品属性 | ProductPropertyController、ProductPropertyValueController、ProductPropertyDO、ProductPropertyValueDO | 属性项 + 属性值两级管理 |
| 商品评价 | ProductCommentController、ProductCommentService、ProductCommentDO | 评价、回复、可见性 |
| 收藏与历史 | AppFavoriteController、AppProductBrowseHistoryController、ProductFavoriteDO、ProductBrowseHistoryDO | App 端用户行为 |
| 跨模块 RPC | ProductCategoryApi、ProductCommentApi、ProductSkuApi、ProductSpuApi | 对外暴露商品能力 |

文件统计：128 个 .java 源文件，2132 个原生节点，488 个可分析符号。

## 2. 架构与依赖

详见 [architecture.md](architecture.md)。核心依赖：
- Spring Boot + Spring MVC + Spring Security
- MyBatis-Plus（Mapper、TypeHandler、LambdaQueryWrapper）
- Spring 事务（`@Transactional`）
- MapStruct（`ProductSpuConvert`）
- Lombok（`@Data`、`@Builder`）
- Hutool、Guava
- yudao-framework（CommonResult、PageResult、BaseDO、ErrorCode）
- 跨模块：`ProductCategoryService` ↔ `ProductSpuService`（`@Lazy` 解循环）

## 3. 业务能力

详见 [business-flows.md](business-flows.md)。核心 9 个流程：
1. 商品品牌 CRUD
2. 商品分类树 CRUD（含层级校验与级联删除保护）
3. 商品 SPU 5-Tab 状态流转（RECYCLE/DISABLE/ENABLE）
4. SPU 复合表单保存（含 SKU 校验、价格汇总）
5. 商品评价创建、回复、可见性切换
6. 商品属性项与属性值两级管理
7. 用户收藏与浏览历史
8. App 端 SPU 列表查询与详情（含子分类展开）
9. 跨模块 RPC 校验与查询

## 4. 数据模型

详见 [data-model.md](data-model.md)。9 类持久化实体：

| 实体 | 表名 | 关键字段 |
|---|---|---|
| ProductBrandDO | product_brand | name(唯一)、status |
| ProductCategoryDO | product_category | parentId(树形)、status |
| ProductSpuDO | product_spu | categoryId、brandId、price(分)、status、sliderPicUrls(JSON) |
| ProductSkuDO | product_sku | spuId、properties(JSON)、price(分)、stock |
| ProductPropertyDO | product_property | name(唯一) |
| ProductPropertyValueDO | product_property_value | propertyId、name |
| ProductCommentDO | product_comment | spuId、orderItemId(唯一)、scores、visible |
| ProductFavoriteDO | product_favorite | userId、spuId(联合唯一) |
| ProductBrowseHistoryDO | product_browse_history | userId、spuId、userDeleted |

## 5. 状态机

详见 [state-machines.md](state-machines.md)。5 类状态机：
1. SPU 销售状态（RECYCLE -1 / DISABLE 0 / ENABLE 1）
2. 分类层级（顶级、一级、二级；SPU 必须挂二级）
3. 评价可见性（visible × replyStatus）
4. 通用启用/禁用（0/1）
5. SKU 规格类型（单规格/多规格）

## 6. 错误处理

详见 [error-handling.md](error-handling.md)。分层错误处理：
- Controller 层：@Valid、@PreAuthorize
- Service 层：业务异常 + @Transactional 回滚
- Repository 层：数据库唯一索引兜底
- 24 条业务错误码（1-008-000-000 段）

## 7. 数据库视图

详见 [database.md](database.md)。9 张核心表 + 关系 + 索引建议 + 缓存策略。

## 8. 关键 API 端点（53 个 admin + 12 个 app）

| 子域 | 端点数 | 关键端点 |
|---|---|---|
| SPU | 9 | POST /product/spu/create、PUT /product/spu/update、DELETE /product/spu/delete、GET /product/spu/page、GET /product/spu/get-detail、GET /product/spu/get-count、GET /product/spu/list-all-simple、GET /product/spu/list、GET /product/spu/export-excel |
| 分类 | 5 | POST /product/category/create、PUT /product/category/update、DELETE /product/category/delete、GET /product/category/get、GET /product/category/list |
| 品牌 | 7 | POST /product/brand/create、PUT /product/brand/update、DELETE /product/brand/delete、GET /product/brand/get、GET /product/brand/page、GET /product/brand/list、GET /product/brand/list-all-simple |
| 属性 | 12 | 各 6 个端点（属性项/属性值） |
| 评价 | 4 | GET /product/comment/page、PUT /product/comment/update-visible、PUT /product/comment/reply、POST /product/comment/create |
| 收藏/历史 | 16 | App 端各端点 |

## 9. 跨模块 RPC（4 组共 20 个方法）

| RPC | 方法数 | 消费方 |
|---|---|---|
| ProductCategoryApi | 5 | yudao-module-promotion（CouponTemplateServiceImpl.validateProductScope、RewardActivityServiceImpl.validateProductScope） |
| ProductSpuApi | 4 | 订单、营销、装修模块 |
| ProductSkuApi | 4 | 订单模块（TradeOrderUpdateServiceImpl 等） |
| ProductCommentApi | 7 | 订单模块（TradeOrderUpdateServiceImpl） |

## 10. 权限点

| 模块 | 权限点 | 说明 |
|---|---|---|
| brand | product:brand:query / create / update / delete | 商品品牌 |
| category | product:category:query / create / update / delete | 商品分类 |
| property | product:property:query / create / update / delete | 商品属性 |
| comment | product:comment:query / create / update | 商品评价 |
| spu | product:spu:query / create / update / delete / export | 商品 SPU |
| sku | product:sku:query / create / update / delete | 商品 SKU |

## 11. 异常码一览

| 段 | 子域 | 条数 |
|---|---|---|
| 1-008-001-000 | 商品分类 | 6 |
| 1-008-002-000 | 商品品牌 | 3 |
| 1-008-003-000 | 商品属性项 | 3 |
| 1-008-004-000 | 商品属性值 | 2 |
| 1-008-005-000 | 商品 SPU | 5 |
| 1-008-006-000 | 商品 SKU | 5 |
| 1-008-007-000 | 商品评价 | 2 |
| 1-008-008-000 | 商品收藏 | 2 |

## 12. source_nodes 追溯

- 8 个 controller 类（admin）+ 5 个 controller 类（app）
- 9 个 service 接口 + 9 个 service 实现类
- 9 个 Mapper 接口
- 4 个 RPC Api 接口 + 4 个 RPC Api 实现
- 5 个 Convert 接口
- 9 个 DO 实体 + 1 个 BaseDO 引用
- 3 个枚举
- 1 个 ErrorCodeConstants 接口
- 1 个 ProductWebConfiguration
- 54 个 HTTP endpoint + 20 个 RPC method
- 40 个 controller_method + 15 个 app_controller_method
- 169 个 service_method
- 40 个 repository_method
- 19 个 convert_method
- 28 个 vo_request + 18 个 vo_response + 5 个 dto
- 共 488 个可分析符号
