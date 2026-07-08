# 数据模型：商城商品中心后端 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
证据：evidence/backend-package-yudao-module-product/{nodes,typecards}.json
覆盖：9 个 DO 实体 + 字段映射 + 业务约束

---

## 1. 实体总览

| 实体 | 表名 | 主键策略 | 主键序列 | 特殊字段类型 |
|---|---|---|---|---|
| ProductBrandDO | product_brand | 自增 | product_brand_seq | 基础字段 |
| ProductCategoryDO | product_category | 自增 | product_category_seq | 树形 parentId |
| ProductSpuDO | product_spu | 自增 | product_spu_seq | Jackson JSON、IntegerList |
| ProductSkuDO | product_sku | 自增 | product_sku_seq | Jackson JSON（properties） |
| ProductPropertyDO | product_property | 自增 | product_property_seq | 基础字段 |
| ProductPropertyValueDO | product_property_value | 自增 | product_property_value_seq | 基础字段 |
| ProductCommentDO | product_comment | 自增 | product_comment_seq | Jackson JSON（picUrls） |
| ProductFavoriteDO | product_favorite | 自增 | product_favorite_seq | 基础字段 |
| ProductBrowseHistoryDO | product_browse_history | 自增 | product_browse_history_seq | 基础字段 |

所有 DO 继承 `cn.iocoder.yudao.framework.mybatis.core.dataobject.BaseDO`，包含公共字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 主键（由各自 DO 重新声明 + `@TableId`） |
| createTime | DateTime | 创建时间 |
| updateTime | DateTime | 更新时间 |
| createBy | Long | 创建人 ID |
| updateBy | Long | 更新人 ID |
| deleted | Boolean | 逻辑删除标志（MyBatis-Plus 软删除） |

## 2. 实体详细说明

### 2.1 ProductSpuDO（商品 SPU）

**表名**：`product_spu`（`@TableName(value = "product_spu", autoResultMap = true)`）

**用途**：商品聚合根，承载商品主信息、价格汇总、销量统计等。一对多关联 ProductSkuDO。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | SPU 编号（自增） |
| name | String | 商品名称 |
| keyword | String | 关键字（搜索用） |
| introduction | String | 商品简介 |
| description | String | 商品详情（富文本） |
| categoryId | Long | 商品分类编号（关联 ProductCategoryDO.id） |
| brandId | Long | 商品品牌编号（关联 ProductBrandDO.id） |
| picUrl | String | 商品封面图 |
| sliderPicUrls | List\<String\> | 商品轮播图（JacksonTypeHandler JSON 反序列化） |
| sort | Integer | 排序字段（值越大越靠前） |
| status | Integer | 商品状态（枚举 ProductSpuStatusEnum） |
| specType | Boolean | 规格类型：false=单规格、true=多规格 |
| price | Integer | 商品价格（单位：分；取所有 SKU 单价最小值） |
| marketPrice | Integer | 市场价（单位：分；取所有 SKU 市场价最小值） |
| costPrice | Integer | 成本价（单位：分；取所有 SKU 成本价最小值） |
| stock | Integer | 库存（所有 SKU 库存总和） |
| deliveryTypes | List\<Integer\> | 配送方式数组（IntegerListTypeHandler） |
| deliveryTemplateId | Long | 物流配置模板编号（关联 TradeDeliveryExpressTemplateDO） |
| giveIntegral | Integer | 赠送积分 |
| subCommissionType | Boolean | 分销类型：false=默认、true=自行设置 |
| salesCount | Integer | 商品销量 |
| virtualSalesCount | Integer | 虚拟销量 |
| browseCount | Integer | 浏览量 |

**业务规则**：
- `price`/`marketPrice`/`costPrice`/`stock` 字段在 `ProductSpuServiceImpl.initSpuFromSkus` 中基于 SKU 自动计算
- `status` 默认值 `ENABLE(1)`，创建时由 `initSpuFromSkus` 写入
- `salesCount`/`browseCount` 在 `updateBrowseCount` 和 `updateSpuStock` 时增量更新
- 状态机：RECYCLE(-1) → DISABLE(0) → ENABLE(1)，删除仅允许 RECYCLE 状态

**索引建议**（推断）：`categoryId`、`brandId`、`status`、`sort`、`deleted`

---

### 2.2 ProductSkuDO（商品 SKU）

**表名**：`product_sku`

**用途**：商品规格单元，承载具体规格组合、价格、库存、重量体积等。一对一关联 ProductSpuDO。

**字段清单**（关键字段）：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | SKU 编号（自增） |
| spuId | Long | 关联 SPU 编号 |
| properties | List\<ProductPropertyValueDetailRespDTO\> | 规格属性组合（Jackson JSON） |
| price | Integer | 销售价（分） |
| marketPrice | Integer | 市场价（分） |
| costPrice | Integer | 成本价（分） |
| stock | Integer | 库存 |
| weight | Double | 重量（克） |
| volume | Double | 体积（立方厘米） |
| firstBrokeragePrice | Integer | 一级分销佣金（分） |
| secondBrokeragePrice | Integer | 二级分销佣金（分） |
| picUrl | String | SKU 图片 |

**业务规则**：
- `properties` 中每个元素包含 `propertyId`/`propertyName`/`valueId`/`valueName` 四个字段
- SKU 的属性组合在同一 SPU 下必须唯一
- SKU 的属性项数量在同一 SPU 下必须一致
- 删除 SPU 时级联删除所有 SKU

---

### 2.3 ProductCategoryDO（商品分类）

**表名**：`product_category`

**用途**：商品分类树，采用邻接表模型（parentId）实现两级分类（一级 + 二级）。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 分类编号 |
| parentId | Long | 父分类编号（0=顶级） |
| name | String | 分类名称 |
| picUrl | String | 分类图片 |
| sort | Integer | 排序字段 |
| status | Integer | 状态（0=禁用、1=启用） |

**常量**：
- `PARENT_ID_NULL = 0L`：顶级分类的 parentId 值
- `CATEGORY_LEVEL = 2`：商品必须挂载到二级分类及以下

**业务规则**：
- 删除分类前必须无子分类
- 删除分类前必须无关联 SPU
- SPU 关联的分类必须为二级及以下层级
- 父分类本身不能是二级分类

---

### 2.4 ProductBrandDO（商品品牌）

**表名**：`product_brand`

**用途**：商品品牌信息。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 品牌编号 |
| name | String | 品牌名称（全局唯一） |
| picUrl | String | 品牌图片 |
| sort | Integer | 排序字段 |
| description | String | 品牌描述 |
| status | Integer | 状态（0=禁用、1=启用） |

**业务规则**：
- 品牌名称在新增/编辑时全局唯一性校验
- 禁用品牌不可被 SPU 引用

---

### 2.5 ProductPropertyDO / ProductPropertyValueDO（商品属性项与值）

**表名**：`product_property` / `product_property_value`

**用途**：商品规格属性字典。属性项与属性值是一对多关系。

**ProductPropertyDO 字段**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 属性项编号 |
| name | String | 属性项名称（唯一） |
| remark | String | 备注 |

**常量**：`ID_DEFAULT = 0L`（在 ProductPropertyDO 中定义）

**ProductPropertyValueDO 字段**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 属性值编号 |
| propertyId | Long | 关联属性项编号 |
| name | String | 属性值名称（在同属性项下唯一） |
| remark | String | 备注 |

**业务规则**：
- 属性项名称全局唯一
- 属性值名称在同属性项下唯一
- 删除属性项前必须无属性值

---

### 2.6 ProductCommentDO（商品评价）

**表名**：`product_comment`

**用途**：商品评价主记录。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 评价编号 |
| userId | Long | 评价用户 ID |
| userNickname | String | 用户昵称（冗余存储） |
| userAvatar | String | 用户头像（冗余存储） |
| anonymous | Boolean | 是否匿名 |
| orderId | Long | 订单编号 |
| orderItemId | Long | 订单项编号 |
| spuId | Long | SPU 编号 |
| spuName | String | SPU 名称（冗余） |
| skuId | Long | SKU 编号 |
| skuPicUrl | String | SKU 图片（冗余） |
| skuProperties | String | SKU 规格属性（冗余） |
| visible | Boolean | 是否可见 |
| scores | Integer | 总评分（1-5） |
| descriptionScores | Integer | 描述分 |
| benefitScores | Integer | 效果分 |
| content | String | 评价内容 |
| picUrls | List\<String\> | 评价图片（Jackson JSON） |
| replyStatus | Integer | 回复状态（0=未回复、1=已回复） |
| replyUserId | Long | 回复人 ID |
| replyContent | String | 回复内容 |
| replyTime | DateTime | 回复时间 |

**业务规则**：
- 同一订单项只能评价一次（`COMMENT_ORDER_EXISTS`）
- 评价可见性由 `visible` 字段控制
- 商家回复时记录 `replyUserId`/`replyContent`/`replyTime`

---

### 2.7 ProductFavoriteDO（商品收藏）

**表名**：`product_favorite`

**用途**：用户对商品的收藏记录。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 收藏编号 |
| userId | Long | 用户 ID |
| spuId | Long | SPU 编号 |

**业务规则**：
- 同一用户对同一 SPU 只能收藏一次（`FAVORITE_EXISTS`）
- 删除 SPU 时不级联删除收藏（由用户主动取消）

---

### 2.8 ProductBrowseHistoryDO（浏览历史）

**表名**：`product_browse_history`

**用途**：用户浏览 SPU 的足迹记录。

**字段清单**：

| 字段 | 类型 | 说明 |
|---|---|---|
| id | Long | 浏览历史编号 |
| spuId | Long | SPU 编号 |
| userId | Long | 用户 ID |
| userDeleted | Boolean | 用户是否已删除该条历史 |

**业务规则**：
- 每次浏览时 `productSpuService.updateBrowseCount(spuId, 1)` 增加 SPU 浏览量
- 软删除字段 `userDeleted` 区别于 `BaseDO.deleted`

---

## 3. 跨实体关系图

```mermaid
erDiagram
    ProductCategoryDO ||--o{ ProductSpuDO : "categoryId"
    ProductBrandDO ||--o{ ProductSpuDO : "brandId"
    ProductSpuDO ||--o{ ProductSkuDO : "spuId"
    ProductPropertyDO ||--o{ ProductPropertyValueDO : "propertyId"
    ProductSkuDO }o--o{ ProductPropertyValueDO : "properties JSON"
    ProductSpuDO ||--o{ ProductCommentDO : "spuId"
    ProductSkuDO ||--o{ ProductCommentDO : "skuId"
    ProductSpuDO ||--o{ ProductFavoriteDO : "spuId"
    ProductSpuDO ||--o{ ProductBrowseHistoryDO : "spuId"

    ProductCategoryDO {
        Long id PK
        Long parentId
        String name
        Integer sort
        Integer status
    }
    ProductBrandDO {
        Long id PK
        String name UK
        String picUrl
        Integer sort
        Integer description
        Integer status
    }
    ProductSpuDO {
        Long id PK
        Long categoryId FK
        Long brandId FK
        String name
        String keyword
        Integer price
        Integer marketPrice
        Integer costPrice
        Integer stock
        Integer status
        Integer salesCount
        Integer browseCount
    }
    ProductSkuDO {
        Long id PK
        Long spuId FK
        String properties JSON
        Integer price
        Integer stock
        Integer firstBrokeragePrice
        Integer secondBrokeragePrice
    }
    ProductPropertyDO {
        Long id PK
        String name UK
        String remark
    }
    ProductPropertyValueDO {
        Long id PK
        Long propertyId FK
        String name
    }
    ProductCommentDO {
        Long id PK
        Long userId
        Long spuId FK
        Long skuId FK
        Long orderId
        Integer scores
        String content
        Boolean visible
    }
    ProductFavoriteDO {
        Long id PK
        Long userId
        Long spuId FK
    }
    ProductBrowseHistoryDO {
        Long id PK
        Long userId
        Long spuId FK
        Boolean userDeleted
    }
```

## 4. 字段类型映射

| 业务类型 | 物理类型 | 说明 |
|---|---|---|
| 主键 | `Long` | 自增 + `@KeySequence` |
| 字符串 | `String` | VARCHAR |
| 价格/金额 | `Integer` | 单位：分 |
| 库存/计数 | `Integer` | 累加使用 `Math::addExact` 防溢出 |
| 状态 | `Integer` | 0=禁用/未上架、1=启用/上架、-1=回收站 |
| 排序 | `Integer` | 越大越靠前 |
| 时间 | `LocalDateTime` | 由 `BaseDO.createTime`/`updateTime` 维护 |
| 列表 | `List<String>` | Jackson JSON 存储 |
| 列表 | `List<Integer>` | IntegerListTypeHandler 存储 |
| 逻辑删除 | `Boolean` | MyBatis-Plus `@TableLogic` |

## 5. 数据完整性约束

| 约束类型 | 实现机制 |
|---|---|
| 主键唯一 | `@TableId` + 数据库自增 |
| 名称唯一 | Service 层 `validateXxxNameUnique` + 数据库唯一索引 |
| 状态合法性 | 枚举 `ProductSpuStatusEnum`、`CommonStatusEnum` |
| 层级约束 | 递归 `getCategoryLevel` |
| 数量约束 | 分类删除前 `selectCountByParentId` / `getSpuCountByCategoryId` |
| 业务唯一 | 收藏 `FAVORITE_EXISTS`、评价 `COMMENT_ORDER_EXISTS` |
| 跨实体引用 | 校验方法 `validateSpuList` / `validateCategoryList` / `validateSku` |
| 价格单位 | 统一"分"（Integer），避免浮点精度问题 |
| 软删除 | `BaseDO.deleted`（MyBatis-Plus `@TableLogic`） |

## 6. source_nodes 追溯

- 9 个 DO 实体节点 ID（class 节点）
- `ProductCategoryDO.PARENT_ID_NULL`、`CATEGORY_LEVEL`、`ProductPropertyDO.ID_DEFAULT` 常量
- `ProductSpuStatusEnum`、`ProductCommentAuditStatusEnum`、`ProductCommentScoresEnum` 枚举
- 共 9 个 entity 类型 + 3 个 enum 类型 + 1 个 framework 配置类
