# 11.6. 热加载资产

在运行时，如果你修改了一个已经加载到游戏中的资产文件（通过 [`AssetServer`]((https://docs.rs/bevy/latest/bevy/asset/struct.AssetServer.html)），Bevy 可以检测到这一点并自动重新加载该资产。这对于快速迭代非常有用。你可以在游戏运行时编辑你的资产，并立即在游戏中看到更改。

并非所有文件格式和使用案例都得到同样好的支持。典型的资产类型，如纹理/图像，应该没有问题，但复杂的 GLTF 或场景文件，或涉及自定义逻辑的资产，可能不支持。

如果你需要在热重载工作流程中运行自定义逻辑，你可以在系统中实现它，使用 [`AssetEvent`](https://docs.rs/bevy/latest/bevy/asset/enum.AssetEvent.html)（[了解更多](./react_to_change_with_asset_events.md)）。

热重载是可选的，必须启用才能工作：

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(AssetPlugin {
            watch_for_changes: true,
            ..Default::default()
        }))
        .run();
}
```

请注意，这需要 Bevy 的 `filesystem_watcher` cargo feature。默认情况下它是启用的，但如果你禁用了默认特性来定制 Bevy，请在需要时确保包含它。

## 着色器

Bevy 还支持对着色器的热重载。你可以编辑自定义的着色器代码，并立即看到更改。

这适用于从文件路径加载的任何着色器，例如在你的材质定义中指定的着色器，或者通过 [`AssetServer`](https://docs.rs/bevy/latest/bevy/asset/struct.AssetServer.html) 加载的着色器。

如果着色器代码不是来自于资产文件，例如如果你在源代码中包含它作为静态字符串，那么它就不能被热重载（原因很明显）。