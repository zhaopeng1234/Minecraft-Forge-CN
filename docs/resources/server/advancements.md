## 进度系统
进度是玩家可以达成的任务，达成后可能推动游戏进程。进度可以基于玩家直接参与的任何行动触发。

原版中的所有进度实现都是通过 JSON 进行数据驱动的。这意味着创建新的进度不一定需要模组，只需要一个[数据包][datapack]即可。关于如何在模组的 `resources` 目录中创建和放置这些进度的完整列表，可以在 [Minecraft 维基百科][wiki] 上找到。此外，进度可以根据可用信息（模组是否加载、物品是否存在等）[有条件地加载并设置默认值][conditional]。

### 进度条件
要解锁一个进度，必须满足指定的条件。条件通过触发器进行跟踪，当执行特定动作（如击杀实体、更改物品栏、繁殖动物等）时，触发器会执行。每当一个进度加载到游戏中时，定义的条件会被读取并作为监听器添加到触发器中。之后会调用一个触发器函数（通常命名为 `#trigger`），该函数会检查所有监听器，以确定当前状态是否满足进度条件。只有在玩家通过完成所有要求获得该进度后，进度的条件监听器才会被移除。

要求被定义为一个字符串数组的数组，代表进度中指定的条件名称。当满足其中一个字符串数组中的所有条件时，进度即完成：
```js
// 在某个进度的 JSON 文件中

// 要满足的条件列表
"criteria": {
  "example_criterion1": { /*...*/ },
  "example_criterion2": { /*...*/ },
  "example_criterion3": { /*...*/ },
  "example_criterion4": { /*...*/ }
},

// 只有在以下情况下，这个进度才会解锁：
// - 条件 1 和 2 都满足
// 或者
// - 条件 3 和 4 都满足
"requirements": [
  [
    "example_criterion1",
    "example_criterion2"
  ],
  [
    "example_criterion3",
    "example_criterion4"
  ]
]
```
原版定义的条件触发器列表可以在 `CriteriaTriggers` 中找到。此外，JSON 格式在 [Minecraft 维基百科][triggers] 上有定义。

#### 自定义条件触发器
可以通过为创建的 `AbstractCriterionTriggerInstance` 子类实现 `SimpleCriterionTrigger` 来创建自定义条件触发器。

##### AbstractCriterionTriggerInstance 子类
`AbstractCriterionTriggerInstance` 表示 `criteria` 对象中定义的单个条件。触发器实例负责保存定义的条件，返回输入是否匹配条件，并将实例写入 JSON 以进行数据生成。

条件通常通过构造函数传递。`AbstractCriterionTriggerInstance` 父类构造函数要求实例将触发器的注册名称和玩家必须满足的条件定义为 `ContextAwarePredicate`。触发器的注册名称应直接提供给父类，而玩家的条件应作为构造函数参数。
```java
// 其中 ID 是触发器的注册名称
public ExampleTriggerInstance(ContextAwarePredicate player, ItemPredicate item) {
  super(ID, player);
  // 保存必须满足的物品条件
}
```
!!! 注意
    通常，触发器实例有一个静态构造函数，允许为数据生成轻松创建这些实例。这些静态工厂方法也可以静态导入，而不是导入类本身。
    ```java
    public static ExampleTriggerInstance instance(ContextAwarePredicate player, ItemPredicate item) {
      return new ExampleTriggerInstance(player, item);
    }
    ```
此外，应该重写 `#serializeToJson` 方法。该方法应将实例的条件添加到其他 JSON 数据中。
```java
@Override
public JsonObject serializeToJson(SerializationContext context) {
  JsonObject obj = super.serializeToJson(context);
  // 将条件写入 JSON
  return obj;
}
```
最后，应该添加一个方法，该方法接受当前数据状态并返回用户是否满足必要条件。玩家的条件已经通过 `SimpleCriterionTrigger#trigger(ServerPlayer, Predicate)` 进行了检查。大多数触发器实例将此方法命名为 `#matches`。
```java
// 此方法对于每个实例都是唯一的，因此不进行重写
public boolean matches(ItemStack stack) {
  // 由于 ItemPredicate 匹配物品堆，因此输入是一个物品堆
  return this.item.matches(stack);
}
```

##### SimpleCriterionTrigger
`SimpleCriterionTrigger<T>` 子类（其中 `T` 是触发器实例的类型）负责指定触发器的注册名称、创建触发器实例，以及一个检查触发器实例并在成功时运行附加监听器的方法。

触发器的注册名称通过 `#getId` 提供。这应该与提供给触发器实例的注册名称匹配。

触发器实例通过 `#createInstance` 创建。此方法从 JSON 中读取条件。
```java
@Override
public ExampleTriggerInstance createInstance(JsonObject json, ContextAwarePredicate player, DeserializationContext context) {
  // 从 JSON 中读取条件：物品
  return new ExampleTriggerInstance(player, item);
}
```
最后，定义一个方法来检查所有触发器实例，并在满足条件时运行监听器。此方法接受 `ServerPlayer` 以及 `AbstractCriterionTriggerInstance` 子类中匹配方法定义的任何其他数据。此方法应在内部调用 `SimpleCriterionTrigger#trigger` 以正确处理检查所有监听器。大多数触发器实例将此方法命名为 `#trigger`。
```java
// 此方法对于每个触发器都是唯一的，因此不进行重写
public void trigger(ServerPlayer player, ItemStack stack) {
  this.trigger(player,
    // AbstractCriterionTriggerInstance 子类中的条件检查方法
    triggerInstance -> triggerInstance.matches(stack)
  );
}
```
之后，应在 `FMLCommonSetupEvent` 期间使用 `CriteriaTriggers#register` 注册一个实例。

!!! 重要
    由于 `CriteriaTriggers#register` 方法不是线程安全的，必须通过 `FMLCommonSetupEvent#enqueueWork` 将其加入同步工作队列。

#### 调用触发器
每当执行正在检查的动作时，应该调用 `SimpleCriterionTrigger` 子类定义的 `#trigger` 方法。
```java
// 在执行动作的某段代码中
// 其中 EXAMPLE_CRITERIA_TRIGGER 是自定义条件触发器
public void performExampleAction(ServerPlayer player, ItemStack stack) {
  // 执行动作的代码
  EXAMPLE_CRITERIA_TRIGGER.trigger(player, stack);
}
```

### 进度奖励
当一个进度完成时，可以给予奖励。这些奖励可以是经验值、战利品表、合成配方书中的配方，或者作为创造模式玩家执行的一个[函数]。
```js
// 在某个进度的 JSON 文件中
"rewards": {
  "experience": 10,
  "loot": [
    "minecraft:example_loot_table",
    "minecraft:example_loot_table2"
    // ...
  ],
  "recipes": [
    "minecraft:example_recipe",
    "minecraft:example_recipe2"
    // ...
  ],
  "function": "minecraft:example_function"
}
```

[datapack]: https://minecraft.wiki/w/Data_pack
[wiki]: https://minecraft.wiki/w/Advancement/JSON_format
[conditional]: ./conditional.md#implementations
[function]: https://minecraft.wiki/w/Function_(Java_Edition)
[triggers]: https://minecraft.wiki/w/Advancement/JSON_format#List_of_triggers
