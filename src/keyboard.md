# 9.1. 键盘

这一页展示了如何处理键盘按键的按下和释放。

请注意：Mac 上的 Command 键对应于 PC 上的 Super/Windows 键。

与[鼠标按钮]()类似，键盘输入可以作为 [`ButtonInput`](https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html) 资源、[事件](./events.md)和[运行条件](./run_condition.md)使用。选择最适合你的用例的模式

## 检测按键状态

对于游戏来说，最常见的是你可能对特定的已知按键感兴趣，并需要检测它们何时被按下或释放。你可以使用 `ButtonInput<KeyCode>` 资源来检查特定的按键。

- 使用 `.pressed(…)/.released(…)` 来检查一个键是否被按下 这些方法在按键处于相应状态的每一帧都会返回 `true`。

- 使用 `.just_pressed(…)/.just_released(…) `来检测按键的实际按下/释放动作 这些方法只在按键按下/释放发生的那一帧更新时返回 `true`。

```rust
fn keyboard_input(
    keys: Res<ButtonInput<KeyCode>>,
) {
    if keys.just_pressed(KeyCode::Space) {
        // Space was pressed
    }
    if keys.just_released(KeyCode::ControlLeft) {
        // Left Ctrl was released
    }
    if keys.pressed(KeyCode::KeyW) {
        // W is being held down
    }
    // we can check multiple at once with `.any_*`
    if keys.any_pressed([KeyCode::ShiftLeft, KeyCode::ShiftRight]) {
        // Either the left or right shift are being held down
    }
    if keys.any_just_pressed([KeyCode::Delete, KeyCode::Backspace]) {
        // Either delete or backspace was just pressed
    }
}
```

若要迭代当前被按住或已被按下/释放的任何键：

```rust
fn keyboard_iter(
    keys: Res<ButtonInput<KeyCode>>,
) {
    for key in keys.get_pressed() {
        println!("{:?} is currently held down", key);
    }
    for key in keys.get_just_pressed() {
        println!("{:?} was pressed", key);
    }
    for key in keys.get_just_released() {
        println!("{:?} was released", key);
    }
}
```

## 运行条件

另一种工作流程是在系统中添加[运行条件](./run_condition.md)，以便只在适当的输入发生时运行它们。

强烈建议你编写自己的运行条件，这样你可以检查任何你想要的内容，支持可配置的绑定等。

对于原型设计，Bevy 提供了一些[内置的运行条件](./input_handling.md#运行条件)：

```rust
use bevy::input::common_conditions::*;

app.add_systems(Update, (
    handle_jump
        .run_if(input_just_pressed(KeyCode::Space)),
    handle_shooting
        .run_if(input_pressed(KeyCode::Enter)),
));
```

## 键盘事件

要获取所有键盘活动，你可以使用 `KeyboardInput` 事件：

```rust
fn keyboard_events(
    mut evr_kbd: EventReader<KeyboardInput>,
) {
    for ev in evr_kbd.read() {
        match ev.state {
            ButtonState::Pressed => {
                println!("Key press: {:?} ({:?})", ev.key_code, ev.logical_key);
            }
            ButtonState::Released => {
                println!("Key release: {:?} ({:?})", ev.key_code, ev.logical_key);
            }
        }
    }
}
```

### 物理 [KeyCode](https://docs.rs/bevy/latest/bevy/input/keyboard/enum.KeyCode.html) vs 逻辑 [Key](https://docs.rs/bevy/latest/bevy/input/keyboard/enum.Key.html)

当你按下一个键时，该事件包含两个重要的信息：

1. `KeyCode`，它总是代表键盘上的一个特定键，而不考虑操作系统的布局或语言设置。

2. `Key`，它包含了操作系统解释的键的逻辑意义。

当你想要实现游戏机制时，你应该使用 `KeyCode`。这将给你提供可靠的键绑定，它们总是有效的，包括对配置了多种键盘布局的多语言用户。

当你想要实现文本/字符输入时，你应该使用 `Key`。这可以给你提供Unicode字符，你可以将它们附加到你的文本字符串中，让你的用户能够像在其他应用程序中一样打字。

如果你想处理键盘上的特殊功能键或媒体键，也可以通过逻辑键 `Key` 来实现。

## 文本输入

这是一个简单的示例，展示了如何将文本输入到字符串中（此处存储为 [`Local`](./local_resources.md)）。

```rust
use bevy::input::ButtonState;
use bevy::input::keyboard::{Key, KeyboardInput};

fn text_input(
    mut evr_kbd: EventReader<KeyboardInput>,
    mut string: Local<String>,
) {
    for ev in evr_kbd.read() {
        // We don't care about key releases, only key presses
        if ev.state == ButtonState::Released {
            continue;
        }
        match &ev.logical_key {
            // Handle pressing Enter to finish the input
            Key::Enter => {
                println!("Text input: {}", &*string);
                string.clear();
            }
            // Handle pressing Backspace to delete last char
            Key::Backspace => {
                string.pop();
            }
            // Handle key presses that produce text characters
            Key::Character(input) => {
                // Ignore any input that contains control (special) characters
                if input.chars().any(|c| c.is_control()) {
                    continue;
                }
                string.push_str(&input);
            }
            _ => {}
        }
    }
}
```

请注意，我们如何实现对退格键和回车键等键的特殊处理。你可以轻松地为应用程序中有意义的其他键添加特殊处理，如箭头键或Esc键。

对于我们的文本产生有用字符的键，它们以小的Unicode字符串形式输入。在某些语言中，每次按键可能会输入多个字符。

注意：为了支持使用复杂脚本语言（如东亚语言）的国际用户或使用手写识别等辅助方法的用户的文本输入，除了键盘输入，你还需要支持 [IME 输入](./ime.md)。

## 键盘聚焦

如果你正在做一些高级的事情，比如缓存状态以检测多键序列或键的组合，那么如果 Bevy 操作系统窗口在键盘输入过程中失去焦点（比如使用 Alt-Tab 或类似的操作系统窗口切换机制），你可能会陷入不一致的状态。

如果你正在做这样的事情，并且你认为你的算法可能会陷入困境，Bevy 提供了一个 [`KeyboardFocusLost`](https://docs.rs/bevy/latest/bevy/input/keyboard/struct.KeyboardFocusLost.html) 事件，让你知道何时应该重置你的状态。

```rust
use bevy::input::keyboard::KeyboardFocusLost;

fn detect_special_sequence(
    mut evr_focus_lost: EventReader<KeyboardFocusLost>,
    mut remembered_keys: Local<Vec<KeyCode>>,
) {
    // Imagine we need to remeber a sequence of keypresses
    // for some special gameplay reason.
    // TODO: implement that; store state in `remembered_keys`

    // But it might go wrong if the user Alt-Tabs, we need to reset
    if !evr_focus_lost.is_empty() {
        remembered_keys.clear();
        evr_focus_lost.clear();
    }
}
```