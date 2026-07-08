# 序列图 F1：品牌 CRUD

入口：brand/index.vue + brand/BrandForm.vue
源文件：src/views/mall/product/brand/index.vue, src/views/mall/product/brand/BrandForm.vue
source_nodes：component:fbb82fd3451fdc7dcf941a9a95fdcb8b, component:2d7a9b88d8c6b9d362467ec2d6beba0e

```mermaid
sequenceDiagram
  actor User
  participant List as 品牌列表<br/>(index.vue)
  participant Form as 品牌表单<br/>(BrandForm.vue)
  participant API as ProductBrandApi
  participant Server as 后端

  User->>List: 进入页面
  List->>List: onMounted
  List->>API: getBrandParam(queryParams)
  API->>Server: GET /product/brand/page
  Server-->>API: {list, total}
  API-->>List: data
  List->>User: 渲染表格 + 分页

  User->>List: 点击"新增"
  List->>Form: formRef.value.open('create')
  Form->>Form: resetForm()
  Form-->>User: 弹出 Dialog
  User->>Form: 填写表单
  User->>Form: 点击"确定"
  Form->>Form: formRef.value.validate()
  alt 校验通过
    Form->>API: createBrand(data)
    API->>Server: POST /product/brand/create
    Server-->>API: 200
    API-->>Form: success
    Form->>User: message.success('创建成功')
    Form-->>List: emit('success')
    Form->>Form: dialogVisible = false
    List->>List: getList()
  else 校验失败
    Form->>User: 内联错误提示
  end

  User->>List: 点击行内"编辑"
  List->>Form: formRef.value.open('update', id)
  Form->>API: getBrand(id)
  API->>Server: GET /product/brand/get
  Server-->>API: BrandVO
  API-->>Form: formData
  Form-->>User: 弹出 Dialog（回显）
  User->>Form: 修改后"确定"
  Form->>API: updateBrand(data)
  API->>Server: PUT /product/brand/update
  Server-->>API: 200
  API-->>Form: success
  Form-->>List: emit('success')
  List->>List: getList()

  User->>List: 点击行内"删除"
  List->>List: message.delConfirm()
  alt 二次确认通过
    List->>API: deleteBrand(id)
    API->>Server: DELETE /product/brand/delete
    Server-->>API: 200
    API-->>List: success
    List->>User: message.success('删除成功')
    List->>List: getList()
  else 用户取消
    List->>List: catch {} 静默
  end
```
