### 进度生成
通过构建一个新的 `AdvancementProvider` 并提供 `AdvancementSubProvider`，可以为模组生成[进度]。进度可以手动创建并提供，或者为了方便起见，使用 `Advancement$Builder` 创建。该提供者必须 [添加][datagen] 到 `DataGenerator` 中。

!!! 注意
    Forge 为 `AdvancementProvider` 提供了一个扩展，名为 `ForgeAdvancementProvider`，它在生成进度方面集成得更好。因此，本文档将使用 `ForgeAdvancementProvider` 以及子提供者接口 `ForgeAdvancementProvider$AdvancementGenerator`。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        output -> new ForgeAdvancementProvider(
          output,
          event.getLookupProvider(),
          event.getExistingFileHelper(),
          // 生成进度的子提供者
          List.of(subProvider1, subProvider2, /*...*/)
        )
    );
}
```

### `ForgeAdvancementProvider$AdvancementGenerator`
`ForgeAdvancementProvider$AdvancementGenerator` 负责生成进度，它包含一个方法，该方法接受注册表查找、写入器（`Consumer<Advancement>`）和现有文件帮助器。

```java
// 在 ForgeAdvancementProvider$AdvancementGenerator 的某个子类中或作为 lambda 引用

@Override
public void generate(HolderLookup.Provider registries, Consumer<Advancement> writer, ExistingFileHelper existingFileHelper) {
  // 在此处构建进度
}
```

### `Advancement$Builder`
`Advancement$Builder` 是一个方便的实现，用于创建要生成的 `Advancement`。它允许定义父进度、显示信息、进度完成时的奖励以及解锁进度的要求。创建 `Advancement` 时，仅需要指定要求。

虽然不是必需的，但有一些重要的方法需要了解：

| 方法 | 描述 |
| :---: | --- |
| `parent` | 设置此进度直接关联的父进度。可以指定进度的名称，或者如果是模组开发者生成的进度，也可以直接指定进度本身。 |
| `display` | 设置要显示在聊天、提示框和进度屏幕上的信息。 |
| `rewards` | 设置此进度完成时获得的奖励。 |
| `addCriterion` | 为进度添加一个条件。 |
| `requirements` | 指定条件是否必须全部返回 `true`，或者至少有一个必须返回 `true`。还可以使用额外的重载方法来混合这些操作。 |

一旦 `Advancement$Builder` 准备好构建，应该调用 `#save` 方法，该方法接受写入器、进度的注册表名称以及用于检查所提供的父进度是否存在的文件帮助器。

```java
// 在某个 ForgeAdvancementProvider$AdvancementGenerator#generate(registries, writer, existingFileHelper) 方法中
Advancement example = Advancement.Builder.advancement()
  .addCriterion("example_criterion", triggerInstance) // 进度如何解锁
  .save(writer, name, existingFileHelper); // 将数据添加到构建器
```

[advancements]: ../../resources/server/advancements.md
[datagen]: ../index.md#data-providers
[conditional]: ../../resources/server/conditional.md
