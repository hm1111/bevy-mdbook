# 14.24. System Piping

你可以将多个Rust函数组合成一个Bevy系统。

你可以创建能够接受输入并产生输出的函数，然后将它们连接在一起，作为一个更大的系统运行。这被称为“系统管道”。

你可以将其视为创建由多个构建块组成的“模块化”系统。这样，你可以在多个系统中重用一些通用代码/逻辑。

请注意，系统管道不是系统间通信的方式。如果你想在系统之间传递数据，应该使用[事件](./events.md)。

你的函数将被组合在一起，Bevy 会将它们视为一个拥有所有组合系统参数以进行数据访问的单个大系统。

## 示例：处理 `Result`s

一个有用的系统管道应用是能够返回错误（允许使用Rust的 `?` 运算符），然后有一个单独的函数来处理它们：

```rust
fn net_receive(mut netcode: ResMut<MyNetProto>) -> std::io::Result<()> {
    netcode.send_updates(/* ... */)?;
    netcode.receive_updates(/* ... */)?;

    Ok(())
}

fn handle_io_errors(
    In(result): In<std::io::Result<()>>,
    // we can also have regular system parameters
    mut commands: Commands,
) {
    if let Err(e) = result {
        eprintln!("I/O error occurred: {}", e);
        // Maybe spawn some error UI or something?
        commands.spawn((/* ... */));
    }
}
```

这样的函数不能单独作为系统添加（Bevy不知道如何处理输入/输出）。通过将它们“管道”在一起，我们创建了一个有效的系统，我们可以添加：

```rust
app.add_systems(FixedUpdate, net_receive.pipe(handle_io_errors));
```

## 性能警告

警惕Bevy将整个链视为一个单一的大系统，拥有所有组合的系统参数及其各自的数据访问要求。这意味着并行性可能会受到限制，从而影响性能。

如果你创建了多个“管道系统”，它们都包含一个共同的函数，而这个函数包含任何可变的访问，那么这将阻止它们全部并行运行！