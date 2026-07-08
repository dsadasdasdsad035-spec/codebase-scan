# 数据模型汇总：商城商品中心前端

入口：frontend-mall-product（单入口）
证据：`./entries/frontend-mall-product/data-model.md`

---

## 9 个前端实体

| # | 实体 | 主要字段 | 关系 |
|---|---|---|---|
| 1 | BrandVO | id, name, picUrl, sort, status, description | 1:N Spu |
| 2 | CategoryVO | id, parentId, name, picUrl, sort, status | 自引用树 + 1:N Spu |
| 3 | PropertyVO | id, name, remark | 1:N PropertyValue |
| 4 | PropertyValueVO | id, propertyId, name, remark | N:1 Property |
| 5 | CommentVO | id, spuId, skuId, userId, userNickname, userAvatar, content, descriptionScores, benefitScores, picUrls, replyContent, visible | N:1 Spu, N:1 Sku |
| 6 | Spu | id, name, categoryId, brandId, keyword, introduction, picUrl, sliderPicUrls, specType, subCommissionType, skus, deliveryTypes, deliveryTemplateId, description, sort, giveIntegral, virtualSalesCount, status | 1:N Sku, 1:N Comment |
| 7 | Sku | id, spuId, name, price, marketPrice, costPrice, barCode, picUrl, stock, weight, volume, firstBrokeragePrice, secondBrokeragePrice | N:1 Spu |
| 8 | PropertyAndValues | propertyId, propertyName, values[] | (组合键) |
| 9 | RuleConfig | name, rule, message | (校验配置) |

详见 [entries/frontend-mall-product/data-model.md](entries/frontend-mall-product/data-model.md)
