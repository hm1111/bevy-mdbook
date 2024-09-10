# Non-Send Resources

non-send 代表着数据只能被应用的主线程访问。比如：Rust中被`!Send`标记的数据。

有些库具有不能被其他线程安全使用的接口，一个常见的例子是图形，窗口，音频，网络，文件系统等等各种底层的操作系统接口。如果你在做一些更高级的事情，比如为了与这些东西交互而创建一个 Bevy 插件，你可能会遇到这种需求。

通常地，Bevy会利用多个cpu内核并使用线程池来运行你的系统(system)。但是有时候你需要确保一些代码只能在主线程中运行，或者以多线程的方式来访问一个不安全的数据。

## Non-Send System and Data Access

使用`NonSend<T>`和`NonSendMut<T>`系统参数，它们的行为就像`Res<T>`和`ResMut<T>`一样。

这允许你访问ECS资源(ECS resources)，不同的是，这些参数的存在会迫使Bevy shedules在主线程中运行系统(system)。这确保了数据永远不必在线程之间传送或从不同的线程访问。

## 自定义Non-Send Resource

通常来说，为了插入resource,它的类型必须是`Send`的。但是，Bevy可以独立跟踪non-send资源，这些资源只能使用`NonSend<T>`/`NonSendMut<T>`访问。

不允许使用`Commands`插入non-send资源，只能使用`World`。这意味着你只能在独占系统(exclusive system)，`FromWorld` impl或者在app builder中初始化这些资源。

如果你需要一个只能在主线程中运行的系统，并且不关心存储的数据，可以使用`NonSendMarker`作为资源。