# 12. 音频

Bevy 提供了一个（虽然有点简陋，但仍然有用的）基于 ECS 的音频框架。本章将教你如何使用它。

你可以在游戏中播放音效和音乐，并控制音量。它有一个基本的“空间音频”实现，可以根据实体的变换在立体声中平移声音。如果你想从代码合成声音、从某个地方流式传输数据，或用于任何其他自定义用例，你还可以实现自己的自定义音频数据源。

Bevy 的音频支持还有第三方替代方案：

1. [bevy_kira_audio](https://github.com/NiklasEi/bevy_kira_audio)：使用 [kira](https://github.com/tesselode/kira)；提供更丰富的功能和播放控制；

2. [bevy_oddio](https://github.com/harudagondi/bevy_oddio)：使用 [oddio](https://github.com/Ralith/oddio)；似乎提供更先进的3D空间声音;

3. [bevy_fundsp](https://github.com/harudagondi/bevy_fundsp)：使用 [fundsp](https://github.com/SamiPerttu/fundsp)；用于高级声音合成和效果。

（Bevy 的官方音频是基于 [rodio](https://github.com/RustAudio/rodio) 库的。）

如你所见，Rust 音频生态系统相当分散。有许多后端库，都提供了不同的功能组合，没有一个特别全面。它们都有些不成熟。你必须做出选择。

音频是一个迫切需要改进的领域。如果你是一个热情的音频开发者，考虑加入 [Discord](https://discord.gg/bevy) 并帮助开发吧！