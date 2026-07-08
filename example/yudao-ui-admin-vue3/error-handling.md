# 错误处理汇总：商城商品中心前端

入口：frontend-mall-product（单入口）
证据：`./entries/frontend-mall-product/error-handling.md`

---

## 9 类错误处理

| # | 类别 | 关键点 |
|---|---|---|
| 1 | 表单校验 | Element Plus 内联 + 复合表单 Tab 自动跳转 + 抛错截断 |
| 2 | 二次确认取消 | 删除/状态变更/导出 try-catch 静默 + Switch 回滚 |
| 3 | 加载状态保护 | try-finally + loading 拒绝 + 异常回滚 |
| 4 | SKU 价格校验 | ruleConfig 4 字段（stock ≥ 1，price/marketPrice/costPrice ≥ 0.01） |
| 5 | 路由参数 | propertyId / categoryId / id 三种解析 |
| 6 | 分↔元换算 | formatToFraction / floatToFixed2 / convertToInteger / fenToYuan |
| 7 | 轮播图 URL 化 | typeof item === 'object' ? item.url : item |
| 8 | API 错误 | axios 拦截器统一处理 |
| 9 | 越界与回滚 | 删除仅回收站可见 + 详情模式只读 |

详见 [entries/frontend-mall-product/error-handling.md](entries/frontend-mall-product/error-handling.md)
