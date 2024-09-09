# 14.19. System sets

system sets能够很容易为多个system应用相同的配置，例如：运行条件，执行顺序。

你添加到system sets中的配置会自动应用到属于这个system sets的所有system。

一个system可以属于多个system sets，同时它也将应用这些system sets的配置。你也可以为单独的system添加额外的配置。


## 匿名sets
最简单的sets就是使用`.add_systems`方法添加多个system的元组，这个元组就是一个system sets。

## 命名sets

这种更正式，也更强大。

你可以创建结构体/枚举作为一个标签/标识符，并派生`SystemSet` trait以及其他必要的trait，之后就可以在其他地方引用这个system sets。

对于单个system set，你可以创建一个空结构体；对于多个有关联的system set，你可以创建一个枚举，每一个枚举成员都是一个单独的system sets。

你也可以使用`.configure_sets`为system sets指定运行条件和执行顺序。

命名系统集(Named sets)的主要用例是逻辑组织，以便你可以管理系统并引用整个组。

### 搭配Plugins

命名sets与plugins一起也非常有用。当你在编写plugin时，你可以公开（pub）一些system sets类型，以允许其他开发者控制plugin中的事物如何运行，或者他们与你的plugin有关的事物如何运行。这样你就不必公开任何单个系统(System)。

## 常见陷阱

system sets的配置只存储在单个shedule中，当你在其他shedule中使用同一个system set，它将不会使用你在其他shedule中的配置。