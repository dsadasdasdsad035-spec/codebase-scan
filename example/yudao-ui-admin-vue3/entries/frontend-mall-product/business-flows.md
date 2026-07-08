# 业务流程：商城商品中心 (frontend-mall-product)

入口 ID：frontend-mall-product
证据：evidence/frontend-mall-product/{nodes,typecards}.json
覆盖：5 大子域核心业务流程 + 跨子域组件复用

---

## 流程 1：品牌 CRUD 流程

**触发条件**：用户进入商品中心 → 点击左侧菜单"商品品牌"

**输入**：搜索条件（name/status/createTime）；新增/编辑表单字段

**输出**：分页品牌列表；新增/编辑/删除结果

**主路径**：

1. `onMounted` 触发 `getList`
2. `getList` 调 `ProductBrandApi.getBrandParam(queryParams)` 获取 `{list, total}`
3. 列表更新 `list.value` 与 `total.value`，el-table 渲染
4. 用户操作：
   - **新增**：点击新增 → `openForm('create')` → `BrandForm.open('create')` → 弹出 Dialog → 校验 → `createBrand` → `emit('success')` → 列表 refresh
   - **编辑**：点击行内编辑 → `openForm('update', id)` → `BrandForm.open('update', id)` → `getBrand(id)` 回显 → 校验 → `updateBrand` → emit('success') → refresh
   - **删除**：点击行内删除 → `message.delConfirm()` 二次确认 → `deleteBrand(id)` → `message.success('删除成功')` → refresh
5. 搜索/重置：`handleQuery` / `resetQuery` 调 `getList`

**失败路径**：
- 校验失败：表单内联错误提示，不调 API
- API 失败：列表 loading=false，错误由 axios 拦截器统一提示
- 删除二次确认取消：不调 API，列表无变化

**热点调用**：`getList`（搜索/分页/刷新都会触发）

**source_nodes**：`component:fbb82fd3451fdc7dcf941a9a95fdcb8b`、`component:2d7a9b88d8c6b9d362467ec2d6beba0e`、`function:a7b69ca7e3cedfd7627766502b02168b`、`function:9d410f93c4e3fb316471e4011bc33645`、`function:983b518f40fda455789509dcb62ad231`、`function:1b470cebdbb9c91f91e22c51714b130d`、`function:fca2bb4e4ef22a93a183643ecf5f888d`

---

## 流程 2：商品分类树 CRUD 流程

**触发条件**：用户进入"商品分类"页

**输入**：分类名称；上级分类（必选，0=顶级）；图片；排序；状态

**输出**：树形分类列表（默认展开）；新增/编辑/删除结果

**主路径**：

1. `onMounted` → `getList` → `getCategoryList(queryParams)` → `handleTree(data, 'id', 'parentId')` 构树
2. 行操作：
   - **编辑**：同品牌流程
   - **查看商品**（仅 parentId > 0 的二级分类）：`handleViewSpu(id)` → `router.push({name: 'ProductSpu', query: {categoryId: id}})` 跳转 SPU 列表
   - **删除**：同品牌流程
3. 表单：CategoryForm.open 时还会异步加载 `getCategoryList({parentId: 0})` 供"上级分类"下拉

**跨入口调用**：`handleViewSpu` 触发 router push 到 ProductSpu 入口（spu/index.vue），将分类 ID 作为 query 参数传递。

**source_nodes**：`component:f69675a209b3865375b7e42f60c85466`、`component:415071f3d41649815096ab65c9a139f3`、`function:983b518f40fda455789509dcb62ad231`、`function:9d410f93c4e3fb316471e4011bc33645`、`function:9d410f93c4e3fb316471e4011bc33645`（注意 handleViewSpu 与 openForm 同名不同实现）

---

## 流程 3：商品属性与属性值两级管理流程

**触发条件**：用户在"商品属性"列表点击行内"属性值"按钮

**输入**：属性项 ID（路由参数 propertyId）；属性值名称

**输出**：属性值分页列表

**主路径**：

1. 列表页 (`property/index.vue`)：
   - 搜索属性名称 + 创建时间
   - 行内"属性值"按钮：`goValueList(id)` → `router.push({name: 'ProductPropertyValue', params: {propertyId: id}})`
2. 跳转属性值列表页 (`property/value/index.vue`)：
   - `queryParams.propertyId = params.propertyId`（禁用下拉显示）
   - `onMounted` → `getList` → `getPropertyValuePage` + `getProperty(propertyId)` 加载属性项名称
   - 行操作：编辑 `openForm('update', propertyId, id)` / 删除 `deletePropertyValue`
3. 跨页面数据流：propertyId 通过路由 params 传递，子页面通过 `getProperty(propertyId)` 单独加载属性项元信息

**source_nodes**：`component:30b1c90977ec826a83ee2f1777895b5d`、`component:43ccda97fa2906aab36871cb7fca6606`、`component:3c079e65e1a015eb488ef8ed4aebf6b6`、`component:1ba7f951ac58dcb25c3f4a691ecd155e`

---

## 流程 4：商品评价管理流程

**触发条件**：用户进入"商品评价"页

**输入**：回复状态/商品名称/用户名称/订单编号/评论时间多条件过滤

**输出**：评价列表；回复弹窗；显示/隐藏切换

**主路径**：

1. 列表查询 `getCommentPage`
2. **回复**：行内"回复"按钮 → `handleReply(id)` → `replyFormRef.value.open(id)` → ReplyForm 显示 → `replyComment({id, replyContent})` → emit('success') → refresh
3. **显示/隐藏 Switch**：`handleVisibleChange(row)`：
   - 二次确认 `message.confirm(visible ? '是否显示评论？' : '是否隐藏评论？')`
   - 确认 → `updateCommentVisible({id, visible})` → refresh
   - 取消 → `row.visible = !changedValue` 回滚 UI
4. **添加虚拟评论**：CommentForm 弹窗，spuId 必填，skuId 联动 SpuShowcase + SkuTableSelect

**source_nodes**：`component:8b462134c11b251f030d72d983c2b803`、`component:c60546efcc9ed0b1d4527bb2ee73663d`、`component:e3823f838eb365c67f0bd5c29489bd91`、`function:983b518f40fda455789509dcb62ad231`（handleVisibleChange）

---

## 流程 5：SPU 5-Tab 状态流转

**触发条件**：用户进入"商品列表"页（SPU）

**输入**：搜索条件；Tab 切换；上架/下架/回收/恢复/删除操作

**输出**：SPU 列表 + 5 个 Tab 计数 + 状态变更

**主路径**：

1. `onMounted` 并行加载：
   - `getTabsCount()`：5 个 Tab 的商品数
   - `getSpuPage(queryParams)`：当前 Tab 的列表
   - `getCategoryList({})` + `handleTree`：分类树（用于 `formatCategoryName`）
2. Tab 切换：`handleTabClick(tab)` → `queryParams.tabType = tab.paneName` → `getList`
3. **上架/下架 Switch**：`handleStatusChange(row)`：
   - 二次确认 → `updateStatus({id, status})` → `getTabsCount()` + `getList()` 刷新
   - 异常：回滚 `row.status` 到原值
4. **回收/恢复**：`handleStatus02Change(row, newStatus)`：
   - newStatus = RECYCLE → 移入回收站 Tab（tabType=4）
   - newStatus = DISABLE → 从回收站恢复到仓库（tabType=1）
5. **详情/修改**：路由 push 到 ProductSpuDetail / ProductSpuEdit
6. **导出**：`exportSpu(queryParams)` → `download.excel(data, '商品列表.xls')`

**失败路径**：
- 状态变更异常：自动回滚 row.status
- 用户取消二次确认：不调 API

**source_nodes**：`component:766a92ffa67135de01a5219da9b57bf5`、`function:983b518f40fda455789509dcb62ad231`（handleStatusChange）、`function:983b518f40fda455789509dcb62ad231`（handleStatus02Change）、`function:983b518f40fda455789509dcb62ad231`（handleDelete）、`function:983b518f40fda455789509dcb62ad231`（handleExport，源码中实际为 handleExport 独立函数，但 codegraph 节点按 name 聚合）、`function:fca2bb4e4ef22a93a183643ecf5f888d`（handleQuery）、`function:1b470cebdbb9c91f91e22c51714b130d`（resetQuery）、`function:9d410f93c4e3fb316471e4011bc33645`（openForm，路由跳转版）

---

## 流程 6：SPU 5-Tab 复合表单提交流程

**触发条件**：用户进入"商品发布"或"修改商品"页（ProductSpuAdd/ProductSpuEdit/ProductSpuDetail）

**输入**：5 个 Tab 表单（基础/价格库存/物流/详情/其他）

**输出**：createSpu / updateSpu；跳转回列表

**主路径**：

1. 路由判断：name === 'ProductSpuDetail' → `isDetail.value = true`
2. 若 `params.id`：调 `getSpu(id)` 加载完整 SPU 数据
3. 价格分↔元换算：
   - 详情模式：`floatToFixed2`（保留 2 位小数）
   - 编辑模式：`formatToFraction`（分转元）
4. 用户编辑各 Tab 子表单
5. 点击"保存"：
   - `submitForm` 依次 `validate` 5 个子表单（info/sku/delivery/description/other）
   - 任一校验失败：emit('update:activeName', 该 Tab)，抛错截断后续校验
   - 全部通过：`cloneDeep` 深拷贝，处理 SKU 命名（用 SPU.name）、价格元转分、轮播图 URL 化
   - 调 `createSpu`（无 id）/ `updateSpu`（有 id）
   - 关闭当前 tab 并 push 列表
6. 点击"返回"：`delView(currentRoute)` + `push({name: 'ProductSpu'})`

**跨子组件交互**：
- SkuForm → ProductAttributes（多规格属性勾选） → @success 触发 skuListRef.generateTableData 重新生成 SKU 列表
- SkuForm → ProductPropertyAddForm（添加新属性弹窗）
- InfoForm → ProductCategorySelect（分类级联）

**source_nodes**：`component:be487fd29883255ef6a9971a99c1c589`、`component:68c9a279711c588e6c9f52ac19abc2be`、`component:18a9cdf6cda3537530eea5e2dc6b6492`、`component:13391245e64d2417c7bc0b6da4718a78`、`component:7a6ff01634b3ee0a30a8a774844408c9`、`component:70c3384cad029eedab3d527723aaebb4`、`component:61fc70313ceef9ac0c4570dba9d9ac51`、`component:58b810df1c6c35195b854d4b8e616178`

---

## 流程 7：跨子域 SPU/SKU 组件复用

**触发源**：CommentForm、SpuForm

**复用路径**：
- `SpuShowcase`：下拉中显示 SPU 缩略图 + 名称 → 选择后回传 ID
- `SpuTableSelect`：弹窗中按条件选择 SPU → 回传 Spu 对象
- `SkuTableSelect`：弹窗中按 spuId 过滤 SKU → 回传 Sku 对象
- `ProductCategorySelect`：树形单/多选分类

**source_nodes**：`component:e66cfb70aadcb8c836cba8d713911f31`、`component:8aa636172dabd1f67e1b4ded472f0715`、`component:71096ed3227a58af4005a43b8f2b0711`、`component:90f10902441d322000edf9da69f88370`
