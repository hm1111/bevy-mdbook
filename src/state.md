# 14.20. State

它能够让你构建应用的运行时"流"。

你可以使用状态实现以下功能：

1. 菜单屏幕或加载屏幕；

2. 暂停/恢复游戏；

3. 不同的游戏模式；

4. ......

在每一个状态(state)中，你可以运行不同的系统。你还可以添加在进入或退出状态时运行的设置和清理系统。

## 定义状态

定义一个枚举类型，并派生 `States` trait以及一系必需的标准 Rust trait。

```rust
#[derive(States, Debug, Clone, PartialEq, Eq, Hash)]
enum MyAppState {
    LoadingScreen,
    MainMenu,
    InGame,
}

#[derive(States, Default, Debug, Clone, PartialEq, Eq, Hash)]
enum MyGameModeState {
    #[default]
    NotInGame,
    Singleplayer,
    Multiplayer,
}

#[derive(States, Default, Debug, Clone, PartialEq, Eq, Hash)]
enum MyPausedState {
    #[default]
    Paused,
    Running,
}
```

你可以拥有多个正交状态！如果你想独立跟踪多个事物，需要创建多个类型！

之后你需要把状态添加到应用中：

```rust
// 指定初始值
app.insert_state(MyAppState::LoadingScreen);

// 或者使用默认值
app.init_state::<MyGameModeState>();
app.init_state::<MyPausedState>();
```

## 为不同的状态运行不同的系统

如果你希望某些系统只在特定的状态下运行，Bevy 提供了一个 `in_state` 运行条件。你可以将它添加到你的系统中。你可能还需要创建系统集（`system sets`）来帮助你对多个系统进行分组并一次性控制它们。

```rust
// configure some system sets to help us manage our systems
// (note: it is per-schedule, so we also need it for FixedUpdate
// if we plan to use fixed timestep)
app.configure_sets(Update, (
    MyMainMenuSet
        .run_if(in_state(MyAppState::MainMenu)),
    MyGameplaySet
        // note: you can check for a combination of different states
        .run_if(in_state(MyAppState::InGame))
        .run_if(in_state(MyPausedState::Running)),
));
app.configure_sets(FixedUpdate, (
    // configure the same set here, so we can use it in both
    // FixedUpdate and Update
    MyGameplaySet
        .run_if(in_state(MyAppState::InGame))
        .run_if(in_state(MyPausedState::Running)),
    // configure a bunch of different sets only for FixedUpdate
    MySingleplayerSet
        // inherit configuration from MyGameplaySet and add extras
        .in_set(MyGameplaySet)
        .run_if(in_state(MyGameModeState::Singleplayer)),
    MyMultiplayerSet
        .in_set(MyGameplaySet)
        .run_if(in_state(MyGameModeState::Multiplayer)),
));

// now we can easily add our different systems
app.add_systems(Update, (
    update_loading_progress_bar
        .run_if(in_state(MyAppState::LoadingScreen)),
    (
        handle_main_menu_ui_input,
        play_main_menu_sounds,
    ).in_set(MyMainMenuSet),
    (
        camera_movement,
        play_game_music,
    ).in_set(MyGameplaySet),
));
app.add_systems(FixedUpdate, (
    (
        player_movement,
        enemy_ai,
    ).in_set(MySingleplayerSet),
    (
        player_net_sync,
        enemy_net_sync,
    ).in_set(MyMultiplayerSet),
));

// of course, if we need some global (state-independent)
// setup to run on app startup, we can still use Startup as usual
app.add_systems(Startup, (
    load_settings,
    setup_window_icon,
));
```

Bevy 还会为你的状态类型的每个可能值创建特殊的 `OnEnter`、`OnExit` 和` OnTransition` schedule。使用它们可以为特定状态执行设置和清理。你添加到它们的任何系统将在状态每次更改为/从相应值更改时运行一次。

```rust
// do the respective setup and cleanup on state transitions
app.add_systems(OnEnter(MyAppState::LoadingScreen), (
    start_load_assets,
    spawn_progress_bar,
));
app.add_systems(OnExit(MyAppState::LoadingScreen), (
    despawn_loading_screen,
));
app.add_systems(OnEnter(MyAppState::MainMenu), (
    setup_main_menu_ui,
    setup_main_menu_camera,
));
app.add_systems(OnExit(MyAppState::MainMenu), (
    despawn_main_menu,
));
app.add_systems(OnEnter(MyAppState::InGame), (
    spawn_game_map,
    setup_game_camera,
    spawn_enemies,
));
app.add_systems(OnEnter(MyGameModeState::Singleplayer), (
    setup_singleplayer,
));
app.add_systems(OnEnter(MyGameModeState::Multiplayer), (
    setup_multiplayer,
));
// ...
```
### 搭配Plugins

这在与插件一起使用时也很有用。你可以在一个地方设置项目的所有状态类型，然后不同的插件可以将其系统添加到相关状态。

你还可以制作可配置的插件，这样就可以指定它们应该将系统添加到哪个状态：

```rust
pub struct MyPlugin<S: States> {
    pub state: S,
}

impl<S: States> Plugin for MyPlugin<S> {
    fn build(&self, app: &mut App) {
        app.add_systems(Update, (
            my_plugin_system1,
            my_plugin_system2,
            // ...
        ).run_if(in_state(self.state.clone())));
    }
}
```

现在，您可以在将插件添加到应用程序时配置插件：

```rust
app.add_plugins(MyPlugin {
    state: MyAppState::InGame,
});
```

当你仅仅使用插件来帮助你的项目进行内部组织管理，并且你清楚每个状态应该包含哪些系统时，你可能不需要像上面所示那样麻烦地使插件可配置。只需直接硬编码状态/将系统添加到正确的状态中即可。

## 控制状态

你可以使用 `State<T>` 资源来检查当前的状态：

```rust
fn debug_current_gamemode_state(state: Res<State<MyGameModeState>>) {
    eprintln!("Current state: {:?}", state.get());
}
```

还可以使用 `NextState<T>` 来切换到另一个状态。

```rust
fn toggle_pause_game(
    state: Res<State<MyPausedState>>,
    mut next_state: ResMut<NextState<MyPausedState>>,
) {
    match state.get() {
        MyPausedState::Paused => next_state.set(MyPausedState::Running),
        MyPausedState::Running => next_state.set(MyPausedState::Paused),
    }
}

// if you have multiple states that must be set correctly,
// don't forget to manage them all
fn new_game_multiplayer(
    mut next_app: ResMut<NextState<MyAppState>>,
    mut next_mode: ResMut<NextState<MyGameModeState>>,
) {
    next_app.set(MyAppState::InGame);
    next_mode.set(MyGameModeState::Multiplayer);
}
```

状态转换将排队等待，以便在下一帧更新周期中执行。

## 状态转换

每个帧更新时，都会运行一个名为` StateTransition` 的 schedule。Bevy 会检查 `NextState<T>` 中是否有任何新状态排队，并为你执行转换。

转换涉及以下几个步骤：

1. 发送 `StateTransitionEvent` 事件；

2. 运行 `OnExit(old_state)` schedule；

3. 运行 `OnTransition { from: old_state, to: new_state }` schedule； 

4. 运行 `OnEnter(new_state)` schedule；

`StateTransitionEvent` 在任何不考虑状态运行，但它想知道是否发生了转换的系统中很有用。你可以使用它来检测状态转换。

`StateTransition` schedule 在 `PreUpdate` 之后运行，但在 `FixedMain`（固定时间步长）和 `Update`（你的游戏系统通常所在的地方）之前运行。

因此，状态转换发生在当前帧的游戏逻辑之前。

如果你觉得每帧一次的状态转换不够，你可以添加额外的转换点，方法是在你喜欢的任何地方添加 Bevy 的 `apply_state_transition` 系统。

```rust
// Example: also do state transitions for MyPausedState
// before MyGameplaySet on each fixed timestep run
app.add_systems(
    FixedUpdate,
    apply_state_transition::<MyPausedState>
        .before(MyGameplaySet)
);
```

## 已知的陷阱

### 系统集配置是per-schedule的

这与配置系统集时适用的一般注意事项相同。

请注意，`app.configure_sets()` 是per-schedule的！如果你在一个调度中配置了一些集合，该配置不会延续到其他调度。

因为状态是如此调度繁重，你必须特别小心。不要仅仅因为你配置了一个集合，就认为你可以在任何地方使用它。

例如，你从 `Update` 和 `FixedUpdate` 得到的集合将不会在你的各种状态转换的 `OnEnter/OnExit` 中工作。

### 事件

这与适用于任何具有运行条件且要接收事件的系统的一般注意事项相同。

当在不总是运行的系统中接收事件时，例如在暂停状态期间，你将错过接收系统未运行时发送的任何事件！

为了缓解这个问题，你可以实现一个[自定义的清理策略](./manual_event_clearing.md)，以手动管理相关事件类型的生命周期。

