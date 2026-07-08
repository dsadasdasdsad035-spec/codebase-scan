# 类体系：商城商品中心后端核心实体

入口：backend-package-yudao-module-product
证据：entries/backend-package-yudao-module-product/data-model.md

---

## 实体类继承关系

```mermaid
classDiagram
    class BaseDO {
        +Long id
        +LocalDateTime createTime
        +LocalDateTime updateTime
        +Long createBy
        +Long updateBy
        +Boolean deleted
    }

    class ProductBrandDO {
        +Long id
        +String name
        +String picUrl
        +Integer sort
        +String description
        +Integer status
    }

    class ProductCategoryDO {
        +Long id
        +Long parentId
        +String name
        +String picUrl
        +Integer sort
        +Integer status
        +PARENT_ID_NULL = 0L
        +CATEGORY_LEVEL = 2
    }

    class ProductSpuDO {
        +Long id
        +String name
        +String keyword
        +String introduction
        +String description
        +Long categoryId
        +Long brandId
        +String picUrl
        +List~String~ sliderPicUrls
        +Integer sort
        +Integer status
        +Boolean specType
        +Integer price
        +Integer marketPrice
        +Integer costPrice
        +Integer stock
        +List~Integer~ deliveryTypes
        +Long deliveryTemplateId
        +Integer giveIntegral
        +Boolean subCommissionType
        +Integer salesCount
        +Integer virtualSalesCount
        +Integer browseCount
    }

    class ProductSkuDO {
        +Long id
        +Long spuId
        +List~ProductPropertyValueDetailRespDTO~ properties
        +Integer price
        +Integer marketPrice
        +Integer costPrice
        +Integer stock
        +Double weight
        +Double volume
        +Integer firstBrokeragePrice
        +Integer secondBrokeragePrice
        +String picUrl
    }

    class ProductPropertyDO {
        +Long id
        +String name
        +String remark
        +ID_DEFAULT = 0L
    }

    class ProductPropertyValueDO {
        +Long id
        +Long propertyId
        +String name
        +String remark
    }

    class ProductCommentDO {
        +Long id
        +Long userId
        +String userNickname
        +String userAvatar
        +Boolean anonymous
        +Long orderId
        +Long orderItemId
        +Long spuId
        +String spuName
        +Long skuId
        +String skuPicUrl
        +String skuProperties
        +Boolean visible
        +Integer scores
        +Integer descriptionScores
        +Integer benefitScores
        +String content
        +List~String~ picUrls
        +Integer replyStatus
        +Long replyUserId
        +String replyContent
        +LocalDateTime replyTime
    }

    class ProductFavoriteDO {
        +Long id
        +Long userId
        +Long spuId
    }

    class ProductBrowseHistoryDO {
        +Long id
        +Long spuId
        +Long userId
        +Boolean userDeleted
    }

    BaseDO <|-- ProductBrandDO
    BaseDO <|-- ProductCategoryDO
    BaseDO <|-- ProductSpuDO
    BaseDO <|-- ProductSkuDO
    BaseDO <|-- ProductPropertyDO
    BaseDO <|-- ProductPropertyValueDO
    BaseDO <|-- ProductCommentDO
    BaseDO <|-- ProductFavoriteDO
    BaseDO <|-- ProductBrowseHistoryDO
```

## 实体关联关系

```mermaid
classDiagram
    class ProductCategoryDO
    class ProductBrandDO
    class ProductSpuDO
    class ProductSkuDO
    class ProductPropertyDO
    class ProductPropertyValueDO
    class ProductCommentDO
    class ProductFavoriteDO
    class ProductBrowseHistoryDO

    ProductCategoryDO "1" --> "*" ProductSpuDO : categoryId
    ProductBrandDO "1" --> "*" ProductSpuDO : brandId
    ProductSpuDO "1" --> "*" ProductSkuDO : spuId
    ProductPropertyDO "1" --> "*" ProductPropertyValueDO : propertyId
    ProductSkuDO "*" --> "*" ProductPropertyValueDO : properties (JSON)
    ProductSpuDO "1" --> "*" ProductCommentDO : spuId
    ProductSkuDO "1" --> "*" ProductCommentDO : skuId
    ProductSpuDO "1" --> "*" ProductFavoriteDO : spuId
    ProductSpuDO "1" --> "*" ProductBrowseHistoryDO : spuId
```

## 控制器 / 服务 / 仓储分层

```mermaid
classDiagram
    class ProductSpuController {
        -ProductSpuService productSpuService
        -ProductSkuService productSkuService
        +createProductSpu()
        +updateSpu()
        +updateStatus()
        +deleteSpu()
        +getSpuDetail()
        +getSpuSimpleList()
        +getSpuList()
        +getSpuPage()
        +getSpuCount()
        +exportSpuList()
    }

    class ProductSpuService {
        <<interface>>
        +createSpu(reqVO)
        +updateSpu(reqVO)
        +updateSpuStatus(reqVO)
        +deleteSpu(id)
        +getSpu(id)
        +getSpuList(ids)
        +getSpuPage(reqVO)
        +getTabsCount()
        +validateSpuList(ids)
        +updateSpuStock(stockIncrCounts)
    }

    class ProductSpuServiceImpl {
        -ProductSpuMapper productSpuMapper
        -ProductSkuService productSkuService
        -ProductBrandService brandService
        -ProductCategoryService categoryService
        +createSpu() @Transactional
        +updateSpu() @Transactional
        +initSpuFromSkus()
        +validateCategory()
    }

    class ProductSpuMapper {
        <<interface>>
        +insert(spu)
        +updateById(spu)
        +selectById(id)
        +selectByIds(ids)
        +selectPage(reqVO)
        +selectCount(...)
        +updateStock(id, incCount)
        +updateBrowseCount(id, incCount)
    }

    ProductSpuController --> ProductSpuService
    ProductSpuService <|.. ProductSpuServiceImpl
    ProductSpuServiceImpl --> ProductSpuMapper
    ProductSpuServiceImpl --> ProductSkuService
    ProductSpuServiceImpl --> ProductBrandService
    ProductSpuServiceImpl --> ProductCategoryService
```

## Convert 类（MapStruct）

```mermaid
classDiagram
    class ProductSpuConvert {
        <<interface>>
        +INSTANCE
        +convert(spu, skus) ProductSpuRespVO
        +convertForSpuDetailRespListVO(spus, skus) List~ProductSpuRespVO~
        +convertList(spus) List~ProductSpuSimpleRespVO~
    }

    class ProductBrandConvert {
        <<interface>>
        +INSTANCE
        +convert(brand) ProductBrandRespVO
        +convertList(brands) List~ProductBrandRespVO~
        +convertList1(brands) List~ProductBrandSimpleRespVO~
    }

    class ProductCategoryConvert {
        <<interface>>
        +INSTANCE
        +convert(category) ProductCategoryRespVO
        +convertList(categories) List~ProductCategoryRespVO~
    }

    class ProductCommentConvert {
        <<interface>>
        +INSTANCE
        +convert(createReqVO) ProductCommentDO
        +convert(comments) List~ProductCommentRespVO~
    }

    class ProductFavoriteConvert {
        <<interface>>
        +INSTANCE
        +convert(reqVO) ProductFavoriteDO
        +convertList(favorites) List~AppFavoriteRespVO~
    }
```

## RPC 暴露类

```mermaid
classDiagram
    class ProductSpuApi {
        <<interface>>
        +getSpuList(ids) List~ProductSpuRespDTO~
        +getSpuMap(ids) Map~Long,ProductSpuRespDTO~
        +validateSpuList(ids) List~ProductSpuRespDTO~
        +getSpu(id) ProductSpuRespDTO
    }

    class ProductCategoryApi {
        <<interface>>
        +validateCategoryList(ids)
        +validateCategory(id)
        +getCategory(id)
        +getEnableCategoryList() List~ProductCategoryRespDTO~
        +getEnableCategoryList(ids) List~ProductCategoryRespDTO~
    }

    class ProductSkuApi {
        <<interface>>
        +getSku(id) ProductSkuRespDTO
        +getSkuList(ids) List~ProductSkuRespDTO~
        +getSkuMap(ids) Map~Long,ProductSkuRespDTO~
        +updateSkuStock(spuId, incrCount) Boolean
    }

    class ProductCommentApi {
        <<interface>>
        +createComment(reqDTO) Long
    }

    class ProductSpuApiImpl
    class ProductCategoryApiImpl
    class ProductSkuApiImpl
    class ProductCommentApiImpl

    ProductSpuApi <|.. ProductSpuApiImpl
    ProductCategoryApi <|.. ProductCategoryApiImpl
    ProductSkuApi <|.. ProductSkuApiImpl
    ProductCommentApi <|.. ProductCommentApiImpl
```

## source_nodes 追溯

- 9 个 DO 实体（class）+ 9 个 service 契约（interface）+ 9 个 service 实现（class）
- 9 个 Mapper（interface）
- 4 个 RPC Api（interface）+ 4 个 RPC ApiImpl（class）
- 5 个 Convert（interface）
- 1 个 ProductWebConfiguration（class）
