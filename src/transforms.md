# 5.2. 变换

变换允许你在游戏世界中放置一个对象。它是对象的“平移(位置/坐标)”、“旋转”和“缩放”的组合。

你可以通过修改"平移"来移动对象，修改"旋转"来旋转对象，修改"缩放"来使对象变小或变大。

## 变换相关的组件

在Bevy中，使用`Transform`和`GlobalTransform`组件来表示对象的变换。

游戏世界中代表一个对象的`entity`都必须拥有这2个组件。Bevy中所有的内置bundle类型都包含它们。如果你想在不使用这些bundle的情况下创建一个entity，你可以使用以下任意一种bundle：

1. `SpatialBundle`：变换+可见性；

2. `TransformBundle`·：只有变换；

### Transform

它是一个包含平移、旋转和缩放的结构体，为了读取和操作这些值，你需要使用`Query`从system中访问。

若一个entity有一个父级，它的`transform`组件就会和父级关联起来，这就意味自己会跟随父级一起平移、旋转和缩放。

### GlobalTransform

它代表着世界中绝对全局位置。

如果某个entity没有父级，那么它的全局变换（`GlobalTransform`）将会与局部变换（`Transform`）相匹配。

`GlobalTransform`的值 Bevy 内部计算和管理的（称为“变换传播”）。

与 `Transform` 不同，`GlobalTransform` 的平移/旋转/缩放不能直接访问。数据是以优化的方式存储的（使用 `Affine3A`），并且在层级结构中可能存在无法表示为简单变换的复杂变换。例如，跨越多级父级的旋转和缩放组合，会导致剪切。

如果你想尝试将 `GlobalTransform` 转换回可操作的平移/旋转/缩放表示，你可以尝试以下方法：

1. `.translation()`;

2. `.to_scale_rotation_translation()`(可能无效)

3. `compute_transform()`(可能无效)

## 变换传播

`Transform`和`GlobalTransform`这两个组件是由 Bevy 内部的一个系统（即“变换传播系统”）同步的，该系统在 `PostUpdate` shedule 中运行。

注意：当你改变 `Transform` 时，`GlobalTransform` 不会立即更新。它们将不同步，直到变换传播系统运行。

如果你需要直接使用 `GlobalTransform`，你应该将你的系统添加到 `PostUpdate` shedule中，并将其顺序设置在 `TransformSystem::TransformPropagate` 之后。

## TransformHelper

如果你需要在一个必须在变换传播之前运行的系统中获取最新的`GlobalTransform`，你可以使用特殊的 `TransformHelper` 系统参数。
它允许你立即按需计算特定实体的`GlobalTransform`。

一个可能有用的例子是，一个系统使相机在屏幕上跟随一个实体。你需要更新相机的变换（这意味着你必须在 Bevy 的变换传播之前进行，这样它就可以考虑到相机的新变换），但你也需要知道你正在跟随的实体的当前最新位置。

`TransformHelper` 在内部的行为类似于两个只读查询。它需要访问 `Parent` 和 `Transform` 组件来完成其工作。这会与我们的其他 `&mut Transform` 查询发生冲突。这就是为什么在上面的示例中我们必须使用参数集(param sets)的原因。

请注意：如果你过度使用 `TransformHelper`，它可能会引发性能问题。它为你计算全局变换，但它不会更新存储在实体 `GlobalTransform` 中的数据。Bevy 稍后仍会在变换传播期间再次进行相同的计算。这会导致重复工作。如果你的系统可以在变换传播后运行，这样它就可以在 Bevy 更新后读取值，那么你应该优先这样做，而不是使用 `TransformHelper`。

