# 错误处理：商城商品中心 (frontend-mall-product)

入口 ID：frontend-mall-product
证据：evidence/frontend-mall-product/{nodes,typecards}.json

---

## 1. 表单校验错误

**触发场景**：用户提交 form 时 `formRef.value.validate()` 失败

**处理**：
- Element Plus 自动在字段下显示红色错误提示（来自 `formRules` 配置）
- 复合表单（spu/form/index.vue）校验失败时：
  - `message.error('【基础设置】不完善，请填写相关信息')` 等本地化提示
  - `emit('update:activeName', 失败Tab名)` 自动切到对应 Tab
  - `throw e` 截断后续子表单校验
- 表单内部 `valid` 检查：若校验未通过直接 return，不调 API

**典型规则**：
- `name: required, blur` 触发
- `picUrl: required, blur` 触发
- `sort: required, blur` 触发
- `spuId / skuId / userAvatar / userNickname / content / scores`: required
- `parentId / categoryId / brandId / keyword / introduction / sliderPicUrls`: required
- `subCommissionType / specType`: required

**source_nodes**：所有 *Form.vue 组件的 `submitForm` 函数节点。

---

## 2. 二次确认取消

**触发场景**：用户点击"删除"/"上架"/"下架"/"回收"/"恢复"/"显隐评论"等危险操作

**处理**：
- 调用 `message.delConfirm()` / `message.confirm(文案)` / `message.exportConfirm()`
- 用户取消 → `try { ... } catch {}` 静默吞掉，不调 API，不更新 UI
- 状态变更类操作（Switch）异常时手动回滚：例如 `row.visible = !changedValue`、`row.status` 翻转

**典型场景**：
- 删除品牌/分类/属性/属性值/SPU
- SPU 上下架/回收/恢复
- 评价显示/隐藏
- Excel 导出
- 复合表单 5 子表单校验失败

**source_nodes**：
- `function:983b518f40fda455789509dcb62ad231`（handleDelete，跨 5 个 index 页）
- `function:983b518f40fda455789509dcb62ad231`（handleStatusChange / handleStatus02Change / handleVisibleChange，按 name 聚合）

---

## 3. 加载状态保护

**触发场景**：异步请求未完成时用户继续操作

**处理**：
- `loading.value = true` 在 `getList` 起始处设置，`finally` 中必置 false
- `formLoading.value = true` 在 `submitForm` 起始处设置，禁用提交按钮 `:disabled="formLoading"`
- 评价显隐操作：`handleVisibleChange` 起始检查 `if (loading.value) return` 直接拒绝
- SPU 状态变更：异常时回滚 `row.status` 字段

**source_nodes**：所有 index.vue + form.vue。

---

## 4. SKU 价格校验

**触发场景**：多规格 SKU 编辑时

**处理**：
- `ruleConfig` 在 `SkuForm.vue` 中集中定义 4 个字段的校验规则
- `stock: arg >= 1`（库存必须 ≥ 1）
- `price: arg >= 0.01`（销售价 ≥ 0.01 元）
- `marketPrice: arg >= 0.01`（市场价 ≥ 0.01 元）
- `costPrice: arg >= 0.01`（成本价 ≥ 0.01 元）
- `SkuList.validateSku()` 在 SPU 表单整体提交时被调用，校验失败抛错

**典型错误提示**：
- "商品库存必须大于等于 1 ！！！"
- "商品销售价格必须大于等于 0.01 元！！！"
- "商品市场价格必须大于等于 0.01 元！！！"
- "商品成本价格必须大于等于 0.00 元！！！"

**source_nodes**：`component:18a9cdf6cda3537530eea5e2dc6b6492`（SkuForm.vue）、`component:7e42462b163ba6dd1fd60611ebe27fc6`（SkuList.vue）

---

## 5. 路由参数处理

**触发场景**：用户通过 URL 直接访问或路由跳转

**处理**：
- `property/value/index.vue`：`queryParams.propertyId = params.propertyId`，若未传则 propertyOptions 为空（下拉禁用但仍可读）
- `spu/index.vue`：`if (route.query.categoryId) queryParams.categoryId = route.query.categoryId`（来自分类页"查看商品"跳转）
- `spu/form/index.vue`：`if ('ProductSpuDetail' === name) isDetail.value = true`（详情模式）
- `spu/form/index.vue`：`const id = params.id as unknown as number` 解析路由 id

**source_nodes**：
- `component:3c079e65e1a015eb488ef8ed4aebf6b6`（value/index.vue）
- `component:766a92ffa67135de01a5219da9b57bf5`（spu/index.vue）
- `component:be487fd29883255ef6a9971a99c1c589`（spu/form/index.vue）

---

## 6. 分↔元换算与精度

**触发场景**：SKU 价格编辑、详情模式回显

**处理**：
- 编辑模式回显：`formatToFraction(item.price)` 等（分转元，UI 显示）
- 详情模式回显：`floatToFixed2(item.price)` 等（保留 2 位小数）
- 提交时：`convertToInteger(item.price)` 等（元转分，持久化到后端）
- 列表显示：`fenToYuan(row.price)`（分转元，加 ¥ 前缀）

**典型调用**：
- `formatToFraction / floatToFixed2` 在 spu/form/index.vue 的 `getDetail` 中按 `isDetail` 区分
- `convertToInteger` 在 spu/form/index.vue 的 `submitForm` 中批量处理
- `fenToYuan` 在 spu/index.vue 的表格列模板中

**source_nodes**：
- `component:be487fd29883255ef6a9971a99c1c589`（spu/form/index.vue）
- `component:766a92ffa67135de01a5219da9b57bf5`（spu/index.vue）

---

## 7. 轮播图 URL 化

**触发场景**：用户在前端选择本地图作为轮播图，提交时需转为 URL 字符串

**处理**：
```ts
const newSliderPicUrls: any[] = []
deepCopyFormData.sliderPicUrls!.forEach((item: any) => {
  typeof item === 'object' ? newSliderPicUrls.push(item.url) : newSliderPicUrls.push(item)
})
deepCopyFormData.sliderPicUrls = newSliderPicUrls
```
- 元素是对象（如前端选图组件返回 `{url, name}`）→ 取 `url`
- 元素是字符串（如已上传图片 URL）→ 直接 push

**source_nodes**：`component:be487fd29883255ef6a9971a99c1c589`（spu/form/index.vue）

---

## 8. API 错误

**处理**：
- 列表/表单 loading 状态在 `try-finally` 兜底置 false
- 业务错误信息通过项目统一 axios 拦截器（`utils/request`）提示，业务代码不直接处理
- 跨页面跳转失败（如路由不存在）由 vue-router 统一处理

---

## 9. 越界与回滚

- 删除 SPU 仅在 `tabType === 4`（回收站）下显示按钮，避免误删
- Switch 状态变更异常时回滚 UI
- 二次确认失败：catch {} 静默
- 详情模式下所有子表单 `:disabled="isDetail"`，InfoForm/SkuForm/DeliveryForm/DescriptionForm/OtherForm 全部只读

**source_nodes**：所有 form.vue 中 `isDetail` 引用。
