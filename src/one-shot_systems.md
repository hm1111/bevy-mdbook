# 14.22. One-Shot Systems

`One-Shot Systems` 翻译为 "一次性系统"。这是计算机编程和系统设计中的一个概念，指的是那些只需要执行一次或者在特定条件下执行一次的系统或操作。

它可能是指在游戏开发中，某些系统（如技能释放、特殊物品使用等）只需要在特定事件触发时执行一次，而不是持续运行或周期性执行。

它是那些你打算自己调用的系统，而不是由引擎自动调用的系统。比如：按下按钮，触发游戏中的特殊物品或技能，等等......

```rust
fn item_handler_health(
    mut q_player: Query<&mut Health, With<Player>>,
) {
    let mut health = q_player.single_mut();
    health.hp += 25.0;
}

fn item_handler_magic_potion(
    mut evw_magic: EventWriter<MyMagicEvent>,
    mut commands: Commands,
) {
    evw_magic.send(MyMagicEvent::Sparkles);
    // 一次性系统运行时，commands会立即执行
    commands.spawn(MySparklesBundle::default());
}
```

## 注册

你不应该将这些系统添加到调度(schedule)中。

相反，你可以将它们注册到世界（World）中，以获得一个 [SystemId](https://docs.rs/bevy/latest/bevy/ecs/system/struct.SystemId.html)。然后，您可以将该 `SystemId` 存储在某个地方，并在以后使用它来运行该系统。

最方便的方法可能是使用 [`FromWorld`](https://docs.rs/bevy/latest/bevy/ecs/world/trait.FromWorld.html) 并将您的 `SystemId`s 放在一个资源（resource）中：

```rust
// 在这个简单例子中，我们在hashmap中使用字符串键来组织我们的系统。
#[derive(Resource)]
struct MyItemSystems(HashMap<String, SystemId>);

impl FromWorld for MyItemSystems {
    fn from_world(world: &mut World) -> Self {
        let mut my_item_systems = MyItemSystems(HashMap::new());

        my_item_systems.0.insert(
            "health".into(),
            world.register_system(item_handler_health)
        );
        my_item_systems.0.insert(
            "magic".into(),
            world.register_system(item_handler_magic_potion)
        );

        my_item_systems
    }
}
```

```rust
app.init_resource::<MyItemSystems>();
```

可选：使用[独占系统](./exclusive_system.md)来注册：

```rust
fn register_item_handler_systems(world: &mut World) {
    let mut my_item_systems = MyItemSystems(HashMap::new());

    my_item_systems.0.insert(
        "health".into(),
        world.register_system(item_handler_health)
    );
    my_item_systems.0.insert(
        "magic".into(),
        world.register_system(item_handler_magic_potion)
    );

    world.insert_resource(my_item_systems);
}
```

或者使用 `app builder` 来注册：

```rust
fn my_plugin(app: &mut App) {
    let mut my_item_systems = MyItemSystems(HashMap::new());

    my_item_systems.0.insert(
        "health".into(),
        app.register_system(item_handler_health)
    );
    my_item_systems.0.insert(
        "magic".into(),
        app.register_system(item_handler_magic_potion)
    );

    app.insert_resource(my_item_systems);
}
```

## 运行

最简单的方法就是使用 [`Commands`](./commands.md)：

```rust
fn trigger_health_item(
    mut commands: Commands,
    systems: Res<MyItemSystems>,
) {
    // TODO: do some logic to implement picking up the health item

    let id = systems.0["health"];
    commands.run_system(id);
}
```

这会将系统放入队列中，等待 Bevy 决定何时应用这些命令。

如果你想立即运行一次性系统，就像调用普通函数那样，你需要[直接访问World](./direct_ecs_world_access.md)。从一个[独占系统](./exclusive_system.md)中执行它：

```rust
fn trigger_magic_item(world: &mut World) {
    // TODO: do some logic to implement picking up the magic item

    let id = world.resource::<MyItemSystems>().0["magic"];
    world.run_system(id).expect("Error Running Oneshot System");

    // Since we are in an exclusive system, we can expect
    // the magic potion to now be in effect!
}
```

无论哪种方式，一次性系统的 `Commands` 都会在运行时自动立即应用。

## 无需注册

一次性系统无需预先注册也是可以运行的：

```rust
world.run_system_once(my_oneshot_system_fn);
```

如果你这样做，Bevy 将无法存储与该系统相关的任何数据：

- [`Local`](./local_resources.md) 将不会保留它们前一次运行的值。

- [`Queries`](./queries.md)将无法缓存它们的查找，导致性能下降。

- 等等。

因此，建议注册你的一次性系统，除非你真的只打算运行它们一次。

## 性能注意事项

运行一次性系统需要独占 World 的访问权限。系统可以有任意参数，Bevy 无法像处理调度中的系统那样验证其数据访问是否与其他系统冲突。因此，不允许多线程。

实际上，这通常不是问题，因为一次性系统的使用场景是很少发生的事件。

但也不要过度使用它们，如果一些事情经常发生，用普通的系统在调度中运行可能更好，然后用运行条件控制它们，反而更合适。