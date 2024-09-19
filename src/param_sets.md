# Param Sets

出于安全原因，一个系统不能有多个参数，这些参数的数据访问可能会在同一数据上发生可变性冲突。

一些例子：

- 多个不兼容的查询。

-  使用 `&World` 同时还有其他系统参数访问特定数据。

- ...

思考这个例子：

```rust
fn reset_health(
    mut q_player: Query<&mut Health, With<Player>>,
    mut q_enemy: Query<&mut Health, With<Enemy>>,
) {
    // ...
}
```

这两个查询都试图可变地访问 `Health`。它们有不同的过滤器，但如果有实体同时具有 `Player` 和 `Enemy` 组件怎么办？如果我们知道这不应该发生，我们可以添加 `Without<Player>` 和 `Without<Enemy>` 过滤器，但如果这在我们的游戏中实际上是有效的呢？

这样的代码将编译（Rust 无法知道 Bevy ECS 的语义），但会导致运行时 `panic`。当 Bevy 尝试运行系统时，它将因系统参数冲突而 `panic`：

```rust
thread 'main' panicked at bevy_ecs/src/system/system_param.rs:225:5:
error[B0001]: Query<&mut game::Health, bevy_ecs::query::filter::With<game::Enemy>> in
system game::reset_health accesses component(s) game::Health in a way that conflicts
with a previous system parameter. Consider using `Without<T>` to create disjoint Queries
or merging conflicting Queries into a `ParamSet`.
```

Bevy 提供了一个解决方案：使用 `ParamSet` 封装任何不兼容的参数：

```rust
fn reset_health(
    // access the health of enemies and the health of players
    // (note: some entities could be both!)
    mut set: ParamSet<(
        Query<&mut Health, With<Enemy>>,
        Query<&mut Health, With<Player>>,
        // also access the whole world ... why not
        &World,
    )>,
) {
    // set health of enemies (use the 1st param in the set)
    for mut health in set.p0().iter_mut() {
        health.hp = 50.0;
    }

    // set health of players (use the 2nd param in the set))
    for mut health in set.p1().iter_mut() {
        health.hp = 100.0;
    }

    // read some data from the world (use the 3rd param in the set)
    let my_resource = set.p2().resource::<MyResource>();
}
```

这样就确保了在同一时间只能使用冲突参数中的一个。Bevy 现在可以愉快地运行我们的系统了。

参数集(param set)的最大参数数量为8。