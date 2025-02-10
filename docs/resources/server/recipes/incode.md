## 非数据包合成配方
并非所有合成配方都足够简单，或者已经迁移到使用数据驱动的合成配方。有些子系统仍需要在代码库中进行修补，以支持添加新的合成配方。

### 酿造合成配方
酿造是少数仍在代码中实现的合成配方之一。酿造合成配方作为 `PotionBrewing` 引导过程的一部分，为其容器、容器配方和药水混合添加。为了扩展现有系统，Forge 允许在 `FMLCommonSetupEvent` 中调用 `BrewingRecipeRegistry#addRecipe` 来添加酿造合成配方。

!!! 警告
    由于 `BrewingRecipeRegistry#addRecipe` 方法不是线程安全的，必须通过 `#enqueueWork` 在同步工作队列中调用它。

默认实现接受一个输入材料、一个催化剂材料和一个标准实现的物品堆输出。此外，也可以提供一个 `IBrewingRecipe` 实例来进行转换。

#### IBrewingRecipe
`IBrewingRecipe` 是一个类似[`Recipe`][recipe]的伪接口，用于检查输入和催化剂是否有效，并在有效时提供相关的输出。这分别通过 `#isInput`、`#isIngredient` 和 `#getOutput` 方法实现。输出方法可以访问输入和催化剂物品堆来构建结果。

!!! 重要
    在 `ItemStack` 或 `CompoundTag` 之间复制数据时，确保使用它们各自的 `#copy` 方法来创建唯一的实例。

没有类似于原版的用于添加额外药水容器或药水混合的包装器。需要添加一个新的 `IBrewingRecipe` 实现来复制此行为。

### 铁砧合成配方
铁砧负责处理受损的输入物品，并在提供一些材料或类似的输入时，减少输入物品的损坏。因此，其系统不容易实现数据驱动。然而，由于铁砧合成配方是一个输入物品加上一定数量的材料，在玩家拥有所需经验等级时得到一个输出物品，因此可以通过 `AnvilUpdateEvent` 修改它以创建一个伪合成配方系统。该事件接受输入物品和材料，并允许模组开发者指定输出物品、经验等级成本以及用于输出的材料数量。也可以通过[取消][cancel]该事件来阻止任何输出。

```java
// 检查左侧和右侧物品是否正确
// 当条件为真时，设置输出物品、经验等级成本和材料数量
public void updateAnvil(AnvilUpdateEvent event) {
  if (event.getLeft().is(...) && event.getRight().is(...)) {
    event.setOutput(...);
    event.setCost(...);
    event.setMaterialCost(...);
  }
}
```

更新事件必须[附加][attached]到 Forge 事件总线。

### 织布机合成配方
织布机负责将染料和图案（来自织布机或物品）应用到旗帜上。虽然旗帜和染料必须分别是 `BannerItem` 或 `DyeItem`，但可以创建自定义图案并在织布机中应用。可以通过[注册][registering]一个 `BannerPattern` 来创建旗帜图案。

!!! 重要
    位于 `minecraft:no_item_required` 标签中的 `BannerPattern` 会作为选项显示在织布机中。不在此标签中的图案必须有一个配套的 `BannerPatternItem` 以及相关标签才能使用。

```java
private static final DeferredRegister<BannerPattern> REGISTER = DeferredRegister.create(Registries.BANNER_PATTERN, "examplemod");

// 传入要通过网络发送的图案名称
public static final BannerPattern EXAMPLE_PATTERN = REGISTER.register("example_pattern", () -> new BannerPattern("examplemod:ep"));
```

[recipe]: ./custom.md#recipe
[cancel]: ../../../concepts/events.md#canceling
[attached]: ../../../concepts/events.md#creating-an-event-handler
[registering]: ../../../concepts/registries.md#registries-that-arent-forge-registries
