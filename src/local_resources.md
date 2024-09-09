# 14.13. 本地资源
本本地资源允许你拥有每个`System`的数据。这些数据不是存储在ECS World中，而是与你的`System`一起存储。在你的`System`之外，没有任何东西可以访问它。该值将在`System`的后续运行中保持。


## `Local<T>`和`ResMut<T>/Res<T>`的相同点和不同点

`Res<T>/ResMut<T>`和`Local<T>`都是Bevy引擎中的资源访问类型。

总的来说，`Res<T>/ResMut<T>`用于访问全局资源，而`Local<T>`用于访问每个系统（`system`）私有的局部资源。 

以下是对这两种类型的详细解释：

**`Res<T>/ResMut<T>`：**

    Res<T>和ResMut<T>是Bevy引擎中用于访问全局资源（Resource）的类型。

    Res<T>表示对资源的只读访问，而ResMut<T>表示对资源的可写访问。

    这些资源存储在Bevy的世界（World）中，可以被多个系统或组件访问。

    它们通常用于存储游戏中的一些全局数据，例如玩家得分、游戏状态等。

**`Local<T>`：**

    Local<T>是用于访问局部变量或上下文数据的类型。

    每个系统（system）可以有自己的Local<T>实例，它不依赖于实体（entity）和组件（component），这意味着它可以在系统中独立使用，而不需要与特定的实体或组件关联。

    Local<T>提供了对特定数据类型的可变访问权限，但其作用域仅限于拥有它的系统（System）。

    它通常用于存储系统内部使用的数据，例如计时器、状态标志或与特定系统操作相关的任何临时数据。


## 指定初始值

`Local<T>` 总是使用默认值初始化，为`Local<T>`指定自定义初始值是不可能的。因此必须实现 `Default` 或者 `FromWorld` trait， 这样会自动初始化。一个`system`可以有多个相同类型的`Local<T>`。

但也有其他方法：

1. 使用闭包

如果你想指定值，可以使用闭包，闭包必须是有效的Bevy `system`（参数都是`system`参数）。这样就可以用闭包"把数据移到函数"。
```rust
#[derive(Default)]
struct MyConfig {
    magic: usize,
}

fn my_system(
    mut cmd: Commands,
    my_res: Res<MyStuff>,
    // note this isn't a valid system parameter
    config: &MyConfig,
) {
    // TODO: do stuff
}

fn main() {
    let config = MyConfig {
        magic: 420,
    };

    App::new()
        .add_plugins(DefaultPlugins)

        // create a "move closure", so we can use the `config`
        // variable that we created above

        // Note: we specify the regular system parameters we need.
        // The closure needs to be a valid Bevy system.
        .add_systems(Update, move |cmd: Commands, res: Res<MyStuff>| {
            // call our function from inside the closure,
            // passing in the system params + our custom value
            my_system(cmd, res, &config);
        })
        .run();
}
```

2. 从 "构造器"中返回一个`system`。
```rust
#[derive(Default)]
struct MyConfig {
    magic: usize,
}

// create a "constructor" function, which can initialize
// our data and move it into a closure that Bevy can run as a system
fn my_system_constructor() -> impl FnMut(Commands, Res<MyStuff>) {
    // create the `MyConfig`
    let config = MyConfig {
        magic: 420,
    };

    // this is the actual system that bevy will run
    move |mut commands, res| {
        // we can use `config` here, the value from above will be "moved in"
        // we can also use our system params: `commands`, `res`
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)

        // note the parentheses `()`
        // we are calling the "constructor" we made above,
        // which will return the actual system that gets added to bevy
        .add_systems(Update, my_system_constructor())

        .run();
}
```