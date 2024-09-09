# 14.14. Exclusive System
它是不会和其他`System`并行运行的`system`，通过`&mut World`参数，它拥有对[ECS World](./direct_ecs_world_access.md)的完全访问权限。

在`exclusive system`中，你对存储在ECS World中的所有数据有完全的控制权。

## 使用exclusive system的一些场景
1. 将一些`component`/`entity`保存到文件中；
2. 用来立即创建/销毁`entity`，添加/移除`component`，而无需添加到队列中等待执行；
3. 使用你自己的控制流逻辑来运行任意`system`和`shedule`；
4. 其他...


## Exclusive System参数
1. `&mut World`：对ECS World的完全访问；
2. `Local<T>`：对本地资源的访问；
3. `&mut SystemState<P>`：对系统状态的访问。模拟常规`System`，让你轻松访问来自 `World` 的数据；如果您的SystemState包括Commands，您必须在完成后调用`.apply()`。只有在那时，通过commands排队的延迟操作才会应用到`World`。

    SystemState 是 Bevy 游戏引擎中用于管理和维护游戏系统状态的一个结构体。它包含了游戏系统运行时所需的所有信息，如当前帧率、窗口大小、输入状态、资源管理器等。

    SystemState 是 Bevy 引擎内部的一个核心组件，它为开发者提供了一个统一的接口来访问和修改游戏状态。 
    
    SystemState 结构体中包含了许多字段，这些字段存储了游戏运行时的各种状态信息。例如，windows 字段存储了当前打开的窗口列表，input 字段存储了用户的输入状态，time 字段存储了当前的时间信息，resources 字段存储了游戏中的所有资源等。 
    
    在 Bevy 引擎中，SystemState 可以通过 app 对象的 state 字段来访问。开发者可以在系统的初始化阶段设置S
    ystemState 的初始值，并在游戏运行过程中通过修改它的字段来影响游戏状态。例如，开发者可以通过更新 time 字段来模拟时间流逝，或者通过修改 input 字段来响应玩家的输入。 
    
    Bevy 引擎中的 SystemState 类型是理解和使用 Bevy 游戏引擎的关键。它提供了一个框架，让开发者能够轻松地管理和维护游戏的状态，确保游戏系统在正确的状态下运行。

4. `&mut QueryState<Q, F = ()>`：对查询状态的访问。模拟常规`Query`，让你轻松访问来自 `World` 的数据；

    在Bevy引擎中，QueryState是一个结构体，它代表了查询状态（Query State），并且被用于执行实体查询（Entity Query）。这个结构体包含了所有执行查询所需的数据和状态信息。

    以下是QueryState结构体的一些主要特点和功能：

    查询过滤：QueryState允许你指定查询的过滤条件。这些条件定义了查询应该包含哪些实体，基于它们的组件（Components）。例如，你可以创建一个查询，只返回具有特定组件（如Player组件）的实体。

    迭代器：结构体内部通常包含一个或多个迭代器，这些迭代器允许你遍历符合条件的实体。通过这些迭代器，你可以访问每个实体的相关组件，从而执行你需要的任何逻辑。

    实体处理：使用QueryState，你可以对查询结果执行各种操作。这些操作可以包括读取组件数据、修改组件数据，或者基于组件数据执行某些逻辑。

    性能优化：QueryState的设计目标之一是高效处理大量的实体和组件。它通常使用有效的数据结构和算法，以确保查询和遍历过程尽可能高效。

    多阶段执行：在Bevy的系统执行流程中，QueryState可以在不同的阶段（如Update阶段）被使用。这允许你在游戏循环的不同时间点执行实体查询，以满足特定的逻辑需求。

    独立性：每个QueryState实例代表一个特定的查询，这意味着你可以同时运行多个不同的查询，每个查询都有自己的状态和迭代器。

    代码组织：使用QueryState有助于将系统中的不同逻辑关注点分离，使得代码更加模块化和易于维护。每个查询可以被看作是一个独立的逻辑单元。

    QueryState类型通常在Bevy的系统（systems）和插件（plugins）中被广泛使用，以实现各种游戏逻辑，如角色AI、物理模拟、状态更新等。通过使用QueryState，开发者可以更好地组织和管理他们的游戏逻辑，确保高效、模块化的代码结构。

## 性能

使用`exclusive system`会牺牲性能，因为在它运行时，其他的`system`，`shedule`都将被停止运行。因此，只在必要时使用`exclusive system`。

当你需要处理大量`entity`时，使用`exclusive system`是一个好的选择。

`Commands`实际上只是一种要求 Bevy 稍后为你独家访问`World`的方式。遍历 `commands` 队列比自己执行独占访问要慢得多。

## exclusive system更快 还是 commands更快？

### exclusive system更快的一些场景
1. 创建/销毁大量的entity；
2. 每一帧都需要执行的逻辑；
3. 其他...

### commands更快的一些场景
需要每一帧都执行检查任务，但是只需要偶尔使用`commands`
1. 当生命值为0时，需要删除entity；
2. 当时间结束时，需要创建/删除entity；
3. 当某个事件发生时，需要添加/删除一些UI元素；
4. 其他...