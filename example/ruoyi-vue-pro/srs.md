# 软件需求规格说明书（SRS）：商城商品中心

技术无关文档，基于业务能力生成 FR/NFR。
入口：backend-package-yudao-module-product（单入口）
证据：./entries/backend-package-yudao-module-product/* + ./architecture.md + ./business-flows.md + ./data-model.md

---

## 1. 范围

本系统为商城"商品中心"提供后端业务能力，覆盖品牌、分类、属性与属性值、评价、商品（含规格）、用户收藏与浏览历史等子域，支持商品从发布到上架/下架/回收的全生命周期管理，并对外提供跨模块的查询与校验服务。

## 2. 业务能力清单

| # | 业务能力 | 入口追溯 |
|---|---|---|
| 1 | 品牌管理 | backend-package-yudao-module-product |
| 2 | 商品分类管理 | backend-package-yudao-module-product |
| 3 | 商品属性与属性值管理 | backend-package-yudao-module-product |
| 4 | 商品评价管理 | backend-package-yudao-module-product |
| 5 | 商品生命周期管理 | backend-package-yudao-module-product |
| 6 | 商品发布与编辑（复合业务） | backend-package-yudao-module-product |
| 7 | 用户收藏与浏览历史 | backend-package-yudao-module-product |
| 8 | 跨模块服务（校验、查询、状态变更） | backend-package-yudao-module-product |

## 3. 功能需求（Functional Requirements）

### FR-1 品牌管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-1.1 | 系统应支持按品牌名称、状态、创建时间查询品牌列表 | ProductBrandController.getBrandPage |
| FR-1.2 | 系统应支持新增品牌，必填项为品牌名称、状态 | ProductBrandController.createBrand |
| FR-1.3 | 系统应支持编辑品牌（按 ID 修改名称、状态、排序、描述） | ProductBrandController.updateBrand |
| FR-1.4 | 系统应支持删除品牌（无关联商品前提下） | ProductBrandController.deleteBrand |
| FR-1.5 | 系统应校验品牌名称全局唯一 | ProductBrandServiceImpl.validateBrandNameUnique |
| FR-1.6 | 系统应支持获取全量启用品牌列表（用于下拉选项） | ProductBrandController.getSimpleBrandList |
| FR-1.7 | 系统应在新增/编辑 SPU 时校验所引用的品牌存在且启用 | ProductSpuServiceImpl → ProductBrandService.validateProductBrand |

### FR-2 商品分类管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-2.1 | 系统应支持按分类名称查询分类列表（树形） | ProductCategoryController.getCategoryList |
| FR-2.2 | 系统应支持新增/编辑/删除分类 | ProductCategoryController.createCategory / updateCategory / deleteCategory |
| FR-2.3 | 分类应支持自引用树形结构（顶级分类 parentId=0） | ProductCategoryDO.parentId |
| FR-2.4 | 系统应校验父分类存在且为顶级（parentId=0） | ProductCategoryServiceImpl.validateParentProductCategory |
| FR-2.5 | 系统应拒绝删除存在子分类的分类 | ProductCategoryServiceImpl.deleteCategory → CATEGORY_EXISTS_CHILDREN |
| FR-2.6 | 系统应拒绝删除绑定商品的分类 | ProductCategoryServiceImpl.deleteCategory → CATEGORY_HAVE_BIND_SPU |
| FR-2.7 | 系统应提供分类层级的递归计算能力（最多 127 层防死循环） | ProductCategoryServiceImpl.getCategoryLevel |
| FR-2.8 | 系统应对外提供分类批量校验能力 | ProductCategoryApiImpl.validateCategoryList |

### FR-3 商品属性与属性值管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-3.1 | 系统应支持属性项的增删改查 | ProductPropertyController |
| FR-3.2 | 系统应支持属性值的增删改查 | ProductPropertyValueController |
| FR-3.3 | 系统应校验属性项名称全局唯一 | ProductPropertyServiceImpl.validatePropertyNameUnique |
| FR-3.4 | 系统应校验属性值名称在同属性项下唯一 | ProductPropertyValueServiceImpl.validatePropertyValueNameUnique |
| FR-3.5 | 系统应拒绝删除有属性值的属性项 | ProductPropertyServiceImpl.deleteProperty → PROPERTY_DELETE_FAIL_VALUE_EXISTS |
| FR-3.6 | 系统应提供属性项的简单列表查询（用于下拉选项） | ProductPropertyController.getPropertySimpleList |
| FR-3.7 | 系统应支持按属性项 ID 查询其下的属性值 | ProductPropertyValueController |

### FR-4 商品评价管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-4.1 | 系统应支持按回复状态、SPU 名称、用户昵称、订单编号、评价时间多条件查询评价 | ProductCommentController.getCommentPage |
| FR-4.2 | 系统应支持新增评价（用户端下单完成后自动调用） | ProductCommentServiceImpl.createComment |
| FR-4.3 | 系统应校验同一订单项只能评价一次 | createComment → COMMENT_ORDER_EXISTS |
| FR-4.4 | 系统应支持商家回复评价 | ProductCommentController.commentReply |
| FR-4.5 | 系统应支持评价的显示/隐藏切换 | ProductCommentController.updateCommentVisible |
| FR-4.6 | 系统应记录评价的回复状态、回复人、回复时间、回复内容 | ProductCommentDO.replyStatus/ReplyUserId/... |
| FR-4.7 | 系统应对外提供评价创建能力（供订单完成时调用） | ProductCommentApi.createComment |

### FR-5 商品生命周期管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-5.1 | 系统应支持按名称、分类、状态等条件查询商品 SPU 列表（分页） | ProductSpuController.getSpuPage |
| FR-5.2 | 商品 SPU 状态应支持三个值：上架、下架、回收站 | ProductSpuStatusEnum |
| FR-5.3 | 系统应支持商品的状态切换（上架↔下架、移入回收站、恢复） | ProductSpuController.updateStatus |
| FR-5.4 | 系统应拒绝从非回收站状态删除商品 | ProductSpuServiceImpl.deleteSpu → SPU_NOT_RECYCLE |
| FR-5.5 | 系统应支持从回收站物理删除商品并级联删除其 SKU | ProductSpuServiceImpl.deleteSpu → productSkuService.deleteSkuBySpuId |
| FR-5.6 | 系统应提供 SPU 列表的 5 个统计指标：销售中、仓库中、售空、警戒库存、回收站 | ProductSpuServiceImpl.getTabsCount |
| FR-5.7 | 系统应支持商品列表的 Excel 导出 | ProductSpuController.exportSpuList |
| FR-5.8 | 系统应支持 SPU 详情查询（含 SKU） | ProductSpuController.getSpuDetail |
| FR-5.9 | 系统应支持按状态获取 SPU 精简列表（用于下拉选项） | ProductSpuController.getSpuSimpleList |
| FR-5.10 | 系统应支持按 ID 列表批量获取 SPU 详情 | ProductSpuController.getSpuList |
| FR-5.11 | 系统应支持 SPU 浏览量的增量更新 | ProductSpuServiceImpl.updateBrowseCount |
| FR-5.12 | 系统应支持 SPU 库存的批量增量更新 | ProductSpuServiceImpl.updateSpuStock |
| FR-5.13 | 系统应对外提供 SPU 校验能力（存在 + 启用） | ProductSpuApiImpl.validateSpuList |

### FR-6 商品发布与编辑

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-6.1 | 系统应支持商品 SPU 的创建（事务原子性：分类校验 + 品牌校验 + SKU 校验 + SPU 插入 + SKU 插入） | ProductSpuServiceImpl.createSpu |
| FR-6.2 | 系统应支持商品 SPU 的更新（保留原状态字段） | ProductSpuServiceImpl.updateSpu |
| FR-6.3 | 系统应校验 SPU 关联的分类层级≥2 | ProductSpuServiceImpl.validateCategory → SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR |
| FR-6.4 | 系统应校验 SKU 列表的属性项数量一致 | ProductSkuServiceImpl.validateSkuList → SPU_ATTR_NUMBERS_MUST_BE_EQUALS |
| FR-6.5 | 系统应校验 SKU 列表的属性组合无重复 | validateSkuList → SKU_PROPERTIES_DUPLICATED |
| FR-6.6 | 系统应支持 SKU 单价、市场价、成本价的最低值自动写入 SPU | ProductSpuServiceImpl.initSpuFromSkus |
| FR-6.7 | 系统应支持 SPU 库存为所有 SKU 库存的总和 | ProductSpuServiceImpl.initSpuFromSkus |
| FR-6.8 | 创建 SPU 时应自动设置默认状态为上架 | initSpuFromSkus |
| FR-6.9 | 系统应支持商品关键字、简介、详情等富文本字段 | ProductSpuDO.keyword/introduction/description |
| FR-6.10 | 系统应支持商品封面图与轮播图（JSON 数组） | ProductSpuDO.picUrl/sliderPicUrls |
| FR-6.11 | 系统应支持单规格与多规格两种 SKU 模式 | ProductSpuDO.specType |

### FR-7 用户收藏与浏览历史

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-7.1 | 系统应支持用户对 SPU 的收藏 | AppFavoriteController.createFavorite |
| FR-7.2 | 系统应校验同一用户对同一 SPU 只能收藏一次 | ProductFavoriteServiceImpl.createFavorite → FAVORITE_EXISTS |
| FR-7.3 | 系统应支持批量收藏与批量取消 | AppFavoriteController.createFavoriteBatch / deleteFavoriteBatch |
| FR-7.4 | 系统应支持用户分页查询自己的收藏列表 | AppFavoriteController.getFavoritePage |
| FR-7.5 | 系统应支持用户浏览 SPU 的足迹记录 | AppProductBrowseHistoryController.createBrowseHistory |
| FR-7.6 | 每次浏览时应同时增加 SPU 的浏览量 | ProductSpuServiceImpl.updateBrowseCount |
| FR-7.7 | 系统应支持用户分页查询与删除自己的浏览历史 | AppProductBrowseHistoryController.getBrowseHistoryPage / deleteBrowseHistory |

### FR-8 跨模块服务

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-8.1 | 系统应对外提供 SPU 批量查询（含 MAP 形式） | ProductSpuApi.getSpuList / getSpuMap |
| FR-8.2 | 系统应对外提供 SPU 校验（存在 + 启用） | ProductSpuApi.validateSpuList |
| FR-8.3 | 系统应对外提供 SPU 单条查询 | ProductSpuApi.getSpu |
| FR-8.4 | 系统应对外提供分类批量校验 | ProductCategoryApi.validateCategoryList |
| FR-8.5 | 系统应对外提供分类单条校验 | ProductCategoryApi.validateCategory |
| FR-8.6 | 系统应对外提供分类查询 | ProductCategoryApi.getCategory |
| FR-8.7 | 系统应对外提供启用分类列表查询 | ProductCategoryApi.getEnableCategoryList |
| FR-8.8 | 系统应对外提供 SKU 查询（含批量、MAP） | ProductSkuApi.getSku / getSkuList / getSkuMap |
| FR-8.9 | 系统应对外提供 SKU 库存更新能力 | ProductSkuApi.updateSkuStock |
| FR-8.10 | 系统应对外提供评价创建能力 | ProductCommentApi.createComment |

## 4. 非功能需求（Non-Functional Requirements）

### NFR-1 权限控制

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-1.1 | 系统应实施基于资源-操作的细粒度权限控制 | @PreAuthorize("@ss.hasPermission('product:*')") |
| NFR-1.2 | 资源类型应包括：brand / category / property / property-value / comment / spu / sku / favorite / browse-history | 权限点前缀 product: |
| NFR-1.3 | 操作类型应至少包括：query / create / update / delete / export | 同上 |

### NFR-2 数据完整性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-2.1 | 系统应保证所有写操作（增删改）在事务边界内执行 | @Transactional(rollbackFor = Exception.class) |
| NFR-2.2 | 系统应通过数据库唯一索引兜底唯一性约束 | uk_name、uk_user_spu、uk_order_item |
| NFR-2.3 | 系统应使用逻辑删除避免物理删除 | BaseDO.deleted + @TableLogic |
| NFR-2.4 | 价格字段应统一使用"分"为单位，避免浮点精度问题 | ProductSpuDO.price/marketPrice/costPrice |
| NFR-2.5 | 库存累加应使用防溢出算术 | ProductSpuServiceImpl → Math::addExact |
| NFR-2.6 | 分类层级递归应设置最大深度防止死循环 | ProductCategoryServiceImpl.getCategoryLevel (Byte.MAX_VALUE) |

### NFR-3 一致性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-3.1 | SPU 状态变更与 SKU 维护应在同一事务内 | createSpu/updateSpu/deleteSpu @Transactional |
| NFR-3.2 | 分类删除前应先校验无子分类、无关联商品 | deleteCategory 多步校验 |
| NFR-3.3 | 评价创建应保证幂等性（同一订单项不能重复评价） | COMMENT_ORDER_EXISTS |
| NFR-3.4 | 收藏创建应保证幂等性（同一用户同一 SPU 不能重复收藏） | FAVORITE_EXISTS |
| NFR-3.5 | 跨模块 RPC 应复用 service 层错误码 | ApiImpl → ServiceException |

### NFR-4 性能

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-4.1 | 系统应支持分页查询（默认 10-20 条/页，最大 100 条/页） | MyBatis-Plus Page |
| NFR-4.2 | 系统应避免 N+1 查询（批量 IN 查询 + Map 转换） | CollectionUtils.convertMap / convertList |
| NFR-4.3 | 系统应支持高频读场景的缓存（字典、用户信息） | 框架级 Redis |
| NFR-4.4 | 库存更新应使用增量更新而非全量替换 | productSpuMapper.updateStock |
| NFR-4.5 | 库存并发场景应使用乐观锁或分布式锁 | 框架级（建议） |

### NFR-5 可观测性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-5.1 | 系统应记录所有 HTTP API 访问日志 | @ApiAccessLog |
| NFR-5.2 | 系统应使用 SLF4J 记录关键业务日志 | service 方法入口、Mapper SQL、异常堆栈 |
| NFR-5.3 | 系统应暴露 Micrometer 指标 | 框架级（HTTP、JVM、DB） |
| NFR-5.4 | 系统应支持 SkyWalking/Zipkin 链路追踪 | 框架级 |
| NFR-5.5 | 业务异常应通过统一异常处理返回结构化错误信息 | GlobalExceptionHandler + ErrorCode |

### NFR-6 安全

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-6.1 | 系统应使用参数化 SQL 防止注入 | MyBatis-Plus LambdaQueryWrapper |
| NFR-6.2 | 系统应记录所有操作人 | BaseDO.createBy / updateBy |
| NFR-6.3 | 用户敏感字段应脱敏 | 框架级 @Sensitive |
| NFR-6.4 | 跨模块 RPC 入口应进行参数校验 | ApiImpl 调用 service 前校验 |

### NFR-7 扩展性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-7.1 | 系统应支持多租户隔离 | 框架级 tenant_id |
| NFR-7.2 | 系统应支持读写分离 | 框架级 MyBatis-Plus 路由 |
| NFR-7.3 | 系统应支持分布式事务（Seata） | 框架级（按需引入） |
| NFR-7.4 | 系统应支持消息队列（RabbitMQ） | 框架级（按需引入） |

## 5. 数据视图（业务对象）

| 对象 | 关键属性 | 生命周期 |
|---|---|---|
| 品牌 | 名称（唯一）、状态、排序、描述 | 创建→启用/禁用→删除 |
| 分类 | 名称、父分类、状态、排序 | 创建→启用/禁用→删除（无子无商品才可删） |
| 属性项 | 名称（唯一）、备注 | 创建→删除（无属性值才可删） |
| 属性值 | 名称、所属属性项、备注 | 创建→删除 |
| 商品 SPU | 名称、分类、品牌、价格、库存、状态、销量、浏览量 | 创建→上架/下架→回收站→删除 |
| 商品 SKU | SPU、规格属性、价格、库存、佣金 | 跟随 SPU 创建/更新/删除 |
| 评价 | 用户、订单、SPU、SKU、评分、内容、可见性、回复 | 创建→可见/隐藏→回复 |
| 收藏 | 用户、SPU | 创建→取消 |
| 浏览历史 | 用户、SPU、是否已删 | 创建→用户主动删除 |

## 6. 状态机视图

| 实体 | 状态字段 | 状态值 | 转移规则 |
|---|---|---|---|
| SPU | status | -1/0/1 | RECYCLE -1 回收站 / DISABLE 0 下架 / ENABLE 1 上架 |
| 分类 | (隐式) | 0/1/2 | 顶级 / 一级 / 二级；SPU 必须挂二级 |
| 评价 | visible | true/false | 默认 false 待审核，管理员可切换 |
| 评价 | replyStatus | 0/1 | 0=未回复、1=已回复 |
| 通用 | status | 0/1 | 0=禁用、1=启用 |
| SKU | specType | true/false | 单规格 / 多规格 |

## 7. 错误码视图

业务错误码集中在 1-008-000-000 段：

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

## 8. 跨模块集成视图

| 上游 | 下游 | 集成点 |
|---|---|---|
| 营销中心 | 本系统 | 营销活动范围校验分类、SPU |
| 交易中心 | 本系统 | 订单完成创建评价、订单创建/取消更新 SKU 库存 |
| 流程中心 | 本系统 | 流程节点引用 SPU |
| 装修中心 | 本系统 | 装修组件引用 SPU 列表 |
| 会员中心 | 本系统 | 获取用户昵称/头像（评价、收藏、历史） |
| 字典中心 | 本系统 | 字典项查询 |

## 9. source_nodes 追溯

本 SRS 由以下证据生成：
- `entries/backend-package-yudao-module-product/architecture.md`
- `entries/backend-package-yudao-module-product/business-flows.md`
- `entries/backend-package-yudao-module-product/data-model.md`
- `entries/backend-package-yudao-module-product/state-machines.md`
- `entries/backend-package-yudao-module-product/error-handling.md`
- `entries/backend-package-yudao-module-product/database.md`
- `entries/backend-package-yudao-module-product/DOCUMENTATION.md`
- `entries/backend-package-yudao-module-product/technical-architecture.md`
- 488 个可分析 TypeCard（evidence/backend-package-yudao-module-product/typecards.json）
- 2132 个原生节点（evidence/backend-package-yudao-module-product/nodes.json）

本 SRS 中所有需求均可追溯到上述证据中的具体节点 ID。
