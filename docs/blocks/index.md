
方块
======

显然，方块对Minecraft世界至关重要。它们构成了所有的地形、结构和机器。如果您对制作mod感兴趣，那么您可能会想要添加一些方块。本页将指导您创建方块，以及您可以使用它们做的一些事情。

创建方块
----------------

### 基本方块

对于不需要特殊功能的简单块（比如鹅卵石、木板等），不需要自定义类。您可以通过使用“BlockBehaviour\$Properties”对象调用“Block”类的初始化方法来创建块。Properties对象可以通过调用“BlockBehaviour\$Properties#of”方法制作，也可以调用它的方法来进行方块的自定义。例如：

- `strength` - 第一个参数为方块的硬度，控制打破块所需的时间。这可以是任意值。作为参考，石头的硬度为1.5，泥土为0.5。如果块应该是坚不可摧的，应该使用-1.0的硬度，请参阅“Blocks#BEDROCK”的定义作为示例。第二个参数为抗性，控制块的防爆性。作为参考，石头的抗性为6.0，泥土为0.5。
- `sound` - 控制块在被击打、破碎或放置时发出的声音。需要`SoundType`参数，请参阅[声音][sounds]页面了解更多详细信息。
- `lightLevel` - 控制块的亮度。参数为使用`BlockState`参数的函数，该函数返回从零到十五的值。
- `friction` - 控制块的滑度。作为参考，冰的滑度为0.98。

所有这些方法都是*可串联的*，这意味着您可以串联调用它们。有关此示例，请参阅`Blocks`类。

!!! 注意
    方块的`CreativeModeTab`没有设置器。如果方块有关联的物品（例如`BlockItem`），则由[`BuildCreativeModeTabContentsEvent`][creativetabs]处理。此外，方块没有翻译键的设置器，因为它是通过`Block#getDescriptionId`从注册表名称生成的。


### 高级方块

当然，上面只允许非常基本的块。如果你想添加功能，比如玩家交互，需要一个自定义类。`Block`类有很多方法，不幸的是，不是每一个都可以在这里记录下来。有关你可以用方块做的事情，请参阅本节的其余页面。


注册方块
-------------------
方块必须被[注册][registering]才能运行。

!!! 重要
  世界中的方块和库存中的“方块”是非常不同的东西。关卡中的方块由`BlockState`表示，其行为由`Block`的实例定义。同时，库存中的物品是`ItemStack`，由`Item`控制。作为`Block`和`Item`之间的桥梁，就有了`BlockItem`类。`BlockItem`是`Item`的子类，它有一个字段`block`保存对它所代表的`Block`的引用。`BlockItem`将“方块”作为物品的一些行为，例如右键单击如何放置块。可以有一个`Block`而没有`BlockItem`。（例如`minecraft:water`存在一个方块，但不是一个物品。因此不可能将其作为一个整体保存在库存中。）

当一个方块被注册时，*只有* 方块本身被注册。该方块不会自动具有`BlockItem`。要为方块创建一个基本`BlockItem`，应该将`BlockItem`的注册表名称设置为其`Block`的注册表名称。也可以使用`BlockItem`的自定义子类。一旦为方块注册了`BlockItem`，就可使用`Block#asItem`检索它。如果`Block`没有对应的`BlockItem`，此方法将返回`Items#AIR`，因此如果您不确定`Block`是否有`BlockItem`，请检查`Block#asItem`是否返回`Items#AIR`。

#### 可选注册方块

过去有几个mod允许用户在配置文件中禁用方块/物品。但是，你不应该这样做。可以注册的块数量没有限制，所以在你的mod中注册所有块！如果你想通过配置文件禁用方块，你应该禁用制作配方。如果你想在创意选项卡中禁用方块，在构建[`BuildCreativeModeTabContentsEvent`][creativetabs]中的内容时使用`FeatureFlag`。


进一步阅读
---------------

有关方块属性的信息，例如用于栅栏、墙壁等原版的方块属性，请参阅[方块状态][blockstates]部分。

[sounds]: ../gameeffects/sounds.md
[creativetabs]: ../items/index.md#创造模式的标签页
[registering]: ../concepts/registries.md#注册的方法
[blockstates]: states.md
