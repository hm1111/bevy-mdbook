# 14.9. Queries

通过使用`Query`参数和可选的`filter`(过滤器)来查询ECS World中你需要的实体，之后你就可以访问和这些实体相关的数据。

## 查询数据

`Query`参数的第一个泛型参数就是你要访问的数据。

`&`表示以共享/只读的方式访问数据,`&mut`表示以独占/可变的方式访问数据。

访问可能不存在的组件时使用`Option`(找到的实体中可能没有这个组件)。

如果想要访问多个组件`,把它们放到元组中。

### 迭代访问数据

为了访问匹配到的实体中的组件，最常见的操作就是迭代查询。

```rust
fn check_zero_health(
    // 访问拥有`Health`和`Transform`组件的实体
    // 以只读的方式访问`Health`和可变的方式访问`Transform`
    // 可选组件：获取`Player`组件，如果存在的话
    mut query: Query<(&Health, &mut Transform, Option<&Player>)>,
) {
    // 获取所有匹配的实体
    for (health, mut transform, player) in &mut query {
        eprintln!("Entity at {} has {} HP.", transform.translation, health.hp);

        // hp为0时，把它的位置居中
        if health.hp <= 0.0 {
            transform.translation = Vec3::ZERO;
        }

        if let Some(player) = player {
            // 当前的实体是`player`
            // 做相关的操作
        }
    }
}
```

除了使用`for`循环，还可以使用`.for_each()`并传入一个在每个实体上运行的闭包。这个语法会被编译器优化，所以性能更好，但是缺乏灵活性且编写麻烦。决定权在你。

```rust
fn enemy_pathfinding(
    mut query_enemies: Query<(&Transform, &mut EnemyAiState)>,
) {
    query_enemies.iter_mut().for_each(|(transform, mut enemy_state)| {
        // TODO: do something with `transform` and `enemy_state`
    })
}
```

如果你想知道查询的实体的 ID，你需要在`Query`中指定`Entity`类型。

```rust
// 添加`Entity`到`Query`来获取Entity ID
fn query_entities(q: Query<(Entity, /* ... */)>) {
    for (e, /* ... */) in q.iter() {
        // `e`是我们访问的实体的Entity ID
    }
}
```

### 访问指定Entity

为了访问一个指定实体中的组件，你需要指定这个实体的 ID：

```rust
if let Ok((health, mut transform)) = query.get_mut(entity) {
    // entity是指定实体的Entity ID
} else {
    // the entity does not have the components from the query
}
```

为了一次性访问多个指定实体的组件，可以使用 `many/many_mut`(`panic`) 或者 `get_many/get_many_mut`(返回`Result`)。指定的实体至少有一个不存在时，会`panic`或返回`Result`。

```rust
#[derive(Resource)]
struct UiHudIndicators {
    // say we have 3 special UI elements
    entities_ui: [Entity; 3],
    entities_text: [Entity; 3],
}

fn update_ui_hud_indicators(
    indicators: Res<UiHudIndicators>,
    query_text: Query<&Text>,
    query_ui: Query<(&Style, &BackgroundColor)>,
) {
    // we can get everything as an array
    if let Ok(my_texts) = query_text.get_many(indicators.entities_text) {
        // the entities exist and match the query
        // TODO: something with `my_texts[0]`, `my_texts[1]`, `my_texts[2]`
    } else {
        // query unsuccessful
    };

    // we can use "destructuring syntax"
    // if we want to unpack everything into separate variables
    let [(style0, color0), (style1, color1), (style2, color2)] =
        query_ui.many(indicators.entities_ui);

    // TODO: something with all these variables
}
```

### 唯一Entity

如果你知道查询应该只匹配一个实体（即查询预期只会匹配一个单个实体），你可以使用 `single/single_mut`（遇到错误时会`panic`）或者 `get_single/get_single_mut`（返回 `Result`）。这些方法确保存在且仅存在一个候选实体可以匹配你的查询，否则会产生错误。

```rust
fn query_player(mut q: Query<(&Player, &mut Transform)>) {
    let (player, mut transform) = q.single_mut();

    // do something with the player and its transform
}
```

### Combinations(组合)

将多个实体进行两两组合，三三组合，...

注意：当有大量实体时，这个操作会非常耗时。

```rust
fn print_potential_friends(
    q_player_names: Query<&PlayerName>,
) {
    // this will iterate over every possible pair of two entities
    // (that have the PlayerName component)

    for [player1, player2] in q_player_names.iter_combinations() {
        println!("Maybe {} could be friends with {}?", player1.0, player2.0);
    }
}

fn apply_gravity_to_planets(
    mut query: Query<&mut Transform, With<Planet>>,
) {
    // this will iterate over every possible pair of two planets

    // For mutability, we need a different syntax
    let mut combinations = query.iter_combinations_mut();
    while let Some([planet1, planet2]) = combinations.fetch_next() {
        // TODO: calculate the gravity force between the planets
    }
}
```

### Bundles

不能使用bundles来查询。它只能用来帮助你配置拥有一组正确组件的实体。它们仅在生成/插入/移除组件时使用。

查询是针对单个组件的。你需要查询你关心的bundle中的特定组件。

初学者一个常见的错误是查询bundle类型！

## Query Filter (过滤查询)

使用过滤器(filter)来减少你从查询中获得的实体数量。

使用`Query`的第二个泛型参数（可选）来实现过滤，添加`With/Without`来过滤掉是否有指定组件的实体。

```rust
fn debug_player_hp(
    // access the health (and optionally the PlayerName, if present), only for friendly players
    query: Query<(&Health, Option<&PlayerName>), (With<Player>, Without<Enemy>)>,
) {
    // get all matching entities
    for (health, name) in query.iter() {
        if let Some(name) = name {
            eprintln!("Player {} has {} HP.", name.0, health.hp);
        } else {
            eprintln!("Unknown player has {} HP.", health.hp);
        }
    }
}
```

如果你只关心一个实体是否有指定的组件，而不关系组件的具体数据，这将很有用。如果你想要访问其中的数据，把组件放入第一个参数，而不是使用过滤器。


### 绑定多个过滤器

可在`Query`的第二个参数指定多个过滤器：

1. (filter1, filter2, ...)：满足所有过滤器（与逻辑）

2. Or<(filter1, filter2, ...)>：满足任何一个过滤器（或逻辑）

## Query Transmutation(转换)

如果想你在一个`Query`函数中调用另一个`Query`函数(必须兼容)，那么你可以通过使用`QueryLens`来获取你所需的`Query`。

```rust
fn debug_positions(
    query: Query<&Transform>,
) {
    for transform in query.iter() {
        eprintln!("{:?}", transform.translation);
    }
}

fn move_player(
    mut query_player: Query<&mut Transform, With<Player>>,
) {
    // TODO: mutate the transform to move the player

    // say we want to call our debug_positions function

    // first, convert into a query for `&Transform`
    let mut lens = query_player.transmute_lens::<&Transform>();
    debug_positions(lens.query());
}

fn move_enemies(
    mut query_enemies: Query<&mut Transform, With<Enemy>>,
) {
    // TODO: mutate the transform to move our enemies

    let mut lens = query_enemies.transmute_lens::<&Transform>();
    debug_positions(lens.query());
}
```

**注意**：当我们从每个函数中调用 `debug_positions` 时，它将访问不同的实体！即使 `Query<&Transform>` 参数类型没有任何额外的过滤器，它也是通过 `QueryLens` 转换创建的，因此它只能访问原始查询派生的实体和组件。如果我们将 `debug_positions` 添加到 Bevy 中作为常规系统，它将访问所有实体的转换。

**另请注意**：这会有一些性能开销；转换操作不是免费的。Bevy 通常会在系统的多次运行中缓存一些查询元数据。当你创建新的查询时，它必须复制这些元数据。

#### Query Joining (查询连接)

你可以结合两个查询来创建一个新的查询，该查询只允许访问同时满足这两个查询的实体，从而产生一组组合的组件。在Bevy中，这通常被称为"查询合并"或"查询连接"。

```rust
fn query_join(
    mut query_common: Query<(&Transform, &Health)>,
    mut query_player: Query<&PlayerName, With<Player>>,
    mut query_enemy: Query<&EnemyAiState, With<Enemy>>,
) {
    let mut player_with_common:
        QueryLens<(&Transform, &Health, &PlayerName), With<Player>> =
            query_player.join_filtered(&mut query_common);

    for (transform, health, player_name) in &player_with_common.query() {
        // TODO: do something with all these components
    }

    let mut enemy_with_common:
        QueryLens<(&Transform, &Health, &EnemyAiState), With<Enemy>> =
            query_enemy.join_filtered(&mut query_common);

    for (transform, health, enemy_ai) in &enemy_with_common.query() {
        // TODO: do something with all these components
    }
}
```

**注意**：最终的查询不能访问原始查询所不能访问的数据。如果你尝试使用`With/Without`过滤器，它将不起作用。