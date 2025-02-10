## 标签
标签是游戏中用于将相关事物分组并提供快速成员检查的通用对象集合。

### 声明自己的分组
标签在你的模组的[数据包][datapack]中声明。例如，一个标识符为 `modid:foo/tagname` 的 `TagKey<Block>` 将引用 `/data/<modid>/tags/blocks/foo/tagname.json` 处的标签。`Block`、`Item`、`EntityType`、`Fluid` 和 `GameEvent` 的标签在文件夹位置使用复数形式，而其他所有注册表使用单数形式（`EntityType` 使用 `entity_types` 文件夹，而 `Potion` 使用 `potion` 文件夹）。

同样，你可以通过声明自己的 JSON 文件来追加或覆盖其他领域（如原版）中声明的标签。例如，要将你自己模组的树苗添加到原版树苗标签中，你可以在 `/data/minecraft/tags/blocks/saplings.json` 中指定，并且如果 `replace` 选项为 `false`，原版将在重载时将所有内容合并为一个标签。

如果 `replace` 为 `true`，则在指定 `replace` 的 JSON 文件之前的所有条目将被移除。

列出的不存在的值将导致标签出错，除非该值使用 `id` 字符串和 `required` 布尔值设置为 `false` 列出，如下例所示：

```js
{
  "replace": false,
  "values": [
    "minecraft:gold_ingot",
    "mymod:my_ingot",
    {
      "id": "othermod:ingot_other",
      "required": false
    }
  ]
}
```

有关基本语法的描述，请参阅 [原版维基百科][tags]。

Forge 还对原版语法进行了扩展。你可以声明一个与 `values` 数组格式相同的 `remove` 数组。这里列出的任何值都将从标签中移除。这是原版 `replace` 选项的更细粒度版本。

### 在代码中使用标签
所有注册表的标签会在登录和重载时自动从服务器发送到任何远程客户端。`Block`、`Item`、`EntityType`、`Fluid` 和 `GameEvent` 是特殊情况，因为它们有 `Holder`，允许通过对象本身访问可用的标签。

!!! 注意
    侵入式 `Holder` 可能会在未来版本的 Minecraft 中被移除。如果移除了，下面的方法可以用来查询相关的 `Holder`。

#### ITagManager
Forge 包装的注册表通过 `ITagManager` 提供了一个额外的辅助工具来创建和管理标签，该工具可以通过 `IForgeRegistry#tags` 获得。可以使用 `#createTagKey` 或 `#createOptionalTagKey` 创建标签。也可以分别使用 `#getTag` 或 `#getReverseTag` 检查标签或注册表对象。

##### 自定义注册表
自定义注册表在构造其 `DeferredRegister` 时可以分别通过 `#createTagKey` 或 `#createOptionalTagKey` 创建标签。然后可以使用调用 `DeferredRegister#makeRegistry` 获得的 `IForgeRegistry` 检查它们的标签或注册表对象。

#### 引用标签
有四种创建标签包装器的方法：

| 方法 | 适用对象 |
| :---: | :--- |
| `*Tags#create` | `BannerPattern`、`Biome`、`Block`、`CatVariant`、`DamageType`、`EntityType`、`FlatLevelGeneratorPreset`、`Fluid`、`GameEvent`、`Instrument`、`Item`、`PaintingVariant`、`PoiType`、`Structure` 和 `WorldPreset`，其中 `*` 代表这些类型之一。 |
| `ITagManager#createTagKey` | Forge 包装的原版注册表，注册表可以从 `ForgeRegistries` 获得。 |
| `DeferredRegister#createTagKey` | 自定义 Forge 注册表。 |
| `TagKey#create` | 没有 Forge 包装的原版注册表，注册表可以从 `Registry` 获得。 |

注册表对象可以通过它们的 `Holder` 或分别通过 `ITag`/`IReverseTag` 检查它们的标签或注册表对象，分别适用于原版或 Forge 注册表对象。

原版注册表对象可以使用 `Registry#getHolder` 或 `Registry#getHolderOrThrow` 获取其关联的 `Holder`，然后使用 `Holder#is` 比较注册表对象是否有某个标签。

Forge 注册表对象可以使用 `ITagManager#getTag` 或 `ITagManager#getReverseTag` 获取其标签定义，然后分别使用 `ITag#contains` 或 `IReverseTag#containsTag` 比较注册表对象是否有某个标签。

持有标签的注册表对象在其注册表对象或状态感知类中有一个名为 `#is` 的方法，用于检查对象是否属于某个标签。

例如：
```java
public static final TagKey<Item> myItemTag = ItemTags.create(new ResourceLocation("mymod", "myitemgroup"));

public static final TagKey<Potion> myPotionTag = ForgeRegistries.POTIONS.tags().createTagKey(new ResourceLocation("mymod", "mypotiongroup"));

public static final TagKey<VillagerType> myVillagerTypeTag = TagKey.create(Registries.VILLAGER_TYPE, new ResourceLocation("mymod", "myvillagertypegroup"));

// 在某个方法中：

ItemStack stack = /*...*/;
boolean isInItemGroup = stack.is(myItemTag);

Potion potion = /*...*/;
boolean isInPotionGroup  = ForgeRegistries.POTIONS.tags().getTag(myPotionTag).contains(potion);

ResourceKey<VillagerType> villagerTypeKey = /*...*/;
boolean isInVillagerTypeGroup = BuiltInRegistries.VILLAGER_TYPE.getHolder(villagerTypeKey).map(holder -> holder.is(myVillagerTypeTag)).orElse(false);
```

### 约定
有几个约定有助于促进生态系统中的兼容性：
- 如果有适合你的方块或物品的原版标签，将其添加到该标签中。请参阅 [原版标签列表][taglist]。
- 如果有适合你的方块或物品的 Forge 标签，将其添加到该标签中。Forge 声明的标签列表可以在 [GitHub][forgetags] 上查看。
- 如果你觉得有一组事物应该由社区共享，请使用 `forge` 命名空间而不是你的模组 ID。
- 标签命名约定应遵循原版约定。特别是，物品和方块分组使用复数形式而不是单数形式（例如 `minecraft:logs`、`minecraft:saplings`）。
- 物品标签应根据其类型分类到子目录中（例如 `forge:ingots/iron`、`forge:nuggets/brass` 等）。

### 从 OreDictionary 迁移
- 对于合成配方，标签可以直接在原版合成配方格式中使用（见下文）。
- 对于在代码中匹配物品，请参阅上文。
- 如果你要声明一种新的物品分组类型，请遵循以下命名约定：
  - 使用 `domain:type/material`。当名称是所有模组开发者都应采用的通用名称时，使用 `forge` 域名。
  - 例如，黄铜锭应注册在 `forge:ingots/brass` 标签下，钴块应注册在 `forge:nuggets/cobalt` 标签下。

### 在合成配方和进度中使用标签
原版直接支持标签。有关使用详情，请参阅 [合成配方][recipes] 和 [进度][advancements] 的相应原版维基页面。

[datapack]: ./index.md
[tags]: https://minecraft.wiki/w/Tag#JSON_format
[taglist]: https://minecraft.wiki/w/Tag#List_of_tags
[forgetags]: https://github.com/MinecraftForge/MinecraftForge/tree/1.19.x/src/generated/resources/data/forge/tags
[recipes]: https://minecraft.wiki/w/Recipe#JSON_format
[advancements]: https://minecraft.wiki/w/Advancement
