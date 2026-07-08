# 错误处理：商城商品中心后端 (backend-package-yudao-module-product)

入口 ID：backend-package-yudao-module-product
证据：evidence/backend-package-yudao-module-product/{nodes,typecards}.json
覆盖：分层错误处理 + 8 组错误码（24 条）

---

## 1. 错误处理总览

本入口采用"统一异常 + 错误码 + 分层校验"模式：

```
Controller 层
  - @Valid 注解 + Jakarta Validation（参数格式校验）
  - @PreAuthorize 权限校验（Spring Security）
  - 异常自动捕获并返回 CommonResult.error()
        ↓
Service 层
  - ServiceExceptionUtil.exception(ErrorCode) 抛出业务异常
  - @Transactional 事务回滚（rollbackFor = Exception.class）
  - 业务规则校验（存在性、唯一性、状态合法性）
        ↓
Repository 层
  - 数据库唯一索引兜底（避免并发唯一性冲突）
  - MyBatis-Plus 软删除
        ↓
GlobalExceptionHandler（框架级）
  - ServiceException → 业务错误码 + 错误消息
  - ValidationException → 参数校验失败信息
  - 其他异常 → 500
```

## 2. 错误码体系

`ErrorCodeConstants` 接口定义 8 组共 24 条错误码，统一在 1-008-000-000 段：

| 错误码 | 错误消息 | 触发场景 |
|---|---|---|
| 1-008-001-000 | 商品分类不存在 | validateCategory、validateCategoryList、getCategory |
| 1-008-001-001 | 父分类不存在 | validateParentProductCategory |
| 1-008-001-002 | 父分类不能是二级分类 | validateParentProductCategory |
| 1-008-001-003 | 存在子分类，无法删除 | deleteCategory |
| 1-008-001-004 | 商品分类({})已禁用，无法使用 | validateCategory、validateCategoryList |
| 1-008-001-005 | 类别下存在商品，无法删除 | deleteCategory |
| 1-008-002-000 | 品牌不存在 | validateBrandExists |
| 1-008-002-001 | 品牌已禁用 | validateProductBrand |
| 1-008-002-002 | 品牌名称已存在 | validateBrandNameUnique |
| 1-008-003-000 | 属性项不存在 | validatePropertyExists |
| 1-008-003-001 | 属性项的名称已存在 | validatePropertyNameUnique |
| 1-008-003-002 | 属性项下存在属性值，无法删除 | deleteProperty |
| 1-008-004-000 | 属性值不存在 | validatePropertyValueExists |
| 1-008-004-001 | 属性值的名称已存在 | validatePropertyValueNameUnique |
| 1-008-005-000 | 商品 SPU 不存在 | validateSpuExists、validateSpuList |
| 1-008-005-001 | 商品分类不正确，原因：必须使用第二级的商品分类及以下 | validateCategory（SPU 创建/更新） |
| 1-008-005-002 | 商品 SPU 保存失败，原因：优惠劵不存在 | (保留，TODO) |
| 1-008-005-003 | 商品 SPU【{}】不处于上架状态 | validateSpuList |
| 1-008-005-004 | 商品 SPU 不处于回收站状态 | deleteSpu |
| 1-008-006-000 | 商品 SKU 不存在 | validateSku |
| 1-008-006-001 | 商品 SKU 的属性组合存在重复 | validateSkuList |
| 1-008-006-002 | 一个 SPU 下的每个 SKU，其属性项必须一致 | validateSkuList |
| 1-008-006-003 | 一个 SPU 下的每个 SKU，必须不重复 | (保留) |
| 1-008-006-004 | 商品 SKU 库存不足 | (订单场景) |
| 1-008-007-000 | 商品评价不存在 | replyComment、updateCommentVisible |
| 1-008-007-001 | 订单的商品评价已存在 | createComment |
| 1-008-008-000 | 该商品已经被收藏 | createFavorite |
| 1-008-008-001 | 商品收藏不存在 | (保留) |

## 3. 异常处理机制

### 3.1 业务异常抛出

通过 `ServiceExceptionUtil.exception(ErrorCode)` 静态方法抛出：

```java
import static cn.iocoder.yudao.framework.common.exception.util.ServiceExceptionUtil.exception;

if (category == null) {
    throw exception(CATEGORY_NOT_EXISTS);
}
```

参数化错误消息：

```java
throw exception(CATEGORY_DISABLED, category.getName());
// 错误消息："商品分类(${name})已禁用，无法使用"
```

### 3.2 事务回滚

涉及多表修改的方法统一使用 `@Transactional(rollbackFor = Exception.class)`：

```java
@Transactional(rollbackFor = Exception.class)
public Long createSpu(ProductSpuSaveReqVO createReqVO) {
    // 多步：validateCategory + validateBrand + validateSku + initSpuFromSkus + insert + createSkuList
    // 任何一步抛出异常，全部回滚
}
```

涉及事务的方法：
- `ProductSpuServiceImpl.createSpu / updateSpu / deleteSpu / updateSpuStock / updateSpuStatus`
- `ProductCategoryServiceImpl.createCategory / updateCategory / deleteCategory`
- `ProductBrandServiceImpl.createBrand / updateBrand / deleteBrand`
- `ProductPropertyServiceImpl.createProperty / updateProperty / deleteProperty`
- `ProductPropertyValueServiceImpl.createPropertyValue / updatePropertyValue / deletePropertyValue`
- `ProductCommentServiceImpl.createComment / replyComment / updateCommentVisible`
- `ProductSkuServiceImpl.createSkuList / updateSkuList / deleteSkuBySpuId`

### 3.3 Controller 层参数校验

通过 `@Valid` + Jakarta Validation 注解：

```java
public CommonResult<Long> createProductSpu(@Valid @RequestBody ProductSpuSaveReqVO createReqVO) {
    return success(productSpuService.createSpu(createReqVO));
}
```

`ProductSpuSaveReqVO` 等 VO 中使用 `@NotNull`、`@Size`、`@Min`、`@Max` 等注解做格式校验。

校验失败时由全局异常处理器返回 400 状态码和详细错误信息。

### 3.4 权限校验

通过 Spring Security `@PreAuthorize` 注解：

```java
@PreAuthorize("@ss.hasPermission('product:spu:create')")
public CommonResult<Long> createProductSpu(...) { ... }
```

权限点格式：`product:{module}:{action}`

action 枚举：
- `query`：查询
- `create`：新增
- `update`：编辑
- `delete`：删除
- `export`：导出

## 4. 跨模块错误传播

跨模块 RPC 调用时，业务异常会通过 `@RestController` 自动序列化为 JSON 错误响应：

```json
{
  "code": 1008001000,
  "data": null,
  "msg": "商品分类不存在"
}
```

调用方可通过 `code` 字段判断错误类型并作出处理。

## 5. 错误处理模式

### 5.1 存在性校验模式

```java
private ProductSpuDO validateSpuExists(Long id) {
    ProductSpuDO spuDO = productSpuMapper.selectById(id);
    if (spuDO == null) {
        throw exception(SPU_NOT_EXISTS);
    }
    return spuDO;
}
```

适用：所有"先查再改"的方法。

### 5.2 唯一性校验模式

```java
private void validateBrandNameUnique(String name) {
    ProductBrandDO brand = brandMapper.selectByName(name);
    if (brand != null) {
        throw exception(BRAND_NAME_EXISTS);
    }
}
```

适用：新增/编辑时的名称唯一性检查。

### 5.3 状态校验模式

```java
if (ObjectUtil.notEqual(spuDO.getStatus(), ProductSpuStatusEnum.RECYCLE.getStatus())) {
    throw exception(SPU_NOT_RECYCLE);
}
```

适用：状态机相关操作。

### 5.4 集合校验模式

```java
public void validateCategoryList(Collection<Long> ids) {
    if (CollUtil.isEmpty(ids)) {
        return;
    }
    List<ProductCategoryDO> list = productCategoryMapper.selectByIds(ids);
    Map<Long, ProductCategoryDO> categoryMap = CollectionUtils.convertMap(list, ProductCategoryDO::getId);
    ids.forEach(id -> {
        // 逐个校验
    });
}
```

适用：批量校验场景。

### 5.5 跨实体引用校验

```java
if (productSpuService.getSpuCountByCategoryId(id) > 0) {
    throw exception(CATEGORY_HAVE_BIND_SPU);
}
```

适用：删除前校验关联关系。

### 5.6 业务规则校验

```java
private void validateCategory(Long id) {
    ProductCategoryDO category = productCategoryMapper.selectById(id);
    if (category == null) {
        throw exception(CATEGORY_NOT_EXISTS);
    }
    if (Objects.equals(category.getStatus(), CommonStatusEnum.DISABLE.getStatus())) {
        throw exception(CATEGORY_DISABLED, category.getName());
    }
}
```

适用：跨实体的有效性校验。

## 6. 失败场景与错误码映射

| 失败场景 | 触发方法 | 错误码 |
|---|---|---|
| 删除有子分类的分类 | deleteCategory | CATEGORY_EXISTS_CHILDREN |
| 删除有商品的分类 | deleteCategory | CATEGORY_HAVE_BIND_SPU |
| 删除非回收站的 SPU | deleteSpu | SPU_NOT_RECYCLE |
| 跨模块引用下架 SPU | validateSpuList | SPU_NOT_ENABLE |
| SPU 关联非二级分类 | createSpu / updateSpu | SPU_SAVE_FAIL_CATEGORY_LEVEL_ERROR |
| 父分类不是一级 | createCategory / updateCategory | CATEGORY_PARENT_NOT_FIRST_LEVEL |
| SKU 属性组合重复 | createSpu / updateSpu | SKU_PROPERTIES_DUPLICATED |
| SKU 属性项不一致 | createSpu / updateSpu | SPU_ATTR_NUMBERS_MUST_BE_EQUALS |
| 品牌名称重复 | createBrand / updateBrand | BRAND_NAME_EXISTS |
| 删除有属性值的属性项 | deleteProperty | PROPERTY_DELETE_FAIL_VALUE_EXISTS |
| 同一订单重复评价 | createComment | COMMENT_ORDER_EXISTS |
| 重复收藏 | createFavorite | FAVORITE_EXISTS |

## 7. source_nodes 追溯

- 1 个 ErrorCodeConstants 接口节点（包含 24 条 ErrorCode 常量）
- 各 service 类中的 `exception(...)` 调用点
- `@Transactional` 注解的方法
- `@PreAuthorize` 注解的 controller 方法
- `@Valid` 注解的 controller 方法
