### 数据包注册表对象生成
可以通过构建一个新的 `DatapackBuiltinEntriesProvider` 并提供一个包含要注册的新对象的 `RegistrySetBuilder` 来为模组生成数据包注册表对象。该提供者必须 [添加][datagen] 到 `DataGenerator` 中。

!!! 注意
    `DatapackBuiltinEntriesProvider` 是 `RegistriesDatapackGenerator` 的 Forge 扩展，它能正确处理对现有数据包注册表对象的引用，而不会导致条目出错。因此，本文档将使用 `DatapackBuiltinEntriesProvider`。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        output -> new DatapackBuiltinEntriesProvider(
          output,
          event.getLookupProvider(),
          // 包含要生成的数据包注册表对象的构建器
          new RegistrySetBuilder().add(/* ... */),
          // 要生成数据包注册表对象的模组 ID 集合
          Set.of(MOD_ID)
        )
    );
}
```

### `RegistrySetBuilder`
`RegistrySetBuilder` 负责构建游戏中要使用的所有数据包注册表对象。该构建器可以为一个注册表添加新条目，然后向该注册表注册对象。

首先，可以通过调用构造函数初始化一个新的 `RegistrySetBuilder` 实例。然后，可以调用 `#add` 方法（该方法接受注册表的 `ResourceKey`、一个包含 `BootstapContext` 以注册对象的 `RegistryBootstrap` 消费者，以及一个可选的 `Lifecycle` 参数来指示注册表的当前生命周期状态）来处理特定注册表的注册。

```java
new RegistrySetBuilder()
  // 创建配置功能
  .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
    // 在此处注册配置功能
  })
  // 创建放置功能
  .add(Registries.PLACED_FEATURE, bootstrap -> {
    // 在此处注册放置功能
  });
```

!!! 注意
    通过 Forge 创建的数据包注册表也可以使用此构建器生成其对象，只需传入关联的 `ResourceKey`。

### 使用 `BootstapContext` 进行注册
构建器提供的 `BootstapContext` 中的 `#register` 方法可用于注册对象。它接受表示对象注册表名称的 `ResourceKey`、要注册的对象，以及一个可选的 `Lifecycle` 参数来指示注册表对象的当前生命周期状态。

```java
public static final ResourceKey<ConfiguredFeature<?, ?>> EXAMPLE_CONFIGURED_FEATURE = ResourceKey.create(
  Registries.CONFIGURED_FEATURE,
  new ResourceLocation(MOD_ID, "example_configured_feature")
);

// 在某个常量位置或参数中
new RegistrySetBuilder()
  // 创建配置功能
  .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
    // 在此处注册配置功能
    bootstrap.register(
      // 配置功能的资源键
      EXAMPLE_CONFIGURED_FEATURE,
      new ConfiguredFeature<>(
        Feature.ORE, // 创建一个矿石功能
        new OreConfiguration(
          List.of(), // 不做任何处理
          8 // 最多以 8 个为一组生成
        )
      )
    );
  })
  // 创建放置功能
  .add(Registries.PLACED_FEATURE, bootstrap -> {
    // 在此处注册放置功能
  });
```

### 数据包注册表对象查找
有时，数据包注册表对象可能希望使用其他数据包注册表对象或包含数据包注册表对象的标签。在这种情况下，可以使用 `BootstapContext#lookup` 查找另一个数据包注册表，以获取一个 `HolderGetter`。从那里，可以通过传递关联的键，使用 `#getOrThrow` 获取数据包注册表对象的 `Holder$Reference` 或标签的 `HolderSet$Named`。

```java
public static final ResourceKey<ConfiguredFeature<?, ?>> EXAMPLE_CONFIGURED_FEATURE = ResourceKey.create(
  Registries.CONFIGURED_FEATURE,
  new ResourceLocation(MOD_ID, "example_configured_feature")
);

public static final ResourceKey<PlacedFeature> EXAMPLE_PLACED_FEATURE = ResourceKey.create(
  Registries.PLACED_FEATURE,
  new ResourceLocation(MOD_ID, "example_placed_feature")
);

// 在某个常量位置或参数中
new RegistrySetBuilder()
  // 创建配置功能
  .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
    // 在此处注册配置功能
    bootstrap.register(
      // 配置功能的资源键
      EXAMPLE_CONFIGURED_FEATURE,
      new ConfiguredFeature(/* ... */)
    );
  })
  // 创建放置功能
  .add(Registries.PLACED_FEATURE, bootstrap -> {
    // 在此处注册放置功能

    // 获取配置功能注册表
    HolderGetter<ConfiguredFeature<?, ?>> configured = bootstrap.lookup(Registries.CONFIGURED_FEATURE);

    bootstrap.register(
      // 放置功能的资源键
      EXAMPLE_PLACED_FEATURE,
      new PlacedFeature(
        configured.getOrThrow(EXAMPLE_CONFIGURED_FEATURE), // 获取配置功能
        List.of() // 对放置位置不做任何处理
      )
    )
  });
```

[datagen]: ../index.md#data-providers
