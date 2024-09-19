# 14.23. Internal Parallelism

内部并行性是指系统内部的多线程处理。

Bevy 中通常的多线程处理是在可能的情况下并行运行每个系统（当与其他系统没有冲突的数据访问时）。这被称为“外部并行性”。

然而，有时你需要编写一个系统，它必须处理大量的实体或事件。在这种情况下，简单的查询或事件迭代将无法扩展以充分利用 CPU。

Bevy 提供了一个解决方案：并行迭代。Bevy 将自动将所有实体/事件分成大小合适的批次，并在单独的 CPU 线程上迭代每个批次，调用你提供的函数/闭包。

如果只有少数实体/事件，Bevy 将自动回退到单线程迭代，其行为将与你正常迭代时完全相同。在实体/事件数量较少时，这比多线程处理更快。

即使并行迭代应该能够根据实体/事件的数量自动做出正确的决策，但它的使用更为复杂，并不总是合适的，因为你必须在闭包内部完成所有操作，并且存在其他限制。

此外，如果你的系统不太可能遇到大量的实体/事件，那么就不要费心使用它，只需正常迭代你的查询和事件即可。

## 并行查询迭代

查询支持并行迭代，它允许你使用 cpu 多线程来处理大量实体。

```rust
fn my_particle_physics(
    mut q_particles: Query<(&mut Transform, &MyParticleState), With<MyParticle>>,
) {
    q_particles.par_iter_mut().for_each(|(mut transform, my_state)| {
        my_state.move_particle(&mut transform);
    });
}
```

并行迭代的一个限制是，Rust 的安全特性不允许你跨 CPU 线程共享可变借用。因此，不可能修改当前实体组件之外的任何数据。

如果你需要修改共享数据，可以使用类似于 `Mutex` 的东西，但要注意增加的开销。它很容易抵消并行迭代带来的任何好处。

## 并行命令

如果你需要使用命令(commands)，可以使用 `ParallelCommands` 系统参数。它允许你从并行迭代闭包内访问 `Commands`。

```rust
fn my_particle_timers(
    time: Res<Time>,
    mut q_particles: Query<(Entity, &mut MyParticleState), With<MyParticle>>,
    par_commands: ParallelCommands,
) {
    q_particles.par_iter_mut().for_each(|(e_particle, mut my_state)| {
        my_state.timer.tick(time.delta());

        if my_state.timer.finished() {
            par_commands.command_scope(|mut commands| {
                commands.entity(e_particle).despawn();
            })
        }
    });
}
```

然而，一般来说，在 Bevy 中使用命令(commands)是一种低效的方式，并且它们在处理大量实体时扩展性不佳。如果你需要在大量实体上生成/销毁或插入/移除组件，你可能应该从一个[独占系统](./exclusive_system.md)中去做，而不是使用命令。

在上面的例子中，我们更新存储在许多实体上的计时器(timer)，并使用命令来销毁任何时间已经过去的实体。这是命令的一个很好的用途，因为计时器需要为所有实体计时，但只有少数实体可能需要同时销毁。

## 并行事件迭代

`EventReader<T>` 为事件提供了并行迭代，它允许你通过 cpu 多线程来处理大量事件。

```rust
fn handle_many_events(
    mut evr: EventReader<MyEvent>,
) {
    evr.par_read().for_each(|ev| {
        // TODO: do something with `ev`
    });
}
```

并行事件迭代一个缺点是，你不能用它来处理需要按顺序处理的事件。在使用并行迭代时，事件的顺序变得不确定。

然而，如果你使用 `.for_each_with_id`，你的闭包将被赋予一个 `EventId`，这是一个顺序索引，用于指示你当前正在处理的事件。这可以帮助你知道你在事件队列中的位置，即使你仍然以未定义的顺序处理事件。

并行事件处理的另一个缺点是，通常你需要为能够响应事件而更改某些数据，但是，在安全的 Rust 中，不可能跨 CPU 线程共享对任何事物的可变访问权限。因此，对于大多数用例，并行事件处理是不可能的。

如果你使用像 `Mutex` 这样的东西来共享数据访问权限，同步开销可能会严重影响性能，你还不如使用常规的单线程事件迭代。

## 控制批次大小

批量大小和并行任务的数量是使用智能算法自动选择的，这些算法基于需要处理的实体/事件的数量，以及Bevy ECS如何在内存中存储/组织实体/组件数据。然而，它假定你为每个实体所做的工作量/计算量大致相同。

如果你发现你想要手动控制批量大小，你可以使用 `BatchingStrategy` 指定最小值和最大值。

```rust
fn par_iter_custom_batch_size(
    q: Query<&MyComponent>,
) {
    q.par_iter().batching_strategy(
        BatchingStrategy::new()
            // whatever fine-tuned values you come up with ;)
            .min_batch_size(256)
            .max_batch_size(4096)
    ).for_each(|my_component| {
        // TODO: do some heavy work
    });

    q.par_iter().batching_strategy(
        // fixed batch size
        BatchingStrategy::fixed(1024)
    ).for_each(|my_component| {
        // TODO: do some heavy work
    });
}
```

## 并行处理任意数据

内部并行性不仅限于 ECS 构造，比如实体/组件或事件。还可以并行处理切片（或任何可以作为切片引用的东西，如 `Vec`）。如果你有一个大的任意数据缓冲区，这就是为你准备的。

使用 `.par_splat_map/.par_splat_map_mut` 在多个并行任务中分配工作。如果将任务数指定为 `None`，则会自动使用可用的 CPU 线程总数。

使用 `.par_chunk_map/.par_chunk_map_mut` 手动指定特定的块大小。

在这两种情况下，你都需要提供一个闭包来处理每个块（子切片）。它将获得其块的起始索引及其块切片的引用。你可以从闭包中返回值，它们将被连接并作为 `Vec` 返回给调用点。

```rust
use bevy::tasks::{ParallelSlice, ParallelSliceMut};

fn parallel_slices(/* ... */) {
    // say we have a big vec with a bunch of data
    let mut my_data = vec![Something; 10000];

    // and we want to process it across the number of
    // available CPU threads, splitting it into equal chunks
    my_data.par_splat_map_mut(ComputeTaskPool::get(), None, |i, data| {
        // `i` is the starting index of the current chunk
        // `data` is the sub-slice / chunk to process
        for item in data.iter_mut() {
            process_thing(item);
        }
    });

    // Example: we have a bunch of numbers
    let mut my_values = vec![10; 8192];

    // Example: process it in chunks of 1024
    // to compute the sums of each sequence of 1024 values.
    let sums = my_values.par_chunk_map(ComputeTaskPool::get(), 1024, |_, data| {
        // sum the current chunk of 1024 values
        let sum: u64 = data.iter().sum();
        // return it out of the closure
        sum
    });

    // `sums` is now a `Vec<u64>` containing
    // the returned value from each chunk, in order
}
```

当你在 Bevy 系统中使用这个 API 时，在 `ComputeTaskPool` 上生成你的任务。

这个 API 在你进行[后台计算](./background_computation.md)以获得额外的并行性时也很有用。在这种情况下，请使用 `AsyncComputeTaskPool` 代替。

## 作用域内任务

作用域内任务实际上是上述所有抽象（并行迭代器和切片）的基础。如果之前讨论的抽象对你没有用，你可以通过自己生成作用域任务来实现任何自定义处理流程。

作用域任务允许你从父函数中借用任何你想要的东西。[作用域](https://docs.rs/bevy/latest/bevy/tasks/struct.Scope.html)将等待任务返回，然后再返回到父函数。这确保你的并行任务不会比父函数更长寿，从而实现“内部并行”。

为了获得性能提升，请确保你的每个任务都有大量且大致相似的工作要做。如果你的任务完成得非常快，并行的开销可能会超过收益。

```rust
use bevy::tasks::ComputeTaskPool;

fn my_system(/* ... */) {
    // say we have a bunch of variables
    let mut a = Something;
    let mut b = Something;
    let mut more_things = [Something; 5];

    // and we want to process the above things in parallel
    ComputeTaskPool::get().scope(|scope| {
        // spawn our tasks using the scope:
        scope.spawn(async {
            process_thing(&mut a);
        });
        scope.spawn(async {
            process_thing(&mut b);
        });

        // nested spawning is also possible:
        // you can use the scope from within a task,
        // to spawn more tasks
        scope.spawn(async {
            for thing in more_things.iter_mut() {
                scope.spawn(async {
                    process_thing(thing);
                })
            }
            debug!("`more_things` array done processing.");
        });
    });

    // at this point, after the task pool scope returns,
    // all our tasks are done and everything has been processed
}
```

当你在 Bevy 系统中使用这个 API 时，在 `ComputeTaskPool` 上生成你的任务。

这个 API 在你进行[后台计算](./background_computation.md)以获得额外的并行性时也很有用。在这种情况下，请使用``AsyncComputeTaskPool` 代替。