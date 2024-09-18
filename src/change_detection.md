# 14.21. Change Detection

Bevy 允许你轻松地检测数据何时发生变化。你可以使用这一点来执行响应变化的操作。

主要用例之一是优化——通过仅在相关数据更改时执行操作来避免不必要的工作。

另一个用例是在更改时触发特殊操作，例如配置某些内容或发送数据到某个地方。

## 组件的更改检测

### 过滤

你可以创建一个仅在实体上的特定组件被修改时才返回这些实体的查询。

使用查询过滤器：

1. `Added<T>`：检测新的组件实例

    - 是否在现有实体上添加了该组件；

    - 或者是否生成了具有该组件的新实体。

2. `Changed<T>`：检测组件实例是否已被更改

    - 当组件发生变化时触发；

    - 也会在组件新添加时触发（与`Added<T>`相同）。

（如果你想对组件的移除做出反应，请参阅[移除检测](#移除检测)。它的工作方式不同。）

```rust
// 当玩家的生命值或经验值发生变化时，打印他们的统计信息
fn debug_stats_change(
    query: Query<
        // components
        (&Health, &PlayerXp),
        // filters
        (Without<Enemy>, Or<(Changed<Health>, Changed<PlayerXp>)>), 
    >,
) {
    for (health, xp) in query.iter() {
        eprintln!(
            "hp: {}+{}, xp: {}",
            health.hp, health.extra, xp.0
        );
    }
}

// 检测新的敌人并打印他们的健康状况
fn debug_new_hostiles(
    query: Query<(Entity, &Health), Added<Enemy>>,
) {
    for (entity, health) in query.iter() {
        eprintln!("Entity {:?} is now an enemy! HP: {}", entity, health.hp);
    }
}
```

### 检测

如果你想像往常一样访问所有的实体，无论它们是否被修改，但是你只是想知道组件是否被改变过，对于不可变访问，你需要使用特殊的查询参数 `Ref<T>` 而不是 `&`。

对于可变访问，更改检测方法(change detection method)始终可用 (因为当你在 Bevy 查询中使用 `&mut` 时，实际上返回的是一个特殊的 `Mut<T>` 类型)。

```rust
// 当健康发生变化时，使精灵闪烁红色
fn debug_damage(
    mut query: Query<(&mut Sprite, Ref<Health>)>,
) {
    for (mut sprite, health) in query.iter_mut() {
        // 在当前帧检测健康是否改变
        if health.is_changed() {
            eprintln!("HP is: {}", health.hp);
            // 也可以检测精灵是否被改变过
            if !sprite.is_changed() {
                sprite.color = Color::RED;
            }
        }
    }
}
```

## 资源的更改检测

在查询中使用`ResMut<T>`和`Res<T>`参数来检测资源的更改。

```rust
fn check_res_changed(
    my_res: Res<MyResource>,
) {
    if my_res.is_changed() {
        // do something
    }
}

fn check_res_added(
    // 使用Option，当资源不存在时，不会panic
    my_res: Option<Res<MyResource>>,
) {
    if let Some(my_res) = my_res {
        // the resource exists

        if my_res.is_added() {
            // it was just added
            // do something
        }
    }
}
```

更改检测由 `DerefMut` 触发。简单地通过可变查询访问组件，或通过 `ResMut` 访问资源，而不实际执行 `&mut` 访问，不会触发它。这使得更改检测非常准确。

注意：如果你调用一个 Rust 函数，它接受 `&mut T `（可变借用），那也算数！即使该函数实际上没有执行任何更改，它也会触发更改检测。

此外，当你改变一个组件时，Bevy 不会检查新值是否真的与旧值不同。它将始终触发更改检测。如果您想避免这种情况，需要自己检查。

```rust
fn update_player_xp(
    mut query: Query<&mut PlayerXp>,
) {
    for mut xp in query.iter_mut() {
        let new_xp = maybe_lvl_up(&xp);

        // 如果新值与旧值相同，则不执行任何更改检测
        if new_xp != *xp {
            *xp = new_xp;
        }
    }
}
```

变更检测是按系统粒度进行的，并且是可靠的。一个系统只有在以前没有看到过这些变化时才会检测到它们。

和事件不一样，当系统只是偶尔执行时，你也不用担心错过改变。

### 可能的陷阱

注意帧延迟/帧滞后，当改变系统(changing system)在检测系统(detecting system)之前运行，检测系统只能在下一帧中检测到改变。若想在同一帧中检测改变，你需要使用[精准的系统顺序](./system_order_of_excution.md)。

## 移除检测

移除检测比较特殊，不同于改变检测，移除后的数据将不存在，后续也就没法追踪它的元数据了。

### 组件的移除检测

你可以在当前帧检查被移除的组件。这些数据会在每帧更新结束时被清除。你必须确保你的检测系统在（或在另一个运行在其后的调度中）执行移除操作的系统之后运行。

注意：移除检测也包括被销毁的实体！

使用 `RemovedComponents<T>` 特殊系统参数类型。在内部，它使用事件实现，行为类似于 `EventReader`，但它会提供被移除组件的实体的 ID。

```rust
fn detect_removals(
    mut removals: RemovedComponents<EnemyIsTrackingPlayer>,
    // ... (maybe Commands or a Query ?) ...
) {
    for entity in removals.read() {
        // do something with the entity
        eprintln!("Entity {:?} had the component removed.", entity);
    }
}
```

### 资源的移除检测

Bevy 没有为资源的移除检测提供 API。但可以使用 `Option` 和单独的 `Local` 系统参数来解决此问题，从而有效地实现你自己的检测。

```rust
fn detect_removed_res(
    my_res: Option<Res<MyResource>>,
    mut my_res_existed: Local<bool>,
) {
    if let Some(my_res) = my_res {
        // the resource exists!

        // remember that!
        *my_res_existed = true;

        // (... you can do something with the resource here if you want ...)
    } else if *my_res_existed {
        // the resource does not exist, but we remember it existed!
        // (it was removed)

        // forget about it!
        *my_res_existed = false;

        // ... do something now that it is gone ...
    }
}
```

请注意，由于此检测是系统本地的，因此不必在同一帧更新期间进行。