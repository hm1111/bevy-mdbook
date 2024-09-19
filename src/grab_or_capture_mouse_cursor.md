# 10.3. 抓取/捕获鼠标光标

在某些类型的游戏中，你可能希望将鼠标限制在窗口内，以防止在游戏过程中鼠标离开窗口。

这种行为有两种变化（[`CursorGrabMode`](https://docs.rs/bevy/latest/bevy/window/enum.CursorGrabMode.html)）：

- `Confined`（限制）允许鼠标移动，但只能在窗口的边界内移动。

- `Locked`（锁定）将鼠标固定在一个位置，不允许其移动。

    - 相对[鼠标运动](./mouse.md##鼠标运动)事件仍然有效。

抓取光标：

```rust
use bevy::window::{CursorGrabMode, PrimaryWindow};

fn cursor_grab(
    mut q_windows: Query<&mut Window, With<PrimaryWindow>>,
) {
    let mut primary_window = q_windows.single_mut();

    // if you want to use the cursor, but not let it leave the window,
    // use `Confined` mode:
    primary_window.cursor.grab_mode = CursorGrabMode::Confined;

    // for a game that doesn't use the cursor (like a shooter):
    // use `Locked` mode to keep the cursor in one place
    primary_window.cursor.grab_mode = CursorGrabMode::Locked;

    // also hide the cursor
    primary_window.cursor.visible = false;
}
```

释放光标：

```rust
fn cursor_ungrab(
    mut q_windows: Query<&mut Window, With<PrimaryWindow>>,
) {
    let mut primary_window = q_windows.single_mut();

    primary_window.cursor.grab_mode = CursorGrabMode::None;
    primary_window.cursor.visible = true;
}
```

你应该在游戏进行时抓取光标，并在玩家暂停游戏/退出到菜单或其他情况时释放它。

对于相对鼠标移动，你应该使用[鼠标运动](./mouse.md##鼠标运动)而不是[光标输入](./mouse.md##鼠标光标位置)来实现你的游戏玩法。

## 平台差异

macOS 本身不支持 `Confined` 模式。Bevy 会回退到 `Locked `模式。如果你想支持 macOS 并且想使用[光标输入](./mouse.md##鼠标光标位置)，你可能需要实现一个“虚拟光标”来代替。

Windows 本身不支持 `Locked` 模式。Bevy 会回退到 `Confined` 模式。你可以通过每帧重新定位光标来模拟锁定行为：

```rust
#[cfg(target_os = "windows")]
fn cursor_recenter(
    mut q_windows: Query<&mut Window, With<PrimaryWindow>>,
) {
    let mut primary_window = q_windows.single_mut();
    let center = Vec2::new(
        primary_window.width() / 2.0,
        primary_window.height() / 2.0,
    );
    primary_window.set_cursor_position(Some(center));
}
```

```rust
#[cfg(target_os = "windows")]
app.add_systems(Update, cursor_recenter);
```