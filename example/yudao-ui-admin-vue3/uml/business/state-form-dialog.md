# 状态机：表单弹窗显隐

入口：所有 *Form.vue 组件
source_nodes：所有 dialogVisible 引用

```mermaid
stateDiagram-v2
  [*] --> Hidden : ref 默认 false
  Hidden --> Visible : open(type, id?)<br/>dialogVisible = true
  Visible --> Hidden : submitForm 成功
  Visible --> Hidden : 用户点击"取消"<br/>dialogVisible = false
  Visible --> Hidden : 用户点击关闭 X
  Hidden --> [*] : 组件销毁
```

详见 [state-machines.md § 2](../../state-machines.md)

**约束**：
- open 时必先 `resetForm()` 清空表单
- 若是 `type=update` 且 `id` 有值：调对应 API 加载数据
- submitForm 成功：emit('success')，弹窗关闭
