# 11.5. 跟踪资产加载进度

有一些优秀的社区插件可以帮助解决这个问题。另外，本页也将向你展示如何自己动手实现。

如果你想检查各种资产文件的状态，可以从 [`AssetServer`](https://docs.rs/bevy/latest/bevy/asset/struct.AssetServer.html) 进行轮询。它会告诉你资产是否已加载、仍在加载、未加载或遇到错误。

要检查单个资产，可以使用 `asset_server.get_load_state(...)`，并提供一个句柄或路径来引用该资产。