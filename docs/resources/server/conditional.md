## 条件加载数据
有时，模组开发者可能希望在不将另一个模组明确设为依赖项的情况下，利用该模组的信息来包含数据驱动的对象。还有些情况是，当存在其他模组的条目时，用这些条目替换某些对象。这可以通过条件子系统来实现。

### 实现方式
目前，条件加载功能已应用于合成配方和进度系统。对于任何条件式的合成配方或进度，会加载一个条件与数据的配对列表。如果列表中某个数据对应的条件为真，则返回该数据；否则，丢弃该数据。

```js
{
  // 对于合成配方，需要指定类型，因为它们可能有自定义序列化器
  // 进度系统不需要此类型
  "type": "forge:conditional",
  
  "recipes": [ // 如果是进度系统，则为 'advancements'
    {
      // 要检查的条件
      "conditions": [
        // 列表中的条件是逻辑与关系
        {
          // 条件 1
        },
        {
          // 条件 2
        }
      ],
      "recipe": { // 如果是进度系统，则为 'advancement'
        // 如果所有条件都满足，则使用此合成配方
      }
    },
    {
      // 如果前一个条件失败，则检查下一个条件
    }
  ]
}
```
条件加载的数据还可以通过 `ConditionalRecipe$Builder` 和 `ConditionalAdvancement$Builder` 进行[数据生成][datagen]。

### 条件类型
通过将 `type` 设置为由 [`IConditionSerializer#getID`][serializer] 指定的条件名称来指定条件。

#### 真和假
布尔条件不包含数据，直接返回条件的预期值。它们分别用 `forge:true` 和 `forge:false` 表示。
```js
// 对于某个条件
{
  // 始终返回 true（如果是 'forge:false' 则返回 false）
  "type": "forge:true"
}
```

#### 非、与和或
布尔运算符条件包含要操作的条件，并应用相应的逻辑。它们分别用 `forge:not`、`forge:and` 和 `forge:or` 表示。
```js
// 对于某个条件
{
  // 反转存储条件的结果
  "type": "forge:not",
  "value": {
    // 一个条件
  }
}
```
```js
// 对于某个条件
{
  // 将存储的条件进行逻辑与运算（如果是 'forge:or' 则进行逻辑或运算）
  "type": "forge:and",
  "values": [
    {
      // 第一个条件
    },
    {
      // 第二个要进行逻辑与运算的条件（如果是 'forge:or' 则进行逻辑或运算）
    }
  ]
}
```

#### 模组已加载
`ModLoadedCondition` 在当前应用程序中加载了指定 ID 的模组时返回 true。用 `forge:mod_loaded` 表示。
```js
// 对于某个条件
{
  "type": "forge:mod_loaded",
  // 如果 'examplemod' 已加载，则返回 true
  "modid": "examplemod"
}
```

#### 物品已注册
`ItemExistsCondition` 在当前应用程序中注册了指定物品时返回 true。用 `forge:item_exists` 表示。
```js
// 对于某个条件
{
  "type": "forge:item_exists",
  // 如果 'examplemod:example_item' 已注册，则返回 true
  "item": "examplemod:example_item"
}
```

#### 标签为空
`TagEmptyCondition` 在指定的物品标签中没有物品时返回 true。用 `forge:tag_empty` 表示。
```js
// 对于某个条件
{
  "type": "forge:tag_empty",
  // 如果 'examplemod:example_tag' 是一个没有条目的物品标签，则返回 true
  "tag": "examplemod:example_tag"
}
```

### 创建自定义条件
可以通过实现 `ICondition` 及其关联的 `IConditionSerializer` 来创建自定义条件。

#### ICondition
任何条件只需要实现两个方法：

| 方法 | 描述 |
| :---: | :--- |
| `getID` | 条件的注册名称。必须与 [`IConditionSerializer#getID`][serializer] 相同。仅用于[数据生成][datagen]。 |
| `test` | 如果条件满足，则返回 true。 |

!!! 注意
    每个 `#test` 方法都可以访问一个表示游戏状态的 `IContext` 对象。目前，只能从注册表中获取标签。

#### IConditionSerializer
序列化器需要实现三个方法：

| 方法 | 描述 |
| :---: | :--- |
| `getID` | 条件的注册名称。必须与 [`ICondition#getID`][condition] 相同。 |
| `read` | 从 JSON 中读取条件数据。 |
| `write` | 将给定的条件数据写入 JSON。 |

!!! 注意
    与 Minecraft 中的其他序列化器实现类似，条件序列化器不负责写入或读取序列化器的类型。

之后，应声明一个静态实例来保存初始化的序列化器，然后在 `RecipeSerializer` 的 `RegisterEvent` 期间或 `FMLCommonSetupEvent` 期间使用 `CraftingHelper#register` 进行注册。
```java
// 在某个序列化器类中
public static final ExampleConditionSerializer INSTANCE = new ExampleConditionSerializer();

// 在某个处理类中
public void registerSerializers(RegisterEvent event) {
  event.register(ForgeRegistries.Keys.RECIPE_SERIALIZERS,
    helper -> CraftingHelper.register(INSTANCE)
  );
}
```

!!! 重要
    如果使用 `FMLCommonSetupEvent` 来注册条件序列化器，由于 `CraftingHelper#register` 不是线程安全的，必须通过 `FMLCommonSetupEvent#enqueueWork` 将其加入同步工作队列。

[datagen]: ../../datagen/server/recipes.md
[serializer]: #iconditionserializer
[condition]: #icondition
