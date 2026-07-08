# 状态机：评论可见性

入口：comment/index.vue
source_nodes：component:8b462134c11b251f030d72d983c2b803, function:983b518f40fda455789509dcb62ad231（handleVisibleChange）

```mermaid
stateDiagram-v2
  [*] --> Hidden : null → false（getList 兜底）
  Hidden --> Visible : Switch 切换<br/>二次确认"是否显示评论？"<br/>updateCommentVisible
  Visible --> Hidden : Switch 切换<br/>二次确认"是否隐藏评论？"<br/>updateCommentVisible
  Visible --> Hidden : 用户取消二次确认<br/>row.visible = !changedValue 回滚
  Hidden --> Visible : 用户取消二次确认<br/>row.visible = !changedValue 回滚
```

详见 [state-machines.md § 3](../../state-machines.md)
