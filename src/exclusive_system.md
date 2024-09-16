# 14.14. Exclusive System

它是不会和其他`System`并行运行的`system`，通过`&mut World`参数，它拥有对[ECS World](./direct_ecs_world_access.md)的完全访问权限。

在`exclusive system`中，你对存储在ECS World中的所有数据有完全的控制权。

## 使用exclusive system的一些场景

1. 将各种实体(entiy)和组件(component)转储到文件中，以实现游戏存档文件的保存和加载，或者从编辑器导出场景等功能；

2. 用来立即创建/销毁`entity`，添加/移除`component`，而无需添加到队列中等待执行；

3. 使用你自己的控制流逻辑来运行任意`system`和`shedule`；

4. 其他...


## 可用于独占系统的参数

1. `&mut World`：对ECS World的完全访问；

2. `Local<T>`：对系统本地资源的访问；

3. `&mut SystemState<P>`：让你轻松访问 `World` 中的数据，`P`是系统参数；如果`SystemState`包括`Commands`，你必须在完成后调用`.apply()`。只有这样，通过`commands`排队的延迟操作才会立即应用到`World`中；

4. `&mut QueryState<Q, F = ()>`：对查询状态的访问。模拟常规`Query`，让你轻松访问 `World` 中的数据；

`SystemState`用来模拟一个常规系统，你可以给它传递常规的系统参数。这使得你可以在函数体内模拟一个常规的系统，并且可以将这个模拟限制在函数体的特定范围内，从而使得代码更加灵活。

`QueryState`则是`SystemState`的一个简化版本，它专门用于单个查询（Query）。如果你只需要查询某些数据，而不需要对ECS世界进行修改，那么`QueryState`是一个更简单的选择。它提供了一种更直接的方式来访问ECS世界中的数据，而不需要通过完整的`SystemState`来获取。

## 性能

它会限制并行和多线程，当独占系统运行时，其他任何事物都不能访问`ECS World`，整个`shedule`都将被停止运行，以适应独占系统这很容易引入性能瓶颈。

一般来说，你应该避免使用独占系统，除非你需要做一些只有它们才能做的事情。

但是，如果你需要处理大量`entity`时，并且使用了`commands`，这时使用独占系统是一个更好的选择。

`Commands`实际上是一种要求 Bevy 稍后为你独占访问`World`的方式。遍历 `commands` 队列比自己执行独占访问要慢得多。

## exclusive system更快 还是 commands更快？

### exclusive system更快的一些场景

1. 创建/销毁大量的entity；

2. 每一帧都需要执行的逻辑；

3. 其他...

### commands更快的一些场景

需要每一帧都执行检查任务，但只需要偶尔使用`commands`：

1. 当生命值为0，需要删除entity时；

2. 当时间结束，需要创建/删除entity时；

3. 当某个事件发生，需要添加/删除一些UI元素时；

4. 其他...