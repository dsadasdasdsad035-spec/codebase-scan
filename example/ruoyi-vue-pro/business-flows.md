# 业务流程：商城商品中心后端 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
证据：evidence/backend-package-yudao-module-product/{nodes,typecards}.json
覆盖：7 大子域核心业务流程 + 跨模块 RPC 编排

---

## 流程 1：商品品牌 CRUD 流程

**触发条件**：管理员在管理后台进入"商品品牌"模块

**输入**：搜索条件（name/status/createTime）；新增/编辑表单字段

**输出**：分页品牌列表；新增/编辑/删除结果

**主路径**：

1. `ProductBrandController.getBrandPage` → `ProductBrandService.getBrandPage` → `ProductBrandMapper.selectPage`
2. 新增 `createBrand(createReqVO)` → `ProductBrandServiceImpl.createBrand`：
   - `validateBrandNameUnique(createReqVO.getName())` 校验品牌名称唯一性（`BRAND_NAME_EXISTS`）
   - `ProductBrandMapper.insert(brand)`
3. 编辑 `updateBrand(updateReqVO)` → `ProductBrandServiceImpl.updateBrand`：
   - `validateBrandExists(updateReqVO.getId())` 校验存在性
   - `validateBrandNameUnique` 校验名称唯一
   - `ProductBrandMapper.updateById(brand)`
4. 删除 `deleteBrand(id)` → `ProductBrandServiceImpl.deleteBrand`：
   - `validateBrandExists(id)` 校验存在性
   - `ProductBrandMapper.deleteById(id)`
5. 查询 `getBrand(id)` → `validateBrandExists` → `selectById`
6. 简单列表 `getSimpleBrandList` → `selectList` 全表查询，按 status 过滤

**失败路径**：
- 品牌不存在：`BRAND_NOT_EXISTS`（1-008-002-000）
- 品牌已禁用：`BRAND_DISABLED`（1-008-002-001）
- 品牌名称重复：`BRAND_NAME_EXISTS`（1-008-002-002）

**热点调用**：`getBrandPage`（分页/搜索）、`getBrand`（详情回显）

**权限点**：`product:brand:query`、`product:brand:create`、`product:brand:update`、`product:brand:delete`

**source_nodes**：`class:4aa361545fe282f8ec2b0ad3364b73b4`、`method:createBrand`、`method:updateBrand`、`method:deleteBrand`、`method:getBrand`

---

## 流程 2：商品分类树 CRUD 流程

**触发条件**：管理员在管理后台进入"商品分类"模块

**输入**：分类名称；上级分类（必选，0=顶级）；图片；排序；状态

**输出**：树形分类列表（`handleTree` 构造）；新增/编辑/删除结果

**主路径**：

1. `ProductCategoryController.getCategoryList` → `ProductCategoryService.getCategoryList(listReqVO)` → `ProductCategoryMapper.selectList(listReqVO)`
2. 新增 `createCategory(createReqVO)` → `ProductCategoryServiceImpl.createCategory`：
   - `validateParentProductCategory(createReqVO.getParentId())` 校验父分类
     - 若 `parentId = PARENT_ID_NULL(0L)`：跳过（顶级分类）
     - 若父分类不存在：`CATEGORY_PARENT_NOT_EXISTS`（1-008-001-001）
     - 若父分类本身是二级：`CATEGORY_PARENT_NOT_FIRST_LEVEL`（1-008-001-002）
   - `ProductCategoryMapper.insert(category)`
3. 编辑 `updateCategory` → `validateProductCategoryExists` + `validateParentProductCategory` + `updateById`
4. 删除 `deleteCategory(id)` → `ProductCategoryServiceImpl.deleteCategory`：
   - `validateProductCategoryExists(id)` 校验存在性
   - `productCategoryMapper.selectCountByParentId(id) > 0` → `CATEGORY_EXISTS_CHILDREN`（1-008-001-003）
   - `productSpuService.getSpuCountByCategoryId(id) > 0` → `CATEGORY_HAVE_BIND_SPU`（1-008-001-005）
   - `deleteById(id)`
5. 校验方法 `validateCategoryList(ids)`（RPC 暴露）：
   - 校验每个分类存在、状态为 ENABLE
   - 校验层级 `getCategoryLevel(id) >= CATEGORY_LEVEL(2)` → `SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR`
6. 层级计算 `getCategoryLevel(id)`：递归向上找父节点，累加层级，最多 127 层防止死循环
7. 跨模块暴露：`ProductCategoryApi.validateCategoryList`（被 yudao-module-promotion 的 `validateProductScope` 等调用）

**失败路径**：
- 分类不存在：`CATEGORY_NOT_EXISTS`（1-008-001-000）
- 父分类不存在：`CATEGORY_PARENT_NOT_EXISTS`（1-008-001-001）
- 父分类不能是二级：`CATEGORY_PARENT_NOT_FIRST_LEVEL`（1-008-001-002）
- 存在子分类：`CATEGORY_EXISTS_CHILDREN`（1-008-001-003）
- 分类已禁用：`CATEGORY_DISABLED`（1-008-001-004）
- 分类下存在商品：`CATEGORY_HAVE_BIND_SPU`（1-008-001-005）
- 分类层级不够：`SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR`（1-008-005-001）

**热点调用**：`getCategoryList`（树形构建）、`validateCategoryList`（跨模块校验）

**权限点**：`product:category:query`、`product:category:create`、`product:category:update`、`product:category:delete`

**source_nodes**：`class:ProductCategoryController`、`method:createCategory`、`method:deleteCategory`、`method:validateCategoryList`、`method:getCategoryLevel`

---

## 流程 3：商品 SPU 5-Tab 状态流转流程

**触发条件**：管理员在管理后台进入"商品 SPU"模块，5 个 Tab 分别为：销售中、仓库中、售空、警戒库存、回收站

**输入**：搜索条件；状态变更请求；删除请求

**输出**：分页 SPU 列表；状态变更/删除结果

**主路径**：

1. Tab 计数 `ProductSpuController.getSpuCount` → `ProductSpuServiceImpl.getTabsCount`：
   - `FOR_SALE`：统计 `status = ENABLE(1)` 的数量
   - `IN_WAREHOUSE`：统计 `status = DISABLE(0)` 的数量
   - `SOLD_OUT`：统计 `stock = 0` 的数量
   - `ALERT_STOCK`：统计全部数量（占位）
   - `RECYCLE_BIN`：统计 `status = RECYCLE(-1)` 的数量
2. 分页 `getSpuPage(pageVO)` → `ProductSpuMapper.selectPage(pageVO)`
3. 状态变更 `updateStatus` → `ProductSpuServiceImpl.updateSpuStatus`：
   - `validateSpuExists(updateReqVO.getId())` 校验存在
   - `updateById` 更新 status 字段
4. 删除 `deleteSpu(id)` → `ProductSpuServiceImpl.deleteSpu`：
   - `validateSpuExists(id)` 校验存在
   - 状态校验：必须为 RECYCLE 状态，否则 `SPU_NOT_RECYCLE`（1-008-005-004）
   - `productSpuMapper.deleteById(id)`
   - `productSkuService.deleteSkuBySpuId(id)` 级联删除 SKU
5. 跨模块校验 `validateSpuList(ids)`（RPC 暴露）：
   - 查询所有 SPU
   - 校验每个 SPU 存在 → `SPU_NOT_EXISTS`（1-008-005-000）
   - 校验每个 SPU 状态为 ENABLE → `SPU_NOT_ENABLE`（1-008-005-003）

**失败路径**：
- SPU 不存在：`SPU_NOT_EXISTS`
- SPU 未上架：`SPU_NOT_ENABLE`
- SPU 未处于回收站状态：`SPU_NOT_RECYCLE`

**状态机**：RECYCLE(-1) → DISABLE(0) → ENABLE(1)，删除仅允许从 RECYCLE 出发

**热点调用**：`getSpuPage`（分页/搜索）、`getTabsCount`（Tab 计数）

**权限点**：`product:spu:query`、`product:spu:update`、`product:spu:delete`、`product:spu:export`

**source_nodes**：`class:ProductSpuController`、`method:deleteSpu`、`method:updateSpuStatus`、`method:getTabsCount`、`enum:ProductSpuStatusEnum`

---

## 流程 4：SPU 复合表单保存流程

**触发条件**：管理员创建或编辑 SPU

**输入**：商品基本信息（名称、关键字、简介、详情、分类、品牌、图片、轮播图）+ SKU 列表

**输出**：SPU ID（创建）/成功（更新）

**主路径**：

1. `ProductSpuController.createProductSpu(createReqVO)` / `updateSpu(updateReqVO)` → 触发 service 方法
2. `ProductSpuServiceImpl.createSpu`（事务：`@Transactional(rollbackFor = Exception.class)`）：
   - `validateCategory(createReqVO.getCategoryId())` 校验分类
     - `categoryService.validateCategory(id)` 校验分类存在且启用
     - `categoryService.getCategoryLevel(id) >= CATEGORY_LEVEL(2)` 校验层级 → `SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR`
   - `brandService.validateProductBrand(createReqVO.getBrandId())` 校验品牌
   - `productSkuService.validateSkuList(skuSaveReqList, createReqVO.getSpecType())` 校验 SKU
     - 校验属性组合无重复 → `SKU_PROPERTIES_DUPLICATED`
     - 校验每个 SKU 属性项一致 → `SPU_ATTR_NUMBERS_MUST_BE_EQUALS`
   - `BeanUtils.toBean` 将 VO 转换为 DO
   - `initSpuFromSkus(spu, skuSaveReqList)` 基于 SKU 初始化 SPU 字段：
     - `price = min(SKU prices)` 单价最低
     - `marketPrice = min(SKU marketPrices)` 市场价最低
     - `costPrice = min(SKU costPrices)` 成本价最低
     - `stock = sum(SKU stocks)` 库存总数
     - 默认 `status = ENABLE(1)` 上架
     - 默认 `salesCount = 0` 销量
     - 默认 `browseCount = 0` 浏览量
   - `productSpuMapper.insert(spu)` 插入 SPU
   - `productSkuService.createSkuList(spu.getId(), skuSaveReqList)` 插入 SKU 列表
3. `updateSpu` 流程类似，区别在于 `validateSpuExists` 校验存在性，并保留原 status 字段

**失败路径**：
- 分类不正确：`SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR`
- 分类/品牌不存在或禁用：`CATEGORY_NOT_EXISTS`、`CATEGORY_DISABLED`、`BRAND_NOT_EXISTS`、`BRAND_DISABLED`
- SKU 属性组合重复：`SKU_PROPERTIES_DUPLICATED`
- SKU 属性项不一致：`SPU_ATTR_NUMBERS_MUST_BE_EQUALS`
- SKU 库存不足：`SKU_STOCK_NOT_ENOUGH`（订单场景触发）

**事务边界**：`createSpu`/`updateSpu`/`deleteSpu` 均在 `@Transactional` 内，任何子操作失败整体回滚

**source_nodes**：`method:createSpu`、`method:updateSpu`、`method:initSpuFromSkus`、`method:validateCategory`、`method:validateSkuList`

---

## 流程 5：商品评价创建与回复流程

**触发条件**：用户在订单完成后创建评价；管理员回复评价；管理员切换评价可见性

**输入**：评价内容（描述分、效果分、内容、图片）；回复内容；可见性状态

**输出**：评价 ID；分页评价列表；回复结果

**主路径**：

1. **创建评价** `ProductCommentServiceImpl.createComment`：
   - 通过 RPC `memberUserApi.getMemberUser(userId)` 获取用户昵称/头像
   - 校验同一订单是否已有评价 → `COMMENT_ORDER_EXISTS`（1-008-007-001）
   - `productSkuService.validateSku(skuId, spuId)` 校验 SKU 属于该 SPU
   - `productCommentMapper.insert(comment)` 插入评价
2. **管理端创建** `ProductCommentController.createComment` 与 App 端共用同一 service 方法
3. **分页查询** `getCommentPage` → `ProductCommentMapper.selectPage`
4. **商家回复** `replyComment(replyReqVO)` → `ProductCommentServiceImpl.replyComment`：
   - 校验评价存在 → `COMMENT_NOT_EXISTS`（1-008-007-000）
   - 设置 `replyUserId`、`replyContent`、`replyTime`
   - `updateById`
5. **可见性切换** `updateCommentVisible(visibleReqVO)` → `ProductCommentServiceImpl.updateCommentVisible`：
   - 校验评价存在
   - 更新 `visible` 字段
6. **跨模块调用**：App 端下单完成后 `ProductCommentApi.createComment`（被 TradeOrderUpdateServiceImpl 调用）

**失败路径**：
- 评价不存在：`COMMENT_NOT_EXISTS`
- 订单已评价：`COMMENT_ORDER_EXISTS`

**状态机**：评价可见性 `visible`（true=显示、false=隐藏）+ 回复状态 `replyStatus`（0=未回复、1=已回复）

**source_nodes**：`class:ProductCommentController`、`class:AppProductCommentController`、`method:createComment`、`method:replyComment`、`method:updateCommentVisible`

---

## 流程 6：商品属性项与属性值两级管理流程

**触发条件**：管理员在管理后台维护商品属性字典

**输入**：属性项名称、备注；属性值名称、属性项 ID

**输出**：分页属性项/属性值列表；CRUD 结果

**主路径**：

1. **属性项 CRUD** `ProductPropertyController`：
   - `getPropertyPage` → `selectPage`
   - `createProperty(createReqVO)` → `validatePropertyNameUnique(name)` → `insert`
   - `updateProperty` → `validatePropertyExists(id)` + `validatePropertyNameUnique` → `updateById`
   - `deleteProperty(id)` → `validatePropertyExists` + 校验无属性值 → `deleteById`
2. **属性值 CRUD** `ProductPropertyValueController`：
   - `getPropertyValuePage` → `selectPage`
   - `createPropertyValue` → `validatePropertyValueNameUnique` → `insert`
   - `updatePropertyValue` / `deletePropertyValue` 类似
3. **简单列表** `getPropertySimpleList` → 按 status 过滤查询 ENABLE 状态的属性项

**失败路径**：
- 属性项不存在：`PROPERTY_NOT_EXISTS`（1-008-003-000）
- 属性项名称已存在：`PROPERTY_EXISTS`（1-008-003-001）
- 属性项下存在属性值：`PROPERTY_DELETE_FAIL_VALUE_EXISTS`（1-008-003-002）
- 属性值不存在：`PROPERTY_VALUE_NOT_EXISTS`（1-008-004-000）
- 属性值名称已存在：`PROPERTY_VALUE_EXISTS`（1-008-004-001）

**source_nodes**：`class:ProductPropertyController`、`class:ProductPropertyValueController`、`method:validatePropertyNameUnique`、`method:validatePropertyValueNameUnique`

---

## 流程 7：用户收藏与浏览历史流程

**触发条件**：App 端用户对商品进行收藏或浏览

**输入**：用户 ID、SPU ID

**输出**：收藏结果、浏览历史列表

**主路径**：

1. **收藏** `AppFavoriteController.createFavorite`：
   - `ProductFavoriteServiceImpl.createFavorite(userId, spuId)`：
     - 校验 `productSpuService.validateSpuExists(spuId)` → `SPU_NOT_EXISTS`
     - 校验是否已收藏（`selectCountByUserIdAndSpuId`）→ `FAVORITE_EXISTS`（1-008-008-000）
     - `insert(favorite)`
2. **批量收藏** `createFavoriteBatch`：
   - 循环单条收藏逻辑
3. **取消收藏** `deleteFavorite` / `deleteFavoriteBatch` → `deleteByUserIdAndSpuId`
4. **分页查询** `getFavoritePage` → `ProductFavoriteMapper.selectPage`
5. **浏览历史** `AppProductBrowseHistoryController.createBrowseHistory`：
   - `ProductBrowseHistoryServiceImpl.createBrowseHistory`：
     - `productSpuService.validateSpuExists(spuId)`
     - `productSpuService.updateBrowseCount(spuId, 1)` 增加 SPU 浏览量
     - `insertBrowseHistory(userId, spuId)`
6. **分页查询历史** `getBrowseHistoryPage` → `selectPage`
7. **删除历史** `deleteBrowseHistory` / `deleteBrowseHistoryList`

**失败路径**：
- 商品不存在：`SPU_NOT_EXISTS`
- 已收藏：`FAVORITE_EXISTS`
- 收藏不存在：`FAVORITE_NOT_EXISTS`（1-008-008-001）

**source_nodes**：`class:AppFavoriteController`、`class:AppProductBrowseHistoryController`、`method:createFavorite`、`method:createBrowseHistory`

---

## 流程 8：App 端 SPU 列表查询与详情流程

**触发条件**：App 端用户浏览商品列表或查看商品详情

**输入**：分类 ID（可空）；分页参数

**输出**：分页 SPU 列表（精简/详情）；SPU 详情

**主路径**：

1. **SPU 列表（App 端）** `AppProductSpuController.getSpuPage`：
   - `ProductSpuServiceImpl.getSpuPage(AppProductSpuPageReqVO)`：
     - 若 `categoryId` 不为空：递归查找所有子分类 ID（`categoryService.getCategoryList({parentId, status=ENABLE})`）
     - 若 `categoryIds` 不为空：同样展开子分类
     - `productSpuMapper.selectPage(pageVO, categoryIds)` 分页查询
2. **SPU 详情（App 端）** `AppProductSpuController.getSpuDetail`：
   - `ProductSpuServiceImpl.getSpu(id, includeDeleted=false)` 默认不查已删除
   - `productSkuService.getSkuListBySpuId(spuId)` 加载 SKU
   - `ProductSpuConvert.INSTANCE.convert(spu, skus)` 转换为 RespVO
3. **SPU 精简列表** `getSpuSimpleList` → `getSpuListByStatus(ENABLE)` → 按 sort 倒序
4. **跨模块调用**：交易下单时通过 `ProductSpuApi.getSpuMap(spuIds)` 批量获取 SPU 信息
5. **库存扣减**：`ProductSkuApi.updateSkuStock(spuId, incrCount)` 在订单创建/取消时被调用

**热点调用**：`getSpuPage`（分页）、`getSpuDetail`（详情）

**source_nodes**：`class:AppProductSpuController`、`method:getSpuPage`、`method:getSpuDetail`、`interface:ProductSpuApi`、`method:updateSkuStock`

---

## 流程 9：跨模块 RPC 校验与查询流程

**触发条件**：其他业务模块（yudao-module-promotion、yudao-module-trade、yudao-module-bpm 等）需要校验或查询商品中心数据

**输入**：来自其他模块的 RPC 请求

**输出**：校验结果或数据

**主路径**：

1. **分类校验** `ProductCategoryApiImpl.validateCategoryList(ids)`：
   - 被 `CouponTemplateServiceImpl.validateProductScope` 等促销模块调用
   - 被 `RewardActivityServiceImpl.validateProductScope` 等营销模块调用
   - 校验分类存在、状态为 ENABLE、层级合规
2. **SPU 校验** `ProductSpuApiImpl.validateSpuList(ids)`：
   - 被订单创建、营销活动、装修模块等业务调用
   - 校验 SPU 存在、状态为 ENABLE
3. **SKU 查询** `ProductSkuApiImpl.getSku / getSkuList / getSkuMap`：
   - 被订单模块、购物车、营销模块调用
   - 返回 SKU 详情（含价格、库存、规格属性）
4. **SKU 库存更新** `ProductSkuApiImpl.updateSkuStock`：
   - 被订单创建（扣减库存）、订单取消（恢复库存）调用
5. **评价创建** `ProductCommentApiImpl.createComment`：
   - 被 `TradeOrderUpdateServiceImpl` 在订单完成后调用
   - 写入评价记录

**失败路径**：各 RPC 方法复用 service 层的失败码，错误信息透传至调用方

**source_nodes**：`class:ProductCategoryApiImpl`、`class:ProductSpuApiImpl`、`class:ProductSkuApiImpl`、`class:ProductCommentApiImpl`
