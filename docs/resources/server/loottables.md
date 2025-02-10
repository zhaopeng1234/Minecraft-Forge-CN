## 战利品表
战利品表是一种逻辑文件，用于规定在各种操作或场景发生时应该发生什么。虽然原版系统仅处理物品生成，但该系统可以扩展以执行任意数量的定义操作。

### 数据驱动的战利品表
原版中的大多数战利品表都是通过 JSON 进行数据驱动的。这意味着创建新的战利品表不一定需要模组，只需要一个[数据包][datapack]即可。关于如何在模组的 `resources` 文件夹中创建和放置这些战利品表的完整列表，可以在[Minecraft 维基百科][wiki]上找到。

### 使用战利品表
战利品表通过其 `ResourceLocation` 进行引用，该位置指向 `data/<命名空间>/loot_tables/<路径>.json`。可以使用 `LootDataResolver#getLootTable` 方法获取与该引用关联的 `LootTable`，其中 `LootDataResolver` 可以通过 `MinecraftServer#getLootData` 方法获得。

战利品表总是根据给定的参数生成。`LootParams` 包含战利品表生成所在的游戏世界、用于更好生成的幸运值、定义场景上下文的 `LootContextParam` 以及激活时应发生的任何动态信息。可以使用 `LootParams$Builder` 构建器的构造函数创建 `LootParams`，并通过传递 `LootContextParamSet` 调用 `LootParams$Builder#create` 方法来构建它。

战利品表可能还有一些上下文信息。`LootContext` 接受构建好的 `LootParams`，并可以设置一些随机种子实例。通过 `LootContext$Builder` 构建器创建上下文，并通过传递一个可空的 `ResourceLocation`（表示要使用的随机实例）调用 `LootContext$Builder#create` 方法来构建它。

`LootTable` 可以使用以下可用方法之一来生成 `ItemStack`，这些方法可能接受 `LootParams` 或 `LootContext`：

| 方法 | 描述 |
| :---: | :--- |
| `getRandomItemsRaw` | 消耗战利品表生成的物品。 |
| `getRandomItems` | 返回战利品表生成的物品。 |
| `fill` | 用生成的战利品表填充一个容器。 |

!!! 注意
    战利品表是为生成物品而构建的，因此这些方法期望对 `ItemStack` 进行一些处理。

### 附加功能
Forge 为战利品表提供了一些额外的功能，以便更好地控制系统。

#### `LootTableLoadEvent`
`LootTableLoadEvent` 是在 Forge 事件总线上触发的一个[事件]，每当加载战利品表时就会触发。如果该事件被取消，则将加载一个空的战利品表。

!!! 重要
    **不要**通过此事件修改战利品表的掉落物。这些修改应该使用[全局战利品修改器][glm]来完成。

#### 战利品池名称
可以使用 `name` 键为战利品池命名。任何未命名的战利品池将以 `custom#` 为前缀，后跟该池的哈希码。

```js
// 对于某个战利品池
{
  "name": "example_pool", // 战利品池将被命名为 'example_pool'
  "rolls": {
    // ...
  },
  "entries": {
    // ...
  }
}
```

#### 抢夺附魔修改器
除了抢夺附魔外，战利品表现在还会受到 Forge 事件总线上的 `LootingLevelEvent` 的影响。

#### 附加上下文参数
Forge 扩展了某些参数集，以考虑可能适用的缺失上下文。`LootContextParamSets#CHEST` 现在允许使用 `LootContextParams#KILLER_ENTITY`，因为矿车箱子是可以被破坏（或“击杀”）的实体。`LootContextParamSets#FISHING` 也允许使用 `LootContextParams#KILLER_ENTITY`，因为当玩家收回钓鱼钩时，它也是一个被收回（或“击杀”）的实体。

#### 熔炼时获得多个物品
使用 `SmeltItemFunction` 时，熔炼配方现在将返回结果中的实际物品数量，而不是单个熔炼物品（例如，如果一个熔炼配方返回 3 个物品，并且有 3 个掉落物，那么结果将是 9 个熔炼物品，而不是 3 个）。

#### 战利品表 ID 条件
Forge 添加了一个额外的 `LootItemCondition`，允许某些物品在特定的战利品表中生成。这通常在[全局战利品修改器][glm]中使用。

```js
// 在某个战利品池或池条目中
{
  "conditions": [
    {
      "condition": "forge:loot_table_id",
      // 当战利品表是泥土的战利品表时应用
      "loot_table_id": "minecraft:blocks/dirt"
    }
  ]
}
```

#### 工具能否执行操作条件
Forge 添加了一个额外的 `LootItemCondition`，用于检查给定的 `LootContextParams#TOOL` 是否可以执行指定的 `ToolAction`。

```js
// 在某个战利品池或池条目中
{
  "conditions": [
    {
      "condition": "forge:can_tool_perform_action",
      // 当工具可以像斧头一样剥去原木的树皮时应用
      "action": "axe_strip"
    }
  ]
}
```

[datapack]: https://minecraft.wiki/w/Data_pack
[wiki]: https://minecraft.wiki/w/Loot_table
[event]: ../../concepts/events.md#creating-an-event-handler
[glm]: ./glm.md
