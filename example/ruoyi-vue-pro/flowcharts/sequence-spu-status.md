# 序列图：商品 SPU 状态流转

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 3

---

## 主流程

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant SkuSvc as ProductSkuServiceImpl
    participant DB as MySQL

    Note over Admin,DB: 上架/下架
    Admin->>Ctrl: PUT /product/spu/update-status
    Note over Ctrl: @PreAuthorize("product:spu:update")
    Ctrl->>SpuSvc: updateSpuStatus(updateReqVO)
    activate SpuSvc
    SpuSvc->>SpuSvc: validateSpuExists(id)
    SpuSvc->>SpuMapper: selectById(id)
    SpuMapper->>DB: SELECT * FROM product_spu WHERE id=?
    DB-->>SpuMapper: spu
    SpuMapper-->>SpuSvc: spu

    SpuSvc->>SpuSvc: spu.setStatus(updateReqVO.getStatus())
    SpuSvc->>SpuMapper: updateById(spu)
    SpuMapper->>DB: UPDATE product_spu SET status=?
    DB-->>SpuMapper: OK
    SpuMapper-->>SpuSvc: OK
    SpuSvc-->>Ctrl: true
    deactivate SpuSvc
    Ctrl-->>Admin: CommonResult<Boolean>(true)
```

## 状态 Tab 计数

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant DB as MySQL

    Admin->>Ctrl: GET /product/spu/get-count
    Ctrl->>SpuSvc: getTabsCount()
    activate SpuSvc

    par 并行 5 个统计
        SpuSvc->>SpuMapper: selectCount(status=ENABLE)
        SpuMapper->>DB: SELECT COUNT(*) WHERE status=1
    and
        SpuSvc->>SpuMapper: selectCount(status=DISABLE)
    and
        SpuSvc->>SpuMapper: selectCount(stock=0)
    and
        SpuSvc->>SpuMapper: selectCount() 总数
    and
        SpuSvc->>SpuMapper: selectCount(status=RECYCLE)
    end

    SpuMapper-->>SpuSvc: 5 个 count
    SpuSvc->>SpuSvc: 包装 Map<Integer, Long>
    SpuSvc-->>Ctrl: counts map
    deactivate SpuSvc
    Ctrl-->>Admin: CommonResult<Map>(counts)
```

## 删除流程（仅 RECYCLE 状态）

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant SkuSvc as ProductSkuServiceImpl
    participant SkuMapper as ProductSkuMapper
    participant DB as MySQL

    Admin->>Ctrl: DELETE /product/spu/delete?id=123
    Ctrl->>SpuSvc: deleteSpu(123)
    activate SpuSvc
    SpuSvc->>SpuMapper: selectById(123)
    SpuMapper->>DB: SELECT * FROM product_spu WHERE id=123
    DB-->>SpuMapper: spu (status=RECYCLE)
    SpuMapper-->>SpuSvc: spu

    alt spu.status == RECYCLE
        SpuSvc->>SpuMapper: deleteById(123)
        SpuMapper->>DB: DELETE FROM product_spu WHERE id=123
        SpuSvc->>SkuSvc: deleteSkuBySpuId(123)
        SkuSvc->>SkuMapper: deleteBySpuId(123)
        SkuMapper->>DB: DELETE FROM product_sku WHERE spu_id=123
        SpuSvc-->>Ctrl: true
    else spu.status != RECYCLE
        Note over SpuSvc: throw SPU_NOT_RECYCLE
        SpuSvc-->>Ctrl: 异常
    end
    deactivate SpuSvc
    Ctrl-->>Admin: CommonResult<Boolean>(true)
```

## 跨模块 SPU 校验

```mermaid
sequenceDiagram
    autonumber
    participant Caller as 业务模块<br/>(yudao-module-promotion 等)
    participant Api as ProductSpuApiImpl
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant DB as MySQL

    Caller->>Api: validateSpuList([1,2,3])
    activate Api
    Api->>SpuSvc: validateSpuList([1,2,3])
    activate SpuSvc
    SpuSvc->>SpuMapper: selectByIds([1,2,3])
    SpuMapper->>DB: SELECT * FROM product_spu WHERE id IN (1,2,3)
    DB-->>SpuMapper: spus
    SpuMapper-->>SpuSvc: spus
    SpuSvc->>SpuSvc: 逐个校验存在 + 状态==ENABLE
    alt 校验通过
        SpuSvc-->>Api: spus list
        Api-->>Caller: spus list
    else SPU 不存在
        SpuSvc-->>Api: throw SPU_NOT_EXISTS
        Api-->>Caller: 异常向上抛出
    else SPU 未上架
        SpuSvc-->>Api: throw SPU_NOT_ENABLE
        Api-->>Caller: 异常向上抛出
    end
    deactivate SpuSvc
    deactivate Api
```

## source_nodes 追溯

- `method:updateSpuStatus`
- `method:deleteSpu`
- `method:getTabsCount`
- `method:validateSpuList`
- `method:deleteSkuBySpuId`
- `enum:ProductSpuStatusEnum`
