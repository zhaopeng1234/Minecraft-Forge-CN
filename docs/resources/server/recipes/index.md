## 合成配方

合成配方是在Minecraft世界中将一定数量的物品转化为其他物品的方式。虽然原版系统仅处理物品转化，但整个系统可以扩展为使用程序员创建的任何对象。

### 数据驱动的合成配方

原版中的大多数合成配方实现都是通过JSON进行数据驱动的。这意味着创建新的合成配方不一定需要模组，只需要一个[数据包][datapack]即可。关于如何在模组的`resources`文件夹中创建和放置这些配方的完整列表，可以在[Minecraft维基百科][wiki]上找到。

合成配方可以作为完成[进度][advancement]的奖励在合成配方书中获得。合成配方进度的父进度始终是`minecraft:recipes/root`，这样它们就不会显示在进度界面上。获得合成配方进度的默认条件是检查用户是否通过使用一次合成配方或通过`/recipe`等命令获得了该配方：

```js
// 在某个合成配方进度的JSON文件中
"has_the_recipe": { // 条件标签
  // 如果examplemod:example_recipe被使用，则条件满足
  "trigger": "minecraft:recipe_unlocked",
  "conditions": {
    "recipe": "examplemod:example_recipe"
  }
}
//...
"requirements": [
  [
    "has_the_recipe"
    // ... 其他用于解锁配方的条件标签，使用逻辑或关系
  ]
]
```

数据驱动的合成配方及其解锁进度可以通过`RecipeProvider`[自动生成][datagen]。

### 合成配方管理器

合成配方通过`RecipeManager`加载和存储。任何与获取可用合成配方相关的操作都由该管理器处理。有两个重要的方法需要了解：

| 方法 | 描述 |
| :---: | :--- |
| `getRecipeFor` | 获取与当前输入匹配的第一个合成配方。 |
| `getRecipesFor` | 获取与当前输入匹配的所有合成配方。 |

每个方法都接受一个`RecipeType`，它表示使用合成配方的方式（合成、熔炼等），一个`Container`，它包含输入的配置，以及当前的游戏世界，该世界会与容器一起传递给`Recipe#matches`方法。

!!! 重要
    Forge提供了`RecipeWrapper`工具类，它扩展了`Container`，用于包装`IItemHandler`并将其传递给需要`Container`参数的方法。

    ```java
    // 在某个包含IItemHandlerModifiable handler的方法中
    recipeManger.getRecipeFor(RecipeType.CRAFTING, new RecipeWrapper(handler), level);
    ```

### 附加功能

Forge为合成配方架构及其实现提供了一些额外的功能，以便更好地控制系统。

#### 合成配方物品堆结果

除了`minecraft:stonecutting`合成配方外，在某些情况下，所有原版合成配方序列化器都会将`result`标签扩展为接受一个完整的`ItemStack`作为`JsonObject`，而不仅仅是物品名称和数量。

```js
// 在某个合成配方的JSON文件中
"result": {
  // 作为结果给予的注册物品的名称
  "item": "examplemod:example_item",
  // 返回的物品数量
  "count": 4,
  // 物品堆的标签数据，也可以是字符串
  "nbt": {
      // 在此处添加标签数据
  }
}
```

!!! 注意
    `nbt`标签也可以是一个包含字符串化NBT（或SNBT）的字符串，用于无法正确表示为JSON对象的数据（如`IntArrayTag`）。

#### 条件合成配方

合成配方及其解锁进度可以根据可用信息（模组是否加载、物品是否存在等）[有条件地加载并设置默认值][conditional]。

#### 更大的合成网格

默认情况下，原版规定合成网格的最大宽度和高度为3x3的正方形。可以在`FMLCommonSetupEvent`中调用`ShapedRecipe#setCraftingSize`方法并传入新的宽度和高度来扩展这个限制。

!!! 警告
    `ShapedRecipe#setCraftingSize`方法**不是线程安全的**。因此，应该通过`FMLCommonSetupEvent#enqueueWork`将其加入同步工作队列。

合成配方中更大的合成网格可以[通过数据生成][datagen]。

#### 材料类型

添加了一些额外的[材料类型][ingredients]，以使合成配方能够有检查标签数据的输入，或者将多个材料组合成一个输入检查器。

[datapack]: https://minecraft.wiki/w/Data_pack
[wiki]: https://minecraft.wiki/w/Recipe
[advancement]: ../advancements.md
[datagen]: ../../../datagen/server/recipes.md
[cap]: ../../../datastorage/capabilities.md
[conditional]: ../conditional.md#implementations
[ingredients]: ./ingredients.md#forge-types
