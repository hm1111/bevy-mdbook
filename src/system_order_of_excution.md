# 14.17. System执行顺序

Bevy 的调度算法旨在通过在可用的CPU线程上并行运行尽可能多的系统(system)来提供最佳性能。

当系统(system)在需要访问的数据上不发生冲突时，并行运行是可能的。但是，当系统(system)需要对一段数据进行可变（独占）访问时，需要访问相同数据的其他系统(system)就不能同时运行。Bevy 根据系统的函数签名（它采用的参数类型）确定所有这些信息（可变访问还是不可变访问）。

默认情况下system的执行顺序是不确定的。Bevy 不关心每个系统何时运行，每一帧它们的执行顺序都可能发生改变。

## 明确执行顺序

如果你需要一个system总是在另一个system之前/之后运行，你可以使用`before`和`after`来指定它们的执行顺序。

当有大量的system需要配置时，使用`before`和`after`来指定它们的执行顺序会变得很麻烦，这时可以使用`system sets`来组织它们。

## 执行顺序很重要吗？？？

根据自己的需求自行决定。

## 循环依赖

当多个系统相互依赖时，重新设计你的游戏逻辑是个好主意。