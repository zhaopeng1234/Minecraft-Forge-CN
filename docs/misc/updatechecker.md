### Forge更新检查器概述
Forge提供了一个非常轻量级、可选加入的更新检查框架。如果任何模组有可用更新，主菜单和模组列表的“模组”按钮上会显示一个闪烁图标，并显示相应的更新日志。它*不会*自动下载更新。

### 开始使用
首先，需要在 `mods.toml` 文件中指定 `updateJSONURL` 参数。该参数的值应该是一个指向更新JSON文件的有效URL。这个文件可以托管在你自己的Web服务器、GitHub或其他任何地方，只要你的模组所有用户都能可靠访问即可。

### 更新JSON格式
JSON本身的格式相对简单，如下所示：
```js
{
  "homepage": "<你的模组主页/下载页面>",
  "<MC版本>": {
    "<模组版本>": "<此版本的更新日志>", 
    // 列出给定Minecraft版本下你的模组的所有版本及其更新日志
    // ...
  },
  "promos": {
    "<MC版本>-latest": "<模组版本>",
    // 声明给定Minecraft版本下你的模组的最新“前沿”版本
    "<MC版本>-recommended": "<模组版本>",
    // 声明给定Minecraft版本下你的模组的最新“稳定”版本
    // ...
  }
}
```
这部分内容比较容易理解，但有几点需要注意：
- `homepage` 下的链接是当模组过时，向用户显示的链接。
- Forge使用内部算法来确定你的模组的一个版本字符串是否比另一个“更新”。大多数版本控制方案应该是兼容的，但如果你担心你的方案是否受支持，可以查看 `ComparableVersion` 类。强烈建议遵循 [Maven版本控制][mvnver]。
- 更新日志字符串可以使用 `\n` 分隔成多行。有些人喜欢先包含一个简略的更新日志，然后链接到提供完整更改列表的外部站点。
- 手动输入数据可能很繁琐。由于Groovy具有原生JSON解析支持，你可以配置 `build.gradle` 在构建发布版时自动更新此文件。这部分内容留给读者作为练习。
- 可以在以下链接找到一些示例：[nocubes][], [Corail Tombstone][corail] 和 [Chisels & Bits 2][chisel]。

### 获取更新检查结果
可以使用 `VersionChecker#getResult(IModInfo)` 获取Forge更新检查器的结果。可以通过 `ModContainer#getModInfo` 获取 `IModInfo`。可以在构造函数中使用 `ModLoadingContext.get().getActiveContainer()`、`ModList.get().getModContainerById(<你的模组ID>)` 或 `ModList.get().getModContainerByObject(<你的模组实例>)` 获取 `ModContainer`。也可以使用 `ModList.get().getModContainerById(<模组ID>)` 获取任何其他模组的 `ModContainer`。返回的对象有一个 `#status` 方法，用于指示版本检查的状态。

| 状态 | 描述 |
| --- | --- |
| `FAILED` | 版本检查器无法连接到提供的URL。 |
| `UP_TO_DATE` | 当前版本等于推荐版本。 |
| `AHEAD` | 如果没有最新版本，当前版本比推荐版本新。 |
| `OUTDATED` | 有新的推荐版本或最新版本。 |
| `BETA_OUTDATED` | 有新的最新版本。 |
| `BETA` | 当前版本等于或比最新版本新。 |
| `PENDING` | 请求的结果尚未完成，稍后应再次尝试。 |

返回的对象还将包含目标版本和 `update.json` 中指定的任何更新日志行。

[mvnver]: ../gettingstarted/versioning.md
[nocubes]: https://cadiboo.github.io/projects/nocubes/update.json
[corail]: https://github.com/Corail31/tombstone_lite/blob/master/update.json
[chisel]: https://github.com/Aeltumn/Chisels-and-Bits-2/blob/master/update.json
