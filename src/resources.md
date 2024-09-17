# 14.6. Resources

`Resources`用来存储全局唯一的数据，而不依赖实体(entity)。它可以在任何时候，任何地方被访问。

## 创建

定义一个结构体/枚举，并派生`Resource` trait。

```rust
#[derive(Resource)]
struct GoalsReached {
    main_goal: bool,
    bonus: u32,
}
```

该类型的实例最多只有一个，否则考虑使用 [Entities and Components](./entities_and_components.md)。

Bevy有许多[内置的resources](./Bevy内置类型列表.md#resources资源)，你可以使用它们访问引擎的不同功能。

## 访问

在系统(system)中使用`Res/ResMut`参数访问`Resouces`。

```rust
fn my_system(
    // 当resources不存在时，程序会panic
    mut goals: ResMut<GoalsReached>,
    other: Res<MyOtherResource>,
    // 当resource可能不存在时，使用`Option`
    mut fancy: Option<ResMut<MyFancyResource>>,
) {
    if let Some(fancy) = &mut fancy {
        // TODO: do things with `fancy`
    }
    // TODO: do things with `goals` and `other`
}
```

## 管理

1. 如果需要在运行时创建/移除`resource`，可以使用`Commands`参数。

```rust
fn my_setup(mut commands: Commands, /* ... */) {
    // 添加resource(使用指定值来创建)
    commands.insert_resource(GoalsReached { main_goal: false, bonus: 100 });
    // 添加resource(使用默认值来创建)
    commands.init_resource::<MyFancyResource>();
    // 移除resource
    commands.remove_resource::<MyOtherResource>();
}
```

2. 也可通过[独占系统](./exclusive_system.md)来管理。

```rust
fn my_setup2(world: &mut World) {
    // The same methods as with Commands are also available here,
    // but we can also do fancier things:

    // 检查resource是否存在
    if !world.contains_resource::<MyFancyResource>() {
        // 获取resource或添加resource
        let _bonus = world.get_resource_or_insert_with(
            || GoalsReached { main_goal: false, bonus: 100 }
        ).bonus;
    }
}
```

3. 也可以使用`app builder`来添加，这意味 resource 在程序启动时就存在了。

```rust
App::new()
    .add_plugins(DefaultPlugins)
    .insert_resource(StartingLevel(3))
    .init_resource::<MyFancyResource>()
    // ...
```

## 初始化Resources

如果你想使用 `.init_resource` 来添加 resource ，有以下2种方法来提供 resource 的默认值。

1. 派生/实现 `Default` trait。
    
    第1步：定义结构体/枚举；

    第2步：派生/实现 `Default` trait；

```rust
// 简单派生，所有字段都被设置为其类型的默认值
#[derive(Resource, Default)]
struct GameProgress {
    game_completed: bool,
    secrets_unlocked: u32,
}

#[derive(Resource)]
struct StartingLevel(usize);

// 其他值则需要手动实现`Default` trait
impl Default for StartingLevel {
    fn default() -> Self {
        StartingLevel(1)
    }
}

// 在枚举类型中，你可以指定默认成员
#[derive(Resource, Default)]
enum GameMode {
    Tutorial,
    #[default]
    Singleplayer,
    Multiplayer,
}
```

2. 针对更复杂的初始化，需要实现 `FromWorld` trait。

```rust
#[derive(Resource)]
struct MyFancyResource { /* stuff */ }

impl FromWorld for MyFancyResource {
    fn from_world(world: &mut World) -> Self {
        // You have full access to anything in the ECS World from here.

        // For example, you can access (and mutate!) other things:
        {
            let mut x = world.resource_mut::<MyOtherResource>();
            x.do_mut_stuff();
        }

        // You can load assets:
        let font: Handle<Font> = world.resource::<AssetServer>().load("myfont.ttf");

        MyFancyResource { /* stuff */ }
    }
}
```

**注意**：过度使用 `FromWorld` trait容易导致项目更难维护。

## 使用建议

选择实体/组件(entity/component)还是资源(resource)，通常取决于你希望如何访问数据：是从全局任何地方访问（资源），还是使用ECS模式（实体/组件）。

即使游戏中只有一个特定的事物（例如单人游戏中的玩家），使用实体而不是资源是一个很好的选择，因为实体是由多个组件组成的，其中一些组件可以与其他实体通用。这可以使游戏逻辑更加灵活。例如，你可以拥有一个适用于玩家和敌人的“健康/伤害系统”。

### Settings

resource的一个常见用法是存储应用的设置/配置。但是，如果它是在运行时无法更改并且只在初始化插件(plugin)时使用的内容，请考虑将其放在插件中，而不是`resources`中。

### Caches

资源在以下情况下也很有用：如果你想以一种更易于或更有效地访问的方式存储一些数据。例如，保存一组资源句柄，或者使用自定义数据结构来更高效地表示游戏地图，而不是使用实体和组件等。

实体和组件虽然很灵活，但并不一定适合所有用例。如果你想以其他方式表示数据，请随意这样做。只需创建一个资源并将其放在那里即可。

### Interfacing with external libraries

如果你想将一些外部的非Bevy软件集成到Bevy应用中，创建一个资源来保存其状态/数据会非常方便。

例如，如果你想使用一个外部的物理或音频引擎，你可以将它的所有数据放在一个资源中，并写一些系统来调用它的函数。这可以给你一个让Bevy代码与它交互的简单方法。

如果外部代码不是线程安全的（在Rust语言中称为`!Send`），这对于非Rust库（如C ++和操作系统级库）来说很常见，你应该使用[Non-Send](./non_send.md) Bevy资源代替。这将确保任何触及它的Bevy系统都将在主线程上运行。