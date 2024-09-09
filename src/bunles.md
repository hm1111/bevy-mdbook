# 14.8. Bundles
可认为`Bundles`是创建`Entity`的模板。通过同一个`Bundle`创建的`Entity`具有相同的`Component`，从而简化创建`Entity`的流程。

相对于直接创建`Entity`，使用`Bundle`的好处：

- 减少代码量
- 减少出错的可能性，这样如果你创建`Entity`时遗漏了一个`Component`，编译器就会报错；

Bevy提供了一些内置`Bundle`来传建常用的entities,也可自定义`Bundle`;`Bundle`可嵌套其他`Bundle`;

Bevy也会把任意Components的元组作为一个`Bundle`。（没有以上说的`Bundle`所具有的好处了）



## 创建Bundle
定义结构体，每一个字段就是一个`Component`，并派生`Bundle` trait。若想定义默认值，可派生`Default` trait/实现`Default` trait。

### 使用Bundle


## 移除bundle
可以从一个`Entity`中移除指定的`Bundle`，`Entity`中相关的`Component`将被移除。

## 查询
不能为一个`Bundle`使用`Query`，`Bundle`只是方便创建`Entity`，`Query`中仍需要指定具体的`Component`，而不是指定`Bundle`。