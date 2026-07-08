# 序列图：商品 SPU 复合表单保存

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 4

---

## 主体流程

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant CategorySvc as ProductCategoryServiceImpl
    participant BrandSvc as ProductBrandServiceImpl
    participant SkuSvc as ProductSkuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant SkuMapper as ProductSkuMapper
    participant DB as MySQL

    Admin->>Ctrl: POST /product/spu/create
    Note over Ctrl: @PreAuthorize("product:spu:create")<br/>@Valid createReqVO

    Ctrl->>SpuSvc: createSpu(createReqVO)
    activate SpuSvc
    Note over SpuSvc: @Transactional(rollbackFor=Exception)

    SpuSvc->>CategorySvc: validateCategory(categoryId)
    activate CategorySvc
    CategorySvc->>CategorySvc: getCategoryLevel(id) >= 2
    CategorySvc-->>SpuSvc: 通过
    deactivate CategorySvc

    SpuSvc->>BrandSvc: validateProductBrand(brandId)
    activate BrandSvc
    BrandSvc-->>SpuSvc: 通过
    deactivate BrandSvc

    SpuSvc->>SkuSvc: validateSkuList(skuList, specType)
    activate SkuSvc
    SkuSvc->>SkuSvc: 校验属性项一致<br/>校验组合不重复
    SkuSvc-->>SpuSvc: 通过
    deactivate SkuSvc

    SpuSvc->>SpuSvc: initSpuFromSkus(spu, skus)
    Note over SpuSvc: price = min(skuPrice)<br/>marketPrice = min(marketPrice)<br/>costPrice = min(costPrice)<br/>stock = sum(stock)<br/>status = ENABLE(1)<br/>salesCount = 0<br/>browseCount = 0

    SpuSvc->>SpuMapper: insert(spu)
    activate SpuMapper
    SpuMapper->>DB: INSERT INTO product_spu (...)
    DB-->>SpuMapper: spuId
    SpuMapper-->>SpuSvc: spuId
    deactivate SpuMapper

    SpuSvc->>SkuSvc: createSkuList(spuId, skuList)
    activate SkuSvc
    SkuSvc->>SkuMapper: insertBatch(skuList)
    activate SkuMapper
    SkuMapper->>DB: INSERT INTO product_sku (...)
    DB-->>SkuMapper: OK
    SkuMapper-->>SkuSvc: OK
    deactivate SkuMapper
    SkuSvc-->>SpuSvc: OK
    deactivate SkuSvc

    SpuSvc-->>Ctrl: spuId
    deactivate SpuSvc

    Ctrl-->>Admin: CommonResult<Long>(spuId)
```

## 失败场景

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant CategorySvc as ProductCategoryServiceImpl
    participant BrandSvc as ProductBrandServiceImpl
    participant SkuSvc as ProductSkuServiceImpl

    Admin->>Ctrl: POST /product/spu/create (categoryId=1)
    Ctrl->>SpuSvc: createSpu(createReqVO)

    SpuSvc->>CategorySvc: validateCategory(1)
    CategorySvc-->>SpuSvc: ❌ 分类层级不够
    Note over SpuSvc: throw SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR

    SpuSvc-->>Ctrl: 异常向上抛出
    Ctrl-->>Admin: CommonResult.error(1_008_005_001, "...")
```

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductSpuController
    participant SpuSvc as ProductSpuServiceImpl
    participant SkuSvc as ProductSkuServiceImpl

    Admin->>Ctrl: POST /product/spu/create (skus 重复)
    Ctrl->>SpuSvc: createSpu(createReqVO)
    SpuSvc->>SkuSvc: validateSkuList(...)
    SkuSvc-->>SpuSvc: ❌ SKU 属性组合重复
    Note over SpuSvc: throw SKU_PROPERTIES_DUPLICATED
    SpuSvc-->>Ctrl: 异常向上抛出
    Ctrl-->>Admin: CommonResult.error(1_008_006_001, "...")
```

## source_nodes 追溯

- `method:createSpu` — 事务入口
- `method:validateCategory` — 分类层级校验
- `method:initSpuFromSkus` — 价格汇总
- `method:validateSkuList` — SKU 校验
- `method:createSkuList` — SKU 批量插入
