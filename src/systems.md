# 14.5. Systems

`systems`是实现游戏逻辑的地方，实际上就是Rust函数。这些函数只能接受**特定类型的参数**，否则会出现编译2错误。

一些参数：
1. Res/ResMut：访问Resources；
2. Query：访问Components；
3. Commands：创建/销毁entities，components，resources；
4. EventWriter/EventReader：事件相关；
5. 其他...
- [可用参数列表](https://docs.rs/bevy/latest/bevy/ecs/system/trait.SystemParam.html#implementors)

最多接受16个参数，若需要更多的参数，可使用元组。

为了运行这些System，需要使用`app builder`来配置。


### 独占system

它拥有ECS World的完全访问权限，可以访问任意所需的数据，但是不能并行执行。

### One-Shot Systems

当你不想要Bevy执行你的`system`，而只想给自己使用时，那么可以使用`One-Shot systems`，这时不用把`system`添加到`shedules`中。
- [One-Shot Systems](https://bevy-cheatbook.github.io/programming/one-shot-systems.html)
