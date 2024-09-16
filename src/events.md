# 14.11. Events

`Events` 是系统之间彼此沟通的工具，用于在系统之间发送数据。

`Events` 是非常有用的抽象层，帮助你解耦。你可以分离不同的功能块，更容易明确`system`的责任范围。

## 创建event

定义一个结构体/枚举，并派生`Event` trait，之后就可以在系统中使用该类型，同时还需要通过`app builder`将该类型注册到应用中。

## 使用event

创建event之后，任何系统都可以接收和发送该类型的数据：

1. 发送数据：使用`EventWriter<T>`参数。

2. 接收数据：使用`EventReader<T>`参数。

```rust
#[derive(Event)]
struct LevelUpEvent(Entity);

fn player_level_up(
    mut ev_levelup: EventWriter<LevelUpEvent>,
    query: Query<(Entity, &PlayerXp)>,
) {
    for (entity, xp) in query.iter() {
        if xp.0 > 1000 {
            ev_levelup.send(LevelUpEvent(entity));
        }
    }
}

fn debug_levelups(
    mut ev_levelup: EventReader<LevelUpEvent>,
) {
    for ev in ev_levelup.read() {
        eprintln!("Entity {:?} leveled up!", ev.0);
    }
}
```

## 注册event

使用`app builder`注册event类型：

`app.add_event::<YourEvent>();`

## 使用建议

事件应该是你的首选数据流工具。由于事件可以从任何系统发送并由多个系统接收，它们非常通用。

事件可以是一个非常有用的抽象层。它们允许你解耦事物，因此你可以分离不同的功能，并更容易地推理出哪个系统负责什么。

你可以想象一下，即使在上面展示的简单 "player up level" 示例中，使用事件也可以让我们轻松地扩展我们假设的游戏，使其具备更多功能。如果我们想显示一个花哨的升级效果或动画、更新UI或其他任何功能，我们可以添加更多的系统来读取事件并执行各自的操作。如果 player_level_up 系统只是检查了玩家经验值并直接管理了玩家等级，而没有通过事件，那么这将给未来游戏的开发带来不便。

## 它是如何工作的？

当注册event类型的时候，Bevy会创建一个`Events<T>` resource，类似于一个后台存储的event队列。 Bevy也提供了一个定期清理事件(event)的 "event maintenance" 系统，防止事件累积从而耗尽内存。

Bevy确保event在当前帧和下一帧都保留（帧更新循环/固定时间步循环），这样即使它们在当前帧没有得到处理,你的系统仍可在下一帧读取event，之后它们才会被自动清理掉。你要确保系统会一直运行，这样它们才能有机会处理这些事件。需要注意的是，当你的系统添加了运行条件，且你的系统没在运行的时候，那么可能会错过这些事件。

如果不喜欢这种自动清理方式，你可以手动控制event何时清理(如果忘记清理，会有内存泄露/浪费的风险)。

`EventWriter<T>`系统参数只是用来可变地访问`Events<T>` resource，并向队列添加事件的语法糖。

`EventReader<T>`稍复杂一点：它不可变地访问event，还会存储一个用来跟踪所读取的event的数量的计数器，这就是它需要使用`mut`关键字的原因。

`Event<T>`在内部使用`Vec`来存储event，发送event等同于向`Vec`添加event。这种方式非常快速，且开销低。在 Bevy 中，事件通常是实现功能的最有效方式，比使用[改变检测(change detection)](./change_detection.md)更好。

## 可能的陷阱

小心**帧延迟/帧滞后**。当接收系统先于发送系统运行是有可能发生这种情况的。此时接收系统在下一帧更新时只有一次机会接收event。如果要确保event被立即/同帧处理，可以[指定系统的执行顺序](./system_order_of_excution.md)。

当system有运行条件(run condition)，它可能会在没有运行时错过一些event。当你的system没有在每帧或固定时间步至少检查一次event，event将被丢失。

如果你希望event持续时间的更长，你可以实现[自定义清理/管理策略](./manual_event_clearing.md)。 但是你也只能处理你自己的event类型，不能处理Bevy内置的event类型。