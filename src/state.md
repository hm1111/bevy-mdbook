# 14.20. State

它能够让你构建应用的运行时"流"。

在每一个state中，你可以让不同的system运行。你也可以在进入/退出一个state时运行构建/清理system。

## 定义state

定义一个枚举类型，并派生`States` trait以及其他必要的trait，然后使用`app builder`注册。

## 为不同的states运行不同的systems

你可以使用`in_state`运行条件(run condition)为state添加system/system sets。

Bevy 会创建特殊的 `OnEnter`、`OnExit` 和 `OnTransition` shedule来为特定状态(state)的每个可能值执行设置和清理任务。你添加到它们的任何系统(system)都将在每次将状态更改为相应的值或从相应的值更改时运行一次。

### 搭配Plugins(Todo)

## 控制states

你可以使用`State<T>` resource来检查当前的state是哪个，还可以使用`NextState<T>`来切换到另一个state。

## 状态转换

在每一帧中，会有一个`StateTransition` shedule执行。Bevy会检查是否有新的state在`NextState<T>`中排队，如果有，Bevy会将当前的state设置为新的state。

转换过程：
1. `StateTransitionEvent` 事件(event)被发送；

2. `OnExit(old_state)` shedule被执行；

3. `OnTransition(from: old_state, to: new_state)` shedule被执行；

4. `OnEnter(new_state)` shedule被执行；


在任何与状态无关但想了解是否发生了状态转换的系统中，`StateTransitionEvent` 非常有用。你可以使用它来检测状态转换 。

`StateTransition` shedule运行在`PreUpdate`之后，`FixedMain`/`Update`之前。因此，状态转换发生在当前帧的游戏逻辑之前。

如果你想每帧运行多次状态转换，你可以使用`apply_state_transition` system 在需要的地方添加转换点。


## 已知的陷阱

### System set configuration is per-schedule (Todo)

### Event (Todo)