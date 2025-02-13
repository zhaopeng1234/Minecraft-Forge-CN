### 音效定义生成
通过继承 `SoundDefinitionsProvider` 并实现 `#registerSounds` 方法，可以为模组生成 `sounds.json` 文件。实现之后，必须将该提供者 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成客户端资源时运行
        event.includeClient(),
        output -> new MySoundDefinitionsProvider(output, MOD_ID, event.getExistingFileHelper())
    );
}
```

### 添加音效
可以通过 `#add` 方法指定音效名称和定义来生成音效定义。音效名称可以从 [`SoundEvent`][soundevent]、`ResourceLocation` 或字符串中获取。

!!! 警告
    提供的音效名称总是假定命名空间是传递给提供者构造函数的模组 ID。不会对音效名称的命名空间进行验证！

#### `SoundDefinition`
可以使用 `#definition` 方法创建 `SoundDefinition`。该定义包含定义一个音效实例的数据。

定义指定了一些方法：

| 方法 | 描述 |
| :---: | --- |
| `with` | 添加一个或多个音效，当该定义被选中时可能会播放这些音效。 |
| `subtitle` | 设置该定义的翻译键。 |
| `replace` | 当值为 `true` 时，会移除其他 `sounds.json` 中已为该定义定义的音效，而不是追加到它们后面。 |

#### `SoundDefinition$Sound`
可以使用 `SoundDefinitionsProvider#sound` 方法指定提供给 `SoundDefinition` 的音效。这些方法接受音效的引用，如果指定的话，还接受一个 `SoundType`。

`SoundType` 可以是以下两种值之一：

| 音效类型 | 定义 |
| :---: | --- |
| `SOUND` | 指定位于 `assets/<命名空间>/sounds/<路径>.ogg` 的音效引用。 |
| `EVENT` | 指定 `sounds.json` 中定义的另一个音效名称的引用。 |

从 `SoundDefinitionsProvider#sound` 创建的每个 `Sound` 都可以指定关于如何加载和播放所提供音效的额外配置：

| 方法 | 描述 |
| :---: | --- |
| `volume` | 设置音效的音量缩放比例，必须大于 `0`。 |
| `pitch` | 设置音效的音高缩放比例，必须大于 `0`。 |
| `weight` | 设置该音效在被选中时播放的可能性。 |
| `stream` | 当值为 `true` 时，从文件中读取音效而不是将其加载到内存中。对于长音效（如背景音乐、音乐唱片等）推荐使用。 |
| `attenuationDistance` | 设置音效可以被听到的方块数。 |
| `preload` | 当值为 `true` 时，资源包加载后立即将音效加载到内存中。 |

```java
// 在某个 SoundDefinitionsProvider#registerSounds 方法中
this.add(EXAMPLE_SOUND_EVENT, definition()
  .subtitle("sound.examplemod.example_sound") // 设置翻译键
  .with(
    sound(new ResourceLocation(MODID, "example_sound_1")) // 设置第一个音效
      .weight(4) // 有 4 / 5 = 80% 的播放概率
      .volume(0.5), // 将此音效的所有音量缩放为原来的一半
    sound(new ResourceLocation(MODID, "example_sound_2")) // 设置第二个音效
      .stream() // 流式播放该音效
  )
);

this.add(EXAMPLE_SOUND_EVENT_2, definition()
  .subtitle("sound.examplemod.example_sound") // 设置翻译键
  .with(
    sound(EXAMPLE_SOUND_EVENT.getLocation(), SoundType.EVENT) // 添加 'EXAMPLE_SOUND_EVENT' 中的音效
      .pitch(0.5) // 将此音效的所有音高缩放为原来的一半
  )
);
```

[datagen]: ../index.md#data-providers
[soundevent]: ../../gameeffects/sounds.md#creating-sound-events
