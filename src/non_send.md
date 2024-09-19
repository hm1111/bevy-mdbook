# Non-Send Resources

Non-Send 指的是那些只能被应用程序的主线程访问的数据类型。在 Rust 中，这样的数据被标记为 `!Send`，意味着它们不具备 `Send` 特征。

一些（通常是系统）库具有不能从其他线程安全使用的接口。例如窗口管理、图形、音频等各种底层操作系统接口。如果你正在做一些高级的事情，比如为了与这些东西交互而创建一个 Bevy 插件，你可能会遇到这种需求。

通常，Bevy 通过在线程池中运行所有系统来工作，从而利用多个 CPU 核心。但是，有时你需要确保某些代码总是在主线程上运行，或者以多线程的方式访问不安全的数据。

## Non-Send 系统和数据访问

要实现这一功能，你可以使用 `NonSend<T>` 或 `NonSendMut<T>` 系统参数。这些参数的行为类似于 `Res<T>` 或 `ResMut<T>`，允许你访问 ECS 资源（某些数据的全局单例），但不同之处在于，这些参数的存在会迫使 Bevy 调度程序始终在主线程上运行系统。这确保了数据永远不必在线程之间传送或从不同的线程访问。

一个这样的资源示例是 Bevy 中的 [WinitWindows](https://docs.rs/bevy/latest/bevy/winit/struct.WinitWindows.html)。这是窗口实体背后的底层，通常用于窗口管理，它让你可以更直接地访问操作系统的窗口管理功能。

```rust
fn setup_raw_window(
    q_primary: Query<Entity, With<PrimaryWindow>>,
    mut windows: NonSend<WinitWindows>
) {
    let raw_window = windows.get_window(q_primary.single());
    // do some special things using `winit` APIs
}
```

```rust
// just add it as a normal system;
// Bevy will notice the NonSend parameter
// and ensure it runs on the main thread
app.add_systems(Startup, setup_raw_window);
```

## 自定义 Non-Send 资源

通常来说，为了插入资源,它的类型必须是 `Send` 的。但是，Bevy可以独立跟踪non-send资源，这些资源只能使用`NonSend<T>`/`NonSendMut<T>`访问。

不允许使用`Commands`插入non-send资源，只能使用`World`。这意味着你只能在独占系统(exclusive system)，`FromWorld` impl或者在app builder中初始化这些资源。

如果你需要一个只能在主线程中运行的系统，并且不关心存储的数据，可以使用`NonSendMarker`作为资源。

通常来说，为了插入资源，这些资源的类型必须是 `Send` 的。

Bevy 可以独立跟踪 `Non-Send` 资源，这些资源只能使用 `NonSend<T>` 或 `NonSendMut<T> `进行访问。

不允许使用 `Commands` 插入 `Non-Send` 资源，因为 `Commands` 用于在系统中调度实体的创建和修改，而这些操作通常需要在线程间安全地传递数据。相反，你只能[直接访问 `World`](./direct_ecs_world_access.md) 来插入这些资源。这意味着你只能在[独占系统](./exclusive_system.md)（exclusive system）、[`FromWorld`](https://docs.rs/bevy/latest/bevy/ecs/world/trait.FromWorld.html) impl 或者在 [`app builder`](./the_app.md) 中初始化这些资源。

```rust
fn setup_platform_audio(world: &mut World) {
    // assuming `OSAudioMagic` is some primitive that is not thread-safe
    let instance = OSAudioMagic::init();

    world.insert_non_send_resource(instance);
}
```

```rust
app.add_systems(Startup, setup_platform_audio);
```

或者，对于简单的事情，如果你不需要一个成熟的系统：

```rust
app.insert_non_send_resource(OSAudioMagic::init());
```

如果你只需要编写一个必须在主线程上运行的系统，但实际上你没有任何数据需要存储，你可以使用` NonSendMarker` 作为一个虚拟的标记。

```rust
fn my_main_thread_system(
    marker: NonSend<NonSendMarker>,
    // ...
) {
    // TODO: do stuff ...
}
```