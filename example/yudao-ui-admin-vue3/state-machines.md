# 状态机汇总：商城商品中心前端

入口：frontend-mall-product（单入口）
证据：`./entries/frontend-mall-product/state-machines.md`

---

## 6 类状态机

| # | 状态机 | 字段 | 状态值 | 关键流转 |
|---|---|---|---|---|
| 1 | SPU 销售状态 | Spu.status | ENABLE=1 / DISABLE=0 / RECYCLE=-1 | DISABLE ↔ ENABLE（Switch）；任一 → RECYCLE（handleStatus02Change）；RECYCLE → DISABLE（恢复）；仅 tabType=4 可物理删除 |
| 2 | 表单弹窗显隐 | dialogVisible | true / false | open()/submitForm/取消/X |
| 3 | 评论可见性 | CommentVO.visible | true / false | Switch 切换 + 二次确认 + 异常回滚 |
| 4 | SPU 规格类型 | Spu.specType | true / false | 切换时重置 propertyList + skus |
| 5 | SPU 分销类型 | Spu.subCommissionType | true / false | 切换时重置所有 SKU 佣金为 0 |
| 6 | 加载状态 | loading / formLoading | true / false | try-finally 兜底 |

详见 [entries/frontend-mall-product/state-machines.md](entries/frontend-mall-product/state-machines.md)
