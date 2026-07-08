# 序列图：用户收藏与浏览历史

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 7

---

## 用户收藏商品

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户 (App)
    participant Ctrl as AppFavoriteController
    participant FavSvc as ProductFavoriteServiceImpl
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant FavMapper as ProductFavoriteMapper
    participant DB as MySQL

    User->>Ctrl: POST /app/favorite/create
    Note over Ctrl: @PreAuthorize 用户已登录
    Ctrl->>FavSvc: createFavorite(userId, spuId)
    activate FavSvc

    FavSvc->>SpuSvc: validateSpuExists(spuId)
    activate SpuSvc
    SpuSvc->>SpuMapper: selectById(spuId)
    SpuMapper->>DB: SELECT * FROM product_spu WHERE id=?
    DB-->>SpuMapper: spu
    SpuMapper-->>SpuSvc: spu
    alt spu == null
        Note over SpuSvc: throw SPU_NOT_EXISTS
    else spu 存在
        SpuSvc-->>FavSvc: spu
    end
    deactivate SpuSvc

    FavSvc->>FavMapper: selectCountByUserIdAndSpuId(userId, spuId)
    FavMapper->>DB: SELECT COUNT(*) FROM product_favorite<br/>WHERE user_id=? AND spu_id=? AND deleted=0
    DB-->>FavMapper: count
    FavMapper-->>FavSvc: count

    alt count > 0
        Note over FavSvc: throw FAVORITE_EXISTS
    else count == 0
        FavSvc->>FavMapper: insert(favorite)
        FavMapper->>DB: INSERT INTO product_favorite (user_id, spu_id)
        DB-->>FavMapper: favoriteId
        FavSvc-->>Ctrl: favoriteId
    end
    deactivate FavSvc
    Ctrl-->>User: CommonResult<Long>(favoriteId)
```

## 批量收藏

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户
    participant Ctrl as AppFavoriteController
    participant FavSvc as ProductFavoriteServiceImpl
    participant SpuSvc as ProductSpuServiceImpl
    participant FavMapper as ProductFavoriteMapper

    User->>Ctrl: POST /app/favorite/create-batch<br/>{spuIds: [1, 2, 3]}
    Ctrl->>FavSvc: createFavoriteBatch(userId, [1, 2, 3])
    activate FavSvc

    loop 对每个 spuId
        FavSvc->>SpuSvc: validateSpuExists(spuId)
        FavSvc->>FavMapper: selectCountByUserIdAndSpuId
        alt 已收藏
            Note over FavSvc: 跳过<br/>(不抛异常)
        else 未收藏
            FavSvc->>FavMapper: insert(favorite)
        end
    end

    FavSvc-->>Ctrl: 成功数
    deactivate FavSvc
    Ctrl-->>User: CommonResult<Integer>(successCount)
```

## 浏览足迹记录（同时更新 SPU 浏览量）

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户
    participant Ctrl as AppProductBrowseHistoryController
    participant HistSvc as ProductBrowseHistoryServiceImpl
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant HistMapper as ProductBrowseHistoryMapper
    participant DB as MySQL

    User->>Ctrl: POST /app/browse-history/create<br/>{spuId: 100}
    Ctrl->>HistSvc: createBrowseHistory(userId, 100)
    activate HistSvc

    HistSvc->>SpuSvc: validateSpuExists(100)
    activate SpuSvc
    SpuSvc->>SpuMapper: selectById(100)
    SpuMapper-->>SpuSvc: spu
    SpuSvc-->>HistSvc: spu
    deactivate SpuSvc

    par 并行更新
        HistSvc->>SpuSvc: updateBrowseCount(100, 1)
        SpuSvc->>SpuMapper: updateBrowseCount(100, 1)
        SpuMapper->>DB: UPDATE product_spu<br/>SET browse_count = browse_count + 1<br/>WHERE id=100
    and
        HistSvc->>HistMapper: insertBrowseHistory(userId, 100)
        HistMapper->>DB: INSERT INTO product_browse_history<br/>(user_id, spu_id)
    end

    HistSvc-->>Ctrl: true
    deactivate HistSvc
    Ctrl-->>User: CommonResult<Boolean>(true)
```

## 浏览历史分页查询

```mermaid
sequenceDiagram
    autonumber
    participant User as 用户
    participant Ctrl as AppProductBrowseHistoryController
    participant HistSvc as ProductBrowseHistoryServiceImpl
    participant HistMapper as ProductBrowseHistoryMapper
    participant Convert as ProductBrowseHistoryConvert
    participant DB as MySQL

    User->>Ctrl: GET /app/browse-history/page
    Ctrl->>HistSvc: getBrowseHistoryPage(pageReqVO)
    HistSvc->>HistMapper: selectPage(pageReqVO)
    HistMapper->>DB: SELECT * FROM product_browse_history<br/>WHERE user_id=? AND user_deleted=0 AND deleted=0<br/>ORDER BY create_time DESC<br/>LIMIT ?, ?
    DB-->>HistMapper: histories + total
    HistMapper-->>HistSvc: PageResult<HistoryDO>
    HistSvc->>Convert: convert(histories)
    Convert-->>HistSvc: historiesRespVO
    HistSvc-->>Ctrl: PageResult<HistoryRespVO>
    Ctrl-->>User: CommonResult<PageResult<HistoryRespVO>>
```

## source_nodes 追溯

- `method:createFavorite` — 单条收藏
- `method:createFavoriteBatch` — 批量收藏
- `method:deleteFavorite` / `deleteFavoriteBatch` — 取消收藏
- `method:createBrowseHistory` — 浏览历史
- `method:updateBrowseCount` — 浏览量更新
- `method:getFavoritePage` / `getBrowseHistoryPage` — 分页查询
- `class:AppFavoriteController`
- `class:AppProductBrowseHistoryController`
