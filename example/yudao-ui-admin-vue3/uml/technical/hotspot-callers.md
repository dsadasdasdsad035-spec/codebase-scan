# 热点调用：跨页同名函数

入口：frontend-mall-product
证据：codegraph callers 输出（已做按 filePath 过滤 + 节点去重）

---

## 1. handleDelete（5 个 index 页同名）

按 filePath 区分：

| 文件 | kind | 行为 |
|---|---|---|
| brand/index.vue | function | 二次确认 → deleteBrand(id) → refresh |
| category/index.vue | function | 二次确认 → deleteCategory(id) → refresh |
| property/index.vue | function | 二次确认 → deleteProperty(id) → refresh |
| property/value/index.vue | function | 二次确认 → deletePropertyValue(id) → refresh |
| comment/index.vue | function | 二次确认 → CommentApi.deleteComment（隐含，源码中通过 visible 切换替代） |
| spu/index.vue | function | 二次确认 → deleteSpu(id) → getTabsCount + getList |

调用方节点：function:983b518f40fda455789509dcb62ad231
被调用 API：deleteBrand / deleteCategory / deleteProperty / deletePropertyValue / deleteSpu

## 2. openForm（4 个 index 页同名）

| 文件 | 行为 |
|---|---|
| brand/index.vue | formRef.value.open(type, id) |
| category/index.vue | formRef.value.open(type, id) |
| property/index.vue | formRef.value.open(type, id) |
| property/value/index.vue | formRef.value.open(type, propertyId, id) |
| spu/index.vue | router.push({name: 'ProductSpuAdd'/'Edit'}) |

调用方节点：function:9d410f93c4e3fb316471e4011bc33645
被调用：formRef.open / router.push

## 3. getList（5 个 index 页同名）

| 文件 | 行为 |
|---|---|
| brand/index.vue | 调 ProductBrandApi.getBrandParam |
| category/index.vue | 调 ProductCategoryApi.getCategoryList |
| property/index.vue | 调 PropertyApi.getPropertyPage |
| property/value/index.vue | 调 PropertyApi.getPropertyValuePage |
| spu/index.vue | 调 ProductSpuApi.getSpuPage |

调用方节点：function:a7b69ca7e3cedfd7627766502b02168b
被调用：5 个分页 API

## 4. handleQuery（5 个 index 页同名）

| 文件 | 行为 |
|---|---|
| brand/index.vue | getList() |
| category/index.vue | getList() |
| property/index.vue | queryParams.pageNo = 1; getList() |
| property/value/index.vue | queryParams.pageNo = 1; getList() |
| spu/index.vue | getList() |
| comment/index.vue | queryParams.pageNo = 1; getList() |

调用方节点：function:fca2bb4e4ef22a93a183643ecf5f888d

## 5. resetQuery（5 个 index 页同名）

| 文件 | 行为 |
|---|---|
| brand/index.vue | queryFormRef.value.resetFields() + handleQuery() |
| category/index.vue | 同上 |
| property/index.vue | 同上 |
| property/value/index.vue | 同上 |
| spu/index.vue | 同上 |
| comment/index.vue | 同上 |

调用方节点：function:1b470cebdbb9c91f91e22c51714b130d

## 6. 跨入口调用

- category/index.vue handleViewSpu → router.push ProductSpu
- spu/index.vue openForm / openDetail → router.push ProductSpuAdd/Edit/Detail

## 7. 跨子域组件复用

- spu/form/SkuForm.vue 从 spu/components/index.ts 导入 { SkuList, getPropertyList, RuleConfig, PropertyAndValues }
- comment/CommentForm.vue 复用 spu/components/SpuShowcase.vue + spu/components/SkuTableSelect.vue
- spu/form/InfoForm.vue 复用 category/components/ProductCategorySelect.vue

---

**热点模式**：
- 5 个 list page 都使用统一模板：getList / handleQuery / resetQuery / openForm / handleDelete
- 二次确认 + 列表刷新是删除的固定范式
- 跨入口导航只有 1 处（分类 → SPU）
- 跨子域组件复用集中在 spu/components（4 个被多页引用）
