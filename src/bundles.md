# 14.8. Bundles

可认为`Bundle`是创建实体(entity)的模板。通过同一个`Bundle`创建的实体具有相同的组件(component)，从而简化创建实体的流程。

相对于直接创建实体，使用`Bundle`可以减少出错的可能性，这样如果你创建实体时遗漏了一个组件，编译器就会报错。

Bevy提供了一些内置`Bundle`来创建常用的实体,也可自定义`Bundle`;`Bundle`可嵌套其他`Bundle`;

## 创建Bundle

定义结构体，每一个字段就是一个组件，并派生`Bundle` trait。

```rust
#[derive(Bundle)]
struct PlayerBundle {
    xp: PlayerXp,
    name: PlayerName,
    health: Health,
    marker: Player,

    // 可以嵌套/包含其他bundle
    sprite: SpriteBundle,
}
```

## 使用Bundle

使用`Bundle`创建实体。

```rust
commands.spawn(PlayerBundle {
    xp: PlayerXp(0),
    name: PlayerName("Player 1".into()),
    health: Health {
        hp: 100.0,
        extra: 0.0,
    },
    marker: Player,
    sprite: SpriteBundle {
        // TODO
        ..Default::default()
    },
});
```

如果你需要默认值，若想定义默认值，可派生`Default` trait 或者 实现`Default` trait。

```rust
impl Default for PlayerBundle {
    fn default() -> Self {
        Self {
            xp: PlayerXp(0),
            name: PlayerName("Player".into()),
            health: Health {
                hp: 100.0,
                extra: 0.0,
            },
            marker: Player,
            sprite: Default::default(),
        }
    }
}

fn do_something_with_bundle(mut commands: Commands) {
    commands.spawn(PlayerBundle {
        name: PlayerName("Player 1".into()),
        ..Default::default()
    });
}
```

## 移除bundle

如果bundle中包含的组件也存在于实体中，那么实体中的对应组件将被删除，否则，会被忽略。

```rust
/// Contains all components to remove when
/// resetting the player between rooms/levels.
#[derive(Bundle)]
struct PlayerResetCleanupBundle {
    status_effect: StatusEffect,
    pending_action: PlayerPendingAction,
    modifier: CurrentModifier,
    low_hp_marker: LowHpMarker,
}
```
```rust
commands.entity(e_player)
    .remove::<PlayerResetCleanupBundle>();
```
## 元组作为bundle

Bevy也会把多个组件的元组作为一个`Bundle`：`(ComponentA, ComponentB, ComponentC)
`。

```rust
commands.spawn((
    SpriteBundle {
        // ...
        ..default()
    },
    Health {
        hp: 50.0,
        extra: 0.0,
    },
    Enemy,
    // ...
));
```

当你需要生成大量的entities时，推荐定义结构体作为bundle来生成，这样可以让你的代码更容易维护。

## 查询

不能为查询指定bundle，`Bundle`只是方便创建实体，`Query`中仍需要指定具体的组件，而不是指定`Bundle`。

错误示例：

```rust
fn my_system(query: Query<&SpriteBundle>) {
  // ...
}
```

正确示例：

```rust
fn my_system(query: Query<(&Transform, &Handle<Image>)>) {
  // ...
}
```