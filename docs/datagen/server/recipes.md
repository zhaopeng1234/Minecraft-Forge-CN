### 配方生成
通过继承 `RecipeProvider` 并实现 `#buildRecipes` 方法，可以为模组生成配方。当消费者接受 `FinishedRecipe` 视图时，配方就会被提供用于数据生成。`FinishedRecipe` 可以手动创建并提供，或者为了方便起见，使用 `RecipeBuilder` 创建。

实现之后，必须将该提供者 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        MyRecipeProvider::new
    );
}
```

### `RecipeBuilder`
`RecipeBuilder` 是一个方便的实现，用于创建要生成的 `FinishedRecipe`。它分别通过 `#unlockedBy`、`#group`、`#save` 和 `#getResult` 方法为解锁、分组、保存和获取配方结果提供基本定义。

!!! 重要
    [配方中的 `ItemStack` 输出][stack] 在原版配方构建器中不受支持。对于现有的原版配方序列化器，必须以不同的方式构建 `FinishedRecipe` 来生成此数据。

!!! 警告
    生成的物品结果必须指定有效的 `RecipeCategory`；否则，将抛出 `NullPointerException`。

除了 [`SpecialRecipeBuilder`] 之外，所有配方构建器都需要指定一个进度条件。如果玩家之前使用过该配方，所有配方都会生成一个解锁该配方的条件。然而，还必须指定一个额外的条件，允许玩家在没有任何先验知识的情况下获得该配方。如果指定的任何条件为真，则玩家将在配方书中获得该配方。

!!! 提示
    配方条件通常使用 `InventoryChangeTrigger` 在用户的物品栏中存在某些物品时解锁其配方。

#### ShapedRecipeBuilder
`ShapedRecipeBuilder` 用于生成有序配方。可以通过 `#shaped` 方法初始化构建器。在保存之前，可以指定配方组、输入符号模式、符号定义的原料以及配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
ShapedRecipeBuilder builder = ShapedRecipeBuilder.shaped(RecipeCategory.MISC, result)
  .pattern("a a") // 创建配方模式
  .define('a', item) // 定义符号代表的物品
  .unlockedBy("criteria", criteria) // 配方如何解锁
  .save(writer); // 将数据添加到构建器
```

##### 额外的验证检查
有序配方在构建之前会进行一些额外的验证检查：
- 必须定义一个模式，并且模式中至少包含一个物品。
- 所有模式行的宽度必须相同。
- 一个符号不能被定义多次。
- 空格字符 (`' '`) 用于表示槽位中没有物品，因此不能被定义。
- 模式必须使用用户定义的所有符号。

#### ShapelessRecipeBuilder
`ShapelessRecipeBuilder` 用于生成无序配方。可以通过 `#shapeless` 方法初始化构建器。在保存之前，可以指定配方组、输入原料以及配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
ShapelessRecipeBuilder builder = ShapelessRecipeBuilder.shapeless(RecipeCategory.MISC, result)
  .requires(item) // 向配方中添加物品
  .unlockedBy("criteria", criteria) // 配方如何解锁
  .save(writer); // 将数据添加到构建器
```

#### SimpleCookingRecipeBuilder
`SimpleCookingRecipeBuilder` 用于生成熔炼、高炉熔炼、烟熏和营火烹饪配方。此外，使用 `SimpleCookingSerializer` 的自定义烹饪配方也可以使用此构建器进行数据生成。可以分别通过 `#smelting`、`#blasting`、`#smoking`、`#campfireCooking` 或 `#cooking` 方法初始化构建器。在保存之前，可以指定配方组和配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
SimpleCookingRecipeBuilder builder = SimpleCookingRecipeBuilder.smelting(input, RecipeCategory.MISC, result, experience, cookingTime)
  .unlockedBy("criteria", criteria) // 配方如何解锁 
  .save(writer); // 将数据添加到构建器
```

#### SingleItemRecipeBuilder
`SingleItemRecipeBuilder` 用于生成切石机配方。此外，使用 `SingleItemRecipe$Serializer` 之类的序列化器的自定义单物品配方也可以使用此构建器进行数据生成。可以分别通过 `#stonecutting` 方法或构造函数初始化构建器。在保存之前，可以指定配方组和配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
SingleItemRecipeBuilder builder = SingleItemRecipeBuilder.stonecutting(input, RecipeCategory.MISC, result)
  .unlockedBy("criteria", criteria) // 配方如何解锁
  .save(writer); // 将数据添加到构建器
```

### 非 `RecipeBuilder` 的构建器
由于缺乏前面提到的所有配方使用的功能，一些配方构建器没有实现 `RecipeBuilder`。

#### SmithingTransformRecipeBuilder
`SmithingTransformRecipeBuilder` 用于生成转换物品的锻造配方。此外，使用 `SmithingTransformRecipe$Serializer` 之类的序列化器的自定义配方也可以使用此构建器进行数据生成。可以分别通过 `#smithing` 方法或构造函数初始化构建器。在保存之前，可以指定配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
SmithingTransformRecipeBuilder builder = SmithingTransformRecipeBuilder.smithing(template, base, addition, RecipeCategory.MISC, result)
  .unlocks("criteria", criteria) // 配方如何解锁
  .save(writer, name); // 将数据添加到构建器
```

#### SmithingTrimRecipeBuilder
`SmithingTrimRecipeBuilder` 用于生成盔甲装饰的锻造配方。此外，使用 `SmithingTrimRecipe$Serializer` 之类的序列化器的自定义升级配方也可以使用此构建器进行数据生成。可以分别通过 `#smithingTrim` 方法或构造函数初始化构建器。在保存之前，可以指定配方解锁条件。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
SmithingTrimRecipe builder = SmithingTrimRecipe.smithingTrim(template, base, addition, RecipeCategory.MISC)
  .unlocks("criteria", criteria) // 配方如何解锁
  .save(writer, name); // 将数据添加到构建器
```

#### SpecialRecipeBuilder
`SpecialRecipeBuilder` 用于为难以限制在配方 JSON 格式中的动态配方（如给盔甲染色、烟花等）生成空的 JSON 文件。可以通过 `#special` 方法初始化构建器。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
SpecialRecipeBuilder.special(dynamicRecipeSerializer)
  .save(writer, name); // 将数据添加到构建器
```

### 条件配方
[条件配方][conditional] 也可以通过 `ConditionalRecipe$Builder` 进行数据生成。可以使用 `#builder` 方法获取构建器。

每个配方的条件可以通过先调用 `#addCondition`，然后在指定所有条件后调用 `#addRecipe` 来指定。程序员可以根据需要多次重复此过程。

在指定所有配方后，可以在最后使用 `#generateAdvancement` 为每个配方添加进度。或者，可以使用 `#setAdvancement` 设置条件进度。

```java
// 在 RecipeProvider#buildRecipes(writer) 方法中
ConditionalRecipe.builder()
  // 为配方添加条件
  .addCondition(...)
  // 添加条件为真时返回的配方
  .addRecipe(...)

  // 为下一个配方添加条件
  .addCondition(...)
  // 添加下一组条件为真时返回的配方
  .addRecipe(...)

  // 创建使用上述配方中的条件和解锁进度的条件进度
  .generateAdvancement()
  .build(writer, name);
```

#### IConditionBuilder
为了简化向条件配方添加条件的过程，而不必手动构造每个条件实例，扩展的 `RecipeProvider` 可以实现 `IConditionBuilder`。该接口添加了一些方法，以便轻松构造条件实例。

```java
// 在 ConditionalRecipe$Builder#addCondition 方法中
(
  // 如果 'examplemod:example_item'
  // 或者 'examplemod:example_item2' 存在
  // 并且
  // 不为 FALSE

  // 方法由 IConditionBuilder 定义
  and( 
    or(
      itemExists("examplemod", "example_item"),
      itemExists("examplemod", "example_item2")
    ),
    not(
      FALSE()
    )
  )
)
```

### 自定义配方序列化器
可以通过创建一个能够构建 `FinishedRecipe` 的构建器来为自定义配方序列化器进行数据生成。完成的配方将配方数据及其解锁进度（如果存在）编码为 JSON。此外，还会指定配方的名称和序列化器，以便知道写入位置以及在加载时可以解码该对象的内容。一旦构建了 `FinishedRecipe`，只需将其传递给 `RecipeProvider#buildRecipes` 提供的 `Consumer` 即可。

!!! 提示
    `FinishedRecipe` 足够灵活，可以对任何对象转换进行数据生成，而不仅仅是物品。

[datagen]: ../index.md#data-providers
[ingredients]: ../../resources/server/recipes/ingredients.md#forge-types
[stack]: ../../resources/server/recipes/index.md#recipe-itemstack-result
[conditional]: ../../resources/server/conditional.md
[special]: #specialrecipebuilder
