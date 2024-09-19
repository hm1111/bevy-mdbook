# 10.2. 更改背景颜色

使用 [`ClearColor`](https://docs.rs/bevy/latest/bevy/prelude/struct.ClearColor.html) 资源来选择默认的背景颜色。这个颜色将被用作所有相机的默认背景颜色，除非被覆盖。

请注意，如果没有相机存在，窗口将是黑色的。您必须生成至少一个相机。

```rust
fn setup(
    mut commands: Commands,
) {
    // this camera will use the default color
    commands.spawn(Camera2dBundle::default());
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        // set the global default clear color
        .insert_resource(ClearColor(Color::rgb(0.9, 0.3, 0.6)))
        .add_systems(Startup, setup)
        .run();
}
```

为了覆盖默认颜色并为指定相机使用不同的颜色，你可以使用 [`Camera`](https://docs.rs/bevy/latest/bevy/prelude/struct.Camera.html) 组件：

```rust
use bevy::render::camera::ClearColorConfig;

// configure the background color (if any), for a specific camera (3D)
commands.spawn(Camera3dBundle {
    camera: Camera {
        // clear the whole viewport with the given color
        clear_color: ClearColorConfig::Custom(Color::rgb(0.8, 0.4, 0.2)),
        ..Default::default()
    },
    ..Default::default()
});

// configure the background color (if any), for a specific camera (2D)
commands.spawn(Camera2dBundle {
    camera: Camera {
        // disable clearing completely (pixels stay as they are)
        // (preserves output from previous frame or camera/pass)
        clear_color: ClearColorConfig::None,
        ..Default::default()
    },
    ..Default::default()
});
```

所有这些位置（特定相机上的组件、全局默认资源）都可以在运行时进行修改，Bevy 将使用您的新颜色。使用资源更改默认颜色将把新颜色应用于所有未指定自定义颜色的现有相机，而不仅仅是新生成的相机。