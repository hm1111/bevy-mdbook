# 5.5. 日志/控制台消息

`LogPlugin` 插件提供了一个 `App` 实例的日志记录系统，它是`DefaultPlugins` 的一部分。

## 日志等级

日志等级明确了日志信息的重要性，还可通过它对日志信息进行过滤。

日志等级有以下几种（等级由高到低）：

1. `off`：关闭所有日志信息；

2. `error`：打印导致程序崩溃的错误信息；

3. `warn`：打印可能导致程序崩溃的警告信息；

4. `info`：打印一般的信息；

5. `debug`：用于开发阶段，打印你的代码正在做什么的调试信息；

6. `trace`：打印非常详细的调试信息。

## 打印自定义的日志信息

使用与日志等级同名的宏来打印自定义日志信息。[更多信息请查看 std::fmt](https://doc.rust-lang.org/std/fmt/)

例如：

```rust
error!("Unknown condition!");
warn!("Something unusual happened!");
info!("Entered game level: {}", level_id);
debug!("x: {}, state: {:?}", x, state);
trace!("entity transform: {:?}", transform);
```

## 过滤日志信息

通过配置`LogPlugin`来控制你想要的日志信息。

```rust
use bevy::log::LogPlugin;

app.add_plugins(DefaultPlugins.set(LogPlugin {
    filter: "info,wgpu_core=warn,wgpu_hal=warn,mygame=debug".into(),
    level: bevy::log::Level::DEBUG,
}));
```

`filter`字段 是一个字符串，指定了不同 Rust 模块/crates的日志级别启用规则列表。在上面的示例中，字符串的意思是：默认显示到 `info` 级别，将 wgpu_core 和 wgpu_hal 限制在 `warn` 级别，对于 mygame 显示 `debug` 级别。

所有高于指定级别的日志也会被启用。所有低于指定级别的日志将被禁用，这些被禁用的消息将不会被显示。

`level`字段 是一个全局限制，规定了要使用的最低日志级别。低于该级别的消息将被忽略，从而避免了大部分的性能开销。

### 环境变量

当运行应用程序时，你可以使用`RUST_LOG`环境变量来覆盖`filter`字段的值。

`RUST_LOG=warn,mygame=debug ./mygame.exe`

对`RUST_LOG="debug" cargo run`使用相同的环境变量可能会出现意想不到的结果，比如：它还会过滤来自`cargo`的debug信息。

### 针对调试版本和发行版本的不同设置

如果你想在 Rust 代码的调试构建和发布构建中执行不同的操作，一个简单的方法是使用“调试断言”的条件编译。

[在开发阶段为什么不建议使用`release`模式？](./low_performance.md)

在Windows系统中，Bevy应用程序运行时默认会打开一个控制台窗口来显示日志信息，如果你不希望在发布版本中显示日志信息，[查看这里](./windows.md#disabling-the-windows-console)。

## 性能影响

打印消息到控制台是一个相对较慢的操作。

然而，如果你没有打印大量的消息，就不必担心速度问题。只需避免在代码的性能敏感部分（如内部循环）中大量打印消息。

在发布版本中，你可以禁用`trance`和`debug`等日志级别。