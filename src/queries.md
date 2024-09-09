# 14.9. Queries
用来过滤符合条件的`Entity`。

## 访问数据

`Query`中第一个泛型参数就是你要访问的数据。

`&`表示共享/只读的数据,`&mut`表示独占/可变数据。

访问可能不存在的`component`时使用`Option`(找到的`entity`中可能没有这个`component`)。

如果想要访问多个`component`,把它们放到元组中。

不能为`Query`的参数指定`Bundle`。`Bundle`只是用来创建/修改/删除`Entity`的，不能用来访问数据。


### for循环访问数据

`Query`是一个`Iterator`，所以你可以使用`for`循环来遍历它。你也可以调用`for_each`方法并且传递一个闭包来处理每个`entity`。

### 访问指定的Entity

如果想访问指定的`Entity`，你需要知道`Entity`的ID。你可以在`Query`中指定`Entity`类型得到`Entity`的`ID`。

`fn do_something(queries: Query<(Entity, /* ... */)>)`

### 访问单个Entity

如果你知道`query`只会匹配一个`entity`,你可以使用`single/single_mut`(panic on error) 或者 `get_single/get_single_mut`(返回`Result`)。这些方法用于结果只会是一个`entity`的情况,但是如果有多个`entity`会导致运行时错误。

### 一次性获取所有Entity的数据
使用`many/many_mut `(`panic`)或者 `get_many/get_many_mut`(返回`Result`)来一次性获取所有`Entity`的数据


### Combinations(组合)
多个entity之间的两两组合，三三组合，...

方法：`iter_combinations/iter_combinations_mut`

#### Query过滤器（缩小查询范围）
使用`Query`的第二个泛型参数（可选）来过滤，使用`With/Without`来限定`Entity`是否有对应的`Component`。

##### 什么时候使用？

如果你不关心这些`component`中存储的数据,而是只希望找到的实体有没有指定的`component`,这会很有用。 如果你想要数据,把`component`放到第一个泛型参数中,而不是使用过滤器。(不想要使用数据时，放入第二个参数；没有数据时，放入第二个参数)

##### 绑定多个过滤器
1. (filter1, filter2, ...)：满足所有过滤器（与逻辑）
2. Or<(filter1, filter2, ...)>：满足任何一个过滤器（或逻辑）

#### Query Transmutation(转换)
如果想你在一个`Query`函数中调用另一个`Query`函数，那么你可以通过使用`QueryLens`来获取你所需的`Query`。
```rust
fn debug_positions(query: Query<Transform>) {
    todo!()
}

fn move_player(query: Query<&mut Transform, With<Player>>) {
    todo!()
    let mut lens = query.transmute_lens::<&Transform>();
    debug_positions(lens.query());
}
```
`Query`会被转换成一个`QueryLens`，然后你可以调用它的`query`方法来获取`Query`，传递给另一个`Query`函数。

*该操作有性能开销，所以你应该只在需要的时候使用它。*


#### Query Joining
把2个查询结果合并成一个它们共有的部分。