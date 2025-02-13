## 数据生成器
数据生成器是一种以编程方式生成模组的资源和数据的方式。它允许在代码中定义这些文件的内容并自动生成，而无需担心具体细节。

数据生成器系统由主类 `net.minecraft.data.Main` 加载。可以传递不同的命令行参数来定制收集哪些模组的数据、考虑哪些现有文件等等。负责数据生成的类是 `net.minecraft.data.DataGenerator`。

MDK（Minecraft Development Kit）的 `build.gradle` 中的默认配置添加了 `runData` 任务来运行数据生成器。

### 现有文件
对于数据生成中未生成的所有纹理或其他数据文件的引用，都必须引用系统上的现有文件。这是为了确保所有引用的纹理都在正确的位置，以便发现并纠正拼写错误。

`ExistingFileHelper` 类负责验证这些数据文件的存在。可以从 `GatherDataEvent#getExistingFileHelper` 中获取一个实例。

`--existing <文件夹路径>` 参数允许在验证文件存在时使用指定的文件夹及其子文件夹。此外，`--existing-mod <模组ID>` 参数允许使用已加载模组的资源进行验证。默认情况下，`ExistingFileHelper` 仅可使用原版数据包和资源。

### 生成器模式
数据生成器可以配置为运行 4 种不同的数据生成，这些生成通过命令行参数进行配置，并且可以通过 `GatherDataEvent#include***` 方法进行检查。

* **客户端资源**：
  * 在 `assets` 中生成仅客户端使用的文件：方块/物品模型、方块状态 JSON 文件、语言文件等。
  * 使用参数 `--client`，通过 `#includeClient` 检查。
* **服务器数据**：
  * 在 `data` 中生成仅服务器使用的文件：合成配方、进度、标签等。
  * 使用参数 `--server`，通过 `#includeServer` 检查。
* **开发工具**：
  * 运行一些开发工具：将 SNBT 转换为 NBT 以及反向转换等。
  * 使用参数 `--dev`，通过 `#includeDev` 检查。
* **报告**：
  * 转储所有已注册的方块、物品、命令等。
  * 使用参数 `--reports`，通过 `#includeReports` 检查。

可以使用 `--all` 参数包含所有生成器。

### 数据提供者
数据提供者是实际定义将生成和提供哪些数据的类。所有数据提供者都实现了 `DataProvider` 接口。Minecraft 为大多数资源和数据提供了抽象实现，因此模组开发者只需扩展并覆盖指定的方法。

当创建数据生成器时，`GatherDataEvent` 会在模组事件总线上触发，并且可以从该事件中获取 `DataGenerator`。使用 `DataGenerator#addProvider` 创建并注册数据提供者。

#### 客户端资源
* [`net.minecraftforge.common.data.LanguageProvider`][langgen] - 用于[语言字符串][lang]；实现 `#addTranslations` 方法。
* [`net.minecraftforge.common.data.SoundDefinitionsProvider`][soundgen] - 用于 [`sounds.json`][sounds]；实现 `#registerSounds` 方法。
* [`net.minecraftforge.client.model.generators.ModelProvider<?>`][modelgen] - 用于[模型]；实现 `#registerModels` 方法。
    * [`ItemModelProvider`][itemmodelgen] - 用于物品模型。
    * [`BlockModelProvider`][blockmodelgen] - 用于方块模型。
* [`net.minecraftforge.client.model.generators.BlockStateProvider`][blockstategen] - 用于方块状态 JSON 文件及其方块和物品模型；实现 `#registerStatesAndModels` 方法。

#### 服务器数据
**以下类位于 `net.minecraftforge.common.data` 包中**：
* [`GlobalLootModifierProvider`][glmgen] - 用于[全局战利品修改器][glm]；实现 `#start` 方法。
* [`DatapackBuiltinEntriesProvider`][datapackregistriesgen] 用于数据包注册表对象；将 `RegistrySetBuilder` 传递给构造函数。

**以下类位于 `net.minecraft.data` 包中**：
* [`loot.LootTableProvider`][loottablegen] - 用于[战利品表][loottable]；将 `LootTableProvider$SubProviderEntry` 传递给构造函数。
* [`recipes.RecipeProvider`][recipegen] - 用于[合成配方]及其解锁进度；实现 `#buildRecipes` 方法。
* [`tags.TagsProvider`][taggen] - 用于[标签]；实现 `#addTags` 方法。
* [`advancements.AdvancementProvider`][advgen] - 用于[进度]；将 `AdvancementSubProvider` 传递给构造函数。

[langgen]: ./client/localization.md
[lang]: https://minecraft.wiki/w/Language
[soundgen]: ./client/sounds.md
[sounds]: https://minecraft.wiki/w/Sounds.json
[modelgen]: ./client/modelproviders.md
[models]: ../resources/client/models/index.md
[itemmodelgen]: ./client/modelproviders.md#itemmodelprovider
[blockmodelgen]: ./client/modelproviders.md#blockmodelprovider
[blockstategen]: ./client/modelproviders.md#block-state-provider
[glmgen]: ./server/glm.md
[glm]: ../resources/server/glm.md
[datapackregistriesgen]: ./server/datapackregistries.md
[loottablegen]: ./server/loottables.md
[loottable]: ../resources/server/loottables.md
[recipegen]: ./server/recipes.md
[recipes]: ../resources/server/recipes/index.md
[taggen]: ./server/tags.md
[tags]: ../resources/server/tags.md
[advgen]: ./server/advancements.md
[advancements]: ../resources/server/advancements.md
