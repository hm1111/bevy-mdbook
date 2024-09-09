# 14.15. Direct ECS World Access

`World`是Bevy ECS存储数据和相关元数据的地方，它包含了所有的entity、component、resource等。

常规`systems`被*system参数*限制了能从`World`中访问的数据，但可以使用`Commands`间接操作`World`。

如果你想自定义控制流，`direct world access`就很有作用了。

Bevy也有多种直接访问`World`的方式,允许你完全地控制和自由操作Bevy ECS中存储的数据:
1. `Exclusive System`
2. `FromWorld` impls
3. App builder
4. 手动创建`World`
5. 自定义`Commands`

直接访问`world`可以做的事情：
1. 生成/销毁`entity`，添加/删除`resource`/`component`等等（立即执行）；
2. 访问`entity`/`component`/`resouce`；
3. 手动运行任意`systems`/`shedules`；

## 使用World

### [SystemState](https://docs.rs/bevy/latest/bevy/ecs/system/struct.SystemState.html#impl-FromWorld-for-SystemState%3CParam%3E)

使用 `SystemState` 是最简单的方法。

这是一种“模仿系统(System)”的类型，其行为方式与具有各种参数的系统(System)相同。所有相同的行为，如查询、变更检测，甚至命令都可用。你可以使用任何系统参数。

它还跟踪任何持久状态，用于变更检测或缓存以提高性能。因此，如果你计划多次重用同一个 `SystemState`，你应该将其存储在某处，而不是每次都创建一个新的。每次调用`.get(world)`时，它的行为就像系统(System)的另一次“运行”。

如果你使用`Commands`，可以选择何时将它们应用于`World`。你需要在 `SystemState` 上手动调用.`apply(world)`来应用它们。


### 运行Systems

### 运行Shedules

如果你想运行许多系统(System)（这是一个常见的用例，例如测试），最简单的方法是构建一个临时`Shedule`。这样，你可以重用Bevy在运行系统时通常所执行的所有调度逻辑。它们将以多线程等方式运行。

如果你想定制控制流程，这也很有用。例如，Bevy的状态和固定时间步长抽象就是这样实现的。`exclusive system`可以包含循环、if/else分支等来实现奇特的算法并根据需要运行整个系统调度。

### 元数据导航

`World`中包含了许多用于导航到所有数据（entity，component，archtype，...）的元数据。