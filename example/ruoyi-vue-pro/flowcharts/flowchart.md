# 主业务流图：商城商品中心（后端）

入口：backend-package-yudao-module-product
证据：./entries/backend-package-yudao-module-product/business-flows.md

---

## 整体业务架构

```mermaid
graph TB
  subgraph "Admin 端 (管理后台)"
    BrandCtrl[ProductBrandController<br/>7 endpoints]
    CategoryCtrl[ProductCategoryController<br/>5 endpoints]
    SpuCtrl[ProductSpuController<br/>9 endpoints]
    PropertyCtrl[ProductPropertyController<br/>6 endpoints]
    PropertyValueCtrl[ProductPropertyValueController<br/>6 endpoints]
    CommentCtrl[ProductCommentController<br/>4 endpoints]
  end

  subgraph "App 端 (用户端)"
    AppCategoryCtrl[AppCategoryController<br/>3 endpoints]
    AppSpuCtrl[AppProductSpuController<br/>4 endpoints]
    AppCommentCtrl[AppProductCommentController<br/>2 endpoints]
    AppFavoriteCtrl[AppFavoriteController<br/>7 endpoints]
    AppHistoryCtrl[AppProductBrowseHistoryController<br/>4 endpoints]
  end

  subgraph "Service 层"
    BrandSvc[ProductBrandService]
    CategorySvc[ProductCategoryService]
    SpuSvc[ProductSpuService]
    SkuSvc[ProductSkuService]
    PropertySvc[ProductPropertyService]
    PropertyValueSvc[ProductPropertyValueService]
    CommentSvc[ProductCommentService]
    FavoriteSvc[ProductFavoriteService]
    HistorySvc[ProductBrowseHistoryService]
  end

  subgraph "Repository 层"
    BrandMapper[ProductBrandMapper]
    CategoryMapper[ProductCategoryMapper]
    SpuMapper[ProductSpuMapper]
    SkuMapper[ProductSkuMapper]
    PropertyMapper[ProductPropertyMapper]
    PropertyValueMapper[ProductPropertyValueMapper]
    CommentMapper[ProductCommentMapper]
    FavoriteMapper[ProductFavoriteMapper]
    HistoryMapper[ProductBrowseHistoryMapper]
  end

  subgraph "RPC 暴露"
    CategoryApi[ProductCategoryApi<br/>5 methods]
    SpuApi[ProductSpuApi<br/>4 methods]
    SkuApi[ProductSkuApi<br/>4 methods]
    CommentApi[ProductCommentApi<br/>1 method]
  end

  BrandCtrl --> BrandSvc --> BrandMapper
  CategoryCtrl --> CategorySvc --> CategoryMapper
  SpuCtrl --> SpuSvc --> SpuMapper
  SpuCtrl --> SkuSvc --> SkuMapper
  PropertyCtrl --> PropertySvc --> PropertyMapper
  PropertyValueCtrl --> PropertyValueSvc --> PropertyValueMapper
  CommentCtrl --> CommentSvc --> CommentMapper

  AppCategoryCtrl --> CategorySvc
  AppSpuCtrl --> SpuSvc
  AppCommentCtrl --> CommentSvc
  AppFavoriteCtrl --> FavoriteSvc --> FavoriteMapper
  AppHistoryCtrl --> HistorySvc --> HistoryMapper

  CategoryApi --> CategorySvc
  SpuApi --> SpuSvc
  SkuApi --> SkuSvc
  CommentApi --> CommentSvc

  SpuSvc -.->|@Lazy 循环依赖| CategorySvc
  CategorySvc -.->|@Lazy 循环依赖| SpuSvc
  SpuSvc --> BrandSvc
  SpuSvc --> SkuSvc
```

## 跨模块 RPC 消费关系

```mermaid
graph LR
  subgraph "本模块 (yudao-module-product)"
    CategoryApi[ProductCategoryApi]
    SpuApi[ProductSpuApi]
    SkuApi[ProductSkuApi]
    CommentApi[ProductCommentApi]
  end

  subgraph "yudao-module-promotion"
    CouponSvc[CouponTemplateServiceImpl<br/>validateProductScope]
    RewardSvc[RewardActivityServiceImpl<br/>validateProductScope]
  end

  subgraph "yudao-module-trade"
    TradeOrderSvc[TradeOrderUpdateServiceImpl]
    OrderCreate[订单创建/取消]
  end

  subgraph "yudao-module-bpm"
    BpmFlow[流程编排]
  end

  CouponSvc -.->|调用| CategoryApi
  RewardSvc -.->|调用| CategoryApi
  TradeOrderSvc -.->|调用| CommentApi
  TradeOrderSvc -.->|调用| SkuApi
  OrderCreate -.->|调用| SkuApi
  BpmFlow -.->|调用| SpuApi
```

## 数据写入流（SPU 复合保存）

```mermaid
graph TD
  Start([createSpu 入口]) --> V1[validateCategory<br/>分类存在+启用+层级≥2]
  V1 --> V2[validateProductBrand<br/>品牌存在+启用]
  V2 --> V3[validateSkuList<br/>属性项一致+组合不重复]
  V3 --> Init[initSpuFromSkus<br/>价格汇总+默认状态]
  Init --> InsSPU[productSpuMapper.insert]
  InsSPU --> InsSKU[productSkuService.createSkuList]
  InsSKU --> End([返回 SPU ID])
  V1 -.->|失败| Ex1[抛 CATEGORY_NOT_EXISTS /<br/>CATEGORY_DISABLED /<br/>SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR]
  V2 -.->|失败| Ex2[抛 BRAND_NOT_EXISTS /<br/>BRAND_DISABLED]
  V3 -.->|失败| Ex3[抛 SPU_ATTR_NUMBERS_MUST_BE_EQUALS /<br/>SKU_PROPERTIES_DUPLICATED]
```

## SPU 状态机流转

```mermaid
stateDiagram-v2
  [*] --> ENABLE: createSpu<br/>默认上架
  ENABLE --> DISABLE: updateStatus(0)
  DISABLE --> ENABLE: updateStatus(1)
  ENABLE --> RECYCLE: updateStatus(-1)
  DISABLE --> RECYCLE: updateStatus(-1)
  RECYCLE --> [*]: deleteSpu<br/>级联删除 SKU
  ENABLE --> SPU_NOT_ENABLE: validateSpuList<br/>校验失败
```

## 评价处理流

```mermaid
graph TD
  Start([createComment 入口]) --> Idem[校验订单项未评价<br/>COMMENT_ORDER_EXISTS]
  Idem --> UserInfo[MemberUserApi.getMemberUser<br/>获取昵称/头像]
  UserInfo --> SkuCheck[validateSku<br/>skuId 属于 spuId]
  SkuCheck --> Insert[productCommentMapper.insert]
  Insert --> End([返回])

  Start2([replyComment]) --> CheckExist[校验评价存在]
  CheckExist --> SetReply[设置 replyStatus=1<br/>replyUserId<br/>replyContent<br/>replyTime]
  SetReply --> Update1[updateById]

  Start3([updateCommentVisible]) --> CheckExist2[校验评价存在]
  CheckExist2 --> SetVis[设置 visible]
  SetVis --> Update2[updateById]
```

## source_nodes 追溯

- 8 个 admin controller + 5 个 app controller 节点
- 9 个 service + 9 个 service_contract 节点
- 9 个 repository 节点
- 4 个 rpc_api + 4 个 rpc_contract 节点
- 54 个 endpoint 节点 + 20 个 rpc_method 节点
