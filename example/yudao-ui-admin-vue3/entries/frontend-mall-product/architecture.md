# 架构说明：商城商品中心前端 (frontend-mall-product)

入口 ID：frontend-mall-product
入口类型：frontend_root
源码范围：yudao-ui-admin-vue3/src/views/mall/product/**
证据：evidence/frontend-mall-product/{inventory,nodes,typecards}.json

---

## 1. 入口职责 (Responsibilities)

本入口为商城商品中心后台管理前端，承接后端 `yudao-module-mall/yudao-module-product` 模块。共 5 大子域、25 个独立 page 级组件 + 5 个 spu 子表单 + 1 个 components 入口。

| 子域 | 路径前缀 | 核心 page | 职责 |
|---|---|---|---|
| 商品品牌 | `brand/` | index, BrandForm | 品牌 CRUD，关联 SPU |
| 商品分类 | `category/` | index, CategoryForm, components/ProductCategorySelect | 分类树 CRUD，分类选择器复用 |
| 商品属性 | `property/`, `property/value/` | index, PropertyForm, value/index, ValueForm | 属性项与属性值两级管理 |
| 商品评价 | `comment/` | index, CommentForm, ReplyForm | 评价查询、虚拟评价创建、回复 |
| SPU/SKU | `spu/`, `spu/form/`, `spu/components/` | index, form/index, 6 form 子组件, 4 components | 商品发布、5 Tab 复合表单、SKU 编辑器、SPU/SKU 选择器 |

## 2. 依赖 (Dependencies)

### 2.1 内部业务 API（src/api/mall/product/*）
- `@/api/mall/product/brand` → ProductBrandApi (getBrandParam/getBrand/createBrand/updateBrand/deleteBrand)
- `@/api/mall/product/category` → ProductCategoryApi (getCategoryList/getCategory/createCategory/updateCategory/deleteCategory)
- `@/api/mall/product/property` → PropertyApi (getPropertyPage/getPropertyValuePage/getProperty/getPropertyValue/createProperty/updateProperty/createPropertyValue/updatePropertyValue/deleteProperty/deletePropertyValue)
- `@/api/mall/product/comment` → CommentApi (getCommentPage/getComment/createComment/replyComment/updateCommentVisible)
- `@/api/mall/product/spu` → ProductSpuApi (getSpu/getSpuPage/createSpu/updateSpu/updateStatus/deleteSpu/exportSpu/getTabsCount/Sku/Spu 类型)

### 2.2 框架与公共组件
- Element Plus（el-form / el-table / el-dialog / el-tree-select / el-cascader / el-switch / el-rate / el-image / el-tabs）
- 项目内部：
  - `@/utils/dict` (DICT_TYPE, getIntDictOptions)
  - `@/utils/formatTime` (dateFormatter)
  - `@/utils/tree` (defaultProps, handleTree, treeToString)
  - `@/utils/constants` (CommonStatusEnum, ProductSpuStatusEnum)
  - `@/utils` (fenToYuan, formatToFraction, floatToFixed2, convertToInteger, copyValueToTarget)
  - `@/utils/download` (Excel 下载)
  - `@/utils/propTypes`, `@/utils/is`
  - `@/components/ImageViewer` (createImageViewer)
  - `@/components/UploadImg`, `UploadImgs`
  - `@/components/ContentWrap`, `Pagination`, `Dialog`, `Icon`
  - `@/store/modules/tagsView` (delView)
  - `vue-types` (oneOfType)
  - `lodash-es` (cloneDeep)
  - `@element-plus/icons-vue` (RefreshRight)

### 2.3 跨入口依赖
- `@/views/erp/...` 等其他业务模块的复用（如无直接证据）
- 路由：`ProductBrand`、`ProductCategory`、`ProductProperty`、`ProductPropertyValue`、`ProductComment`、`ProductSpu`、`ProductSpuAdd`、`ProductSpuEdit`、`ProductSpuDetail`

## 3. 集成状态 (Integration Status)

**已接入主流程**：本入口全部 25 个 page + 5 个 spu 子表单 + 1 个 components 入口均已通过：
- components 入口被 SkuForm 通过 `import { SkuList, getPropertyList, RuleConfig, PropertyAndValues } from '@/views/mall/product/spu/components/index'` 显式引用
- 所有 form 弹窗通过 ref 暴露 `open` 方法供列表页调用
- 所有 form 提交成功后 emit('success') 通知列表页 refresh
- CommentForm 与 SPU 域通过 `SpuShowcase` / `SkuTableSelect` 跨子域组件复用

**独立子模块**：4 个 spu components（SkuList / SkuTableSelect / SpuShowcase / SpuTableSelect）均为封装通用 UI 控件，可被其他业务模块复用。

**外部依赖**：后端 yudao-module-product 提供的 9 类业务 API；Element Plus 组件库；项目内基础工具与公共组件。

## 4. source_nodes 追溯

- 25 个 page 组件节点 ID（详见 ui-layout.md）
- 5 个 spu 子表单节点 ID
- 4 个 spu 公共组件节点 ID
- 1 个 spu/components/index.ts 模块入口节点
- 共 141 个可分析符号，覆盖 5 大子域
