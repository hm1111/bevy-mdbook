# 11.4. 使用资产事件对更改做出反应

如果你需要在资产创建、修改或删除时执行特定的操作，你可以创建一个对 [`AssetEvent`](https://docs.rs/bevy/latest/bevy/asset/enum.AssetEvent.html) 事件做出反应的系统。

```rust
#[derive(Resource)]
struct MyMapImage {
    handle: Handle<Image>,
}

fn fixup_images(
    mut ev_asset: EventReader<AssetEvent<Image>>,
    mut assets: ResMut<Assets<Image>>,
    map_img: Res<MyMapImage>,
) {
    for ev in ev_asset.iter() {
        match ev {
            AssetEvent::Created { handle } => {
                // a texture was just loaded or changed!

                // 该可变访问会导致另一个AssetEvent被发送
                let texture = assets.get_mut(handle).unwrap();
                // ^ unwrap is OK, because we know it is loaded now

                if *handle == map_img.handle {
                    // it is our special map image!
                } else {
                    // it is some other image
                }
            }
            AssetEvent::Modified { handle } => {
                // 图片被修改了
            }
            AssetEvent::Removed { handle } => {
                // 图片未被加载
            }
        }
    }
}
```

**注意**：如果你正在处理修改事件(Modified event)并且对数据进行了可变访问，那么 `.get_mut ` 将会为同一资产触发另一个修改事件。如果你不小心，这可能会导致无限循环！（由你自己的系统引起的事件）。