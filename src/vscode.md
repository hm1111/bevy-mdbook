# 3.2.1. VSCode

## 安装Rust Analyzer

在VSCode中搜索并安装Rust Analyzer插件。

## 提升Rust Analyzer性能

若你在`.cargo/config.toml`中指定了第三方的链接器，Rust Analyzer可能会忽略它。
若想让Rust Analyzer使用这些链接器，你需要在VSCode中打开设置（`Ctrl +,`），搜索`rust-analyzer`，并将以下设置添加到`settings.json`文件中：

Windows：
```json
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=rust-lld.exe"
}
```

Linux(lld)：
```json
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=clang -Clink-arg=-fuse-ld=lld"
}
```

Linux(mold):
```json
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=clang -Clink-arg=-fuse-ld=mold"
}
```
注：经本人测试，添加了上述代码后，每次修改代码后运行，都会导致重新编译，反而大大增加了编译时间。（可能有误，大家可自行测试）

## CARGO_MANIFEST_DIR

当运行Bevy应用时，它会在`BEVY_ASSET_ROOT`和`CARGO_MANIFEST_DIR`环境变量中查找`assets`目录。这可以让运行在终端中的`cargo run`正常工作。

若`assets`目录没有在`BEVY_ASSET_ROOT`或`CARGO_MANIFEST_DIR`中，Bevy就会在二进制程序所在的目录中查找`assets`目录。这对发行版本很方便。但是针对开发版本，二进制程序是在`target`目录中，Bevy是找不到`assets`目录的。

（剩下的暂时不翻译）