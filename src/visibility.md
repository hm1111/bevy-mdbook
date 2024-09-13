# 5.3. 可见性

可见性被用来控制一个事物是否被渲染。如果你想让游戏世界中的实体不被显示出来，你可以选择隐藏它。

## 可见性组件

在Bevy中，通过多个组件来表示可见性的：

1. `Visibility`

2. `InheritedVisibility`

3. `ViewVisibility`

游戏世界中能被渲染的实体必须有这三个组件。Bevy中所有的内置bundle都包含了这三个组件。

如果你不想使用这些bundle来创建实体，可以使用以下这些bundle来创建：

1. `SpatialBundle`：变换 + 可见性；

2. `VisibilityBundle`：可见性；

如果你没能正确地使用它们，这些实体将不会被渲染。

### Visibility

可见性是“面向用户的切换开关”。在这里，你可以为当前实体指定你想要的内容：

1. `Inherited`：继承父实体的可见性；

2. `Hidden`：总是隐藏当前实体；

3. `Visible`：总是显示当前实体；

若一个实体有父级，当父级丢失了visibility组件时，它的行为就和它没有父级一样。

### InheritedVisibility

继承可见性（`InheritedVisibility`）表示当前实体根据其父实体的可见性所应具有的状态。

继承可见性的值应被视为只读。它由Bevy内部管理，类似于变换传播的方式。一个“可见性传播”系统在`PostUpdate` shedule中运行。

如果你想读取当前帧的最新值，你应该将你的系统添加到`PostUpdate` shedule中，并将其顺序设置在`VisibilitySystems::VisibilityPropagate`之后。

### ViewVisibility

`ViewVisibility` 表示 Bevy 对该实体是否需要渲染的最终决定。

`ViewVisibility` 的值是只读的，它由 Bevy 内部管理。

它用于“剔除”：如果实体不在任何相机或光源的范围内，它就不需要被渲染，因此 Bevy 会隐藏它以提高性能。

每个帧结束，经过“可见性传播”后，Bevy 会检查哪些实体可以被哪些视图（相机或光源）看到，并将结果存储在这些组件中。如果你想读取当前帧的最新值，你应该将你的系统添加到 `PostUpdate` 调度中，并将其顺序设置在 `VisibilitySystems::CheckVisibility` 之后。