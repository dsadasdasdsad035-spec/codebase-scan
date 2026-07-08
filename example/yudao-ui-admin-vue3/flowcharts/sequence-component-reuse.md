# 序列图 F7：跨子域 SPU/SKU 组件复用

入口：spu/components/* + comment/CommentForm.vue
source_nodes：component:e66cfb70aadcb8c836cba8d713911f31, component:8aa636172dabd1f67e1b4ded472f0715, component:71096ed3227a58af4005a43b8f2b0711, component:90f10902441d322000edf9da69f88370

```mermaid
sequenceDiagram
  participant SpuForm as spu/form/index.vue
  participant InfoF as InfoForm
  participant SkuF as SkuForm
  participant CForm as comment/CommentForm
  participant CatSel as ProductCategorySelect
  participant SpuShow as SpuShowcase
  participant SpuTbl as SpuTableSelect
  participant SkuTbl as SkuTableSelect
  participant API as ProductCategoryApi / ProductSpuApi

  rect rgb(240,248,255)
    Note over SpuForm,SkuTbl: 复合表单中的复用路径
    SpuForm->>InfoF: 渲染基础设置 Tab
    InfoF->>CatSel: 使用分类级联
    CatSel->>API: getCategoryList({parentId})
    API-->>CatSel: CategoryVO[]
    CatSel-->>InfoF: 选中 categoryId

    SpuForm->>SkuF: 渲染价格库存 Tab
    SkuF->>SpuShow: 表单内选 SPU（备用）
    SpuShow->>API: 加载 SPU 列表
    API-->>SpuShow: Spu[]
    SpuShow-->>SkuF: 选中的 spuId

    SpuForm->>SpuTbl: 备用（页面级 SPU 选择）
    SpuTbl-->>SpuForm: 选中的 Spu

    SpuForm->>SkuTbl: 备用（页面级 SKU 选择）
    SkuTbl->>API: getSpuPage / getSkuList(spuId)
    API-->>SkuTbl: 列表
    SkuTbl-->>SpuForm: 选中的 Sku
  end

  rect rgb(255,250,240)
    Note over CForm,API: 评价表单中的复用路径
    CForm->>SpuShow: 选择 SPU
    SpuShow-->>CForm: 选中的 spuId
    CForm->>CForm: 显示 SKU 选择按钮
    CForm->>SkuTbl: skuTableSelectRef.value.open()
    SkuTbl->>API: 加载该 spuId 下的 SKU
    API-->>SkuTbl: Sku[]
    SkuTbl-->>CForm: 选中的 Sku
    CForm->>CForm: formData.skuId = sku.id + skuData 更新
  end
```
