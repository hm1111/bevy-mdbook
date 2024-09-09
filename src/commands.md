# 14.10. Commands
在`System`中，我们可以通过`Commands`来创建/销毁`entity`，添加/移出`entity`中的`Component`，管理`Resource`。

操作不会立马生效，因为在其他`System`并行执行的时候修改数据是不安全的，它们会被放入队列中等安全之后再执行。
通常来说，`Commands`被用于每个`Shedule`的结尾。

每个`Shedule`中的`System`的执行顺序是不确定的，除非你明确指定了`before`和`after`。

### 自定义Commands
`Commands`还可以作为一种便捷的方式来执行任何需要对 ECS World 的完全访问权限的自定义操作。

### one-shot systems
一种能立即执行而无需排队等待执行的`System`。

### 扩展Commands API
1. 自定义Command类型

    创建一个自定义类型，并实现`Command` trait。

2. 为Bevy `Commands`类型实现自定义trait

    创建一个trait，并为`Commands`实现这个triat

