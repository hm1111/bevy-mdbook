# 14.12. Plugins

随着项目的增长，把它分成多个模块是很有必要的。在Bevy中，你可以使用“插件”来实现这一点。

在Bevy游戏引擎中，插件是一种组织和管理游戏逻辑的方式。你可以理解插件是能添加到`app builder`中的所有事物的集合，它可以包含系统(systems)、资源(resources)、组件(components)、事件(events)等。当你使用`App::add_plugins(...)`添加一个插件时，Bevy会将插件中的所有内容合并到你的应用程序（App）中。

这种合并是扁平化的，意味着插件中的所有内容都会被添加到App的顶层，而不会创建任何嵌套的结构。例如，如果你有一个插件包含一个系统和一个资源，这两个元素将直接添加到App的系统列表和资源列表中。

这种设计提供了极大的灵活性，允许开发者以模块化的方式构建游戏。每个插件可以专注于特定的功能领域，并且可以轻松地在不同的项目中重用。此外，Bevy的插件系统还允许开发者覆盖或修改插件中的默认行为，以满足特定项目的需求。

对于你自己项目中的内部组织，插件的主要价值在于不必将所有 Rust 类型和函数声明为 `pub`，这样就可以从 `main`函数中访问它们并添加到应用中。它允许你能够在任意地方给应用添加内容，就像Rust模块/Rust文件/Rust crates一样。

Bevy 提供了一些内置的Plugins，它们可以让你快速的开始开发游戏。你还可以自定义插件，以满足你的特定需求。

## 自定义Plugins

2种方法：

1. 创建一个带`&mut App`参数的函数。(最简单的方法)

```rust
fn my_plugin(app: &mut App) {
    app.init_resource::<MyCustomResource>();
    app.add_systems(Update, (
        do_some_things,
        do_other_things,
    ));
}
```

2. 定义一个结构体，并实现`Plugin` trait。这种方法的好处就是，如果你想让你的插件可配置，你可以使用配置参数或泛型来扩展它。同时，你还可以以`&mut`的方式来访问App，这样你可以添加任何你需要的东西。

```rust
struct MyPlugin;

impl Plugin for MyPlugin {
    fn build(&self, app: &mut App) {
        app.init_resource::<MyOtherResource>();
        app.add_event::<MyEvent>();
        app.add_systems(Startup, plugin_init);
        app.add_systems(Update, my_system);
    }
}
```

## 将Plugin添加到App中

插件创建好之后，你可以在其他地方将插件添加到app中(通常在`main`函数中)。

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins((
            my_plugin, // the `fn`-based plugin
            MyPlugin,  // the `struct`-based plugin
        ))
        .run();
}
```

## 使用建议

1. 为不同的状态([States](./state.md))创建不同的Plugins；

2. 为各种子系统(SubSystems)创建不同的Plugins，比如物理/输入处理等等；

## PluginGroup

`PluginGroup` 是一个插件的集合，它允许你在一个地方定义一组插件，并在需要时将它们一次性添加到应用中。这对于组织和管理插件非常有用。

给app插入`plugin`时，你可以禁用一部分插件。请注意，这只是简单地禁用了功能，但它不会真正删除代码，也不能避免应用程序的二进制文件的膨胀，被禁用的插件仍然会被编译进你的程序中。

如果你想减小应用程序的二进制文件的体积，你应该禁用 Bevy 的默认cargo 功能，或者单独依赖 Bevy 的各个子库。

### 自定义PluginGroup

定义一个结构体，并实现`PluginGroup` trait。

```rust
use bevy::app::PluginGroupBuilder;

struct MyPluginGroup;

impl PluginGroup for MyPluginGroup {
    fn build(self) -> PluginGroupBuilder {
        PluginGroupBuilder::start::<Self>()
            .add(FooPlugin)
            .add(BarPlugin)
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(MyPluginGroup)
        .run();
}
```

## Plugin配置

`Plugin`也是一个存储 在程序初始化/启动期间所需的设置/配置 的好地方。对于那些在运行时可能变化的设置，推荐放进`resource`中。

```rust
struct MyGameplayPlugin {
    /// Should we enable dev hacks?
    enable_dev_hacks: bool,
}

impl Plugin for MyGameplayPlugin {
    fn build(&self, app: &mut App) {
        // add our gameplay systems
        app.add_systems(Update, (
            health_system,
            movement_system,
        ));
        // ...

        // if "dev mode" is enabled, add some hacks
        if self.enable_dev_hacks {
            app.add_systems(Update, (
                player_invincibility,
                free_camera,
            ));
        }
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(MyGameplayPlugin {
            // change to true for dev testing builds
            enable_dev_hacks: false,
        })
        .run();
}
```

被添加到`PluginGroup`中`plugin`也可以被配置。Bevy `DefaultPlugins`中的很多`plugin`就是这样工作的。

```rust
use bevy::window::WindowResolution;

App::new()
    .add_plugins(DefaultPlugins.set(
        // here we configure the main window
        WindowPlugin {
            primary_window: Some(Window {
                resolution: WindowResolution::new(800.0, 600.0),
                // ...
                ..Default::default()
            }),
            ..Default::default()
        }
    ))
    .run();
```

## 把插件发布到crates.io

Plugin给你了一个很好的途径来发布基于bevy的库，其他人可以轻松集成进他们的项目。

如果你想把插件发布成公共crate，你需要了解[插件开发指南](https://bevyengine.org/learn/quick-start/plugin-development/)。

别忘了在官网上提交进入[Bevy Assets](https://bevyengine.org/assets)的申请，这样大家可以更容易地找到你的插件。在[Github 仓库](https://github.com/bevyengine/bevy-assets)提交一个PR就行。

如果你对前沿的Bevy(main)感兴趣，[看这里](./using_bleeding_edge_bevy.md)。