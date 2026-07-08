# 序列图：商品分类树 CRUD 与跨模块校验

入口：backend-package-yudao-module-product
来源：business-flows.md 流程 2

---

## 分类新增

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductCategoryController
    participant Svc as ProductCategoryServiceImpl
    participant Mapper as ProductCategoryMapper
    participant DB as MySQL

    Admin->>Ctrl: POST /product/category/create
    Note over Ctrl: @PreAuthorize("product:category:create")
    Ctrl->>Svc: createCategory(createReqVO)
    activate Svc

    Svc->>Svc: validateParentProductCategory(parentId)
    alt parentId == 0
        Note over Svc: 顶级分类，跳过父分类校验
    else parentId != 0
        Svc->>Mapper: selectById(parentId)
        Mapper->>DB: SELECT * FROM product_category WHERE id=?
        alt 父分类不存在
            DB-->>Mapper: null
            Note over Svc: throw CATEGORY_PARENT_NOT_EXISTS
        else 父分类存在但 parentId != 0
            Note over Svc: throw CATEGORY_PARENT_NOT_FIRST_LEVEL
        else 父分类是一级
            Note over Svc: 通过
        end
    end

    Svc->>Svc: BeanUtils.toBean(createReqVO, ProductCategoryDO.class)
    Svc->>Mapper: insert(category)
    Mapper->>DB: INSERT INTO product_category (...)
    DB-->>Mapper: categoryId
    Mapper-->>Svc: categoryId
    Svc-->>Ctrl: categoryId
    deactivate Svc
    Ctrl-->>Admin: CommonResult<Long>(categoryId)
```

## 分类删除（多重保护）

```mermaid
sequenceDiagram
    autonumber
    participant Admin as 管理员
    participant Ctrl as ProductCategoryController
    participant CatSvc as ProductCategoryServiceImpl
    participant CatMapper as ProductCategoryMapper
    participant SpuSvc as ProductSpuServiceImpl
    participant SpuMapper as ProductSpuMapper
    participant DB as MySQL

    Admin->>Ctrl: DELETE /product/category/delete?id=5
    Ctrl->>CatSvc: deleteCategory(5)
    activate CatSvc

    CatSvc->>CatMapper: selectById(5)
    CatMapper->>DB: SELECT * FROM product_category WHERE id=5
    DB-->>CatMapper: category
    CatMapper-->>CatSvc: category

    alt 分类不存在
        Note over CatSvc: throw CATEGORY_NOT_EXISTS
    else 分类存在
        CatSvc->>CatMapper: selectCountByParentId(5)
        CatMapper->>DB: SELECT COUNT(*) FROM product_category WHERE parent_id=5
        DB-->>CatMapper: count

        alt count > 0
            Note over CatSvc: throw CATEGORY_EXISTS_CHILDREN
        else 无子分类
            CatSvc->>SpuSvc: getSpuCountByCategoryId(5)
            activate SpuSvc
            SpuSvc->>SpuMapper: selectCount(categoryId=5)
            SpuMapper->>DB: SELECT COUNT(*) FROM product_spu WHERE category_id=5
            DB-->>SpuMapper: spuCount
            SpuMapper-->>SpuSvc: spuCount
            SpuSvc-->>CatSvc: spuCount
            deactivate SpuSvc

            alt spuCount > 0
                Note over CatSvc: throw CATEGORY_HAVE_BIND_SPU
            else 无商品
                CatSvc->>CatMapper: deleteById(5)
                CatMapper->>DB: DELETE FROM product_category WHERE id=5
                DB-->>CatMapper: OK
                CatSvc-->>Ctrl: true
            end
        end
    end
    deactivate CatSvc
    Ctrl-->>Admin: CommonResult<Boolean>(true)
```

## 跨模块分类校验（营销活动范围）

```mermaid
sequenceDiagram
    autonumber
    participant Promotion as CouponTemplateServiceImpl
    participant CatApi as ProductCategoryApi
    participant Impl as ProductCategoryApiImpl
    participant Svc as ProductCategoryServiceImpl
    participant Mapper as ProductCategoryMapper
    participant DB as MySQL

    Promotion->>CatApi: validateCategoryList([10, 20, 30])
    activate CatApi
    CatApi->>Impl: validateCategoryList([10, 20, 30])
    activate Impl
    Impl->>Svc: validateCategoryList([10, 20, 30])
    activate Svc

    Svc->>Mapper: selectByIds([10, 20, 30])
    Mapper->>DB: SELECT * FROM product_category WHERE id IN (10, 20, 30)
    DB-->>Mapper: categories
    Mapper-->>Svc: categories

    loop 对每个 id
        alt 分类不存在
            Note over Svc: throw CATEGORY_NOT_EXISTS
        else 分类已禁用
            Note over Svc: throw CATEGORY_DISABLED
        else 层级 < 2
            Note over Svc: throw SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR
        else 通过
            Note over Svc: 继续
        end
    end

    Svc-->>Impl: 通过
    deactivate Svc
    Impl-->>CatApi: 通过
    deactivate Impl
    CatApi-->>Promotion: 通过
    deactivate CatApi
    Note over Promotion: 继续优惠券模板保存
```

## 层级递归计算

```mermaid
sequenceDiagram
    autonumber
    participant Caller as 调用方
    participant Svc as ProductCategoryServiceImpl
    participant Mapper as ProductCategoryMapper
    participant DB as MySQL

    Caller->>Svc: getCategoryLevel(15)
    activate Svc

    Svc->>Svc: level = 1

    loop 最多 127 次
        Svc->>Mapper: selectById(id)
        Mapper->>DB: SELECT * FROM product_category WHERE id=?
        DB-->>Mapper: category
        Mapper-->>Svc: category

        alt category == null or category.parentId == 0
            Note over Svc: break
        else
            Svc->>Svc: level++<br/>id = category.parentId
        end
    end

    Svc-->>Caller: level (>= 2 即合规)
    deactivate Svc
```

## source_nodes 追溯

- `method:createCategory` — 创建分类
- `method:updateCategory` — 更新分类
- `method:deleteCategory` — 删除分类
- `method:validateCategoryList` — 批量校验
- `method:validateParentProductCategory` — 父分类校验
- `method:getCategoryLevel` — 层级递归
- `interface:ProductCategoryApi`
- `class:ProductCategoryApiImpl`
- `class:ProductCategoryServiceImpl`
