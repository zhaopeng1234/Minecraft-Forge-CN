### 标签生成
通过继承 `TagsProvider` 并实现 `#addTags` 方法，可以为模组生成[标签]。实现之后，必须将该提供者 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        // 继承自 net.minecraftforge.common.data.BlockTagsProvider
        output -> new MyBlockTagsProvider(
          output,
          event.getLookupProvider(),
          MOD_ID,
          event.getExistingFileHelper()
        )
    );
}
```

### `TagsProvider`
标签提供者有两种用于生成标签的方法：通过 `#tag` 方法创建包含对象和其他标签的标签，或者通过 `#getOrCreateRawBuilder` 方法使用其他对象类型的标签来生成标签数据。

!!! 注意
    通常情况下，除非某个注册表包含来自不同注册表的对象表示（方块有物品表示，以便在物品栏中获取方块），否则提供者不会直接调用 `#getOrCreateRawBuilder` 方法。

当调用 `#tag` 方法时，会创建一个 `TagAppender` 对象，它作为一个可链式调用的消费者，用于向标签中添加元素：

| 方法 | 描述 |
| :---: | --- |
| `add` | 通过资源键将一个对象添加到标签中。 |
| `addOptional` | 通过名称将一个对象添加到标签中。如果该对象不存在，则在加载时会跳过该对象。 |
| `addTag` | 通过标签键将一个标签添加到另一个标签中。内部标签中的所有元素现在都成为外部标签的一部分。 |
| `addOptionalTag` | 通过名称将一个标签添加到另一个标签中。如果该标签不存在，则在加载时会跳过该标签。 |
| `replace` | 当值为 `true` 时，所有之前从其他数据包添加到该标签的条目都将被丢弃。如果在此之后加载另一个数据包，它仍会将条目追加到该标签中。 |
| `remove` | 通过名称或键从标签中移除一个对象或标签。 |

```java
// 在某个 TagProvider#addTags 方法中
this.tag(EXAMPLE_TAG)
  .add(EXAMPLE_OBJECT) // 向标签中添加一个对象
  .addOptional(new ResourceLocation("othermod", "other_object")) // 向标签中添加另一个模组的对象

this.tag(EXAMPLE_TAG_2)
  .addTag(EXAMPLE_TAG) // 向标签中添加另一个标签
  .remove(EXAMPLE_OBJECT) // 从该标签中移除一个对象
```

!!! 重要
    如果模组的标签对另一个模组的标签是软依赖（另一个模组在运行时可能存在也可能不存在），则应该使用可选方法来引用其他模组的标签。

#### 现有提供者
Minecraft 为某些注册表提供了一些标签提供者，可以继承这些提供者。此外，一些提供者还包含额外的辅助方法，以便更轻松地创建标签。

| 注册表对象类型 | 标签提供者 |
| :---: | --- |
| `Block` | `BlockTagsProvider`\* |
| `Item` | `ItemTagsProvider` |
| `EntityType` | `EntityTypeTagsProvider` |
| `Fluid` | `FluidTagsProvider` |
| `GameEvent` | `GameEventTagsProvider` |
| `Biome` | `BiomeTagsProvider` |
| `FlatLevelGeneratorPreset` | `FlatLevelGeneratorPresetTagsProvider` |
| `WorldPreset` | `WorldPresetTagsProvider` |
| `Structure` | `StructureTagsProvider` |
| `PoiType` | `PoiTypeTagsProvider` |
| `BannerPattern` | `BannerPatternTagsProvider` |
| `CatVariant` | `CatVariantTagsProvider` |
| `PaintingVariant` | `PaintingVariantTagsProvider` |
| `Instrument` | `InstrumentTagsProvider` |
| `DamageType` | `DamageTypeTagsProvider` |

\* `BlockTagsProvider` 是 Forge 添加的 `TagsProvider`。

##### `ItemTagsProvider#copy`
方块有物品表示，以便在物品栏中获取它们。因此，许多方块标签也可以作为物品标签。为了轻松生成与方块标签具有相同条目的物品标签，可以使用 `#copy` 方法，该方法接受要复制的方块标签和要复制到的物品标签。

```java
// 在 ItemTagsProvider#addTags 方法中
this.copy(EXAMPLE_BLOCK_TAG, EXAMPLE_ITEM_TAG);
```

### 自定义标签提供者
可以通过继承 `TagsProvider` 子类来创建自定义标签提供者，该子类接受用于生成标签的注册表键。

```java
public RecipeTypeTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> registries, ExistingFileHelper fileHelper) {
  super(output, Registries.RECIPE_TYPE, registries, MOD_ID, fileHelper);
}
```

#### 内在持有者标签提供者
一种特殊类型的 `TagProvider` 是 `IntrinsicHolderTagsProvider`。当使用此提供者通过 `#tag` 方法创建标签时，可以使用对象本身通过 `#add` 方法将其自身添加到标签中。为此，在构造函数中提供一个函数，将对象转换为其 `ResourceKey`。

```java
// `IntrinsicHolderTagsProvider` 的子类
public AttributeTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> registries, ExistingFileHelper fileHelper) {
  super(
    output,
    ForgeRegistries.Keys.ATTRIBUTES,
    registries,
    attribute -> ForgeRegistries.ATTRIBUTES.getResourceKey(attribute).get(),
    MOD_ID,
    fileHelper
  );
}
```

[tags]: ../../resources/server/tags.md
[datagen]: ../index.md#data-providers
[custom]: ../../concepts/registries.md#creating-custom-forge-registries
