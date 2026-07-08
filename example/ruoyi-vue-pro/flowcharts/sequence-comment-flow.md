# 序列图：商品评价创建与回复

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 5

---

## 用户创建评价（订单完成后自动调用）

```mermaid
sequenceDiagram
    autonumber
    participant Order as 订单模块<br/>(TradeOrderUpdateServiceImpl)
    participant Api as ProductCommentApi
    participant Impl as ProductCommentApiImpl
    participant Svc as ProductCommentServiceImpl
    participant UserApi as MemberUserApi
    participant SkuSvc as ProductSkuServiceImpl
    participant Mapper as ProductCommentMapper
    participant DB as MySQL

    Order->>Api: createComment(commentReqDTO)
    Note over Api: @PostMapping 触发<br/>订单完成事件
    Api->>Impl: createComment(commentReqDTO)
    activate Impl

    Impl->>Svc: createComment(commentReqDTO)
    activate Svc

    Svc->>UserApi: getMemberUser(userId)
    activate UserApi
    UserApi-->>Svc: {nickname, avatar}
    deactivate UserApi

    Svc->>Svc: 校验订单项未评价
    Note over Svc: 抛 COMMENT_ORDER_EXISTS

    Svc->>SkuSvc: validateSku(skuId, spuId)
    activate SkuSvc
    SkuSvc-->>Svc: 通过
    deactivate SkuSvc

    Svc->>Svc: BeanUtils.toBean(commentReqDTO, ProductCommentDO.class)
    Svc->>Svc: setUserNickname/setUserAvatar

    Svc->>Mapper: insert(comment)
    activate Mapper
    Mapper->>DB: INSERT INTO product_comment (...)
    DB-->>Mapper: commentId
    Mapper-->>Svc: commentId
    deactivate Mapper

    Svc-->>Impl: commentId
    deactivate Svc
    Impl-->>Api: commentId
    deactivate Impl
    Api-->>Order: commentId
```

## 商家回复评价

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductCommentController
    participant Svc as ProductCommentServiceImpl
    participant Mapper as ProductCommentMapper
    participant DB as MySQL

    Admin->>Ctrl: PUT /product/comment/reply
    Note over Ctrl: @PreAuthorize("product:comment:update")
    Ctrl->>Svc: replyComment(replyReqVO)
    activate Svc

    Svc->>Mapper: selectById(replyReqVO.getId())
    Mapper->>DB: SELECT * FROM product_comment WHERE id=?
    DB-->>Mapper: comment
    Mapper-->>Svc: comment

    alt comment 存在
        Svc->>Svc: setReplyUserId / setReplyContent / setReplyTime
        Svc->>Svc: setReplyStatus(1)
        Svc->>Mapper: updateById(comment)
        Mapper->>DB: UPDATE product_comment SET reply_...
        DB-->>Mapper: OK
        Mapper-->>Svc: OK
        Svc-->>Ctrl: true
    else comment 不存在
        Svc-->>Ctrl: ❌ throw COMMENT_NOT_EXISTS
    end
    deactivate Svc
    Ctrl-->>Admin: CommonResult<Boolean>(true)
```

## 评价可见性切换

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductCommentController
    participant Svc as ProductCommentServiceImpl
    participant Mapper as ProductCommentMapper
    participant DB as MySQL

    Admin->>Ctrl: PUT /product/comment/update-visible
    Note over Ctrl: @PreAuthorize("product:comment:update")
    Ctrl->>Svc: updateCommentVisible(visibleReqVO)
    activate Svc

    Svc->>Mapper: selectById(visibleReqVO.getId())
    Mapper->>DB: SELECT * FROM product_comment WHERE id=?
    DB-->>Mapper: comment
    Mapper-->>Svc: comment

    alt comment 存在
        Svc->>Svc: comment.setVisible(visibleReqVO.getVisible())
        Svc->>Mapper: updateById(comment)
        Mapper->>DB: UPDATE product_comment SET visible=?
        DB-->>Mapper: OK
        Svc-->>Ctrl: true
    else comment 不存在
        Svc-->>Ctrl: ❌ throw COMMENT_NOT_EXISTS
    end
    deactivate Svc
    Ctrl-->>Admin: CommonResult<Boolean>(true)
```

## 评价分页查询

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductCommentController
    participant Svc as ProductCommentServiceImpl
    participant Mapper as ProductCommentMapper
    participant DB as MySQL

    Admin->>Ctrl: GET /product/comment/page?<br/>replyStatus=0&spuName=...
    Ctrl->>Svc: getCommentPage(pageReqVO)
    Svc->>Mapper: selectPage(pageReqVO)
    Mapper->>DB: SELECT * FROM product_comment<br/>WHERE deleted=0<br/>AND (replyStatus=0)<br/>AND (spu_name LIKE ?)<br/>LIMIT ?, ?
    DB-->>Mapper: comments + total
    Mapper-->>Svc: PageResult<Comment>
    Svc-->>Ctrl: PageResult<Comment>
    Ctrl->>Ctrl: BeanUtils.toBean(... RespVO.class)
    Ctrl-->>Admin: CommonResult<PageResult<RespVO>>
```

## source_nodes 追溯

- `method:createComment` — 创建评价
- `method:replyComment` — 商家回复
- `method:updateCommentVisible` — 可见性切换
- `method:getCommentPage` — 分页查询
- `interface:ProductCommentApi`
- `class:ProductCommentApiImpl`
- `class:ProductCommentController`
