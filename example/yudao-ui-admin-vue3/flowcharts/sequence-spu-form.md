# 序列图 F6：SPU 5-Tab 复合表单提交

入口：spu/form/index.vue + 5 form 子组件
source_nodes：component:be487fd29883255ef6a9971a99c1c589, component:68c9a279711c588e6c9f52ac19abc2be, component:18a9cdf6cda3537530eea5e2dc6b6492, component:13391245e64d2417c7bc0b6da4718a78, component:7a6ff01634b3ee0a30a8a774844408c9, component:70c3384cad029eedab3d527723aaebb4, component:61fc70313ceef9ac0c4570dba9d9ac51, component:58b810df1c6c35195b854d4b8e616178

```mermaid
sequenceDiagram
  actor User
  participant Form as SPU 复合表单
  participant Info as InfoForm
  participant Sku as SkuForm
  participant Del as DeliveryForm
  participant Desc as DescriptionForm
  participant Oth as OtherForm
  participant API as ProductSpuApi
  participant TStore as tagsView store
  participant Router as vue-router

  User->>Form: 进入 /spu/form/add 或 /edit/{id} 或 /detail/{id}
  alt 有 id
    Form->>API: getSpu(id)
    API-->>Form: Spu
    alt isDetail=true（详情模式）
      Form->>Form: floatToFixed2 保留 2 位小数
    else 编辑模式
      Form->>Form: formatToFraction 分转元
    end
  end

  User->>Form: 编辑 5 个 Tab 表单
  Form->>Info: validate() 通过后写回 formData
  Form->>Sku: validate() 校验 SKU（ruleConfig 4 字段）
  Form->>Del: validate() 通过后写回 formData
  Form->>Desc: validate() 通过后写回 formData
  Form->>Oth: validate() 通过后写回 formData

  User->>Form: 点击"保存"
  Form->>Form: submitForm()
  Form->>Info: validate()
  alt Info 失败
    Info->>Form: emit('update:activeName', 'info') + throw
    Form->>User: message.error('【基础设置】不完善')
  else Info 通过
    Form->>Sku: validate()
    alt Sku 失败
      Sku->>Form: emit('update:activeName', 'sku') + throw
    else Sku 通过
      Form->>Del: validate()
      alt Del 失败 → 切到 delivery Tab
      else Del 通过
        Form->>Desc: validate()
        alt Desc 失败 → 切到 description Tab
        else Desc 通过
          Form->>Oth: validate()
          alt Oth 失败 → 切到 other Tab
          else Oth 通过
            Form->>Form: cloneDeep(formData)
            Form->>Form: 校验 formData.name 非空
            Form->>Form: skus[].name = formData.name
            Form->>Form: 价格元转分（convertToInteger）
            Form->>Form: sliderPicUrls 对象转 URL
            alt 无 id
              Form->>API: createSpu(data)
              API-->>Form: success
              Form->>User: message.success('创建成功')
            else 有 id
              Form->>API: updateSpu(data)
              API-->>Form: success
              Form->>User: message.success('更新成功')
            end
            Form->>TStore: delView(currentRoute)
            Form->>Router: push ProductSpu
          end
        end
      end
    end
  end

  User->>Form: 点击"返回"
  Form->>TStore: delView(currentRoute)
  Form->>Router: push ProductSpu
```
