# Bevy内置类型列表

## SystemParams(系统参数)

一些可被用来作为系统参数(system parameter)的类型。

**主要分为2大类：**

- 在常规系统中使用的参数

- 在独占系统(exclusive system)中使用的参数

[参考链接](https://bevy-cheatbook.github.io/builtins.html#list-of-bevy-builtins)

每个系统(system)支持最多16个参数，可以把它们放入元组中以支持更多数目的参数，每个元组最多有16个元素，允许元组嵌套。

## Assets(资产)

[一些asset类型](https://bevy-cheatbook.github.io/builtins.html#assets)

## File Formats(文件格式)

使用`cargo features`来启用/关闭对文件格式的支持。同时还提供了一些非官方插件以支持其他的文件格式。

主要的文件格式有：

- 图片

- 音频

- 3D资产

- 着色器

- 字体

- Bevy场景

[参考链接](https://bevy-cheatbook.github.io/builtins.html#file-formats)

## GLTF Asset Labels(GLTF资产标签)

[参考链接](https://bevy-cheatbook.github.io/builtins.html#gltf-asset-labels)

## Shader Imports(TODO)

## wgpu backend

在GLES3和WebGL2中，一些渲染功能是不可用的；WebGPU还处于实验阶段，只有少数浏览器支持。

[参考链接](https://bevy-cheatbook.github.io/builtins.html#wgpu-backends)

## Shedules(计划)

3种内置shedules：

1. `Main`：用来运行其他一系列的shedules（游戏逻辑），每一帧都运行一次；

2. `Extra`：将数据从`Main World`复制到`Render World`，运行在`Main` shedule之后；

3. `Render`：执行渲染、图像相关的任务，运行在`Extra` shedule之后；

第一帧运行的shedules：

1. `PreStartup`

2. `Startup`

3. `PostStartup`

每一帧都运行的shedules：

1. `First`：每一帧开始的初始化任务；

2. `PreUpdate`：在用户逻辑之前运行的引擎内部系统；

3. `StateTransition`：将要执行的状态转换；

4. `RunFixedUpdateLoop`：运行`FixedUpdate` shedule；

5. `Update`：每一帧需要运行的用户逻辑；

6. `PostUpdate`：在用户逻辑之后运行的引擎内部系统；

7. `Last`：每一帧结束时的清理任务；

`FixedUpdate` shedule用于运行需要固定时间间隔的逻辑。

当需要改变状态(change state)时，需要在`StateTransition` shedule中执行`OnEnter`、`OnTransition`和`OnExit` shedule。

`Render` shedule用`RenderSet`来组织。

[Render shedule 相关信息。](https://bevy-cheatbook.github.io/builtins.html#schedules)

## Run Conditions(运行条件 TODO)

## Plugins(插件 TODO)

## Bundles(包)

用来生成不同的entities。

任何数量不超过15个component类型的元素组成的元组都是有效的bundle。

一些内置的bundle：

- 通用的bundles：

    - `SpatialBundle`

    - `TransformBundle`

    - `VisibilityBundle`

- 场景有关的bundles：
    
    - `SceneBundle`

    - `DynamicSceneBundle`

- 音频有关的bundles：

    - `AudioBundle`

    - `SpatialAudioBundle`

    - `AudioSourceBundle`

    - `SpatialAudioSourceBundle`

- Bevy 3D有关的bundles：

    - `Camera3dBundle`

    - `TemporalAntiAliasingBundle`

    - `ScreenSpaceAmbientOcclusionBundle`

    - ......

- Bevy 2D有关的bundles：

    - `Camera2dBundle`

    - `SpriteBundle`

    - `SpriteSheetBundle`

    - ......

- Bevy UI有关的bundles：

    - `NodeBundle`

    - `ButtonBundle`

    - `ImageBundle`

    - ......

## Resources(资源)

### 配置资源

这些资源允许你改变Bevy不同部分如何工作的设置，这些资源通常在`App`的构建阶段设置，在`App`的运行阶段可以被修改。

在运行时不可修改的设置不会使用资源表示,而是通过相应的插件(plugins)进行配置。

其中的一些配置资源：

- `ClearColor`

- `GlobalVolume`

- `AmbientLight`

- .....

### 引擎资源

这些资源在运行时提供对游戏引擎不同功能的访问。

#### 主世界资源(main world resources)

- `Time`

- `FixedTime`

- `AssetServer`

- ......

#### 渲染世界资源

这些资源存在于渲染世界中，能被渲染系统(render system)访问：

- `MainWorld`

- `RenderGraph`

- `PipelineCache`

- ......

#### 底层wgpu资源

使用这些资源，你可以直接通过`wgpu` API来控制GPU。

在主世界和渲染世界都可以使用的资源：

- `RenderDevice`

- `RenderQueue`

- `RenderAdapter`

- `RenderAdapterInfo`


### 输入处理资源

这些资源代表不同输入设备的当前状态，你可以在系统(system)中读取它们来处理用户输入。

- `Input<KeyCode>`

- `Input<MouseButton>`

- `Input<GamepadButton>`

- ......


## Events(事件)

[参考链接](https://bevy-cheatbook.github.io/builtins.html#events)

### 输入事件

输入事件被输入设备激活。

- `MouseButtonInput`

- `MouseWheel`

- `MouseMotion`

- ......


### 引擎事件

应用运行时发生的各种应用内部事件。

- `AssetEvent<T>`

- `HierarchyEvent`

- `AppExit`

### 系统和控制事件

来自操作系统/窗口系统的事件，或者用于控制Bevy的事件。

- `RequestRedraw`

- `FileDragAndDrop`

- `CursorEntered`

- ......


## Components(组件)

[原文链接](https://bevy-cheatbook.github.io/builtins.html#components)

[Components相关文档](https://docs.rs/bevy/latest/bevy/ecs/component/trait.Component.html#implementors)