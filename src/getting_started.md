# 3.1. 开始

和其他Rust库一样，你需要先安装Rust并配置好环境。[Bevy官方设置](https://bevyengine.org/learn/book/getting-started/setup/)

在Linux系统中，你可能还需要安装其他依赖库。[点击此处查看](https://github.com/bevyengine/bevy/blob/main/docs/linux_dependencies.md)

## 安装Rust

[Rust官方链接](https://www.rust-lang.org/learn/get-started)

## 创建项目

此处不翻译，详细内容可查看[Bevy官方链接](https://bevyengine.org/learn/book/getting-started/setup/)

## 项目文档

### 创建并打开项目文档

`cargo doc --open`

当文档创建完成后，会自动使用浏览器打开，该文档可离线使用。项目代码有变动时，需要重新运行该命令才能更新文档。

## 可选的额外设置

使用Rust的默认设置会导致较低的性能，你可以通过配置优化来提升性能。[查看这里提升性能](https://bevy-cheatbook.github.io/pitfalls/performance.html)。

迭代重新编译速度对于保持你的工作效率很重要，因此每次你想测试游戏时，您都不必等待 Rust 编译器重新构建你的项目。Bevy 的[入门页面](https://bevyengine.org/learn/book/getting-started/setup/)提供了有关如何加快编译时间的建议。

一些额外的[开发工具](./bevy_dev_tools_and_editors.md)也很重要。

## 遇到问题？

查看[常见的陷阱](./common_pitfalls.md)可能会找到解决方案。也可以访问[Github Discussions](https://github.com/bevyengine/bevy/discussions)，还有[Discord](https://discord.gg/bevy)。

## GPU驱动

为了更好地运行，Bevy需要DirectX 12(Windows), Vulkan(Windows, Linux, macOS), Metal(macOS)。

OpenGL(GLES3)可以用，但是可能会有一些问题(bugs，不支持的功能，更差的性能, ......)。

确保你的系统有兼容的硬件和驱动，同时，你的用户也需要有兼容的硬件和驱动。

使用WebGL2是支持网络游戏的，并且也能在现代浏览器中运行。但是其性能受限，并且某些Bevy功能可能不支持。新的实验性高性能WebGPU API也得到支持，但是浏览器对它的支持还不够完善。