# 软件需求规格说明书（SRS）：商城商品中心后台

技术无关文档，基于业务能力生成 FR/NFR。
入口：frontend-mall-product（单入口）
证据：./entries/frontend-mall-product/* + ./architecture.md + ./business-flows.md + ./data-model.md

---

## 1. 范围

本系统为商城"商品中心"提供后台管理能力，覆盖品牌、分类、属性与属性值、评价、商品（含规格）五大子域，支持商品从发布到上架/下架/回收的全生命周期管理。

## 2. 业务能力清单

| # | 业务能力 | 入口追溯 |
|---|---|---|
| 1 | 品牌管理 | frontend-mall-product |
| 2 | 商品分类管理 | frontend-mall-product |
| 3 | 商品属性管理 | frontend-mall-product |
| 4 | 商品评价管理 | frontend-mall-product |
| 5 | 商品生命周期管理 | frontend-mall-product |

## 3. 功能需求（Functional Requirements）

### FR-1 品牌管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-1.1 | 系统应支持按品牌名称、状态、创建时间区间查询品牌列表 | brand/index.vue（搜索栏 + 列表） |
| FR-1.2 | 系统应支持新增品牌，必填项为品牌名称、品牌图片、品牌排序 | brand/BrandForm.vue |
| FR-1.3 | 系统应支持编辑品牌（按 ID 加载数据后修改） | brand/BrandForm.vue（type=update） |
| FR-1.4 | 系统应支持删除品牌（需二次确认） | brand/index.vue（handleDelete） |
| FR-1.5 | 品牌列表应展示品牌名称、图片、排序、状态、创建时间 | brand/index.vue（el-table 列） |
| FR-1.6 | 品牌状态应支持启用/禁用二值切换 | brand/BrandForm.vue（CommonStatusEnum） |

### FR-2 商品分类管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-2.1 | 系统应支持按分类名称查询分类树 | category/index.vue |
| FR-2.2 | 系统应支持新增/编辑/删除分类（必填项：上级分类、名称、图片、排序、状态） | category/CategoryForm.vue |
| FR-2.3 | 分类应支持自引用树形结构（顶级分类 parentId=0） | CategoryVO.parentId |
| FR-2.4 | 列表应展示分类名称、图标、排序、状态、创建时间 | category/index.vue（el-table 列） |
| FR-2.5 | 二级分类应支持"查看商品"快捷入口，跳转到该分类下的商品列表 | category/index.vue（handleViewSpu） |

### FR-3 商品属性管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-3.1 | 系统应支持按名称、创建时间查询属性项列表 | property/index.vue |
| FR-3.2 | 系统应支持属性项的增删改（必填项：名称） | property/PropertyForm.vue |
| FR-3.3 | 系统应支持按属性项 ID 查询其下的属性值列表 | property/value/index.vue（route.params.propertyId） |
| FR-3.4 | 系统应支持属性值的增删改（必填项：名称、所属属性项） | property/value/ValueForm.vue |
| FR-3.5 | 属性项列表应展示编号、属性名称、备注、创建时间 | property/index.vue |
| FR-3.6 | 属性值列表应展示编号、属性值名称、备注、创建时间 | property/value/index.vue |

### FR-4 商品评价管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-4.1 | 系统应支持按回复状态、商品名称、用户名称、订单编号、评论时间多条件查询评价 | comment/index.vue（搜索栏） |
| FR-4.2 | 系统应支持添加虚拟评价（必填项：商品、规格、用户头像、用户名称、评论内容、描述星级、服务星级） | comment/CommentForm.vue |
| FR-4.3 | 系统应支持回复评价 | comment/ReplyForm.vue |
| FR-4.4 | 系统应支持评价的显示/隐藏切换（需二次确认） | comment/index.vue（handleVisibleChange） |
| FR-4.5 | 评价列表应展示评论编号、商品信息（封面+名称+规格 tag）、用户名称、商品评分、服务评分、评论内容（含图片）、回复内容、评论时间、是否展示 | comment/index.vue |
| FR-4.6 | 描述星级和服务星级应支持 0-5 星评分 | comment/CommentForm.vue（el-rate） |

### FR-5 商品生命周期管理

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-5.1 | 系统应支持按商品名称、分类级联、创建时间查询商品列表 | spu/index.vue |
| FR-5.2 | 商品列表应提供 5 个状态分组：出售中、仓库中、已售罄、警戒库存、回收站 | spu/index.vue（tabsData） |
| FR-5.3 | 系统应支持商品的上架/下架切换（仅在非回收站状态下可切换） | spu/index.vue（handleStatusChange） |
| FR-5.4 | 系统应支持商品移入回收站（仅非回收站状态下可操作） | spu/index.vue（handleStatus02Change, status=-1） |
| FR-5.5 | 系统应支持从回收站恢复到仓库 | spu/index.vue（handleStatus02Change, status=0） |
| FR-5.6 | 系统应支持商品的物理删除（仅在回收站状态下可操作） | spu/index.vue（handleDelete, tabType=4） |
| FR-5.7 | 系统应支持商品列表的 Excel 导出 | spu/index.vue（handleExport） |
| FR-5.8 | 商品列表应展示编号、商品信息（封面+名称）、价格、销量、库存、排序、销售状态、创建时间 | spu/index.vue（el-table 列） |

### FR-6 商品发布与编辑（复合表单）

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-6.1 | 系统应提供商品发布/编辑的复合表单，包含 5 个分组：基础设置、价格库存、物流设置、商品详情、其它设置 | spu/form/index.vue（el-tabs） |
| FR-6.2 | 基础设置应包含：商品名称、商品分类级联、商品品牌、商品关键字、商品简介、封面图、轮播图 | spu/form/InfoForm.vue |
| FR-6.3 | 价格库存应支持单规格/多规格切换 | spu/form/SkuForm.vue（specType） |
| FR-6.4 | 多规格时应支持添加商品属性（propertyId+valueId 组合）并自动生成 SKU 列表 | spu/form/ProductAttributes.vue + spu/form/SkuList.vue |
| FR-6.5 | 单规格时 SKU 应至少包含：名称、价格、市场价、成本价、条码、图片、库存、重量、体积、一级/二级分销佣金 | spu/form/SkuList.vue |
| FR-6.6 | 物流设置应包含：配送方式、运费模板、重量、体积 | spu/form/DeliveryForm.vue |
| FR-6.7 | 商品详情应支持富文本编辑 | spu/form/DescriptionForm.vue |
| FR-6.8 | 其它设置应包含：排序、赠送积分、虚拟销量 | spu/form/OtherForm.vue |
| FR-6.9 | 复合表单应按顺序校验所有分组，校验失败时自动定位到对应分组 | spu/form/index.vue（submitForm） |
| FR-6.10 | 价格字段应支持元↔分的换算 | spu/form/index.vue（formatToFraction / floatToFixed2 / convertToInteger） |
| FR-6.11 | SKU 名称应在提交时自动回填为 SPU 名称 | spu/form/index.vue（submitForm） |
| FR-6.12 | 详情模式下所有分组应只读 | spu/form/index.vue（isDetail）+ 各 form :disabled="isDetail" |
| FR-6.13 | 轮播图应支持前端的本地图选择对象，提交时转换为 URL 字符串 | spu/form/index.vue（submitForm sliderPicUrls 处理） |

### FR-7 跨子域组件复用

| ID | 需求描述 | 追溯 |
|---|---|---|
| FR-7.1 | 系统应提供商品分类树形选择器，支持单选/多选 | category/components/ProductCategorySelect.vue |
| FR-7.2 | 系统应提供 SPU 下拉展示选择器（封面+名称） | spu/components/SpuShowcase.vue |
| FR-7.3 | 系统应提供 SPU 弹窗表格选择器 | spu/components/SpuTableSelect.vue |
| FR-7.4 | 系统应提供 SKU 弹窗表格选择器（按 spuId 过滤） | spu/components/SkuTableSelect.vue |

## 4. 非功能需求（Non-Functional Requirements）

### NFR-1 权限控制

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-1.1 | 系统应实施基于资源-操作的细粒度权限控制 | 所有 index.vue + form.vue（v-hasPermi） |
| NFR-1.2 | 资源类型应包括：brand / category / property / comment / spu | 权限点前缀 product: |
| NFR-1.3 | 操作类型应至少包括：create / update / delete / export / query | 同上 |

### NFR-2 数据完整性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-2.1 | 所有必填字段应在表单层强制校验 | 所有 *Form.vue formRules |
| NFR-2.2 | 危险操作（删除/状态变更/导出）应实施二次确认 | 所有 index.vue（message.delConfirm/confirm/exportConfirm） |
| NFR-2.3 | 状态变更异常时 UI 应自动回滚到原值 | spu/index.vue（handleStatusChange catch）、comment/index.vue（handleVisibleChange catch） |
| NFR-2.4 | 加载中状态应拒绝用户重复触发操作 | comment/index.vue（handleVisibleChange loading 保护） |
| NFR-2.5 | 复合表单的分组校验失败时应自动定位到失败分组 | spu/form/index.vue（emit update:activeName） |

### NFR-3 价格精度

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-3.1 | 销售价/市场价/成本价应 >= 0.01 元 | spu/form/SkuForm.vue ruleConfig |
| NFR-3.2 | 库存应 >= 1 | spu/form/SkuForm.vue ruleConfig |
| NFR-3.3 | 价格在存储层应使用整数分，在 UI 层应自动进行分↔元换算 | spu/form/index.vue + spu/index.vue（fenToYuan） |

### NFR-4 可观测性

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-4.1 | 列表加载、表单提交应有 loading 状态指示 | 所有 *.vue 的 loading / formLoading ref |
| NFR-4.2 | 异步操作（删除/状态变更/提交）应提供用户可见的成功/失败消息 | 所有 *.vue（message.success/error） |
| NFR-4.3 | 错误应通过统一的应用层拦截器呈现，业务代码不应直接处理网络错误 | utils/request（隐含） |

### NFR-5 路由与导航

| ID | 需求描述 | 追溯 |
|---|---|---|
| NFR-5.1 | 系统应支持通过 URL 参数在不同子域之间跳转 | 路由 ProductSpu、ProductPropertyValue、ProductSpuAdd/Edit/Detail |
| NFR-5.2 | 跨子域导航应携带必要的查询参数（如分类 ID） | spu/index.vue（route.query.categoryId） |
| NFR-5.3 | 复合表单的关闭应同时移除对应路由 tag | spu/form/index.vue（delView + push） |

## 5. 追溯（Traceability）

| FR 簇 | 业务能力 | source_nodes |
|---|---|---|
| FR-1 | 品牌管理 | component:fbb82fd3451fdc7dcf941a9a95fdcb8b, component:2d7a9b88d8c6b9d362467ec2d6beba0e, function:a7b69ca7e3cedfd7627766502b02168b, function:983b518f40fda455789509dcb62ad231, function:1b470cebdbb9c91f91e22c51714b130d, function:9d410f93c4e3fb316471e4011bc33645, function:fca2bb4e4ef22a93a183643ecf5f888d |
| FR-2 | 分类管理 | component:f69675a209b3865375b7e42f60c85466, component:415071f3d41649815096ab65c9a139f3, component:90f10902441d322000edf9da69f88370 |
| FR-3 | 属性管理 | component:30b1c90977ec826a83ee2f1777895b5d, component:43ccda97fa2906aab36871cb7fca6606, component:3c079e65e1a015eb488ef8ed4aebf6b6, component:1ba7f951ac58dcb25c3f4a691ecd155e |
| FR-4 | 评价管理 | component:8b462134c11b251f030d72d983c2b803, component:c60546efcc9ed0b1d4527bb2ee73663d, component:e3823f838eb365c67f0bd5c29489bd91, function:983b518f40fda455789509dcb62ad231 |
| FR-5 | SPU 生命周期 | component:766a92ffa67135de01a5219da9b57bf5, function:983b518f40fda455789509dcb62ad231（handleStatusChange/02Change/handleDelete/handleExport）, function:9d410f93c4e3fb316471e4011bc33645（openForm 路由跳转） |
| FR-6 | SPU 复合表单 | component:be487fd29883255ef6a9971a99c1c589, component:68c9a279711c588e6c9f52ac19abc2be, component:18a9cdf6cda3537530eea5e2dc6b6492, component:13391245e64d2417c7bc0b6da4718a78, component:7a6ff01634b3ee0a30a8a774844408c9, component:70c3384cad029eedab3d527723aaebb4, component:61fc70313ceef9ac0c4570dba9d9ac51, component:58b810df1c6c35195b854d4b8e616178 |
| FR-7 | 跨子域组件 | component:e66cfb70aadcb8c836cba8d713911f31, component:8aa636172dabd1f67e1b4ded472f0715, component:71096ed3227a58af4005a43b8f2b0711, component:90f10902441d322000edf9da69f88370 |
