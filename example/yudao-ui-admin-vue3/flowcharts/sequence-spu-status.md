# 序列图 F5：SPU 5-Tab 状态流转

入口：spu/index.vue
source_nodes：component:766a92ffa67135de01a5219da9b57bf5, function:983b518f40fda455789509dcb62ad231（handleStatusChange/02Change/handleDelete/handleExport 聚合）

```mermaid
sequenceDiagram
  actor User
  participant List as SPU 列表
  participant Tabs as 5 个状态 Tab
  participant API as ProductSpuApi
  participant Server as 后端

  User->>List: 进入 SPU 页
  List->>API: getTabsCount()
  API->>Server: GET /product/spu/tabs-count
  Server-->>API: {0:10, 1:5, 2:0, 3:0, 4:2}
  API-->>List: 计数映射
  List->>List: tabsData 数量填充
  List->>API: getSpuPage(queryParams.value)
  API-->>List: {list, total}
  List->>API: getCategoryList({}) + handleTree
  List->>User: 渲染 Tab + 表格 + 分类树

  User->>Tabs: 切换 Tab
  Tabs->>List: handleTabClick(tab)
  List->>List: queryParams.tabType = tab.paneName
  List->>API: getSpuPage(queryParams)
  API-->>List: 列表
  List->>User: 刷新

  User->>List: 切换上/下架 Switch（status >= 0）
  List->>List: handleStatusChange(row)
  alt 二次确认通过
    List->>API: updateStatus({id, status: row.status})
    API-->>List: success
    List->>API: getTabsCount() 并行
    List->>API: getSpuPage(queryParams) 并行
    API-->>List: 刷新 Tab 计数 + 列表
  else 用户取消
    List->>List: row.status 翻转回原值
  end

  User->>List: 点击"回收"（非回收站 Tab）
  List->>List: handleStatus02Change(row, RECYCLE=-1)
  List->>API: updateStatus({id, status: -1})
  API-->>List: success
  List->>API: getTabsCount() + getSpuPage()
  API-->>List: 刷新

  User->>List: 点击"恢复"（回收站 Tab, tabType=4）
  List->>List: handleStatus02Change(row, DISABLE=0)
  List->>API: updateStatus({id, status: 0})
  API-->>List: success
  List->>API: getTabsCount() + getSpuPage()
  API-->>List: 刷新

  User->>List: 点击"删除"（仅 tabType=4）
  List->>List: message.delConfirm()
  alt 确认
    List->>API: deleteSpu(id)
    API-->>List: success
    List->>API: getTabsCount() + getSpuPage()
    API-->>List: 刷新
  end

  User->>List: 点击"导出"
  List->>List: message.exportConfirm()
  alt 确认
    List->>API: exportSpu(queryParams)
    API->>Server: GET /product/spu/export
    Server-->>API: Excel blob
    API-->>List: data
    List->>List: download.excel(data, '商品列表.xls')
  end

  User->>List: 点击"详情"/"修改"
  List->>List: router.push ProductSpuDetail / ProductSpuEdit
```
