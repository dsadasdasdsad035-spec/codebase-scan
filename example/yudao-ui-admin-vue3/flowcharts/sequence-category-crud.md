# 序列图 F2：分类树 CRUD（含跨入口跳转）

入口：category/index.vue + category/CategoryForm.vue
source_nodes：component:f69675a209b3865375b7e42f60c85466, component:415071f3d41649815096ab65c9a139f3

```mermaid
sequenceDiagram
  actor User
  participant List as 分类列表
  participant Form as 分类表单
  participant Router as vue-router
  participant SpuList as SPU 列表（spu/index.vue）
  participant API as ProductCategoryApi

  User->>List: 进入分类页
  List->>API: getCategoryList(queryParams)
  API-->>List: CategoryVO[]
  List->>List: handleTree(data, 'id', 'parentId')
  List->>User: 渲染树形表格

  User->>List: 点击"新增"
  List->>Form: formRef.value.open('create')
  Form->>API: getCategoryList({parentId: 0})
  API-->>Form: 顶级分类列表
  Form-->>User: 弹出 Dialog（含上级分类下拉）
  User->>Form: 选择上级分类、填写信息、确定
  Form->>API: createCategory(data)
  API-->>Form: success
  Form-->>List: emit('success')
  List->>List: getList()

  User->>List: 点击二级分类"查看商品"
  List->>Router: router.push({name: 'ProductSpu', query: {categoryId: id}})
  Router->>SpuList: 路由切换
  SpuList->>SpuList: onMounted 读 route.query.categoryId
  SpuList->>SpuList: 加载该分类下的 SPU
  SpuList->>User: 展示 SPU 列表

  User->>List: 点击行内"编辑"
  List->>Form: formRef.value.open('update', id)
  Form->>API: getCategory(id)
  API-->>Form: 回显数据
  Form-->>User: 弹出 Dialog
  User->>Form: 修改、确定
  Form->>API: updateCategory(data)
  API-->>Form: success
  Form-->>List: emit('success')
  List->>List: getList()

  User->>List: 点击行内"删除"
  List->>List: message.delConfirm()
  alt 确认
    List->>API: deleteCategory(id)
    API-->>List: success
    List->>List: getList()
  end
```

**关键跨入口调用**：`handleViewSpu` 触发 router.push 到 `ProductSpu`，将 `categoryId` 作为 query 传递。
