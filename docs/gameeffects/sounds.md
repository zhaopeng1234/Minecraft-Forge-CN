声音
======

术语说明
-----------

| 术语           | 描述 |
|----------------|----------------|
| 声音事件  | 触发音效的事件。例如 `minecraft:block.anvil.hit` 或 `botania:spreader_fire`。 |
| 声音类别 | 声音所属的类别，例如 `player`、`block` 或简单的 `master`。声音设置界面中的滑块代表这些类别。 |
| 声音文件   | 磁盘上实际播放的文件：一个 `.ogg` 文件。 |

`sounds.json`
-------------

这个 JSON 文件用于定义声音事件，以及指定它们要播放的声音文件、字幕等信息。声音事件通过 [`资源位置`][loc] 来标识。`sounds.json` 文件应位于资源命名空间的根目录下（`assets/<命名空间>/sounds.json`），它会定义该命名空间下的声音事件（`assets/<命名空间>/sounds.json` 定义了命名空间 `命名空间` 中的声音事件）。

在原版 [维基百科][wiki] 上可以找到完整的规范，但下面这个示例突出了重要部分：

```js
{
  "open_chest": {
    "subtitle": "mymod.subtitle.open_chest",
    "sounds": [ "mymod:open_chest_sound_file" ]
  },
  "epic_music": {
    "sounds": [
      {
        "name": "mymod:music/epic_music",
        "stream": true
      }
    ]
  }
}
```

在顶级对象下，每个键对应一个声音事件。请注意，这里没有给出命名空间，因为它取自 JSON 文件本身的命名空间。每个事件都会指定一个本地化键，当启用字幕时会显示该键对应的内容。最后，会指定要播放的实际声音文件。注意，其值是一个数组；如果指定了多个声音文件，游戏会在每次触发声音事件时随机选择一个进行播放。

这两个示例展示了两种不同的指定声音文件的方式。[维基百科][wiki] 中有详细的说明，但一般来说，像背景音乐或音乐唱片这样的长声音文件应该使用第二种形式，因为 `"stream"` 参数告诉 Minecraft 不要将整个声音文件加载到内存中，而是从磁盘流式传输。第二种形式还可以指定声音文件的音量、音高和权重。

在所有情况下，命名空间为 `命名空间` 且路径为 `路径` 的声音文件的路径是 `assets/<命名空间>/sounds/<路径>.ogg`。因此，`mymod:open_chest_sound_file` 指向 `assets/mymod/sounds/open_chest_sound_file.ogg`，而 `mymod:music/epic_music` 指向 `assets/mymod/sounds/music/epic_music.ogg`。

`sounds.json` 文件可以通过 [数据生成][datagen] 来创建。

创建声音事件
---------------------

为了在服务器上引用声音，必须创建一个在 `sounds.json` 中有对应条目的 `SoundEvent`。然后，这个 `SoundEvent` 必须 [注册][registration]。通常，用于创建声音事件的位置应设置为其注册表名称。

`SoundEvent` 作为声音的引用，会被传递以播放声音。如果一个模组有 API，应该在 API 中公开其 `SoundEvent`。

!!! 注意
    只要声音在 `sounds.json` 中注册，无论是否有引用的 `SoundEvent`，它仍然可以在逻辑客户端上被引用。

播放声音
--------------

原版有很多播放声音的方法，有时不清楚该使用哪一个。

请注意，每个方法都接受一个 `SoundEvent`，即上面注册的那些。此外，“服务器行为”和“客户端行为”指的是各自的 [**逻辑** 端][sides]。

### `Level`

1. <a name="level-playsound-pbecvp"></a> `playSound(Player, BlockPos, SoundEvent, SoundSource, volume, pitch)`
    - 简单地转发到 [重载方法 (2)](#level-playsound-pxyzecvp)，将给定 `BlockPos` 的每个坐标加 0.5。

2. <a name="level-playsound-pxyzecvp"></a> `playSound(Player, double x, double y, double z, SoundEvent, SoundSource, volume, pitch)`
    - **客户端行为**：如果传入的玩家是客户端玩家，则向该客户端玩家播放声音事件。
    - **服务器行为**：向附近的每个人播放声音事件，但 **不包括** 传入的玩家。玩家可以为 `null`。
    - **用途**：这些行为之间的对应关系表明，这两个方法应该从一些玩家发起的代码中调用，这些代码将同时在两个逻辑端运行：逻辑客户端负责向用户播放声音，逻辑服务器负责让其他所有人听到声音，而不会再次向原始用户播放。通过在逻辑服务器上调用该方法并传入 `null` 玩家，它们也可用于在服务器端的任何位置播放任何声音，从而让每个人都能听到。

3. <a name="level-playsound-xyzecvpd"></a> `playLocalSound(double x, double y, double z, SoundEvent, SoundSource, volume, pitch, distanceDelay)`
    - **客户端行为**：仅在客户端世界中播放声音事件。如果 `distanceDelay` 为 `true`，则根据声音与玩家的距离延迟播放。
    - **服务器行为**：无操作。
    - **用途**：此方法仅在客户端有效，因此对于在自定义数据包中发送的声音或其他仅客户端的效果声音很有用。用于播放雷声。

### `ClientLevel`

1. <a name="clientlevel-playsound-becvpd"></a> `playLocalSound(BlockPos, SoundEvent, SoundSource, volume, pitch, distanceDelay)`
    - 简单地转发到 `Level` 的 [重载方法 (3)](#level-playsound-xyzecvpd)，将给定 `BlockPos` 的每个坐标加 0.5。

### `Entity`

1. <a name="entity-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)`
    - 转发到 `Level` 的 [重载方法 (2)](#level-playsound-pxyzecvp)，将玩家参数传入 `null`。
    - **客户端行为**：无操作。
    - **服务器行为**：在该实体的位置向所有人播放声音事件。
    - **用途**：在服务器端从任何非玩家实体发出任何声音。

### `Player`

1. <a name="player-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)`（重写 [`Entity`](#entity-playsound-evp) 中的方法）
    - 转发到 `Level` 的 [重载方法 (2)](#level-playsound-pxyzecvp)，将 `this` 作为玩家传入。
    - **客户端行为**：无操作，参见 [`LocalPlayer`](#localplayer-playsound-evp) 中的重写方法。
    - **服务器行为**：向附近的每个人播放声音，但 *不包括* 该玩家。
    - **用途**：参见 [`LocalPlayer`](#localplayer-playsound-evp)。

### `LocalPlayer`

1. <a name="localplayer-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)`（重写 [`Player`](#player-playsound-evp) 中的方法）
    - 转发到 `Level` 的 [重载方法 (2)](#level-playsound-pxyzecvp)，将 `this` 作为玩家传入。
    - **客户端行为**：直接播放声音事件。
    - **服务器行为**：此方法仅在客户端有效。
    - **用途**：就像 `Level` 中的方法一样，玩家类中的这两个重写方法似乎适用于在两端同时运行的代码。客户端负责向用户播放声音，而服务器负责让其他所有人听到声音，而不会再次向原始用户播放。

[loc]: ../concepts/resources.md#resourcelocation
[wiki]: https://minecraft.wiki/w/Sounds.json
[datagen]: ../datagen/client/sounds.md
[registration]: ../concepts/registries.md#methods-for-registering
[sides]: ../concepts/sides.md
