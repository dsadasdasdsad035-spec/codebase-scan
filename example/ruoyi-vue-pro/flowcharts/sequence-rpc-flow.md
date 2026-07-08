# 序列图：跨模块 RPC 调用

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 9

---

## 整体 RPC 拓扑

```mermaid
graph LR
  subgraph 调用方
    Promotion[yudao-module-promotion<br/>CouponTemplateServiceImpl]
    Trade[yudao-module-trade<br/>TradeOrderUpdateServiceImpl]
    Bpm[yudao-module-bpm]
  end

  subgraph "yudao-module-product/api"
    CatApi[ProductCategoryApi]
    SpuApi[ProductSpuApi]
    SkuApi[ProductSkuApi]
    CommentApi[ProductCommentApi]
  end

  Promotion -->|validateCategoryList| CatApi
  Promotion -->|validateSpuList| SpuApi
  Trade -->|createComment| CommentApi
  Trade -->|updateSkuStock| SkuApi
  Trade -->|getSkuMap| SkuApi
  Bpm -->|validateSpuList| SpuApi
  Bpm -->|getSpuMap| SpuApi
```

## 订单创建时 SKU 库存更新

```mermaid
sequenceDiagram
    autonumber
    participant OrderSvc as TradeOrderServiceImpl
    participant SkuApi as ProductSkuApi
    participant Impl as ProductSkuApiImpl
    participant SkuSvc as ProductSkuServiceImpl
    participant SkuMapper as ProductSkuMapper
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant DB as MySQL

    Note over OrderSvc: 用户提交订单
    OrderSvc->>SkuApi: updateSkuStock(spuId, -1)
    activate SkuApi
    SkuApi->>Impl: updateSkuStock(spuId, -1)
    activate Impl
    Impl->>SkuSvc: updateSkuStock(spuId, -1)
    activate SkuSvc

    SkuSvc->>SkuMapper: updateStock(spuId, -1)
    SkuMapper->>DB: UPDATE product_sku SET stock = stock + (-1)<br/>WHERE spu_id = spuId
    DB-->>SkuMapper: OK
    SkuMapper-->>SkuSvc: OK

    Note over SkuSvc: 同步更新 SPU 库存
    SkuSvc->>SpuSvc: updateSpuStock({spuId: -1})
    activate SpuSvc
    SpuSvc->>SpuMapper: updateStock(spuId, -1)
    SpuMapper->>DB: UPDATE product_spu SET stock = stock + (-1)<br/>WHERE id = spuId
    DB-->>SpuMapper: OK
    SpuMapper-->>SpuSvc: OK
    SpuSvc-->>SkuSvc: OK
    deactivate SpuSvc

    SkuSvc-->>Impl: OK
    deactivate SkuSvc
    Impl-->>SkuApi: OK
    deactivate Impl
    SkuApi-->>OrderSvc: OK
    deactivate SkuApi
```

## 订单取消时 SKU 库存恢复

```mermaid
sequenceDiagram
    autonumber
    participant OrderSvc as TradeOrderServiceImpl
    participant SkuApi as ProductSkuApi
    participant Impl as ProductSkuApiImpl
    participant SkuSvc as ProductSkuServiceImpl
    participant SkuMapper as ProductSkuMapper
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper

    Note over OrderSvc: 订单取消
    OrderSvc->>SkuApi: updateSkuStock(spuId, +1)
    activate SkuApi
    SkuApi->>Impl: updateSkuStock(spuId, +1)
    Impl->>SkuSvc: updateSkuStock(spuId, +1)
    SkuSvc->>SkuMapper: updateStock(spuId, +1)
    SkuSvc->>SpuSvc: updateSpuStock({spuId: +1})
    SpuSvc->>SpuMapper: updateStock(spuId, +1)
    SkuSvc-->>SkuApi: OK
    SkuApi-->>OrderSvc: OK
    deactivate SkuApi
```

## 营销活动范围校验（分类 + SPU）

```mermaid
sequenceDiagram
    autonumber
    participant PromoSvc as CouponTemplateServiceImpl
    participant CatApi as ProductCategoryApi
    participant CatImpl as ProductCategoryApiImpl
    participant CatSvc as ProductCategoryServiceImpl
    participant SpuApi as ProductSpuApi
    participant SpuImpl as ProductSpuApiImpl
    participant SpuSvc as ProductSpuServiceImpl

    Note over PromoSvc: 创建/更新优惠券模板<br/>validateProductScope

    PromoSvc->>CatApi: validateCategoryList(categoryIds)
    activate CatApi
    CatApi->>CatImpl: validateCategoryList(categoryIds)
    CatImpl->>CatSvc: validateCategoryList(categoryIds)
    CatSvc-->>CatImpl: 通过
    CatImpl-->>CatApi: 通过
    CatApi-->>PromoSvc: 通过
    deactivate CatApi

    PromoSvc->>SpuApi: validateSpuList(spuIds)
    activate SpuApi
    SpuApi->>SpuImpl: validateSpuList(spuIds)
    SpuImpl->>SpuSvc: validateSpuList(spuIds)
    SpuSvc-->>SpuImpl: 通过
    SpuImpl-->>SpuApi: 通过
    SpuApi-->>PromoSvc: 通过
    deactivate SpuApi

    Note over PromoSvc: 校验全部通过，保存模板
```

## 订单完成时评价创建

```mermaid
sequenceDiagram
    autonumber
    participant OrderSvc as TradeOrderUpdateServiceImpl
    participant CommentApi as ProductCommentApi
    participant Impl as ProductCommentApiImpl
    participant Svc as ProductCommentServiceImpl
    participant UserApi as MemberUserApi
    participant CommentMapper as ProductCommentMapper

    Note over OrderSvc: 订单状态变为"已完成"<br/>自动调用评价创建
    OrderSvc->>CommentApi: createComment(commentReqDTO)
    activate CommentApi
    CommentApi->>Impl: createComment(commentReqDTO)
    Impl->>Svc: createComment(commentReqDTO)
    activate Svc

    Svc->>UserApi: getMemberUser(userId)
    UserApi-->>Svc: {nickname, avatar}
    Svc->>Svc: 校验订单项未评价<br/>COMMENT_ORDER_EXISTS
    Svc->>Svc: setUserNickname / setUserAvatar
    Svc->>CommentMapper: insert(comment)
    CommentMapper-->>Svc: commentId

    Svc-->>Impl: commentId
    deactivate Svc
    Impl-->>CommentApi: commentId
    CommentApi-->>OrderSvc: commentId
    deactivate CommentApi
```

## source_nodes 追溯

- `interface:ProductCategoryApi` + `class:ProductCategoryApiImpl`
- `interface:ProductSpuApi` + `class:ProductSpuApiImpl`
- `interface:ProductSkuApi` + `class:ProductSkuApiImpl`
- `interface:ProductCommentApi` + `class:ProductCommentApiImpl`
- 跨模块消费方节点（已识别）：
  - `method:validateProductScope` (CouponTemplateServiceImpl)
  - `method:validateProductScope` (RewardActivityServiceImpl)
  - `method:createComment` (TradeOrderUpdateServiceImpl)
