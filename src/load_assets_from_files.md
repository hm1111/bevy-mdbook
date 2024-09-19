# 11.2. 从文件加载资产

使用 [`AssetServer`](https://docs.rs/bevy/latest/bevy/asset/struct.AssetServer.html) 资源从文件加载资产：

```rust
#[derive(Resource)]
struct UiFont(Handle<Font>);

fn load_ui_font(
    mut commands: Commands,
    server: Res<AssetServer>
) {
    let handle: Handle<Font> = server.load("font.ttf");

    // we can store the handle in a resource:
    //  - to prevent the asset from being unloaded
    //  - if we want to use it to access the asset later
    commands.insert_resource(UiFont(handle));
}
```

这会将资产加载排入队列以便在后台进行，并返回一个句柄。资产需要一些时间才能可用。在同一个系统中，你不能立即访问实际数据，但你可以使用句柄。

你可以使用句柄来生成实体，比如 2D 精灵、3D 模型和 UI，即使在资产加载完成之前。当资产准备就绪时，它们将“突然出现”。

请注意，你可以根据需要多次调用 `asset_server.load(…)`，即使资产正在加载或已经加载。它只会为你提供相同的句柄。每次调用它，它都会检查资产的状态，必要时开始加载，并给你一个句柄。

Bevy 支持加载多种资产文件格式，并且可以扩展以支持更多。使用的资产加载器实现是根据文件扩展名选择的。

## 无类型加载

如果你想要一个无类型句柄，可以使用 `asset_server.load_untyped(...)`。

无类型加载是可能的，因为 Bevy 总是使用文件扩展名检测文件类型。

### 加载文件夹

可以使用 `asset_server.load_folder(…)` 加载整个文件夹的资产，无论文件夹内有多少文件。这将返回一个包含所有无类型句柄的 `Vec<HandleUntyped>`。

```rust
#[derive(Resource)]
struct ExtraAssets(Vec<HandleUntyped>);

fn load_extra_assets(
    mut commands: Commands,
    server: Res<AssetServer>,
) {
    if let Ok(handles) = server.load_folder("extra") {
        commands.insert_resource(ExtraAssets(handles));
    }
}
```

加载文件夹的功能并不被所有的I/O后端支持。在 WASM/Web 环境下，它不起作用。

## 资产路径和标签

用来标识文件系统中资产的资产路径实际上是一个特殊的资产路径（[AssetPath]()），它由文件路径和一个标签组成。标签用于在同一文件中包含多个资产的情况。例如，GLTF文件可以包含网格、场景、纹理、材质等。

资产路径可以从字符串创建，标签（如果有）附加在“#”符号后。

```rust
fn load_gltf_things(
    mut commands: Commands,
    server: Res<AssetServer>
) {
    // get a specific mesh
    let my_mesh: Handle<Mesh> = server.load("my_scene.gltf#Mesh0/Primitive0");

    // spawn a whole scene
    let my_scene: Handle<Scene> = server.load("my_scene.gltf#Scene0");
    commands.spawn(SceneBundle {
        scene: my_scene,
        ..Default::default()
    });
}
```

查看 [`GLTF`](./3d_models_and_scenes.md)页面以了解更多3D模型的信息。

## 资产从何处加载？

在桌面平台上，它将资产路径视为相对于名为 `assets` 的文件夹，该文件夹必须位于以下位置之一：

- 游戏可执行文件旁边，用于发布。

- 在开发过程中使用 `cargo` 运行游戏时的 Cargo 项目文件夹中。这通过 `CARGO_MANIFEST_DIR` 环境变量标识。

在 Web 上，它使用 HTTP URL 指向位于游戏 `.wasm` 文件旁边的 `assets` 文件夹内的资产。

还有一些[非官方插件](./community_plugin_ecosystem.md)提供了替代的 I/O 后端实现，例如从归档文件（.zip）内部加载资产、嵌入在游戏可执行文件内部、使用网络协议等，还有许多其他可能性。