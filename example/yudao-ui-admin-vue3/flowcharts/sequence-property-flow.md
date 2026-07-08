# 序列图 F3：属性/属性值两级管理

入口：property/index.vue + property/value/index.vue
source_nodes：component:30b1c90977ec826a83ee2f1777895b5d, component:43ccda97fa2906aab36871cb7fca6606, component:3c079e65e1a015eb488ef8ed4aebf6b6, component:1ba7f951ac58dcb25c3f4a691ecd155e

```mermaid
sequenceDiagram
  actor User
  participant PList as 属性项列表
  participant PForm as 属性项表单
  participant Router as vue-router
  participant VList as 属性值列表
  participant VForm as 属性值表单
  participant API as PropertyApi

  User->>PList: 进入属性项页
  PList->>API: getPropertyPage(queryParams)
  API-->>PList: {list, total}
  PList->>User: 渲染表格

  User->>PList: 点击行内"属性值"
  PList->>Router: push({name: 'ProductPropertyValue', params: {propertyId: id}})
  Router->>VList: 路由切换
  VList->>VList: queryParams.propertyId = params.propertyId
  VList->>API: getPropertyValuePage(queryParams)
  API-->>VList: {list, total}
  VList->>API: getProperty(propertyId)
  API-->>VList: PropertyVO
  VList->>User: 渲染属性值表格（含属性项名称下拉）

  User->>VList: 点击"新增"
  VList->>VForm: formRef.value.open('create', propertyId)
  VForm-->>User: 弹出 Dialog
  User->>VForm: 填写名称、备注、确定
  VForm->>API: createPropertyValue(data)
  API-->>VForm: success
  VForm-->>VList: emit('success')
  VList->>VList: getList()
```

**关键设计**：propertyId 通过路由 params 传递，子页面通过 `getProperty(propertyId)` 单独加载属性项元信息（不会重复从父页传值）。
