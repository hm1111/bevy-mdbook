# 14.19. System sets

**system sets能够很容易为多个system应用相同的配置**，例如：运行条件，执行顺序。

你添加到system sets中的配置会自动应用到属于这个system sets的所有system。

一个system可以属于多个system sets，同时它也将应用这些system sets的配置。你也可以为单独的system添加额外的配置。

![](../_resources/system%20set%201.png)

在Bevy引擎中，System Sets 是一种用于配置和管理多个系统（Systems）执行方式的机制。通过System Sets，开发者可以定义系统执行的具体方式，包括系统的执行优先级、是否并发执行以及是否作为一个整体来调度执行。

具体来说，理解Bevy中的 System Sets 可以从以下几个方面入手：

定义执行方式：System Sets 允许开发者为一组相关的系统定义它们的执行方式。例如，你可以创建一个 RenderingSet，包含所有与渲染相关的系统，并指定它们应该按照特定的顺序执行。

控制执行顺序：可以使用依赖注入（Dependencies）来确保系统之间正确的执行顺序。通过指定系统之间的依赖关系，可以避免竞争条件和确保数据的一致性。

并发执行：System Sets 还允许开发者决定系统是否应该并发执行。这可以通过标记系统为并行（parallel）或串行（serial）来实现。并行系统可以同时在不同的线程上执行，而串行系统则逐个执行。

系统组调度：System Sets 提供了将多个系统作为一个逻辑单元进行调度的能力。这意味着你可以启动、暂停或停止整个 System Set，而不是逐个管理每个系统。

定制化执行流程：通过创建自定义的 System Set，开发者可以构建复杂的执行流程，比如根据游戏状态或用户输入来改变系统执行的顺序或并发程度。

维护性和可读性：使用 System Sets 可以提高代码的维护性和可读性。通过将相关系统组织在一起，使得代码结构更清晰，更易于理解和维护。

性能优化：合理利用 System Sets 可以提高性能。例如，根据系统的优先级来调度它们，可以确保关键系统能够及时执行，而不太重要的系统则可以在后台运行。

扩展性：System Sets 提供了一个灵活的框架，允许开发者轻松地添加、删除或修改系统。随着游戏的发展，你可以很方便地扩展或重构你的系统架构。

在Bevy的架构中，System Sets 是一个关键特性，它提供了一种高级的方式来管理和组织 Systems，使得开发者能够构建健壮、高效和易于维护的游戏引擎。通过灵活地配置 System Sets，开发者可以创建各种复杂的执行模式，以满足不同游戏需求。

## 匿名sets

最简单的sets就是使用`.add_systems`方法添加多个system的元组，这个元组就是一个system sets。

## 命名sets

这种更正式，也更强大。

你可以创建结构体/枚举作为一个标签/标识符，并派生`SystemSet` trait以及其他必要的trait，之后就可以在其他地方引用这个system sets。

对于单个system set，你可以创建一个空结构体；对于多个有关联的system set，你可以创建一个枚举，每一个枚举成员都是一个单独的system sets。

你也可以使用`.configure_sets`为system sets指定运行条件和执行顺序。

命名系统集(Named sets)的主要用例是逻辑组织，以便你可以管理系统并引用整个组。

流程：

1. 创建system sets并为其指定运行条件和执行顺序。

2. 创建一个或多个system，并将其添加到system sets中，根据需要为其添加额外的配置。

### 搭配Plugins

命名sets与plugins一起也非常有用。当你在编写plugin时，你可以公开（pub）一些system sets类型，以允许其他开发者控制plugin中的事物如何运行，或者他们与你的plugin有关的事物如何运行。这样你就不必公开任何单个系统(System)。

## 常见陷阱

system sets 的配置只存储在单个shedule中，当你在其他shedule中使用同一个system set，它将不会使用你在其他shedule中的配置。在`Update` shedule中配置的system sets只在`Update` shedule中生效。