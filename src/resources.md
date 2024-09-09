# 14.6. Resources

`Resources`用来存储全局唯一的数据，而不依赖`entities`。可以在任何时候，任何地方被访问。

### 创建
使用`struct/enum`创建一个结构体/枚举，并派生`Resource` trait。

该类型的实例最多只有一个，否则考虑使用 `entities/components`。

### 访问
在`system`中使用`Res/ResMut`参数访问`Resouces`。

### 管理
在运行时创建/移除`resource`，可以使用`Commands`参数。
可通过[独占system](./独占system.md)来管理。

### 初始化Resources
2种方法：
1. 派生/实现`Default` trait；
    
    第1步：定义结构体/枚举；

    第2步：派生/实现`Default` trait；

2. 实现`FromWorld` trait；（针对复杂的Resources）

### 使用建议
1. 如果有的东西运行时不会改变并且只会在初始化plugin时使用,考虑把他们放在plugin的struct,而不是变成resource。
2. 如果你想集成一些外部非bevy的软件进Bevy app,创建一个resource来保存它的状态/数据是非常方便的。
3. 如果外部代码不是线程安全的（在Rust术语中称为`!Send`），这对于非Rust（例如C++和操作系统级别）的库来说是很常见的， 你应该使用一个非`Send`的Bevy resource。这将确保任何接触该resource的Bevy system都会在主线程上运行。
4. 全局数据使用`Resources`，否则使用`Entities/Components`；

#### Settings
存储应用的设置/配置。

但是，如果它是在运行时无法更改并且仅在初始化插件时使用的内容，请考虑将其放在插件的`struct`中，而不是`resources`中。

#### Caches

### Interfacing with external libraries
集成第三方非Bevy库到Bevy应用中。

若第三方库不是线程安全的，你应该使用 `Non-Send` Bevy Resources来代替。
- [Non-Send](https://bevy-cheatbook.github.io/programming/non-send.html)