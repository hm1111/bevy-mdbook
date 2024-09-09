# 14.16. Shedules

用来管理System，指定System什么时候以及如何执行。

Bevy根据不同的目的设计了不同的`Shedules`，它们会在合适的时候执行。

## Sheduling Systems

如果你需要Bevy中的`System`执行，则需要通过`app builder`把它添加到`Shedules`中。

### Pre-System Configuration

你可以向`System`中添加元数据来决定`System`如何运行。

有以下3种元数据：
1. `run condition`：控制`System`是否运行；

2. `ordering dependencies`：控制`System`的执行顺序；

3. `system sets`：将`System`组织到一起，因此可以将通用配置应用于所有系统；

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

当`Shedule`运行时，运行算法会遵循它的配置来确定`System`是否已准备好运行。当以下的所有情况都为`true`时，就代表System已经准备好运行：

1. 当前没有正在运行的System正在可变地访问相同的数据；

2. `System`所有的ordered "before"都已结束 或者 由于run condition而被跳过；

3. `System`所有的run condition都返回`true`；

当一个System准备好时，它会在一个可用的CPU线程上运行。System默认不是以一个明确的顺序运行的。如果你关心一个System和另一个System的执行顺序，可以使用`before`和`after`来指定它们的执行顺序。

### 动态添加/删除Systems

`Shedule`并不支持在运行时动态添加/删除Systems，你必须在`App`构建时添加所有的Systems，然后使用`run condition`控制它们。

## Bevy的App Structure

Bevy中主要有3个`Shedule`：

1. `Main`：它是Bevy引擎的核心，它包含了游戏逻辑和状态更新的主要系统；

2. `Extract`：它用于准备渲染所需的数据，例如，从各种系统中提取模型、材质和纹理信息。

3. `Render`：它负责执行渲染操作，包括绘制3D模型、应用光照和阴影效果等。

除了这三个主要的Shedules之外，还存在其他的Shedules，这些Shedules通常在`Main`Shedules内部管理和运行。

这些Shedules负责特定的游戏功能或逻辑，例如物理模拟、AI行为或音频处理。它们作为`Main`Shedules的一部分，有助于组织和执行与游戏核心逻辑相关的各种任务。

在一个正常的Bevy应用程序中，`Main`+`Extract`+`Render`Shedule会在一个循环中重复运行。它们一起产生游戏的一帧。每次`Main`运行时，它都会运行一系列其他Shedule。在它的第一次运行中，它也会首先运行一系列的“startup”shedule。

大多数情况下，你只需要处理`Main`shedule的一系列子shedule，其他2个只和图像开发者有关。[有关Extract和Render的更多信息](https://bevy-cheatbook.github.io/gpu/intro.html)

## Main shedule

它是运行程序逻辑的地方，它是一种元shedule，用来以指定顺序运行其他shedule。你不能把System直接添加到Main shedule中，而是添加到Main shedule管理的其他shedule中。

### Bevy提供的一些shedule

1. `First`/`PreUpdate`/`StateTransition`/`RunFixedMainLoop`/`Update`/`PostUpdate`/`Last`：每次`Main`shedule运行时，它们都会运行一次。

2. `PreStartup`/`Startup`/`PostStartup`：在`Main`shedule第一次运行时运行，且只会运行一次；

3. `FixedMain`：它的固定时间步和`Main`shedule是一样的；为了追赶固定时间步长间隔，它会在需要的时候通过`RunFixedMainLoop`执行多次；

4. `FixedFirst`/`FixedPreUpdate`/`FixedUpdate`/`FixedPostUpdate`/`FixedLast`：每次`FixedMain`：它们的时间步长和`Main`shedule的子shedule一样；

5. `OnEnter()`/`OnExit()`/`OnTransition()`：在`State`改变时，它们会通过`StateTransition`执行；

对大部分的应用来说，`Update`/`Startup`/`FixedUpdate`/`State` transition这几个是被使用最多的。

`Update`用来处理每一帧都需要执行的任务；`Startup`用来处理程序启动时需要执行的初始化任务；`FixedUpdate`用来处理固定时间步长间隔需要执行的任务；`State` transition用来处理`State`改变时需要执行的任务。

## 配置shedules

### Single-Threaded Shedules(Todo)

### Defered Appliction(Todo)

## Main shedule Configuration
`Main`shedule的子shedule的执行顺序是由`MainSheduleOrder` resource决定的，如果这些预定义的子shedule不能满足你的要求，你可以定义你自己的shedule。

### 创建自定义shedule
第1步：定义一个结构体/枚举，并派生`SheduleLabel` trait以及其他一些必要的trait；

第2步：使用`app`初始化，并添加到`MainSheduleOrder`resource中；

第3步：添加system到自定义的shedule；