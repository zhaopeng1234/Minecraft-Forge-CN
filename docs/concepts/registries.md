注册
==========

注册是将 MOD 中的对象（如物品、方块、声音等）告知游戏的过程。 注册非常重要，因为如果不注册，游戏将根本无法获取这些对象，这将导致无法解释的行为和游戏崩溃。

游戏中需要注册的大部分事项都由Forge注册中心处理。注册表是一个类似于Map的对象，它为键值赋值。 Forge 使用带有 [资源位置][ResourceLocation] 作为键的注册表来注册对象。 这允许`资源位置`充当对象的 "注册名称"。

每个种类的可注册对象都有各自的注册表。 要查看 Forge 封装的所有注册表，请参见`ForgeRegistries`类。 每一个注册表内的注册名称必须是唯一的。 不过，不同注册表中的名称不会冲突。 例如，有一个`方块`注册表和一个`物品`注册表。 可以用相同的名称各注册一个`方块` 和 `物品` ，不会发生碰撞；但是，如果两个不同的 `方块` 或 `物品` 以相同的名称注册，则第二个对象将覆盖第一个对象。

注册的方法
------------------

注册对象有两种方法：`DeferredRegister`类和生命周期中的`RegisterEvent`。

### 延迟注册`DeferredRegister`

`延迟注册` 是注册对象的推荐方法。 它带来了使用静态初始化器的便利并避免了与静态初始化器相关的问题。 其维护了一个生产者列表，并在`RegisterEvent`中从这些生产者里获取对象。

一个注册自定义方块的示例：

```java
private static final DeferredRegister<Block> BLOCKS = DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);

public static final RegistryObject<Block> ROCK_BLOCK = BLOCKS.register("rock", () -> new Block(BlockBehaviour.Properties.of().mapColor(MapColor.STONE)));

public ExampleMod() {
  BLOCKS.register(FMLJavaModLoadingContext.get().getModEventBus());
}
```

### 注册事件`RegisterEvent`

`注册事件` 是注册对象的第二种方法。该 [事件] 在Mod构造函数之后、配置加载之前针对每个注册表触发。对象通过使用 `#register方法` ，传递注册表键值、注册表对象名称和对象本身来注册的。还有一个额外的`#register方法`的重载，该重载接收一个辅助consumer函数，以注册一个具有给定名称的对象。 建议使用此方法以避免创建不必要的对象。

下面是一个示例：（该事件处理器已在*mod 事件总线*上注册）

```java
@SubscribeEvent
public void register(RegisterEvent event) {
  event.register(ForgeRegistries.Keys.BLOCKS,
    helper -> {
      helper.register(new ResourceLocation(MODID, "example_block_1"), new Block(...));
      helper.register(new ResourceLocation(MODID, "example_block_2"), new Block(...));
      helper.register(new ResourceLocation(MODID, "example_block_3"), new Block(...));
      // ...
    }
  );
}
```

### 非Forge注册表的注册表

并非所有注册表都由 Forge 封装。 这些注册表可以是静态注册表，例如`LootItemConditionType`，可以安全使用。还有一些动态注册表，如`ConfiguredFeature`和其他一些世界生成注册表，它们通常以 JSON 表示。 

!!! 重要
    动态注册表对象**只能**通过数据文件（例如 JSON）注册。 它们**不能**在代码中注册。

`DeferredRegister`的create方法有一个重载，允许开发者指定原版注册表要创建`RegistryObject`的注册表键值。 注册方法和附加到Mod事件总线的方法与其他 `DeferredRegister`的方法相同：
```java
private static final DeferredRegister<LootItemConditionType> REGISTER = DeferredRegister.create(Registries.LOOT_CONDITION_TYPE, "examplemod");

public static final RegistryObject<LootItemConditionType> EXAMPLE_LOOT_ITEM_CONDITION_TYPE = REGISTER.register("example_loot_item_condition_type", () -> new LootItemConditionType(...));
```

!!! 注意
   某些类本身无法注册。 相反，`*Type`类会被注册，并在构造函数中使用。 例如，[`BlockEntity`][blockentity] 有 `BlockEntityType`、 以及 `Entity` 有 `EntityType`. 这些`*Type`类是工厂，只按需创建包含的类型。 这些工厂是通过使用 `*Type`的`Builder`内部类创建的。 例如下面的代码，其中，`REGISTER`指的是一个`DeferredRegister<BlockEntityType>`。
   
```java
public static final RegistryObject<BlockEntityType<ExampleBlockEntity>> EXAMPLE_BLOCK_ENTITY = REGISTER.register(
   "example_block_entity", () -> BlockEntityType.Builder.of(ExampleBlockEntity::new, EXAMPLE_BLOCK.get()).build(null)
);
```

引用注册对象
------------------------------

注册的对象在创建和注册时不应存储在字段中。 每当`RegisterEvent`注册表被触发时，它们总是被新创建和注册。 这是为了在 Forge 的未来版本中允许动态加载和卸载 MOD。

注册对象必须始终通过`RegistryObject`或带有`@ObjectHolder`的字段来引用。

### 使用注册表对象`RegistryObject`

`注册表对象`可用来检索已注册对象的引用。 `DeferredRegister` 将使用这些引用来返回已注册对象的引用。 在其对应的注册表触发 `注册事件` 之后，注册表对象的引用将和 `@ObjectHolder` 注释的对象都会被更新。

要获取 `RegistryObject`、 使用 `资源地址` 和注册表对象调用`RegistryObject`类的`create`方法。 也可以通过提供注册表名称来使用自定义注册表。 将 `RegistryObject` 存储在 `public static final` 字段中，并在需要注册对象时调用 `get`方法 。

使用 `RegistryObject`的示例：

```java
public static final RegistryObject<Item> BOW = RegistryObject.create(new ResourceLocation("minecraft:bow"), ForgeRegistries.ITEMS);

// 假设'neomagicae:mana_type'是一个有效的注册表，而'neomagicae:coffeinum'是该注册表中的一个有效对象
public static final RegistryObject<ManaType> COFFEINUM = RegistryObject.create(new ResourceLocation("neomagicae", "coffeinum"), new ResourceLocation("neomagicae", "mana_type"), "neomagicae"); 
```

### 使用 @ObjectHolder 注解

通过使用`@ObjectHolder`注释类或字段，可将注册表中的注册对象注入到`public static`字段中。 并提供足够的信息来构建 `资源地址` 以识别特定注册表中的特定对象

`@ObjectHolder`的规则如下：
* 如果类使用`@ObjectHolder`进行注解，那么如果没有明确定义，其value字段值将成为类内所有字段的默认命名空间。
* 如果使用`@Mod`对类进行注解，那么如果没有明确定义，modid 将成为类中所有注解字段的默认命名空间。
* 在下列情况下，能够注入一个字段：
  * 至少有以下修饰词`public static`;
  * **字段**使用`@ObjectHolder`注解，并且明确定义了name的值，以及注册表名称已明确定义
* _如果字段没有相应的注册表名称或name的值，则会出现编译时异常_
* _如果结果`资源位置`不完整或无效（路径中的字符无效），则会抛出异常。_
* 如果没有出现其他错误或异常，将注入该字段
* 如果上述所有规则都不适用，则不会采取任何行动（可能会记录一条信息）。

`@ObjectHolder`注释的字段会在 注册表的`RegisterEvent` 事件触发后注入其值，同时注入的还有 `注册表对象`。

!!! 注意
    如果要注入的对象不存在于注册表中，则会记录一条调试信息，并且不会注入任何值。

由于这些规则比较复杂，这里举几个例子：

```java
class Holder {
  

  // 存在注释。 [public static] 为必必须的。 [final]为可选项。
  // 注册表名称已明确定义： "minecraft:enchantment
  // 资源地址已明确定义： "minecraft:flame
  // 会注入：从 [Enchantment] 注册表中注入 "minecraft:flame"
  @ObjectHolder(registryName = "minecraft:enchantment", value = "minecraft:flame")
  public static final Enchantment flame = null;     

  // 字段上没有注释。
  // 因此，该字段被忽略。
  public static final Biome ice_flat = null;        

  // 存在注释。 [public static] 为必必须的。
  // 注册表未在字段中指定。
  // 因此，这将产生一个编译时异常。
  @ObjectHolder("minecraft:creeper")
  public static Entity creeper = null;              

  // 存在注释。 [public static] 为必必须的。 [final]为可选项。
  // 注册表名称已明确定义： "minecraft:potion"
  // 资源地址未在字段中指定
  // 因此，这将产生一个编译时异常。
  @ObjectHolder(registryName = "potion")
  public static final Potion levitation = null;     
}
```

创建自定义Forge注册表
--------------------------------

自定义注册表通常只是键值的简单映射。 这是一种常见的样式；不过，它必须要拥有一个注册表的强依赖。 它还要求任何需要在双方之间同步的数据都必须手动完成。 自定义 Forge 注册表为创建软依赖关系提供了一个简单的替代方案，同时还能实现更好的管理和双方之间的自动同步（除非另有说明）。 由于对象也使用Forge注册表，因此注册也以同样的方式实现了标准化。

自定义Forge注册表是使用 `RegistryBuilder` 作为参数、 通过`新注册事件`或`延迟注册`创建的。 `RegistryBuilder`类接受各种参数（例如注册表的名称、id 范围以及针对注册表上发生的不同事件的各种回调）。在`新注册事件`触发后，新注册表将被注册到`RegistryManager`。

任何新创建的注册表都应使用其相关的[注册方法][registration]来注册相关对象。

### 使用新注册事件NewRegistryEvent

使用 `NewRegistryEvent` 时，使用`RegistryBuilder`调用 `create` 方法，将返回一个supplier封装的注册表。 在`NewRegistryEvent`完成向Mod事件总线发布后，可以访问supplier提供的注册表。 在`NewRegistryEvent`触发之前从supplier获取自定义注册表将导致`null`值。

#### 新数据包注册表
可使用 mod 事件总线上的 `DataPackRegistryEvent`的`NewRegistry` 事件添加新数据包注册表。新数据包注册表通过向该事件的`dataPackRegistry`方法传入 代表注册表名称的`ResourceKey`创建的，以及用于编码和解码 JSON 数据的 `Codec` 。 可提供可选的 `Codec` 以将数据包注册表同步到客户端。

!!! 重要
    数据包注册表注册不能通过 `延迟注册` 创建。 它们只能通过事件创建。

### 使用延迟注册

`DeferredRegister`类再次对上述事件进行了封装。在使用注册表名称和ModId调用`DeferredRegister`的`create`方法重载，创建了常量字段后， 就可以通过`DeferredRegister`的`makeRegistry`方法构建注册表。 该方法接收一个`RegistryBuilder`的supplier，其中包含了其他配置项。 该方法默认情况下已填充`setName`方法。 由于此方法可随时返回，因此将返回supplier封装的 `IForgeRegistry`。 在`NewRegistryEvent`触发之前从supplier获取自定义注册表将导致`null`值。

!!! 重要
    `DeferredRegister`的`makeRegistry`方法 必须在 `DeferredRegister` 通过 `register`方法 添加到Mod事件总线之前调用。 `makeRegistry`方法 也使用 `register` 方法在 `NewRegistryEvent` 期间创建注册表。

处理缺失条目
------------------------
在某些情况下，当Mod更新或对象被删除，某些注册表对象就会不复存在。可以通过第三个注册表事件指定处理缺失映射的操作： `MissingMappingsEvent`。 在此事件中，可以通过使用注册表键值和 mod id调用 `getMappings`方法 或 仅使用注册表键值调用 `#getAllMappings` 来获取缺失映射的列表。

!!! important
    `MissingMappingsEvent` 在 **Forge** 事件总线上触发。

对于每个`映射`，可选择以下四种映射类型之一来处理缺失的条目：

| Action | Description |
| :---:  |     :---    |
| IGNORE | 忽略丢失的条目并放弃映射。 |
|  WARN  | 在日志中生成警告。 |
|  FAIL  | 阻止世界加载。 |
| REMAP  | 将条目重映射到已注册的非空对象。 |

如果未指定任何操作，那么默认操作将通过通知用户缺失的条目以及是否还想加载世界来实现。 除了重映射之外的所有操作都将防止任何其他注册表对象取代现有 id，以防相关条目被添加回游戏中。

[ResourceLocation]: ./resources.md#资源位置resourcelocation
[registration]: #注册的方法
[event]: ./events.md
[blockentity]: ../blockentities/index.md
