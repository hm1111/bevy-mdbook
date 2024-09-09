# 14.12. Plugins
在Bevy游戏引擎中，插件是一种组织和管理游戏逻辑的方式。插件可以包含系统（systems）、资源（resources）、组件（components）、事件（events）等。当你使用`App::add_plugins(...)`添加一个插件时，Bevy会将插件中的所有内容合并到你的应用程序（App）中。

这种合并是扁平化的，意味着插件中的所有内容都会被添加到App的顶层，而不会创建任何嵌套的结构。例如，如果你有一个插件包含一个系统和一个资源，这两个元素将直接添加到App的系统列表和资源列表中。

这种设计提供了极大的灵活性，允许开发者以模块化的方式构建游戏。每个插件可以专注于特定的功能领域，并且可以轻松地在不同的项目中重用。此外，Bevy的插件系统还允许开发者覆盖或修改插件中的默认行为，以满足特定项目的需求。

对于你自己项目中的内部组织，插件的主要价值在于不必将所有 Rust 类型和函数声明为 `pub`，这样就可以从 `fn main` 访问它们并添加到应用程序构建器(App Builder)中。它允许你能够在任意地方给应用添加内容，就像Rust模块/Rust文件/Rust crates一样。

## 内置Plugins
Bevy 提供了一些内置的Plugins， 它们可以让你快速的开始开发游戏。

### 自定义Plugins
2种方法：

1. 创建一个带`&mut App`的函数。

    好处：简单

2. 定义一个类型，实现`Plugin` trait。

    好处：如果你想让你的插件可配置，你可以使用配置参数或泛型来扩展它。

## 使用建议
1. 为不同的状态([States](https://bevy-cheatbook.github.io/programming/states.html))创建不同的Plugins；
2. 为不同的子系统(SubSystems)创建不同的Plugins，比如物理/输入处理等等；

## PluginGroup

PluginGroup 是一个插件的集合，它允许你在一个地方定义一组插件，并在需要时一次性添加它们。这对于组织和管理插件非常有用。

给app插入`plugin`时，你可以禁用一部分插件。请注意，这只是简单地禁用了功能，但它不是真正删除代码以避免应用程序的二进制文件的膨胀，被禁用的插件仍然会被编译进你的程序中。

如果你想给你的编译产物瘦身，你应该禁用 Bevy 的默认cargo features，或者单独使用 Bevy 的各种子库。

### 自定义PluginGroup
自定义类型，并实现`PluginGroup` trait。

## Plugin配置
Plugin也是一个存储 在程序初始化/启动期间所需的设置/配置 的好地方，对于那些运行时可能变化的设置，推荐放进`resource`中。

被添加到`PluginGroup`中`plugin`也可以被配置。bevy `DefaultPlugins`中的很多就是这样工作的。

## 发布plugin到Crates
Plugin给你了一个很好的途径来发布基于bevy的库，让其他人可以轻松集成进他们的项目。

如果你想把plugin发布成公共crate，你需要了解[插件开发指南](https://github.com/bevyengine/bevy/blob/main/docs/plugins_guidelines.md)。

别忘了在官网上提交进入[Bevy Assets](https://bevyengine.org/assets)的申请，这样大家可以更容易地找到你的插件。在[Github 仓库](https://github.com/bevyengine/bevy-assets)提交一个PR就行。

如果你有兴趣支持前沿Bevy(main)，[看这里](https://songpinru.github.io/setup/bevy-git.html#advice-for-plugin-authors)。