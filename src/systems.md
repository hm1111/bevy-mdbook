# 14.5. Systems

系统(system)是一组被Bevy执行的函数，通常使用Rust函数来实现。它是实现游戏逻辑的地方。

这些函数只能接受**特定类型的参数**，这些参数指定了你想访问的数据，如果你使用了不受支持的参数类型，你会得到编译错误。

一些受支持的参数：

1. [Res/ResMut](./resources.md)：访问资源(resource)；

2. [Query](./queries.md)：访问实体/组件(entity/component)；

3. [Commands](./commands.md)：创建/销毁实体、组件、资源等等；

4. [EventWriter/EventReader](./events.md)：发送事件/接收事件；

5. [可用参数列表](https://docs.rs/bevy/latest/bevy/ecs/system/trait.SystemParam.html#implementors)

```rust
fn debug_start(
    // access resource
    start: Res<StartingLevel>
) {
    eprintln!("Starting on level {:?}", *start);
}
```

系统参数可被放入到元组中(可嵌套)，这样有利于管理：

```rust
fn complex_system(
    (a, mut b): (
        Res<ResourceA>,
        ResMut<ResourceB>,
    ),
    (q0, q1, q2): (
        Query<(/* … */)>,
        Query<(/* … */)>,
        Query<(/* … */)>,
    ),
) {
    // …
}
```

每个系统最多接受16个参数，若需要更多的参数，可使用元组。元组最多包含16个成员，但可以无限嵌套。

为了运行这些System，需要使用`app builder`来配置。

一种特殊的系统：[独占系统](./exclusive_system.md)。它拥有ECS World的完全访问权限，可以访问任意所需的数据，但是不能和其他系统并行执行。一般情况下，你应该使用常规平行系统。

```rust
fn reload_game(world: &mut World) {
    // ... access whatever we want from the World
}
```
## 运行

为了让Bevy运行你的系统，你可以使用`app builder`把它添加到应用中。

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        // run these only once at launch
        .add_systems(Startup, (setup_camera, debug_start))
        // run these every frame update
        .add_systems(Update, (move_player, enemies_ai))
        // ...
        .run();
}
```

**注意**：编写了一个新系统，却忘记把它添加到应用中是一个常见错误。如果你的代码并没有如你期望的那样运行，请检查你是否将它添加到了应用中。

系统被包含在shedule中。Update shedule是你添加每一帧都需要运行的系统的地方；Startup shedule是你添加在程序启动时只需要运行一次的系统的地方。

## One-Shot Systems

当你不想要 Bevy 执行你的系统，而只想给自己使用时，可以使用[一次性系统](./one-shot_systems.md)(one-shot systems)，不用把系统添加到`shedules`中。