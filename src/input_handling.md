# 9. 输入处理

Bevy 支持以下输入：

- `keyboard`：检测键盘上的按键按下和释放，或者文本输入

- `Mouse`：

    - `Motion`：移动鼠标，不绑定OS光标；

    - `Cursor`：绝对指针位置；

    - `Buttons`

    - `Scrolling`：鼠标滚轮或触摸板手势；

    - `Touchpad Gestures`：只支持macos/ios；

- `Touchscreen`：多点触控；

- `Gamepad`：使用 `gilrs` 库；

- `Drag and Drop`：只支持文件；

- `IME`：针对多语言用户的高级文本输入；

以下是一些显著的输入设备，目前 Bevy 并不支持：

- 设备倾斜的加速度计和陀螺仪；

- 其他传感器，如温度传感器；

- 在多点触控板上跟踪单个手指，就像在触摸屏上一样；

- 麦克风和其他音频输入设备；

- MIDI（乐器数字接口），但有一个非官方的插件：[`bevy_midi`](https://github.com/BlackPhlox/bevy_midi)。

请注意，这并不意味着 Bevy 将来不会支持这些设备或功能，随着项目的发展，它可能会添加对更多输入设备和传感器的支持。

----

对于大多数输入类型（在合理的情况下），Bevy 提供了两种处理方式：

1. 通过资源（输入资源）检查当前状态，

2. 通过事件（输入事件）。

有些输入仅作为事件提供。

检查状态是使用资源完成的，例如 [`ButtonInput`](https://docs.rs/bevy/latest/bevy/input/struct.ButtonInput.html)（用于按键或按钮等二元输入）、[`Axis`](https://docs.rs/bevy/latest/bevy/input/struct.Axis.html)（用于模拟输入）、[`Touches`](https://docs.rs/bevy/latest/bevy/input/prelude/struct.Touches.html)（用于触摸屏上的手指）等。这种处理输入的方式对于实现游戏逻辑非常方便。在这些情况下，你通常只关心映射到游戏动作的特定输入。你可以检查特定的按钮/键以了解它们何时被按下/释放，或者它们的当前状态是什么。

[事件](./events.md)（[输入事件](./Bevy内置类型列表.md#输入事件)）是一种更低级、更全面的方法。如果你想获取该类输入设备的所有活动，而不仅仅是检查特定的输入，那么就使用事件。

## 输入映射

Bevy 目前还没有提供内置的输入映射方法（配置键位绑定等）。你需要自己想出一种方法，将输入转换为你的游戏/应用中的逻辑动作。

有一些社区制作的插件可能会对此有所帮助：请参阅 [bevy-assets 上的输入部分](https://bevyengine.org/assets/#input)。我个人推荐：[Leafwing Studios 的输入管理器插件](https://github.com/leafwing-studios/leafwing-input-manager)。它有自己的观点，不太可能适用于所有游戏，但如果它适合你，它的质量是非常高的。

为你的游戏构建自己的特定抽象可能是一个好主意。例如，如果你需要处理玩家移动，你可能需要一个系统来读取输入并将其转换为你自己的内部“移动意图/动作事件”，然后另一个系统对这些自定义事件做出反应，实际移动玩家。确保使用[明确的系统顺序](./system_order_of_excution.md)，以避免延迟/帧延迟。

## 运行条件

Bevy 还提供了[运行条件](./run_condition.md)（[请看这里的所有条件](https://docs.rs/bevy/latest/bevy/input/common_conditions/index.html)），如果你希望特定的系统只在特定的键或按钮被按下时运行，你可以将这些运行条件附加到你的系统上。

这样，你可以将输入处理作为系统调度/配置的一部分，避免在 CPU 上运行不必要的代码。

在实际游戏中不建议使用这种方式，因为你必须硬编码键位，这使得用户无法配置键位绑定。

为了支持可配置的键位绑定，你可以实现自己的运行条件，从用户偏好中检查键位绑定。

如果你正在使用 [LWIM 插件](https://github.com/leafwing-studios/leafwing-input-manager)，它也提供了类似的基于运行条件的工作流程支持。