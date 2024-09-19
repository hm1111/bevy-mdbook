# 11.3. 访问资产数据

要从系统中访问实际的资产数据，请使用 [`Assets<T>`](https://docs.rs/bevy/latest/bevy/asset/struct.Assets.html) 资源。

你可以使用句柄来标识所需的资产。

无类型句柄需要 "升级" 为类型句柄。

```rust
#[derive(Resource)]
struct SpriteSheets {
    map_tiles: Handle<TextureAtlas>,
}

fn use_sprites(
    handles: Res<SpriteSheets>,
    atlases: Res<Assets<TextureAtlas>>,
    images: Res<Assets<Image>>,
) {
    // Could be `None` if the asset isn't loaded yet
    if let Some(atlas) = atlases.get(&handles.map_tiles) {
        // do something with the texture atlas
    }
}
```

## 使用代码创建资产

你也可以手动将资产添加到 [`Assets<T>`](https://docs.rs/bevy/latest/bevy/asset/struct.Assets.html) 中。

有时你需要从代码中创建资产，而不是[从文件中加载它们](./load_assets_from_files.md)。这种用例的一些常见示例包括：

- 创建纹理图集

- 创建 3D 或 2D 材质

- 程序生成资产，如图像或 3D 网格

为此，首先创建资产的数据（资产类型的实例），然后将其添加到 `Assets<T>` 资源中，以便 Bevy 存储和跟踪它。你将获得一个[句柄](./handles.md)，用于引用它，就像任何其他资产一样。

```rust
fn add_material(
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    let new_mat = StandardMaterial {
        base_color: Color::rgba(0.25, 0.50, 0.75, 1.0),
        unlit: true,
        ..Default::default()
    };

    let handle = materials.add(new_mat);

    // do something with the handle
}
```