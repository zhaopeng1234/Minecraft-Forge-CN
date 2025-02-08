## `ItemOverrides`

`ItemOverrides` 为 [`BakedModel`][baked] 提供了一种处理 `ItemStack` 状态并返回新的 `BakedModel` 的方式；此后，返回的模型将替换原来的模型。`ItemOverrides` 表示一个任意函数 `(BakedModel, ItemStack, ClientLevel, LivingEntity, int)` → `BakedModel`，这使其适用于动态模型。在原版游戏中，它用于实现物品属性覆盖。

### `ItemOverrides()`
给定一个 `ItemOverride` 列表，构造函数会复制并烘焙这个列表。可以使用 `#getOverrides` 方法访问烘焙后的覆盖项。

### `resolve`
该方法接受一个 `BakedModel`、一个 `ItemStack`、一个 `ClientLevel`、一个 `LivingEntity` 和一个 `int` 类型的参数，以生成另一个用于渲染的 `BakedModel`。这是模型处理其物品状态的地方。

此方法不应改变游戏世界的状态。

### `getOverrides`
返回一个包含此 `ItemOverrides` 使用的所有 [`BakedOverride`][override] 的不可变列表。如果没有适用的覆盖项，则返回空列表。

## `BakedOverride`
这个类表示一个原版物品覆盖项，它包含多个用于物品属性的 `ItemOverrides$PropertyMatcher`，以及在这些匹配器满足条件时要使用的模型。它们是原版物品 JSON 模型的 `overrides` 数组中的对象，示例如下：
```js
{
  // 在原版 JSON 物品模型内部
  "overrides": [
    {
      // 这是一个 ItemOverride
      "predicate": {
        // 这是一个 Map<ResourceLocation, Float>，包含属性名称及其最小值
        "example1:prop": 0.5
      },
      // 这是覆盖项的 'location' 或目标模型，如果上面的谓词匹配，则使用该模型
      "model": "example1:item/model"
    },
    {
      // 这是另一个 ItemOverride
      "predicate": {
        "example2:prop": 1
      },
      "model": "example2:item/model"
    }
  ]
}
```

[baked]: ./bakedmodel.md
[override]: #bakedoverride
