# 14.18. Run Conditions(运行条件)

运行条件用来指示一个系统(system)是否应该运行。

run condtion它是一个返回`bool`之的函数，只允许接收只读的系统参数(system parameter)。

它可被应用于单个系统(system)和整个系统集(system sets)。

当应用于单个系统(system)时，Bevy 将在系统准备好运行之前的最后一刻评估RC(run condition)。如果您将相同的 RC(run condition) 添加到多个系统，Bevy 将为每个系统(system)单独评估RC(run condition)。

当应用于系统集(system sets)时，Bevy将在其中任一个系统(system)运行之前评估RC(run condition)，若评估结果为`false`，则整个系统集(system set)将被跳过。

你可以为一个系统(system)添加多个RC(run condition)，同时它也将继承它所属的系统集(system set)的RC(run condition)，当所有的RC(run condition)都为`true`时，这个系统(system)才会运行。

## 常用的运行条件

Bevy为某些常用场景提供了一些内置的运行条件(run condition)。

1. ECS常用运行条件：[相关文档](https://docs.rs/bevy/0.13.0/bevy/ecs/schedule/common_conditions/index.html)

2. 输入相关运行条件：[相关文档](https://docs.rs/bevy/0.13.0/bevy/input/common_conditions/index.html)

3. 时间相关运行条件：[相关文档](https://docs.rs/bevy/0.13.0/bevy/time/common_conditions/index.html)

4. 其他运行条件

## 一些陷阱

对于那些不是每一帧都运行的系统(system)，当它们未运行时，可能会错过接收事件(event)的机会。为了避免出现这种情况，你可以使用[*自定义清理策略*](https://bevy-cheatbook.github.io/patterns/manual-event-clear.html)来手动管理相关事件(event)的生命周期来解决这个问题。