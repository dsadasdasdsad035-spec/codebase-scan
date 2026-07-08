# 序列图 F4：评价多条件管理

入口：comment/index.vue + CommentForm.vue + ReplyForm.vue
source_nodes：component:8b462134c11b251f030d72d983c2b803, component:c60546efcc9ed0b1d4527bb2ee73663d, component:e3823f838eb365c67f0bd5c29489bd91

```mermaid
sequenceDiagram
  actor User
  participant List as 评价列表
  participant CForm as 添加虚拟评论
  participant RForm as 回复评论
  participant SpuShow as SPU 展示选择
  participant SkuTbl as SKU 表格选择
  participant API as CommentApi

  User->>List: 进入评价页
  List->>API: getCommentPage(queryParams)
  API-->>List: {list, total}
  List->>List: visible == null 兜底为 false
  List->>User: 渲染表格

  User->>List: 点击"添加虚拟评论"
  List->>CForm: formRef.value.open('create')
  CForm-->>User: 弹出 Dialog
  User->>CForm: 点击 Spu 展示选择 → SpuShowcase
  SpuShow-->>CForm: 选中的 spuId
  CForm->>CForm: 显示 SKU 规格选择按钮
  User->>CForm: 点击 SKU 框 → SkuTableSelect.open()
  SkuTbl->>API: 加载该 spuId 下的 SKU
  SkuTbl-->>CForm: 选中的 sku
  CForm->>CForm: 记录 skuId + skuData
  User->>CForm: 填写头像/昵称/内容/星级/图片、确定
  CForm->>API: createComment(data)
  API-->>CForm: success
  CForm-->>List: emit('success')
  List->>List: getList()

  User->>List: 点击行内"回复"
  List->>RForm: replyFormRef.value.open(id)
  RForm-->>User: 弹出 Dialog
  User->>RForm: 填写回复内容、确定
  RForm->>API: replyComment(data)
  API-->>RForm: success
  RForm-->>List: emit('success')
  List->>List: getList()

  User->>List: 切换 visible Switch
  List->>List: handleVisibleChange(row)
  alt 加载中
    List->>User: 拒绝变更
  else 二次确认
    List->>API: updateCommentVisible({id, visible})
    API-->>List: success
    List->>List: getList()
  else 用户取消
    List->>List: row.visible = !changedValue 回滚
  end
```
