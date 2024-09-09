# 14.7. Entities, Componets

### 1. Entities
从概念上说，一个entity代表了一组不同`Component`的值。每一个Component就是一个Rust类型。

从技术上说，一个entity只是一个被用来寻找相关数据的整型ID。

在Bevy中，`Entity`由ID和generation这2个整数组成。这个ID在旧的`Entity`被销毁后，可被新的`Entity`重用，但generation的值会被更改。

可使用`Commands`和`exclusive World access`来创建和销毁`Entity`。若多个`Entity`存在相同的`Component`，可使用[Bundles](./bundles.md)简化生成`Entity`的流程。

### 2. Components
组件是和`Entity`关联的数据。

#### 2.1. 创建
使用Rust结构体/枚举 来定义，并派生`Component` trait。

#### 2.2. 不同类型的Components

##### 2.2.1. Newtype Components
Rust中的Newtype模式。

##### 2.2.2. Marker Components
不保存任何数据，仅仅用来标记entities。

使用Rust中的元组结构体/枚举 来创建。

#### 2.2. 访问 Components
使用`Query`在`System`中访问。通过`Query`，你可以访问任何匹配了query签名的entity的Components值。

#### 2.3. 添加/删除 Components
使用`Commands`/`World`来添加/删除entity中的Components。