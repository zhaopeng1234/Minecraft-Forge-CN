## 全局战利品修改器
全局战利品修改器是一种数据驱动的方法，用于处理对采集掉落物的修改，无需覆盖数十甚至数百个原版战利品表，也无需在不知道可能加载哪些模组的情况下处理与其他模组战利品表的交互效果。全局战利品修改器也支持堆叠，而不是最后加载的生效，这与标签类似。

### 注册全局战利品修改器
你需要以下 4 样东西：
1. **创建 `global_loot_modifiers.json`**：这将告知 Forge 你的修改器信息，其工作方式类似于[标签]。
2. **一个表示你的修改器的序列化 JSON 文件**：它将包含有关你修改的所有数据，并允许数据包调整你的效果。
3. **一个继承 `IGlobalLootModifier` 的类**：这是使你的修改器工作的操作代码。大多数模组开发者可以继承 `LootModifier`，因为它提供了基本功能。
4. **一个用于编码和解码你的操作类的编解码器**：这像其他 `IForgeRegistryEntry` 一样[注册]。

### `global_loot_modifiers.json` 文件
`global_loot_modifiers.json` 表示要加载到游戏中的所有战利品修改器。此文件**必须**放在 `data/forge/loot_modifiers/global_loot_modifiers.json` 中。

!!! 重要
    `global_loot_modifiers.json` 只会在 `forge` 命名空间中被读取。如果该文件位于模组的命名空间下，将被忽略。

`entries` 是要加载的修改器的*有序列表*。指定的[资源位置][resloc]指向 `data/<命名空间>/loot_modifiers/<路径>.json` 中的相关条目。这主要与数据包制作者解决不同模组修改器之间的冲突有关。

`replace` 为 `true` 时，会将行为从向全局列表追加战利品修改器改为完全替换全局列表条目。模组开发者为了与其他模组实现兼容，应使用 `false`。数据包制作者可能希望使用 `true` 来指定他们的覆盖项。

```js
{
  "replace": false, // 必须存在
  "entries": [
    // 表示 'data/examplemod/loot_modifiers/example_glm.json' 中的一个战利品修改器
    "examplemod:example_glm",
    "examplemod:example_glm2"
    // ...
  ]
}
```

### 序列化的 JSON 文件
此文件包含与你的修改器相关的所有潜在变量，包括在修改任何战利品之前必须满足的条件。尽可能避免使用硬编码值，以便数据包制作者可以根据需要调整平衡性。

`type` 表示用于读取相关 JSON 文件的[编解码器]的注册名称。这一项必须始终存在。

`conditions` 应表示此修改器激活所需的战利品表条件。条件应避免硬编码，以便数据包创建者有足够的灵活性来调整标准。这一项也必须始终存在。

!!! 重要
    虽然 `conditions` 应表示修改器激活所需的条件，但这仅在使用 Forge 捆绑类时适用。如果继承 `LootModifier`，所有条件将被**逻辑与**在一起，并检查是否应应用该修改器。

序列化器读取并由修改器定义的任何其他属性也可以指定。

```js
// 在 data/examplemod/loot_modifiers/example_glm.json 中
{
  "type": "examplemod:example_loot_modifier",
  "conditions": [
    // 正常的战利品表条件
    // ...
  ],
  "prop1": "val1",
  "prop2": 10,
  "prop3": "minecraft:dirt"
}
```

### `IGlobalLootModifier` 接口
为了提供全局战利品修改器指定的功能，必须实现 `IGlobalLootModifier` 接口。每次序列化器从 JSON 解码信息并将其提供给此对象时，都会生成这些实例。

要创建新的修改器，需要定义两个方法：`#apply` 和 `#codec`。`#apply` 接受当前要生成的战利品以及上下文信息，如当前游戏世界或其他定义的参数，并返回要生成的掉落物列表。

!!! 注意
    任何一个修改器返回的掉落物列表会按照注册顺序传递给其他修改器。因此，修改后的战利品可以被另一个战利品修改器进一步修改。

`#codec` 返回用于将修改器编码和解码为/从 JSON 的注册[编解码器]。

#### `LootModifier` 子类
`LootModifier` 是 `IGlobalLootModifier` 的抽象实现，为大多数模组开发者提供了易于扩展和实现的基本功能。它通过定义 `#apply` 方法来检查条件，以确定是否修改生成的战利品，扩展了现有接口。

子类实现中有两点需要注意：构造函数必须接受一个 `LootItemCondition` 数组，以及 `#doApply` 方法。

`LootItemCondition` 数组定义了在修改战利品之前必须满足的条件列表。提供的条件将被**逻辑与**在一起，这意味着所有条件都必须为真。

`#doApply` 方法与 `#apply` 方法的工作方式相同，只是它仅在所有条件都返回 `true` 时才执行。

```java
public class ExampleModifier extends LootModifier {

  public ExampleModifier(LootItemCondition[] conditionsIn, String prop1, int prop2, Item prop3) {
    super(conditionsIn);
    // 存储其余参数
  }

  @NotNull
  @Override
  protected ObjectArrayList<ItemStack> doApply(ObjectArrayList<ItemStack> generatedLoot, LootContext context) {
    // 修改战利品并返回新的掉落物
  }

  @Override
  public Codec<? extends IGlobalLootModifier> codec() {
    // 返回用于编码和解码此修改器的编解码器
  }
}
```

### 战利品修改器编解码器
JSON 和 `IGlobalLootModifier` 实例之间的连接器是一个 [`Codec<T>`][codecdef]，其中 `T` 表示要使用的 `IGlobalLootModifier` 的类型。

为了方便起见，提供了一个战利品条件编解码器，可以通过 `LootModifier#codecStart` 轻松添加到类似记录的编解码器中。这用于相关战利品修改器的[数据生成][datagen]。

```java
// 对于某个 DeferredRegister<Codec<? extends IGlobalLootModifier>> REGISTRAR
public static final RegistryObject<Codec<ExampleModifier>> = REGISTRAR.register("example_codec", () ->
  RecordCodecBuilder.create(
    inst -> LootModifier.codecStart(inst).and(
      inst.group(
        Codec.STRING.fieldOf("prop1").forGetter(m -> m.prop1),
        Codec.INT.fieldOf("prop2").forGetter(m -> m.prop2),
        ForgeRegistries.ITEMS.getCodec().fieldOf("prop3").forGetter(m -> m.prop3)
      )
    ).apply(inst, ExampleModifier::new)
  )
);
```

[示例][examples]可以在 Forge 的 Git 仓库中找到，包括精准采集和熔炼效果的示例。

[tags]: ./tags.md
[resloc]: ../../concepts/resources.md#ResourceLocation
[codec]: #the-loot-modifier-codec
[registered]: ../../concepts/registries.md#methods-for-registering
[codecdef]: ../../datastorage/codecs.md
[datagen]: ../../datagen/server/glm.md
[examples]: https://github.com/MinecraftForge/MinecraftForge/blob/1.20.x/src/test/java/net/minecraftforge/debug/gameplay/loot/GlobalLootModifiersTest.java
