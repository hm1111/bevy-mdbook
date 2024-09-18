# 14.10. Commands

`Commands`被用来创建/销毁实体(entity)，添加/移出实体中的组件(component)，管理资源(resource)。

```rust
fn spawn_things(
    mut commands: Commands,
) {
    // 管理资源
    commands.insert_resource(MyResource::new());
    commands.remove_resource::<MyResource>();

    // 使用spawn()创建新的实体
    // 给组件提供所需的数据
    // (通常使用bundle)
    commands.spawn(PlayerBundle {
        name: PlayerName("Henry".into()),
        xp: PlayerXp(1000),
        health: Health {
            hp: 100.0, extra: 20.0
        },
        _p: Player,
        sprite: Default::default(),
    });

    // 你可以使用元组来添加额外的组件或bundle
    // (组件和bundle组成的元组也被认为是bundle)
    // (注意额外的圆括号)
    let my_entity_id = commands.spawn((
        // 添加一些组件
        ComponentA,
        ComponentB::default(),
        // 添加一些bundles
        MyBundle::default(),
        TransformBundle::default(),
    )).id(); // 在结尾调用`.id()`获取 Entity (id)

    // 添加/移除实体中的组件
    commands.entity(my_entity_id)
        .insert(ComponentC::default())
        .remove::<ComponentA>()
        .remove::<(ComponentB, MyBundle)>();

    // 除了指定的组件/bundle，其它的组件/bundle都会被移除
    commands.entity(my_entity_id)
        .retain::<(TransformBundle, ComponentC)>();
}

fn make_all_players_hostile(
    mut commands: Commands,
    // 通过指定Entity组件来获取Entity id，因为需要在指定的entities上执行指令
    query: Query<Entity, With<Player>>,
) {
    for entity in query.iter() {
        commands.entity(entity)
            // 给指定的实体添加组件
            .insert(Enemy)
            // 移除指定的组件
            .remove::<Friendly>();
    }
}

fn despawn_all_enemies(
    mut commands: Commands,
    query: Query<Entity, With<Enemy>>,
) {
    for entity in query.iter() {
        commands.entity(entity).despawn();
    }
}
```

## 操作何时生效？

操作不会立马生效，因为在其他系统并行执行的时候修改数据是不安全的，它们会被放入队列中等安全之后再执行。

在同一个schedule中，你可以添加`.before()/.after()`来指定系统的执行顺序。Bevy会在第二个系统之前执行这些操作，确保第二个系统能看见第一个系统所做的修改。

```rust
app.add_systems(Update, spawn_new_enemies_if_needed);

// `enmy_ai`在运行时会看见新生成的敌人，因为Bevy会应用第一个系统的指令(感谢`.after()`依赖)
app.add_systems(Update, enemy_ai.after(spawn_new_enemies_if_needed));
```

如果没有指定系统的执行顺序，`commands`何时被应用是不确定的。这就导致有些系统可能在下一帧才能看到`commands`所做的修改。

通常来说，`Commands`在每个schedule的结尾才被执行，之后执行的schedule就可以看到这些变化。例如：你可以在`PostUpdate`中看到`Update`生成的实体。

## 自定义Commands

`Commands`还可以执行任何需要对 ECS World 的完全访问权限的自定义操作。你可以将任何自定义代码排队等待运行，就像标准命令一样以延迟的方式工作。

```rust
fn my_system(mut commands: Commands) {
    let x = 420;

    commands.add(move |world: &mut World| {
        // do whatever you want with `world` here

        // note: it's a closure, you can use variables from
        // the parent scope/function
        eprintln!("{}", x);
    });
}
```

如果你想要一些可重用的东西，可以考虑[单次执行系统](./one-shot_systems.md)(one-shot system)。它们是编写常规Bevy系统并按需运行它们的一种方式。

### 扩展Commands API

2种方法：

1. 自定义Command类型。

定义一个自定义类型，并实现`Command` trait。

```rust
use bevy::ecs::world::Command;

struct MyCustomCommand {
    // you can have some parameters
    data: u32,
}

impl Command for MyCustomCommand {
    fn apply(self, world: &mut World) {
        // do whatever you want with `world` and `self.data` here
    }
}

// use it like this
fn my_other_system(mut commands: Commands) {
    commands.add(MyCustomCommand {
        data: 920, // set your value
    });
}
```

2. 为Bevy `Commands` 类型实现自定义trait。

创建一个trait，并为`Commands`类型实现这个tratt。

```rust
pub trait MyCustomCommandsExt {
    // define a method that we will be able to call on `commands`
    fn do_custom_thing(&mut self, data: u32);
}

// implement our trait for Bevy's `Commands`
impl<'w, 's> MyCustomCommandsExt for Commands<'w, 's> {
    fn do_custom_thing(&mut self, data: u32) {
        self.add(MyCustomCommand {
            data,
        });
    }
}

fn my_fancy_system(mut commands: Commands) {
    // now we can call our custom method just like Bevy's `spawn`, etc.
    commands.do_custom_thing(42);
}
```

注意：如果你想要从其他的Rust文件中使用这些扩展方法，你需要先导入相应的trait，否则这些方法将不可用。

`use crate::thing::MyCustomCommandsExt;`