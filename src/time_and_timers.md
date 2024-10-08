# 5.4. 时间和计时器

`Time` 资源是你最基本的全局时间信息来源，你可以从任何需要时间的系统中访问它。你应该从它那里获取所有的时间信息。Bevy 在每帧开始时更新这些值。

## Delta Time

增量时间（delta time）：即从上一个帧更新到当前帧之间经过了多少时间。这告诉你游戏运行的速度，因此你可以调整移动和动画等的速度。这样，无论游戏的帧率如何，一切都可以平滑地进行，并以相同的速度运行。

## Ongoing Time

[时间](https://docs.rs/bevy/latest/bevy/time/struct.Time.html)w可以为你提供自启动以来的总运行时间。如果你需要一个累积的、不断增加的时间测量，就可以使用这个。

## Timer and Stopwatches

Bevy 还提供了一些工具来帮助你跟踪特定的时间间隔或计时：`Timer` 和 `Stopwatch`。你可以创建多个这些实例，来跟踪任何你想要的东西。你可以在你自己的组件或资源类型中使用它们。

`Timer` 和 `Stopwatch` 需要被“滴答”（tick）。你需要有一些系统调用 `.tick(delta)`，它们才能前进，否则它们将处于非活动状态。`delta` 应该来自 `Time` 资源。

### Timer

计时器(Timer)允许你检测何时经过了特定的时间间隔。计时器有一个设定的持续时间。它们可以是“重复”的或“非重复”的。

两种类型都可以手动“重置”（从头开始计算时间间隔）和“暂停”（即使你不断滴答它们，它们也不会前进）。

重复计时器在达到设定的持续时间后会自动重置。

使用`.finished()` 来检测计时器何时达到了其设定的持续时间。如果你只需要在达到持续时间的确切滴答时检测，请使用`.just_finished()`。

请注意，Bevy 的计时器不像典型的现实生活中的计时器（现实计时器朝着零倒计时）。Bevy 的计时器从零开始，向上计数，直到达到设定的持续时间。它们基本上就像带有额外功能的秒表：最大持续时间和可选的自动重置。

### Stopwatch

秒表(stopwatch)允许你追踪从某个特定的时间点开始已经过去了多少时间。

它会持续累积时间，你可以通过.elapsed()或.elapsed_secs()来检查已经过去的时间。你可以随时手动重置它。