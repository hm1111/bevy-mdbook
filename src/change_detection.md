# 14.21. Change Detection

检测数据何时改变，并在数据改变时执行一些任务。

主要用例之一是优化-仅在相关数据更改时才执行，以避免不必要的工作。另一个用例是在更改时发生触发特殊操作。

## Components的改变检测：添加/修改

### 查询

若指定的component被改变时执行查询。

2种情况：

1. `Added<T>`：检测新的component
    
    - 新的component被添加到entity中；
    - 带有指定component的entity被生成；

2. `Changed<T>`：检测component已改变

    - 当component被改变
    - 当新的component被添加到entity中（同`Added<T>`)


如果你只想知道entity中的某些component是否发生了改变，针对不可变访问，你需要在`Query`中使用`Ref<T>`参数代替`&`。针对可变访问，使用`&mut`即可，因为它总是会返回一个`Mut<T>`类型，检测改变的方法总是可用的。

## Resource的改变检测

在`Query`中使用`ResMut<T>`和`Res<T>`参数即可。

更改检测由 `DerefMut` 触发。简单地通过可变查询访问组件，或通过 `ResMut` 访问资源，而不实际执行 `&mut` 访问，不会触发它。这使得更改检测非常准确。

注意：如果你调用一个 Rust 函数，它接受 `&mut T `（可变借用），那也算数！即使该函数实际上没有执行任何更改，它也会触发更改检测。

此外，当你改变一个组件时，Bevy 不会检查新值是否真的与旧值不同。它将始终触发更改检测。如果您想避免这种情况，只需自己检查一。

变更检测是按系统粒度进行的，并且是可靠的。一个系统只有在以前没有看到过这些变化时才会检测到它们。

和event不一样，当sytem只是偶尔执行时，你也不用当心错过改变。

### 可能的陷阱

注意帧延迟/帧滞后，当改变system在检测system之前运行，检测system只能在下一帧中检测到改变。若想在同一帧中检测改变，你需要使用[精准的系统顺序](./system_order_of_excution.md)


## 移除检测

移除检测比较特殊，不同于改变检测，移除后的数据将不存在，后续也就没法追踪它的元数据了。


### Components

将要被移除的数据都是在每一帧的最后才会被移除。你应该确保检测system在移除system之后运行。

使用`RemovedComponents<T>`参数，它的行为和`EventReader`很像，但它给了你component被删除的entity的ID（也包含被删除的entity的ID）。

### Resources

Bevy没有为resource的移除提供检测API。但可以使用 `Option` 和单独的 `Local` 系统参数来解决此问题，从而有效地实现你自己的检测。请注意，由于此检测是系统本地的，因此不必在同一帧更新期间进行。