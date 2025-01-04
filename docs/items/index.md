
物品
=====

与方块一起，项目是大多数mod的关键组成部分。方块构成了你周围的关卡，项目存在于库存中。

创建物品
----------------

### 基础物品

不需要特殊功能的基本物品（比如木棍或糖）不需要自定义类。您可以通过使用`Item$Properties`对象调用`Item`类的构造函数来创建物品。这`Item$Properties`对象可以通过其构造函数创建，并通过调用其方法进行自定义。例如：


|      Method        |                  Description                  |
|:------------------:|:----------------------------------------------|
| `requiredFeatures` | 设置让此物品在`创造模式选项卡`中能被看见所需的`FeatureFlag`。 |
| `durability`       | 设置此物品的最大耐久值。如果超过`0`，则添加两个物品属性“已损坏`damaged`”和“耐久`damage`”。 |
| `stacksTo`         | 设置最大堆叠大小。您不能创建同时拥有可损坏和可堆叠属性的物品。 |
| `setNoRepair`      | 使该物品无法修复，即使它是可损坏的。 |
| `craftRemainder`   | 设置此物品的容器物品，例如熔岩桶使用后给你一个空桶的方式。 |

上面的方法是可链接的，这意味着它们返回`this`以方便串联调用它们。

### 高级物品

如上设置物品的属性仅适用于简单物品，如果需要更复杂的物品，则应子类化`物品`并重载其方法。

## 创造模式选项卡

可以通过[mod事件总线][modbus]上的`BuildCreativeModeTabContentsEvent`将物品添加到`CreativeModeTab`。可以通过`#accept`添加物品而无需任何额外配置。


```java
// 在MOD事件总线上注册
// 假设我们有名为ITEM的RegistryObject<Item> 和名为BLOCK的RegistryObject<Block>
@SubscribeEvent
public void buildContents(BuildCreativeModeTabContentsEvent event) {
  // 添加到原料选项卡
  if (event.getTabKey() == CreativeModeTabs.INGREDIENTS) {
    event.accept(ITEM);
    event.accept(BLOCK); // 接收物品，假设方块已注册物品
  }
}
```

您还可以通过`FeatureFlagSet`中的`FeatureFlag`或决定玩家是否有权限查看创造模式选项卡的布尔值来启用或禁用添加项目。


### 自定义创造模式选项卡

自定义的`CreativeModeTab`必须[注册][registering]。构建器可以通过`CreativeModeTab#builder`创建。选项卡可以设置标题、图标、默认物品和许多其他属性。此外，Forge提供了额外的方法来自定义选项卡的图像、标签和插槽颜色，选项卡应该排在什么顺序等。

```java
// 假设我们有一个名为REGISTRAR的DeferredRegister<CreativeModeTab>
// 假设我们有名为ITEM的RegistryObject<Item> 和名为BLOCK的RegistryObject<Block>
public static final RegistryObject<CreativeModeTab> EXAMPLE_TAB = REGISTRAR.register("example", () -> CreativeModeTab.builder()
  // 设置要显示的选项卡名称
  .title(Component.translatable("item_group." + MOD_ID + ".example"))
  // 设置创造模式选项卡的图标
  .icon(() -> new ItemStack(ITEM.get()))
  // 向选项卡添加物品
  .displayItems((params, output) -> {
    output.accept(ITEM.get());
    output.accept(BLOCK.get());
  })
  .build()
);
```

注册物品
-------------------
物品必须[注册][注册]才能生效。

[modbus]: ../concepts/events.md#Mod事件总线
[registering]: ../concepts/registries.md#注册的方法
