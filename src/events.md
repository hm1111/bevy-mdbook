# 14.11. Events
让`Systems`之间彼此沟通，在`System`中发送数据。

`Events`是非常有用的抽象层，帮助你解耦。你可以分离不同的功能块，更容易明确`system`的责任范围。

## 创建event
创建结构体/枚举，并派生`Event` trait，之后就可以在`System`中使用该类型， 同时还需要通过app builder将该类型注册到应用中。

1。 发送数据：使用`EventWriter<T>`。

2。 接收数据：使用`EventReader<T>`。


## 注册event
`app.add_event::<YourEvent>();`

## 使用建议(待补充)
 

## 如何工作
当注册event类型的时候，Bevy会创建一个`Events<T>` resource，类似于一个后台存储的event队列。 Bevy也提供了一个定期清理event的 "event maintenance" system，防止他们累积从而耗尽内存。

Bevy确保event在当前帧和下一帧都保留（帧更新循环/固定时间步循环），这样即使它们在当前帧没有得到处理,你的system仍可在下一帧读取event，之后它们才会被自动清理掉。如果不喜欢这种自动清理方式，你可以手动控制event何时清理(如果忘记清理，会导致内存泄露/浪费)。

`EventWriter<T>` system参数只是向队列添加event的语法糖，它用来可变地访问`Events<T> resource`。

`EventReader<T>`稍复杂一点：它不可变地访问event，但是它还会存储一个用来追踪所读取的event的数量的计数器，这也是它需要使用`mut`关键字的原因。

`Event<T>`在内部使用`Vec`来存储event，发送event等同于向`Vec`添加event。相比于使用`change detection`，使用event会有更好的性能。

## 可能的陷阱
小心**帧延迟/帧滞后**。接收system先于发送system运行是可能的。此时接收system在下一帧更新时只有一次机会接收event。如果要确保event被立即/同帧处理，可以使用[explicit system ordering](https://bevy-cheatbook.github.io/programming/system-order.html)。

当system有运行条件(run condition)，它可能会在没有运行时错过一些event。当你的system没有在每隔一帧或固定时间步至少检查一次event，event将被丢失。

如果你希望event持续时间的更长，你可以实现[自定义清理/管理策略](https://bevy-cheatbook.github.io/patterns/manual-event-clear.html)。 但是你也只能处理你自己的event类型，不能处理Bevy内置的event类型。