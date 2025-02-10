## 物品属性
物品属性是一种将物品 “属性” 暴露给模型系统的方式。以弓为例，其最重要的属性就是拉开的程度。这些信息随后用于为弓选择模型，从而创建拉弓的动画。

物品属性会为其注册的每个 `ItemStack` 分配一个特定的 `float` 值，原版物品模型定义可以使用这些值来定义 “覆盖”。在这种情况下，物品默认使用某个模型，但如果某个覆盖条件匹配，就会覆盖原模型并使用另一个模型。物品属性之所以有用，主要是因为它们是连续的。例如，弓使用物品属性来定义其拉弓动画。物品模型由 “float” 数值谓词决定，数值范围不限，但通常在 `0.0F` 到 `1.0F` 之间。这使得资源包可以在这个范围内为拉弓动画添加任意数量的模型，而不是局限于动画中的四个 “固定位置” 来添加模型。指南针和时钟也是如此。

### 为物品添加属性
`ItemProperties#register` 用于为特定物品添加属性。`Item` 参数是要附加属性的物品（例如 `ExampleItems#APPLE`）。`ResourceLocation` 参数是赋予该属性的名称（例如 `new ResourceLocation("pull")`）。`ItemPropertyFunction` 是一个函数式接口，它接受 `ItemStack`、所在的 `ClientLevel`（可能为 null）、持有该物品的 `LivingEntity`（可能为 null）以及包含持有实体 id 的 `int` 类型参数（可能为 `0`），返回该属性的 `float` 值。对于模组添加的物品属性，建议使用模组的 mod id 作为命名空间（例如 `examplemod:property`，而不是仅使用 `property`，因为后者实际上表示 `minecraft:property`）。这些操作应在 `FMLClientSetupEvent` 中完成。

还有另一个方法 `ItemProperties#registerGeneric`，用于为所有物品添加属性，由于所有物品都会应用此属性，所以它不接受 `Item` 作为参数。

!!! 重要
    使用 `FMLClientSetupEvent#enqueueWork` 来执行这些任务，因为 `ItemProperties` 中的数据结构不是线程安全的。

!!! 注意
    Mojang 已弃用 `ItemPropertyFunction`，推荐使用其子接口 `ClampedItemPropertyFunction`，它会将结果限制在 `0` 到 `1` 之间。

### 使用覆盖
覆盖的格式可以在 [维基百科][format] 上查看，在 `model/item/bow.json` 中可以找到一个很好的示例。作为参考，以下是一个假设的具有 `examplemod:power` 属性的物品示例。如果没有匹配的值，默认使用当前模型，但如果有多个匹配项，则会选择列表中的最后一个匹配项。

!!! 重要
    谓词适用于所有*大于或等于*给定值的值。

```js
{
  "parent": "item/generated",
  "textures": {
    // 默认
    "layer0": "examplemod:items/example_partial"
  },
  "overrides": [
    {
      // power >=.75
      "predicate": {
        "examplemod:power": 0.75
      },
      "model": "examplemod:item/example_powered"
    }
  ]
}
```

以下是支持代码中的一个假设片段。与旧版本（低于 1.16.x）不同，由于 `ItemProperties` 在服务器端不存在，此操作仅需在客户端完成。

```java
private void setup(final FMLClientSetupEvent event)
{
  event.enqueueWork(() ->
  {
    ItemProperties.register(ExampleItems.APPLE, 
      new ResourceLocation(ExampleMod.MODID, "pulling"), (stack, level, living, id) -> {
        return living!= null && living.isUsingItem() && living.getUseItem() == stack? 1.0F : 0.0F;
      });
  });
}
```

[format]: https://minecraft.wiki/w/Tutorials/Models#Item_models
