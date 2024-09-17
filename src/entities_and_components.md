# 14.7. Entities, Componets

## Entities

从概念上说，一个实体(entity)代表了一组组件(component)的值。每一个组件就是一个Rust类型。

从技术上说，实体只是一个被用来寻找和它关联的数据的整型ID。在Bevy中，`Entity`类型由ID和generation这2个整数组成。这个ID在旧的实体被销毁后，可被新的实体重用，但generation的值会被更改。

可使用 [Commands](./commands.md) 和 [ECS World access](./direct_ecs_world_access.md) 来创建和销毁实体。若多个实体存在相同的组件，可使用[Bundles](./bundles.md)简化生成实体的流程。

```rust
fn setup(mut commands: Commands) {
    // 创建新的实体
    commands.spawn((
        // 初始化组件和bundle
        Enemy,
        Health {
            hp: 100.0,
            extra: 25.0,
        },
        AiMode::Passive,
        // ...
    ));

    // 调用`.id()`来获取Entity ID
    let my_entity = commands.spawn((/* ... */)).id();

    // 销毁实体，移除所有和它有关的数据
    commands.entity(my_entity).despawn();
}
```

##  Components

组件是和实体关联的数据。

### 创建Component类型

定义一个结构体/枚举，并派生`Component` trait。

```rust
#[derive(Component)]
struct Health {
    hp: f32,
    extra: f32,
}

#[derive(Component)]
enum AiMode {
    Passive,
    ChasingPlayer,
}
```

### Newtype Components

使用Newtype模式。使用封装（newtype）结构体可以从简单类型中创建出独特的组件。

```rust
#[derive(Component)]
struct PlayerXp(u32);

#[derive(Component)]
struct PlayerName(String);
```

### Marker Components

使用空结构体标记指定的实体。不保存任何数据，仅仅用来标记实体。

```rust
/// Add this to all menu ui entities to help identify them
#[derive(Component)]
struct MainMenuUI;

/// Marker for hostile game units
#[derive(Component)]
struct Enemy;

/// This will be used to identify the main player entity
#[derive(Component)]
struct Player;

/// Tag all creatures that are currently friendly towards the player
#[derive(Component)]
struct Friendly;
```

### 访问 Components

使用[Query](./queries.md)在[系统](./systems.md)中访问。通过`Query`，你可以访问任何匹配了`Query`签名的实体的组件值。

```rust
fn level_up_player(
    // 获取相关数据，有些组件是只读的，有些可变
    mut query_player: Query<(&PlayerName, &mut PlayerXp, &mut Health), With<Player>>,
) {
    // `single`确保只有一个匹配的实体
    let (name, mut xp, mut health) = query_player.single_mut();
    if xp.0 > 1000 {
        xp.0 = 0;
        health.hp = 100.0;
        health.extra += 25.0;
        info!("Player {} leveled up!", name.0);
    }
}

fn die(
    // `Entity`可被用来获取实体的ID
    query_health: Query<(Entity, &Health)>,
    // 可使用`Commands`销毁实体
    mut commands: Commands,
) {
    // 在循环中检查所有查询到的实体
    for (entity_id, health) in query_health.iter() {
        if health.hp <= 0.0 {
            commands.entity(entity_id).despawn();
        }
    }
}
```

### 添加/删除 Components

可使用[Commands](./commands.md) 或者 [ECS World Access](./direct_ecs_world_access.md)来添加/删除实体中的组件。

```rust
fn make_enemies_friendly(
    query_enemy: Query<Entity, With<Enemy>>,
    mut commands: Commands,
) {
    for entity_id in query_enemy.iter() {
        commands.entity(entity_id)
            .remove::<Enemy>()
            .insert(Friendly);
    }
}
```