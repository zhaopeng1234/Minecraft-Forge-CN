
方块状态
============

过去的行为
---------------------------------------
在Minecraft 1.7和以前的版本中，没有`BlockEntity`但需要存储放置或状态数据的方块使用**元数据**。元数据是与方块一起存储的额外数字，允许方块的旋转、面向，甚至完全独立的行为。

然而，元数据系统是混乱和受限的，因为它只存储为方块ID旁边的数字，除了代码中注释的内容之外没有任何意义。例如，实现一个方块，它可以面向一个方向，位于方块空间的上半部分或下半部分（如楼梯）：

```Java
switch (meta) {
  case 0: { ... } // 面向南边,位于方块的下半部分
  case 1: { ... } // 面向南边,位于方块的上半部分
  case 2: { ... } // 面向北边,位于方块的下半部分
  case 3: { ... } // 面向北边,位于方块的上半部分
  // ... 其他 ...
}
```

因为这些数字本身没有任何意义，除非可以访问源代码和评论，否则没有人能知道它们代表什么。

状态（States）简介
---------------------------------------
在Minecraft 1.8及更高版本中，元数据系统和方块ID系统被弃用，最终被**方块状态系统**取代。方块状态系统从方块的其他行为中抽象出方块属性的细节。

方块的每个*属性*由`Property<?>`的一个实例来描述，方块属性的示例包括工具（`EnumProperty<NoteBlockInstrument>`）、方向（`DirectionProperty`）、充能状态（`Property<Boolean>`）等，属性的每个值都有`Property<T>`中类型`T`的对应值。

使用 “Block ”和 “Property<T>”及对应值的映射，可以构造出一个唯一的组合。这个组合称为`BlockState`。

以前无意义的元数据值体系被更易于解读和处理的方块属性体系所取代，以前一个朝东被充能或按住的石头按钮用“`minecraft:stone_button`加上元数据`9`”来表示，现在，这用“`minecraft:stone_button[facing=east, powered=true]`”来表示。

方块状态(BlockState)的正确使用
---------------------------------------

`BlockState`系统是一个灵活而强大的系统，但它也有局限性。`BlockState`是不可变的，它们属性的所有组合都是在游戏启动时生成的。这意味着一个有许多属性和可能的值的`BlockState`会减慢游戏的加载速度，并让任何试图理解你的方块逻辑的人感到困惑。

并非所有块和情况都需要使用`BlockState`；只有方块的最基本属性应该放入`BlockState`，任何其他情况都最好有一个`BlockEntity`或者是一个单独的`Block`。始终考虑您是否真的需要为您的目的使用`BlockState`。

!!! 注意
    一个好的经验法则是：**如果它有不同的名称，它应该是一个单独的方块**。

一个例子是制作椅子方块：椅子的*方向*应该是*属性*，而不同*类型*的木材应该分成不同的方块。
朝东的“橡木椅”（`oak_chair[facing=east]`）和朝西的“云杉椅”（`spruce_chair[facing=west]`）是不同的方块。

实现`BlockState`
---------------------------------------

在你的方块类中，创建或引用`static final` `Property<?>`对象。您可以自由地创建自己的`Property<?>`实现，但本文没有介绍实现的方法。原版的代码提供了几个方便的实现：

* `IntegerProperty`
    * 实现了`Property<Integer>`。定义了一个包含整数值的属性。
    * 通过调用`IntegerProperty#create(String propertyName, int minimum, int maximum)`创建.
* `BooleanProperty`
    * 实现了`Property<Boolean>`。定义了一个包含布尔值的属性。
    * 通过调用`BooleanProperty#create(String propertyName)`创建.
* `EnumProperty<E extends Enum<E>>`
    * 实现了`Property<E>`。定义了一个可以使用Enum值的属性。
    * 通过调用 `EnumProperty#create(String propertyName, Class<E> enumClass)`创建.
    * 也可以只使用Enum值的一个子集（例如16种`DyeColor`中的4个）参考`EnumProperty#create`的重载.
* `DirectionProperty`
    * 这是一个 `EnumProperty<Direction>`的简单实现
    * 该类还提供了一些方便的谓词。例如，要获取表示基本方向的属性，请调用`DirectionProperty.create("<name>", Direction.Plane.HORIZONTAL)`；要获取X方向，调用`DirectionProperty.create("<name>", Direction.Axis.X)`。

类`BlockStateProperties`包含共享原版的属性，应尽可能使用或引用这些属性，而不是创建自己的属性。

当你有你想要指定的`Property<>`对象时，在你的`Block`类中重载`Block#createBlockStateDefinition(StateDefinition$Builder)`。在该方法中，调用`StateDefinition$Builder#add(...)`，将每个你希望方块拥有的`Property<?>`作为参数传入。

每个方块也会有一个默认状态，它会自动为您选择。您可以通过从构造函数调用`Block#registerDefaultState(BlockState)`方法来更改这个默认状态。当您的方块被放置时，它会变成这个默认状态。`DoorBlock`的一个例子：

```Java
this.registerDefaultState(
  this.stateDefinition.any()
    .setValue(FACING, Direction.NORTH)
    .setValue(OPEN, false)
    .setValue(HINGE, DoorHingeSide.LEFT)
    .setValue(POWERED, false)
    .setValue(HALF, DoubleBlockHalf.LOWER)
);
```
如果您希望更改放置方块时使用的`BlockState`，您可以重载`Block#getStateForPlacement(BlockPlaceContext)`。例如，这可以用来根据玩家放置方块时的位置设置方块的方向。

因为`BlockState`是不可变的，并且它们属性的所有组合都是在游戏启动时生成的，所以调用`BlockState#setValue(Property<T>, T)`将简单地转到`Block`的`StateHolder`，并请求包含您想要的属性值的`BlockState`。

因为所有可能的`BlockState`都是在启动时生成的，所以您可以自由使用引用相等运算符（`==`）来检查两个`BlockState`是否相等。


使用`BlockState`
---------------------
您可以通过调用`BlockState#getValue(Property<?>)`来获取属性的值，并将您想要获取值的属性传递给它。
如果要获得具有不同值集的`BlockState`，只需通过属性及其值调用`BlockState#setValue(Property<T>, T)`。

可以使用`Level#setBlockAndUpdate(BlockPos, BlockState)`，`Level#getBlockState(BlockPos)`在世界中获取和放置`BlockState`。如果放置`Block`，将调用`Block#defaultBlockState()`获取"默认"状态，并后续调用`BlockState#setValue(Property<T>, T)`实现所需状态。
