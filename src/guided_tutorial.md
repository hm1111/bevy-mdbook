# 1.1. 入门指南

Bevy新手者的入门指南。

对官方例子感兴趣？可以查看[这里](https://github.com/bevyengine/bevy/tree/latest/examples#examples)。

遇到问题，需要帮助？访问[Bevy Discussions](https://github.com/bevyengine/bevy/discussions)，或者 [Bevy常见的陷阱](./common_pitfalls.md)。

## 基础

这里包含了每一个Bevy项目都涉及的内容。

- [Bevy设置技巧](./bevy_setup_tips.md)
    - [开始](./getting_started.md)
- [Beyv编程框架](./bevy_programming_framework.md)
    - [ECS介绍](./ECS编程介绍.md)
    - [Entities and Components](./entities_and_components.md)
    - [Bundles](./bundles.md)
    - [Resources](./resources.md)
    - [Systems](./systems.md)
    - [App Builder](./app_builder.md)
    - [Queries](./queries.md)
    - [Commands](./commands.md)
- [游戏引擎基础](./game_engine_fundamentals.md)
    - [坐标系](./coordinate_systems.md)
    - [转换](./transforms.md)
    - [时间和计时器](./time_and_timers.md)
- [一般图形功能](./general_graphics_features.md)
    - [Cameras](./cameras.md)
- [Bevy资产管理](./bevy_assets_management.md)
    - [使用AssetServer加载资产](./load_assets_with_assetserver.md)
    - [句柄](./handles.md)
- [输入处理](./input_handling.md)
    - [Keyboard](./keyboard.md)
    - [Mouse](./mouse.md)
    - [Gamepad](./gamepad.md)
    - [Touchscree](./touchscreen.md)
- [窗口管理](./window_management.md)
    - [窗口属性](./window_properties.md)
    - [更改背景颜色](./change_the_background_color.md)
- [音频](./audio.md)
    - [播放音频](./playing_sounds.md)

## 下一步
- [Bevy设置技巧](./bevy_setup_tips.md)
    - [Bevy开发工具和编辑器](./bevy_dev_tools_and_editors.md)
    - [社区插件生态系统](./community_plugin_ecosystem.md)
- [Bevy编程框架](./bevy_programming_framework.md)
    - [Events](./events.md)
    - [System执行顺序](./system_order_of_excution.md)
    - [运行条件](./run_condition.md)
    - [System Sets](./system_sets.md)
    - [Local Resources](./local_resources.md)
    - [Shedules](./shedules.md)
    - [State](./state.md)
    - [Plugins](./plugins.md)
    - [Change Detection](./change_detection.md)
- [游戏引擎基础](./game_engine_fundamentals.md)
    - [父子层级](./parent_chid_hierarchy.md)
    - [可见性](./visibility.md)
    - [日志/控制台信息](./loging_and_console_messages.md)
- [Bevy资产管理](./bevy_assets_management.md)
    - [访问资产数据](./access_asset_data.md)
    - [资产热加载](./hot_loading_assets.md)
- [输入处理](./input_handling.md)
    - [将光标坐标转换为游戏世界坐标](./convert_cursor_to_world_coordinates.md)
- [音频](./audio.md)
    - [立体声音频](./spatial_audio.md)

## 中级

- [Bevy编程框架](./bevy_programming_framework.md)
    - [直接访问World](./direct_ecs_world_access.md)
    - [独占系统](./exclusive_system.md)
    - [Param Sets](./param_sets.md)
    - [管道系统](./system_piping.md)
- [游戏引擎基础](./game_engine_fundamentals.md)
    - [固定时间步](./fixed_timestep.md)
- [一般图形功能](./general_graphics_features.md)
    - [HDR,Tonemapping](./hdr_and_tonemapping.md)
    - [Bloom](./bloom.md)
- [Bevy资产管理](./bevy_assets_management.md)
    - [通过资产事件对变化做出反应](./react_to_change_with_asset_events.md)
    - [跟踪资产加载进度](./track_asset_loading_progress.md)
- [编程模式](./programming_patterns.md)
    - [编写测试](./write_tests_for_systems.md)
    - [泛型系统](./genric_systems.md)
    - [手动清理事件](./manual_event_clearing.md)
- [窗口管理](./window_management.md)
    - [抓取/捕获鼠标光标](./grab_or_capture_mouse_cursor.md)
- [音频](./audio.md)
    - [自定义音频流](./custom_audio_streams.md)

## 高级

这些主题适用于特定的技术情况。如果您想了解更多关于 Bevy 内部工作原理、使用自定义功能扩展引擎或使用 Bevy 执行其他高级操作，您可以学习它们。

- [Bevy设置技巧](./bevy_setup_tips.md)
    - [自定义Bevy(crates和features)](./customizing_bevy.md)
    - [使用最前沿的Bevy](./using_bleading_edge_bevy.md)
- [Bevy编程框架](./bevy_programming_framework.md)
    - [Non-Send](./non_send.md)
- [编程模式](./programming_patterns.md)
    - [组件存储](./component_storage.md)
- [输入处理](./input_handling.md)
    - [拖放文件](./drag_and_drop_files.md)
    - [用于高级文本输入的IME](./ime_for_advanced_text_input.md)
- [Bevy GPU渲染框架](./bevy_render_framework.md)
    - [渲染架构概述](./rendering_architecture_overview.md)
    - [渲染集](./render_sets.md)