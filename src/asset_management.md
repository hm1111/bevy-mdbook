# 11. 资产管理

资产是游戏引擎处理的数据：包括所有的图像、3D模型、声音、场景、游戏特定的元素（如物品描述）等！

Bevy 有一个灵活的系统，可以异步（在后台，不会导致游戏中的延迟峰值）加载和管理游戏资产。

在你的代码中，你使用[句柄](./handles.md)来引用单个资产。

资产数据可以从[文件加载](./load_assets_from_files.md)，也可以从代码中访问。支持[热重载](./hot_loading_assets.md)，这有助于你在开发过程中，如果资产文件在游戏运行时发生变化，会重新加载这些文件。

如果你想写一些代码来在资产加载完成、被修改或卸载时做一些事情，你可以使用[资产事件](./react_to_change_with_asset_events.md)。

