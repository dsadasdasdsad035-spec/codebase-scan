# 主业务流图：商城商品中心

入口：frontend-mall-product
证据：./entries/frontend-mall-product/business-flows.md

---

## 整体导航关系

```mermaid
graph LR
  BrandList[品牌列表]
  BrandForm[品牌表单]
  CategoryList[分类列表]
  CategoryForm[分类表单]
  PropertyList[属性项列表]
  PropertyForm[属性项表单]
  ValueList[属性值列表]
  ValueForm[属性值表单]
  CommentList[评价列表]
  CommentForm[添加评论表单]
  ReplyForm[回复评论表单]
  SpuList[SPU 列表]
  SpuForm[SPU 复合表单]
  InfoForm[基础设置]
  SkuForm[价格库存]
  DeliveryForm[物流设置]
  DescriptionForm[商品详情]
  OtherForm[其它设置]
  SpuShowcase[SPU 展示选择]
  SpuTableSelect[SPU 表格选择]
  SkuTableSelect[SKU 表格选择]
  ProductCategorySelect[分类选择器]
  SkuList[SKU 列表]
  ProductAttributes[多规格属性]
  PropertyAddForm[添加属性表单]

  BrandList -->|openForm| BrandForm
  CategoryList -->|openForm| CategoryForm
  CategoryList -->|查看商品| SpuList
  PropertyList -->|openForm| PropertyForm
  PropertyList -->|属性值| ValueList
  ValueList -->|openForm| ValueForm
  CommentList -->|添加虚拟评论| CommentForm
  CommentList -->|回复| ReplyForm
  SpuList -->|新增/修改| SpuForm
  SpuForm --> InfoForm
  SpuForm --> SkuForm
  SpuForm --> DeliveryForm
  SpuForm --> DescriptionForm
  SpuForm --> OtherForm
  SkuForm --> SkuList
  SkuForm --> ProductAttributes
  SkuForm --> PropertyAddForm
  CommentForm -.-> SpuShowcase
  CommentForm -.-> SkuTableSelect
  SpuForm -.-> ProductCategorySelect
  SpuForm -.-> SpuShowcase
  SpuForm -.-> SpuTableSelect
  SpuForm -.-> SkuTableSelect
```

---

## 5 子域聚合

```mermaid
graph TD
  Start([用户进入商品中心])
  Start --> Brand[品牌管理]
  Start --> Category[分类管理]
  Start --> Property[属性管理]
  Start --> Comment[评价管理]
  Start --> Spu[SPU 管理]

  Brand --> BrandCRUD[CRUD + 搜索]
  Category --> CategoryCRUD[CRUD + 树形 + 查看商品]
  Property --> PropertyCRUD[属性项 CRUD] --> ValueCRUD[属性值 CRUD]
  Comment --> CommentCRUD[多条件搜索 + 虚拟评论 + 回复 + 显隐]
  Spu --> SpuListFlow[5 Tab 状态流转]
  Spu --> SpuFormFlow[5 子表单复合发布]
  SpuListFlow --> SpuFormFlow
```

---

## 关键流程编号

| 编号 | 流程 | 详细 |
|---|---|---|
| F1 | 品牌 CRUD | sequence-brand-crud.md |
| F2 | 分类树 CRUD | sequence-category-crud.md |
| F3 | 属性/属性值两级 | sequence-property-flow.md |
| F4 | 评价多条件管理 | sequence-comment-flow.md |
| F5 | SPU 5-Tab 状态流转 | sequence-spu-status.md |
| F6 | SPU 5-Tab 复合表单 | sequence-spu-form.md |
| F7 | 跨子域组件复用 | sequence-component-reuse.md |
