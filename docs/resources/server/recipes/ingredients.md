## 材料（Ingredients）
`Ingredient` 是基于物品的输入的谓词处理器，用于检查某个 `ItemStack` 是否满足成为合成配方中有效输入的条件。所有接受输入的[原版合成配方][recipes]都使用一个 `Ingredient` 或一个 `Ingredient` 列表，然后将其合并为一个单一的 `Ingredient`。

### 自定义材料
除了[复合材料][compound]之外，可以通过将 `type` 设置为[材料序列化器][serializer]的名称来指定自定义材料。如果未指定类型，`type` 默认使用原版材料 `minecraft:item`。自定义材料也可以轻松用于[数据生成][datagen]。

#### Forge 提供的额外材料类型
Forge 为开发者提供了一些额外的 `Ingredient` 类型。

##### 复合材料（CompoundIngredient）
虽然功能相同，但复合材料取代了在合成配方中实现材料列表的方式。它们作为一个逻辑或集合工作，即传入的物品堆必须至少符合所提供的材料之一。进行此更改是为了让自定义材料在列表中能正常工作。因此，**无需指定类型**。
```js
// 对于某些输入
[
  // 这些材料中至少有一个必须匹配才能成功
  {
    // 材料
  },
  {
    // 自定义材料
    "type": "examplemod:example_ingredient"
  }
]
```

##### 严格 NBT 材料（StrictNBTIngredient）
`StrictNBTIngredient` 会比较 `ItemStack` 上的物品、耐久度和共享标签（由 `IForgeItem#getShareTag` 定义），以确保完全等效。可以通过将 `type` 指定为 `forge:nbt` 来使用它。
```js
// 对于某些输入
{
  "type": "forge:nbt",
  "item": "examplemod:example_item",
  "nbt": {
    // 添加 NBT 数据（必须与物品堆上的完全匹配）
  }
}
```

##### 部分 NBT 材料（PartialNBTIngredient）
`PartialNBTIngredient` 是[`StrictNBTIngredient`][nbt] 的一种宽松版本，它针对单个或一组物品进行比较，并且仅比较共享标签（由 `IForgeItem#getShareTag` 定义）中指定的键。可以通过将 `type` 指定为 `forge:partial_nbt` 来使用它。
```js
// 对于某些输入
{
  "type": "forge:partial_nbt",

  // 必须指定 'item' 或 'items' 中的一个
  // 如果两者都指定，仅会读取 'item'
  "item": "examplemod:example_item",
  "items": [
    "examplemod:example_item",
    "examplemod:example_item2"
    // ...
  ],

  "nbt": {
    // 仅检查 'key1' 和 'key2' 的等效性
    // 物品堆中的所有其他键将不会被检查
    "key1": "data1",
    "key2": {
      // 数据 2
    }
  }
}
```

##### 交集材料（IntersectionIngredient）
`IntersectionIngredient` 作为一个逻辑与集合工作，即传入的物品堆必须匹配所有提供的材料。必须至少提供两个材料。可以通过将 `type` 指定为 `forge:intersection` 来使用它。
```js
// 对于某些输入
{
  "type": "forge:intersection",

  // 所有这些材料都必须返回 true 才能成功
  "children": [
    {
      // 材料 1
    },
    {
      // 材料 2
    }
    // ...
  ]
}
```

##### 差集材料（DifferenceIngredient）
`DifferenceIngredient` 作为一个集合减法（SUB）工作，即传入的物品堆必须匹配第一个材料，但不能匹配第二个材料。可以通过将 `type` 指定为 `forge:difference` 来使用它。
```js
// 对于某些输入
{
  "type": "forge:difference",
  "base": {
    // 物品堆所属的材料
  },
  "subtracted": {
    // 物品堆不属于的材料
  }
}
```

### 创建自定义材料
可以通过为创建的 `Ingredient` 子类实现 `IIngredientSerializer` 来创建自定义材料。

!!! 提示
    自定义材料应继承 `AbstractIngredient`，因为它提供了一些有用的抽象，便于实现。

#### 材料子类
每个材料子类需要实现三个重要的方法：

| 方法 | 描述 |
| :---: | :--- |
| `getSerializer` | 返回用于读写材料的[序列化器]。 |
| `test` | 如果输入对于此材料有效，则返回 `true`。 |
| `isSimple` | 如果材料根据物品堆的标签进行匹配，则返回 `false`。`AbstractIngredient` 子类需要定义此行为，而 `Ingredient` 子类默认返回 `true`。 |

所有其他定义的方法由开发者根据材料子类的需要使用。

#### IIngredientSerializer
`IIngredientSerializer` 子类型必须实现三个方法：

| 方法 | 描述 |
| :---: | :--- |
| `parse (JSON)` | 将 `JsonObject` 转换为 `Ingredient`。 |
| `parse (Network)` | 读取网络缓冲区以解码 `Ingredient`。 |
| `write` | 将 `Ingredient` 写入网络缓冲区。 |

此外，`Ingredient` 子类应该实现 `Ingredient#toJson` 以便用于[数据生成][datagen]。`AbstractIngredient` 子类将 `#toJson` 设为抽象方法，要求实现该方法。

之后，应声明一个静态实例来保存初始化的序列化器，然后在 `RecipeSerializer` 的 `RegisterEvent` 期间或 `FMLCommonSetupEvent` 期间使用 `CraftingHelper#register` 进行注册。`Ingredient` 子类在 `Ingredient#getSerializer` 中返回序列化器的静态实例。
```java
// 在某个序列化器类中
public static final ExampleIngredientSerializer INSTANCE = new ExampleIngredientSerializer();

// 在某个处理类中
public void registerSerializers(RegisterEvent event) {
  event.register(ForgeRegistries.Keys.RECIPE_SERIALIZERS,
    helper -> CraftingHelper.register(registryName, INSTANCE)
  );
}

// 在某个材料子类中
@Override
public IIngredientSerializer<? extends Ingredient> getSerializer() {
  return INSTANCE;
}
```

!!! 提示
    如果使用 `FMLCommonSetupEvent` 来注册材料序列化器，必须通过 `FMLCommonSetupEvent#enqueueWork` 将其加入同步工作队列，因为 `CraftingHelper#register` 不是线程安全的。

[recipes]: https://minecraft.wiki/w/Recipe#List_of_recipe_types
[nbt]: #strictnbtingredient
[serializer]: #iingredientserializer
[compound]: #compoundingredient
[datagen]: ../../../datagen/server/recipes.md
