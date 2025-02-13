### 战利品表生成
通过构建一个新的 `LootTableProvider` 并提供 `LootTableProvider$SubProviderEntry`，可以为模组生成[战利品表][loottable]。该提供者必须 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        output -> new MyLootTableProvider(
          output,
          // 指定生成所需的表的注册表名称，也可以留空
          Collections.emptySet(),
          // 生成战利品的子提供者
          List.of(subProvider1, subProvider2, /*...*/)
        )
    );
}
```

### `LootTableSubProvider`
每个 `LootTableProvider$SubProviderEntry` 接受一个提供的 `LootTableSubProvider`，该 `LootTableSubProvider` 为给定的 `LootContextParamSet` 生成战利品表。`LootTableSubProvider` 包含一个方法，该方法接受写入器（`BiConsumer<ResourceLocation, LootTable.Builder>`）以生成表。

```java
public class ExampleSubProvider implements LootTableSubProvider {

  // 用于为包装的 Supplier 创建工厂方法
  public ExampleSubProvider() {}

  // 用于生成战利品表的方法
  @Override
  public void generate(BiConsumer<ResourceLocation, LootTable.Builder> writer) {
    // 通过调用 writer#accept 在此处生成战利品表
  }
}
```

然后，可以将该表添加到 `LootTableProvider#getTables` 中，以用于任何可用的 `LootContextParamSet`：

```java
// 在传递给 LootTableProvider 构造函数的列表中
new LootTableProvider.SubProviderEntry(
  ExampleSubProvider::new,
  // 为 'empty' 参数集生成战利品表
  LootContextParamSets.EMPTY
)
```

#### `BlockLootSubProvider` 和 `EntityLootSubProvider` 子类
对于 `LootContextParamSets#BLOCK` 和 `#ENTITY`，有特殊类型（分别为 `BlockLootSubProvider` 和 `EntityLootSubProvider`），它们提供了额外的辅助方法，用于创建和验证是否存在战利品表。

`BlockLootSubProvider` 的构造函数接受一个物品列表，这些物品具有抗爆炸属性，用于确定如果方块被炸掉是否可以生成战利品表；还接受一个 `FeatureFlagSet`，用于确定方块是否启用，以便为其生成战利品表。

```java
// 在某个 BlockLootSubProvider 子类中
public MyBlockLootSubProvider() {
  super(Collections.emptySet(), FeatureFlags.REGISTRY.allFlags());
}
```

`EntityLootSubProvider` 的构造函数接受一个 `FeatureFlagSet`，用于确定实体类型是否启用，以便为其生成战利品表。

```java
// 在某个 EntityLootSubProvider 子类中
public MyEntityLootSubProvider() {
  super(FeatureFlags.REGISTRY.allFlags());
}
```

要使用它们，必须将所有已注册的对象分别提供给 `BlockLootSubProvider#getKnownBlocks` 和 `EntityLootSubProvider#getKnownEntityTypes`。这些方法是为了确保可迭代对象中的所有对象都有一个战利品表。

!!! 提示
    如果使用 `DeferredRegister` 来注册模组的对象，那么可以通过 `DeferredRegister#getEntries` 将条目提供给 `#getKnown*` 方法：

    ```java
    // 在某个 BlockLootSubProvider 子类中，针对某个 DeferredRegister BLOCK_REGISTRAR
    @Override
    protected Iterable<Block> getKnownBlocks() {
      return BLOCK_REGISTRAR.getEntries() // 获取所有已注册的条目
        .stream() // 对流包装的对象
        .flatMap(RegistryObject::stream) // 如果可用，获取对象
        ::iterator; // 创建可迭代对象
    }
    ```

可以通过实现 `#generate` 方法来添加战利品表本身。

```java
// 在某个 BlockLootSubProvider 子类中
@Override
public void generate() {
  // 在此处添加战利品表
}
```

### 战利品表构建器
为了生成战利品表，`LootTableSubProvider` 接受它们作为 `LootTable$Builder`。然后，在 `LootTableProvider$SubProviderEntry` 中设置指定的 `LootContextParamSet`，并通过 `#build` 方法构建。在构建之前，构建器可以指定条目、条件和修饰符，这些会影响战利品表的功能。

!!! 注意
    战利品表的功能非常广泛，本文档不会全面涵盖。相反，将简要介绍每个组件。每个组件的具体子类型可以使用 IDE 找到。它们的实现将留给读者作为练习。

#### LootTable
战利品表是基础对象，可以使用 `LootTable#lootTable` 将其转换为所需的 `LootTable$Builder`。战利品表可以使用指定顺序应用的池列表（通过 `#withPool`）以及函数（通过 `#apply`）来构建，以修改这些池的结果物品。

#### LootPool
战利品池表示一组操作，可以使用 `LootPool#lootPool` 生成 `LootPool$Builder`。每个战利品池可以指定条目（通过 `#add`），这些条目定义了池中要执行的操作；条件（通过 `#when`），这些条件定义了是否应该执行池中操作；以及函数（通过 `#apply`），用于修改条目的结果物品。每个池可以按照指定的次数执行（通过 `#setRolls`）。此外，可以指定额外执行次数（通过 `#setBonusRolls`），这会受到执行者运气的影响。

#### LootPoolEntryContainer
战利品条目定义了被选中时要执行的操作，通常是生成物品。每个条目都有一个关联的、[已注册] `LootPoolEntryType`。它们也有自己关联的构建器，这些构建器继承自 `LootPoolEntryContainer$Builder`。多个条目可以同时执行（通过 `#append`），或者按顺序执行，直到有一个失败（通过 `#then`）。此外，条目在失败时可以默认执行另一个条目（通过 `#otherwise`）。

#### LootItemCondition
战利品条件定义了某些操作执行所需满足的要求。每个条件都有一个关联的、[已注册] `LootItemConditionType`。它们也有自己关联的构建器，这些构建器继承自 `LootItemCondition$Builder`。默认情况下，指定的所有战利品条件都必须返回 `true`，操作才能执行。也可以指定战利品条件，使得只有一个条件返回 `true` 即可（通过 `#or`）。此外，条件的结果输出可以反转（通过 `#invert`）。

#### LootItemFunction
战利品函数在将执行结果传递到输出之前对其进行修改。每个函数都有一个关联的、[已注册] `LootItemFunctionType`。它们也有自己关联的构建器，这些构建器继承自 `LootItemFunction$Builder`。

##### NbtProvider
NBT 提供者是由 `CopyNbtFunction` 定义的一种特殊类型的函数。它们定义了从哪里获取标签信息。每个提供者都有一个关联的、[已注册] `LootNbtProviderType`。

#### NumberProvider
数字提供者确定战利品池执行的次数。每个提供者都有一个关联的、[已注册] `LootNumberProviderType`。

##### ScoreboardNameProvider
计分板提供者是由 `ScoreboardValue` 定义的一种特殊类型的数字提供者。它们定义了从哪个计分板获取要执行的滚动次数。每个提供者都有一个关联的、[已注册] `LootScoreProviderType`。

[loottable]: ../../resources/server/loottables.md
[datagen]: ../index.md#data-providers
[registered]: ../../concepts/registries.md#registries-that-arent-forge-registries
