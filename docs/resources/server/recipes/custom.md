## 自定义合成配方

每个合成配方定义都由三个部分组成：`Recipe` 实现，它保存数据并使用提供的输入处理执行逻辑；`RecipeType`，它表示合成配方将使用的类别或上下文；以及 `RecipeSerializer`，它处理合成配方数据的解码和网络通信。如何使用合成配方由实现者决定。

### 合成配方（Recipe）

`Recipe` 接口描述了合成配方的数据和执行逻辑。这包括匹配输入并提供相关的结果。由于合成配方子系统默认执行物品转换，输入通过 `Container` 子类型提供。

!!! 重要
    传递给合成配方的 `Container` 应被视为其内容是不可变的。任何可变操作都应该在输入的副本上执行，通过 `ItemStack#copy`。

为了能够从管理器中获取合成配方实例，`#matches` 方法必须返回 `true`。此方法检查提供的容器，以确定相关输入是否有效。可以通过调用 `Ingredient#test` 使用 `Ingredient` 进行验证。

如果选择了该合成配方，则使用 `#assemble` 方法构建它，该方法可能会使用输入中的数据来创建结果。

!!! 提示
    `#assemble` 应该始终生成一个唯一的 `ItemStack`。如果不确定 `#assemble` 是否这样做，在返回结果之前调用 `ItemStack#copy`。

大多数其他方法纯粹是为了与合成配方书集成。

```java
public record ExampleRecipe(Ingredient input, int data, ItemStack output) implements Recipe<Container> {
  // 在此实现方法
}
```

!!! 注意
    虽然上面的示例中使用了记录（record），但在你自己的实现中不一定要这样做。

### 合成配方类型（RecipeType）

`RecipeType` 负责定义合成配方将使用的类别或上下文。例如，如果一个合成配方要在熔炉中熔炼，它的类型将是 `RecipeType#SMELTING`。在高炉中爆破的合成配方类型将是 `RecipeType#BLASTING`。

如果现有的类型都不匹配合成配方将使用的上下文，则必须[注册][forge]一个新的 `RecipeType`。

然后，新合成配方子类型中的 `Recipe#getType` 方法必须返回 `RecipeType` 实例。

```java
// 对于某个 RegistryObject<RecipeType> EXAMPLE_TYPE
// 在 ExampleRecipe 中
@Override
public RecipeType<?> getType() {
  return EXAMPLE_TYPE.get();
}
```

### 合成配方序列化器（RecipeSerializer）

`RecipeSerializer` 负责为相关的 `Recipe` 子类型解码 JSON 并进行网络通信。序列化器解码的每个合成配方都作为唯一实例保存在 `RecipeManager` 中。`RecipeSerializer` 必须[注册][forge]。

`RecipeSerializer` 只需要实现三个方法：

| 方法 | 描述 |
| :---: | :--- |
| `fromJson` | 将 JSON 解码为 `Recipe` 子类型。 |
| `toNetwork` | 将 `Recipe` 编码到缓冲区以发送到客户端。不需要编码合成配方标识符。 |
| `fromNetwork` | 从服务器发送的缓冲区中解码 `Recipe`。不需要解码合成配方标识符。 |

然后，新合成配方子类型中的 `Recipe#getSerializer` 方法必须返回 `RecipeSerializer` 实例。

```java
// 对于某个 RegistryObject<RecipeSerializer> EXAMPLE_SERIALIZER
// 在 ExampleRecipe 中
@Override
public RecipeSerializer<?> getSerializer() {
  return EXAMPLE_SERIALIZER.get();
}
```

!!! 提示
    有一些有用的方法可以使合成配方的数据读写更轻松。`Ingredient` 可以使用 `#fromJson`、`#toNetwork` 和 `#fromNetwork`，而 `ItemStack` 可以使用 `CraftingHelper#getItemStack`、`FriendlyByteBuf#writeItem` 和 `FriendlyByteBuf#readItem`。

### 构建 JSON

自定义合成配方的 JSON 文件与其他[合成配方][json]存储在同一位置。指定的 `type` 应该表示 **合成配方序列化器** 的注册名称。任何额外的数据在解码时由序列化器指定。

```js
{
  // 自定义序列化器的注册名称
  "type": "examplemod:example_serializer",
  "input": {
    // 一些材料输入
  },
  "data": 0, // 合成配方所需的一些数据
  "output": {
    // 一些物品堆输出
  }
}
```

### 非物品逻辑

如果物品不用于合成配方的输入或结果的一部分，那么 [`RecipeManager`][manager] 中提供的常规方法将无用。相反，应该为自定义 `Recipe` 实例添加一个额外的方法来测试合成配方的有效性和/或提供结果。从那里，可以通过 `RecipeManager#getAllRecipesFor` 获取该特定 `RecipeType` 的所有合成配方，然后使用新实现的方法检查和/或提供结果。

```java
// 在某个 Recipe 子实现 ExampleRecipe 中

// 检查指定位置的方块是否与存储的数据匹配
boolean matches(Level level, BlockPos pos);

// 创建方块状态以将指定位置的方块设置为该状态
BlockState assemble(RegistryAccess access);

// 在某个管理类中
public Optional<ExampleRecipe> getRecipeFor(Level level, BlockPos pos) {
  return level.getRecipeManager()
    .getAllRecipesFor(exampleRecipeType) // 获取所有合成配方
    .stream() // 遍历所有类型的合成配方
    .filter(recipe -> recipe.matches(level, pos)) // 检查合成配方输入是否有效
    .findFirst(); // 找到第一个输入匹配的合成配方
}
```

### 数据生成

所有自定义合成配方，无论输入或输出数据如何，都可以使用 `RecipeProvider` 创建为 `FinishedRecipe` 进行[数据生成][datagen]。

[forge]: ../../../concepts/registries.md#methods-for-registering
[json]: https://minecraft.wiki/w/Recipe#JSON_format
[manager]: ./index.md#recipe-manager
[datagen]: ../../../datagen/server/recipes.md#custom-recipe-serializers
