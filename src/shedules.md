# 14.16. Shedules

用来管理System，指定System什么时候以及如何执行。

Bevy根据不同的目的设计了不同的`Shedules`，它们会在合适的时候执行。

## Sheduling Systems

如果你需要Bevy中的`System`执行，则需要通过`app builder`把它添加到`Shedules`中。

### Pre-System Configuration

你可以向`System`中添加元数据来决定`System`如何运行。

有以下3种元数据：
1. [run condition](./run_condition.md)：控制`System`在什么情况下运行；

2. [ordering dependencies](./system_order_of_excution.md)：控制`System`的执行顺序；

3. [system sets](./system_sets.md)：将`System`组织到一起，因此可以将通用配置应用于所有系统；

当`Shedule`运行时，运行算法会遵循它的配置来确定`System`是否已准备好运行。当以下的所有情况都为`true`时，就代表System已经准备好运行：

1. 没有其他正在运行的System在可变地访问相同的数据；

2. 所有在它之前执行的系统都已结束 或者 由于它们不满足执行条件而被跳过；

3. `System`所有的执行条件都返回`true`；

当一个System准备好时，它会在一个可用的CPU线程上运行。System默认不是以一个明确的顺序运行的。如果你关心一个System和另一个System的执行顺序，可以使用`before`和`after`来指定它们的执行顺序。

### 动态添加/删除Systems

`Shedule`并不支持在运行时动态添加/删除Systems，你必须在`App`构建时添加所有的Systems，并使用运行条件来控制它们的运行。

## Bevy App Structure

Bevy中主要有3个`Shedule`：

1. `Main`：它是Bevy引擎的核心，它包含了游戏逻辑和状态更新的主要系统；

2. `Extract`：它用于准备渲染所需的数据，例如，从各种系统中提取模型、材质和纹理信息。

3. `Render`：它负责执行渲染操作，包括绘制3D模型、应用光照和阴影效果等。

除了这三个主要的Shedules之外，还存在其他的Shedules，这些Shedules通常在`Main`Shedules内部管理和运行。

这些Shedules负责特定的游戏功能或逻辑，例如物理模拟、AI行为或音频处理。它们作为`Main`Shedules的一部分，有助于组织和执行与游戏核心逻辑相关的各种任务。

在一个正常的Bevy应用程序中，`Main`+`Extract`+`Render`Shedule会在一个循环中重复运行。它们一起生成游戏的一帧。每次`Main`运行时，它都会运行一系列其他Shedule。在它的第一次运行中，它也会首先运行一系列的"startup" shedule。

大多数情况下，你只需要处理`Main`shedule的一系列子shedule，其他2个只和图像开发者有关。[有关Extract和Render的更多信息](https://bevy-cheatbook.github.io/gpu/intro.html)。

## Main shedule

它是运行程序逻辑的地方，它是一种元shedule，用来以指定顺序运行其他shedule。你不能把System直接添加到`Main` shedule中，而是添加到`Main` shedule管理的其他子shedule中。

### Bevy提供了一些子shedule，用来管理你的系统(system)：

1. `First`/`PreUpdate`/`StateTransition`/`RunFixedMainLoop`/`Update`/`PostUpdate`/`Last`：每次`Main` shedule运行时，它们都会运行一次。

2. `PreStartup`/`Startup`/`PostStartup`：在`Main` shedule第一次运行时运行，且只会运行一次；

3. `FixedMain`：它的固定时间步和`Main`shedule是一样的；为了追赶固定时间步长间隔，它会在需要的时候通过`RunFixedMainLoop`执行多次；

4. `FixedFirst`/`FixedPreUpdate`/`FixedUpdate`/`FixedPostUpdate`/`FixedLast`：它们的时间步长和`Main`shedule的子shedule一样；

5. `OnEnter()`/`OnExit()`/`OnTransition()`：在`State`改变时，它们会通过`StateTransition`执行；

对大部分的应用来说，`Update`/`Startup`/`FixedUpdate`/`State transition` 这几个是被使用最多的。

`Update`用来处理每一帧都需要执行的任务；`Startup`用来处理程序启动时需要执行的初始化任务；`FixedUpdate`用来处理固定时间步长间隔需要执行的任务；`State transition`用来处理`State`改变时需要执行的任务。

## 配置shedules

### Single-Threaded Shedules

当你觉得多线程的shedule不能很好地工作时，或者因为其他原因，你可以为每一个shedule关闭它的多线程特性，而只使用单线程来运行。

在单线程shedule中，系统只会在主线程运行，且同一时刻只有一个系统在运行。然而，“就绪算法”依然有效，这意味着系统的执行顺序依然是不明确的。你可以根据需要指定它们的执行顺序。

```rust
// Make FixedUpdate run single-threaded
app.edit_schedule(FixedUpdate, |schedule| {
    schedule.set_executor_kind(ExecutorKind::SingleThreaded);

    // or alternatively: Simple will apply Commands after every system
    schedule.set_executor_kind(ExecutorKind::Simple);
});
```

### 歧义检测（不太理解）

它是一个可选的功能，用来调试和*非确定性*相关的问题。

```rust
// Enable ambiguity warnings for the Update schedule
app.edit_schedule(Update, |schedule| {
    schedule.set_build_settings(ScheduleBuildSettings {
        ambiguity_detection: LogLevel::Warn,
        ..default()
    });
});
```

该警告表示在某些系统组合中，至少有一个系统会可变地访问某些数据（资源或组件），但其他系统没有对该系统有明确的顺序依赖关系。

这种情况可能表明存在错误，因为你无法确定读取数据的系统会在修改数据的系统之前还是之后运行。

你需要根据具体情况决定是否关心这个问题。

### Defered Appliction(Todo)

通常情况下，Bevy 会自动管理`commands`和其他延迟操作的应用位置。如果系统之间存在顺序依赖关系，Bevy 会确保在第二个系统运行之前应用第一个系统的任何挂起的延迟操作。

如果你想禁用这种自动行为并手动管理同步点，你可以这样做:

```rust
app.edit_schedule(Update, |schedule| {
    schedule.set_build_settings(ScheduleBuildSettings {
        auto_insert_apply_deferred: false,
        ..default()
    });
});
```
为了手动创建同步点，把`apply_defered`方法插入到你想要同步的系统之前：
```rust
app.add_systems(
    Update,
    apply_deferred
        .after(MyGameplaySet)
        .before(MyUiSet)
);
app.add_systems(Update, (
    (
        system_a,
        apply_deferred,
        system_b,
    ).chain(),
));
```

## Main shedule Configuration

`Main` shedule的子shedule的执行顺序是由`MainSheduleOrder` resource决定的，如果这些预定义的子shedule不能满足你的要求，你可以定义你自己的shedule。`MainScheduleOrder` 资源(resource)允许你指定主调度器(`Main` shedule)运行子调度器的顺序。

### 创建自定义shedule

第1步：定义一个结构体/枚举，并派生`SheduleLabel` trait以及其他一些必要的trait；

第2步：使用`app`初始化，并添加到`MainSheduleOrder` resource中；

第3步：添加system到自定义的shedule；

```rust
// Ensure the schedule has been created
// (this is technically optional; Bevy will auto-init
// the schedule the first time it is used)
app.init_schedule(PrepareUpdate);

// Add it to the MainScheduleOrder so it runs every frame
// as part of the Main schedule. We want our PrepareUpdate
// schedule to run after StateTransition.
app.world.resource_mut::<MainScheduleOrder>()
    .insert_after(StateTransition, PrepareUpdate);

// Now we can add some systems to our new schedule!
app.add_systems(PrepareUpdate, (
    my_weird_custom_stuff,
));
```