# 14.4. The App

app是你定义构成你项目的所有内容的地方：插件(plugins)、系统(systems)、事件(events)......

通常情况下，你会在项目的 `main` 函数中创建你的 App。然而，你不必在那里添加所有的东西。如果你想从多个地方（如其他Rust文件或crate）向你的应用程序添加内容，你可以使用[插件](./plugins.md)。随着项目的增长，你将需要这样做以保持一切井然有序。

```rust
fn main() {
    App::new()
        // Bevy内置插件
        .add_plugins(DefaultPlugins)

        // 项目自定义插件
        .add_plugins(ui::MyUiPlugin)

        // 事件
        .add_event::<LevelUpEvent>()

        // 在应用启动时需要运行一次的系统
        .add_systems(Startup, spawn_things)

        // 每一帧都需要运行的系统
        .add_systems(Update, (
            camera_follow_player,
            debug_levelups,
            debug_stats_change,
        ))
        // ...

        // 运行app
        .run();
}
```

**注意**：在使用`add_systems/add_plugins/configure_sets`方法时，可以传入元组来一次性添加多个事物。

组件不需要添加到app中。

schedules 不能在运行时被修改；所有系统都必须添加到 app 中。

你可以使用[运行条件](./run_condition.md)来控制单个系统。

你可以使用[`MainscheduleOrder`](https://docs.rs/bevy/latest/bevy/app/struct.MainScheduleOrder.html) [resource](./resources.md)来动态启用/禁用整个 schedule。

## Bevy内置功能

Bevy 游戏引擎的自身功能被表示为一个[插件组](./plugins.md#plugingroup)。每个典型的 Bevy 应用程序必须首先添加它，使用以下方法之一：

1. [DefaultPlugins](https://docs.rs/bevy/latest/bevy/struct.DefaultPlugins.html)：制作一个完整的应用；

2. [MinimalPlugins](https://docs.rs/bevy/latest/bevy/struct.MinimalPlugins.html)：比如：无头服务器；

## 设置数据

通常，你可以从系统中设置你的数据。从常规系统中使用`Commands`，或者使用独占系统来获取完整的World访问权限。

将你的设置系统添加到`Startup` schedule中，以便在启动时初始化一些东西，或者使用[状态](./state.md)进入/退出系统来在菜单、游戏模式、关卡等之间转换时做一些事情。

但是，你也可以直接从应用构建器(app builder)初始化数据。对于资源来说，这很常见，因为它们需要一直存在。你也可以[直接访问World](./direct_ecs_world_access.md)。

```rust
// 使用指定值创建(或覆盖)资源
app.insert_resource(StartingLevel(3));

// 确保资源存在;如果不存在，创建它
app.init_resource::<MyFancyResource>();

// 你能够直接访问World来做任何事情
app.world_mut().spawn(SomeBundle::default());
```

## 退出应用

为关闭应用，可以从任何系统发送一个[`AppExit`](https://docs.rs/bevy/latest/bevy/app/enum.AppExit.html)事件：

```rust
use bevy::app::AppExit;

fn exit_system(mut exit: EventWriter<AppExit>) {
    exit.send(AppExit::Success);
}
```

你可以指定返回给操作系统的退出代码。如果 Bevy 接收到多个 `AppExit` 事件，只有当所有事件都报告成功时，才会返回成功。如果有些事件报告了错误，那么最后一个事件将决定进程的实际退出代码。