# 10.4. 设置窗口图标

你可能想要设置一个自定义的窗口图标。在 Windows 和 Linux 上，这是显示在窗口标题栏（如果有）和任务栏（如果有）的图标图像。

不幸的是，Bevy 尚未提供一种简单且符合人体工程学的内置方法来执行此操作。但是，可以通过 `winit` API 来实现。

这里显示的方法相当棘手。为了节省代码复杂性，我们绕过资产系统，直接使用 `image` 图像库加载文件，而不是使用 Bevy 的资产系统在后台加载图像。

Bevy 正在进行一些工作来添加适当的 API 来实现这一点；请参阅 PR [#1163](https://github.com/bevyengine/bevy/issues/1163)、[#2268](https://github.com/bevyengine/bevy/pull/2268)、[#5488](https://github.com/bevyengine/bevy/issues/5488)、[#8130](https://github.com/bevyengine/bevy/issues/8130) 和 Issue [#1031](https://github.com/bevyengine/bevy/issues/1031)。

这个示例展示了如何从 Bevy 启动系统为主要/主窗口设置图标。

```rust
use bevy::winit::WinitWindows;
use winit::window::Icon;

fn set_window_icon(
    // we have to use `NonSend` here
    windows: NonSend<WinitWindows>,
) {
    // here we use the `image` crate to load our icon data from a png file
    // this is not a very bevy-native solution, but it will do
    let (icon_rgba, icon_width, icon_height) = {
        let image = image::open("my_icon.png")
            .expect("Failed to open icon path")
            .into_rgba8();
        let (width, height) = image.dimensions();
        let rgba = image.into_raw();
        (rgba, width, height)
    };
    let icon = Icon::from_rgba(icon_rgba, icon_width, icon_height).unwrap();

    // do it for all windows
    for window in windows.windows.values() {
        window.set_window_icon(Some(icon.clone()));
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, set_window_icon)
        .run();
}
```

注意：`WinitWindows`是一个 [non-send](./non_send.md) 资源。

注意：你需要将 `winit` 和 `image` 添加到你的项目依赖项中，并且它们必须与 Bevy 使用的版本相同。在 Bevy 0.13 中，这应该是 winit = "0.29" 和 image = "0.24"。如果你不知道该使用哪个版本，你可以使用 `cargo tree` 或检查 Cargo.lock 来查看正确的版本。