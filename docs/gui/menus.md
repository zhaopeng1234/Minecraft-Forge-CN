# 菜单系统

菜单是图形用户界面（GUI）的一种后端类型，它们处理与某些数据容器交互时涉及的逻辑。菜单本身并不是数据容器，而是允许用户间接修改内部数据容器状态的视图。因此，数据容器不应直接与任何菜单耦合，而应传入数据引用以供调用和修改。

## `MenuType`

菜单是动态创建和移除的，因此它们本身不是注册对象。所以，需要注册另一个工厂对象，以便轻松创建和引用菜单的*类型*。对于菜单而言，这些就是 `MenuType`。

`MenuType` 必须[注册][registered]。

### `MenuSupplier`

`MenuType` 通过将 `MenuSupplier` 和 `FeatureFlagSet` 传入其构造函数来创建。`MenuSupplier` 表示一个函数，它接受容器的 ID 和查看菜单的玩家的物品栏，并返回一个新创建的 [`AbstractContainerMenu`][acm]。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final RegistryObject<MenuType<MyMenu>> MY_MENU = REGISTER.register("my_menu", () -> new MenuType(MyMenu::new, FeatureFlags.DEFAULT_FLAGS));

// 在 MyMenu 中，这是一个 AbstractContainerMenu 子类
public MyMenu(int containerId, Inventory playerInv) {
  super(MY_MENU.get(), containerId);
  // ...
}
```

!!! 注意
    容器标识符对于单个玩家是唯一的。这意味着，即使两个不同的玩家正在查看同一个数据容器，相同的容器 ID 在他们那里也代表两个不同的菜单。

`MenuSupplier` 通常负责在客户端创建一个菜单，该菜单使用虚拟数据引用来存储和与服务器数据容器同步的信息进行交互。

### `IContainerFactory`

如果客户端需要额外的信息（例如，数据容器在世界中的位置），则可以使用子类 `IContainerFactory`。除了容器 ID 和玩家物品栏外，它还提供一个 `FriendlyByteBuf`，可以存储从服务器发送的额外信息。可以通过 `IForgeMenuType#create` 使用 `IContainerFactory` 创建 `MenuType`。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final RegistryObject<MenuType<MyMenuExtra>> MY_MENU_EXTRA = REGISTER.register("my_menu_extra", () -> IForgeMenuType.create(MyMenu::new));

// 在 MyMenuExtra 中，这是一个 AbstractContainerMenu 子类
public MyMenuExtra(int containerId, Inventory playerInv, FriendlyByteBuf extraData) {
  super(MY_MENU_EXTRA.get(), containerId);
  // 从缓冲区存储额外数据
  // ...
}
```

## `AbstractContainerMenu`

所有菜单都继承自 `AbstractContainerMenu`。菜单接受两个参数，即 [`MenuType`][mt]，它表示菜单本身的类型，以及容器 ID，它表示当前访问者的菜单的唯一标识符。

!!! 重要
    玩家一次只能打开 100 个唯一的菜单。

每个菜单应该包含两个构造函数：一个用于在服务器上初始化菜单，另一个用于在客户端上初始化菜单。用于在客户端上初始化菜单的构造函数是提供给 `MenuType` 的那个。服务器菜单构造函数包含的任何字段，在客户端菜单构造函数中都应该有一些默认值。

```java
// 客户端菜单构造函数
public MyMenu(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory);
}

// 服务器菜单构造函数
public MyMenu(int containerId, Inventory playerInventory) {
  // ...
}
```

每个菜单实现必须实现两个方法：`#stillValid` 和 [`#quickMoveStack`][qms]。

### `#stillValid` 和 `ContainerLevelAccess`

`#stillValid` 确定给定玩家的菜单是否应该保持打开状态。这通常指向静态的 `#stillValid` 方法，该方法接受一个 `ContainerLevelAccess`、玩家以及此菜单所关联的 `Block`。客户端菜单的此方法必须始终返回 `true`，这也是静态 `#stillValid` 方法的默认行为。此实现会检查玩家是否在数据存储对象所在位置的八格范围内。

`ContainerLevelAccess` 在封闭作用域内提供当前世界和方块的位置。在服务器上构造菜单时，可以通过调用 `ContainerLevelAccess#create` 创建一个新的访问对象。客户端菜单构造函数可以传入 `ContainerLevelAccess#NULL`，它不执行任何操作。

```java
// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, ContainerLevelAccess.NULL);
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, ContainerLevelAccess access) {
  // ...
}

// 假设此菜单与 RegistryObject<Block> MY_BLOCK 关联
@Override
public boolean stillValid(Player player) {
  return AbstractContainerMenu.stillValid(this.access, player, MY_BLOCK.get());
}
```

### 数据同步

有些数据需要同时存在于服务器和客户端，以便向玩家显示。为此，菜单实现了一个基本的数据同步层，以便每当当前数据与上次同步到客户端的数据不匹配时进行更新。对于玩家而言，这会每秒检查一次。

Minecraft 默认支持两种形式的数据同步：通过 `Slot` 同步 `ItemStack`，通过 `DataSlot` 同步整数。`Slot` 和 `DataSlot` 是视图，它们持有对数据存储的引用，假设操作有效，玩家可以在屏幕中修改这些数据。可以在构造函数中通过 `#addSlot` 和 `#addDataSlot` 将它们添加到菜单中。

!!! 注意
    由于 Forge 已弃用 `Slot` 使用的 `Container`，转而使用 [`IItemHandler` 能力][cap]，因此接下来的解释将围绕使用能力变体 `SlotItemHandler` 展开。

`SlotItemHandler` 包含四个参数：表示物品堆所在物品栏的 `IItemHandler`、此槽位具体表示的物品堆的索引，以及该槽位在屏幕上相对于 `AbstractContainerScreen#leftPos` 和 `#topPos` 的左上角渲染位置的 x 和 y 坐标。客户端菜单构造函数应始终提供一个相同大小的空物品栏实例。

在大多数情况下，菜单包含的任何槽位会首先添加，然后是玩家的物品栏，最后是玩家的快捷栏。要从菜单中访问任何单个 `Slot`，必须根据槽位添加的顺序计算索引。

`DataSlot` 是一个抽象类，应该实现一个 getter 和 setter 来引用数据存储对象中存储的数据。客户端菜单构造函数应始终通过 `DataSlot#standalone` 提供一个新实例。

每次初始化新菜单时，都应该重新创建这些以及槽位。

!!! 警告
    虽然 `DataSlot` 存储一个整数，但由于它在网络上传输值的方式，实际上它被限制为一个**短整型**（-32768 到 32767）。整数的高 16 位会被忽略。

```java
// 假设我们有一个来自数据对象的大小为 5 的物品栏
// 假设在服务器菜单每次初始化时都会构造一个 DataSlot

// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, new ItemStackHandler(5), DataSlot.standalone());
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, IItemHandler dataInventory, DataSlot dataSingle) {
  // 检查数据物品栏的大小是否为某个固定值
  // 然后，为数据物品栏添加槽位
  this.addSlot(new SlotItemHandler(dataInventory, /*...*/));

  // 为玩家物品栏添加槽位
  this.addSlot(new Slot(playerInventory, /*...*/));

  // 为处理的整数添加数据槽位
  this.addDataSlot(dataSingle);

  // ...
}
```

#### `ContainerData`

如果需要将多个整数同步到客户端，可以使用 `ContainerData` 来引用这些整数。此接口用作索引查找，每个索引代表一个不同的整数。如果通过 `#addDataSlots` 将 `ContainerData` 添加到菜单中，也可以在数据对象本身中构造 `ContainerData`。该方法会为接口指定的数据数量创建一个新的 `DataSlot`。客户端菜单构造函数应始终通过 `SimpleContainerData` 提供一个新实例。

```java
// 假设我们有一个大小为 3 的 ContainerData

// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
  this(containerId, playerInventory, new SimpleContainerData(3));
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, ContainerData dataMultiple) {
  // 检查 ContainerData 的大小是否为某个固定值
  checkContainerDataCount(dataMultiple, 3);

  // 为处理的整数添加数据槽位
  this.addDataSlots(dataMultiple);

  // ...
}
```

!!! 警告
    由于 `ContainerData` 委托给 `DataSlot`，因此它们也被限制为一个**短整型**（-32768 到 32767）。

#### `#quickMoveStack`

`#quickMoveStack` 是任何菜单都必须实现的第二个方法。每当物品堆被按住 Shift 键点击（即快速移动）时，就会调用此方法，直到物品堆完全移出其先前的槽位，或者没有其他地方可以放置该物品堆为止。该方法返回正在快速移动的槽位中的物品堆
