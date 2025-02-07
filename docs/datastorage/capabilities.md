# 能力系统
能力系统允许以动态和灵活的方式公开功能，而无需直接实现众多接口。

一般来说，每个能力都以接口的形式提供一项功能。

Forge 为方块实体、实体、物品堆、世界和世界区块添加了能力支持。这些能力可以通过事件进行附加，或者在你自己实现这些对象时重写能力相关方法来公开。以下各节将详细解释。

## Forge 提供的能力
Forge 提供了三种能力：`IItemHandler`、`IFluidHandler` 和 `IEnergyStorage`。

### `IItemHandler`
`IItemHandler` 公开了一个处理物品栏槽位的接口。它可以应用于方块实体（如箱子、机器等）、实体（如玩家的额外槽位、生物的物品栏/背包）或物品堆（如便携式背包等）。它用一个便于自动化操作的系统取代了旧的 `Container` 和 `WorldlyContainer`。

### `IFluidHandler`
`IFluidHandler` 公开了一个处理流体存储的接口。它同样可以应用于方块实体、实体或物品堆。

### `IEnergyStorage`
`IEnergyStorage` 公开了一个处理能量容器的接口。它可以应用于方块实体、实体或物品堆。它基于 TeamCoFH 的 RedstoneFlux API。

## 使用现有能力
如前所述，方块实体、实体和物品堆通过 `ICapabilityProvider` 接口实现了能力提供者的功能。该接口添加了 `#getCapability` 方法，可用于查询相关提供者对象中存在的能力。

要获取一个能力，你需要通过其唯一实例来引用它。对于 `IItemHandler`，这个能力主要存储在 `ForgeCapabilities#ITEM_HANDLER` 中，但也可以使用 `CapabilityManager#get` 来获取其他实例引用。
```java
public static final Capability<IItemHandler> ITEM_HANDLER = CapabilityManager.get(new CapabilityToken<>(){});
```
调用 `CapabilityManager#get` 时，会为你关联的类型提供一个非空的能力。匿名的 `CapabilityToken` 允许 Forge 保持软依赖系统，同时仍拥有获取正确能力所需的泛型信息。

!!! important
    即使你随时都有一个非空的能力可用，也不意味着该能力本身已经可以使用或已注册。可以通过 `Capability#isRegistered` 来检查。

`#getCapability` 方法有一个 `Direction` 类型的第二个参数，可用于请求特定面的能力实例。如果传入 `null`，可以假设请求来自方块内部或某个面的概念没有意义的地方，例如不同的维度。在这种情况下，将请求一个不考虑面的通用能力实例。`#getCapability` 的返回类型将对应于传递给该方法的能力中声明的类型的 `LazyOptional`。对于物品处理能力，这是 `LazyOptional<IItemHandler>`。如果某个提供者没有该能力，将返回一个空的 `LazyOptional`。

## 公开能力
要公开一个能力，你首先需要一个底层能力类型的实例。请注意，你应该为每个持有该能力的对象分配一个单独的实例，因为该能力很可能与包含它的对象相关联。

对于 `IItemHandler`，默认实现使用 `ItemStackHandler` 类，该类在构造函数中有一个可选参数，用于指定槽位数量。然而，应避免依赖这些默认实现的存在，因为能力系统的目的是防止在能力不存在的上下文中出现加载错误，因此实例化应该在检查能力是否已注册的保护下进行（见上一节中关于 `CapabilityManager#get` 的说明）。

一旦你有了自己的能力接口实例，你需要通知能力系统的用户你公开了这个能力，并提供该接口引用的 `LazyOptional`。这可以通过重写 `#getCapability` 方法，并将能力实例与你要公开的能力进行比较来实现。如果你的机器根据查询的面有不同的槽位，你可以使用 `side` 参数进行测试。对于实体和物品堆，可以忽略这个参数，但仍然可以将面作为一个上下文，例如玩家不同的盔甲槽位（`Direction#UP` 公开玩家的头盔槽位），或者物品栏中周围方块的情况（`Direction#WEST` 公开熔炉的输入槽位）。不要忘记回退到 `super`，否则现有的附加能力将停止工作。

能力必须在提供者的生命周期结束时通过 `LazyOptional#invalidate` 使其无效。对于拥有的方块实体和实体，可以在 `#invalidateCaps` 方法中使 `LazyOptional` 无效。对于非拥有的提供者，应该将一个提供无效操作的可运行对象传递给 `AttachCapabilitiesEvent#addListener`。
```java
// 在你的方块实体子类的某个地方
LazyOptional<IItemHandler> inventoryHandlerLazyOptional;

// 提供的实例（例如 () -> inventoryHandler）
// 确保延迟初始化，因为初始化应该只在需要时进行
inventoryHandlerLazyOptional = LazyOptional.of(inventoryHandlerSupplier);

@Override
public <T> LazyOptional<T> getCapability(Capability<T> cap, Direction side) {
  if (cap == ForgeCapabilities.ITEM_HANDLER) {
    return inventoryHandlerLazyOptional.cast();
  }
  return super.getCapability(cap, side);
}

@Override
public void invalidateCaps() {
  super.invalidateCaps();
  inventoryHandlerLazyOptional.invalidate();
}
```

!!! tip
    如果在给定对象上只公开一个能力，你可以使用 `Capability#orEmpty` 作为 if/else 语句的替代。
    ```java
    @Override
    public <T> LazyOptional<T> getCapability(Capability<T> cap, Direction side) {
      return ForgeCapabilities.ITEM_HANDLER.orEmpty(cap, inventoryHandlerLazyOptional);
    }
    ```
`Item` 是一个特殊情况，因为它们的能力提供者存储在 `ItemStack` 上。相反，应该通过 `Item#initCapabilities` 附加一个提供者。这应该在物品堆的生命周期内持有你的能力。

强烈建议在代码中直接进行能力检查，而不是试图依赖映射或其他数据结构，因为能力检查可能每秒被许多对象执行，为了避免减慢游戏速度，它们需要尽可能快。

## 附加能力
如前所述，可以使用 `AttachCapabilitiesEvent` 将能力附加到现有的提供者、`Level` 和 `LevelChunk` 上。同一个事件用于所有可以提供能力的对象。`AttachCapabilitiesEvent` 有 5 种有效的泛型类型，提供以下事件：
- `AttachCapabilitiesEvent<Entity>`：仅针对实体触发。
- `AttachCapabilitiesEvent<BlockEntity>`：仅针对方块实体触发。
- `AttachCapabilitiesEvent<ItemStack>`：仅针对物品堆触发。
- `AttachCapabilitiesEvent<Level>`：仅针对世界触发。
- `AttachCapabilitiesEvent<LevelChunk>`：仅针对世界区块触发。

泛型类型不能比上述类型更具体。例如：如果你想将能力附加到 `Player` 上，你必须订阅 `AttachCapabilitiesEvent<Entity>`，然后在附加能力之前确定提供的对象是 `Player`。

在所有情况下，该事件都有一个 `#addCapability` 方法，可用于将能力附加到目标对象上。不是将能力本身添加到列表中，而是添加能力提供者，这些提供者有机会仅从某些面返回能力。虽然提供者只需要实现 `ICapabilityProvider`，但如果能力需要持久存储数据，则可以实现 `ICapabilitySerializable<T extends Tag>`，该接口除了返回能力外，还将提供标签的保存/加载功能。

有关如何实现 `ICapabilityProvider` 的信息，请参考[公开能力][expose]部分。

## 创建自己的能力
可以使用以下两种方式之一注册一个能力：`RegisterCapabilitiesEvent` 或 `@AutoRegisterCapability`。

### `RegisterCapabilitiesEvent`
可以使用 `RegisterCapabilitiesEvent`，通过将能力类型的类提供给 `#register` 方法来注册一个能力。该事件在模组事件总线上[处理]。
```java
@SubscribeEvent
public void registerCaps(RegisterCapabilitiesEvent event) {
  event.register(IExampleCapability.class);
}
```

### `@AutoRegisterCapability`
通过使用 `@AutoRegisterCapability` 注解能力类型来注册一个能力。
```java
@AutoRegisterCapability
public interface IExampleCapability {
  // ...
}
```

## 持久化世界区块和方块实体的能力
与世界、实体和物品堆不同，世界区块和方块实体只有在被标记为脏时才会写入磁盘。因此，为世界区块或方块实体实现具有持久状态的能力时，应确保每当其状态改变时，将其所有者标记为脏。

`ItemStackHandler` 通常用于方块实体的物品栏，它有一个可重写的方法 `void onContentsChanged(int slot)`，用于将方块实体标记为脏。
```java
public class MyBlockEntity extends BlockEntity {

  private final IItemHandler inventory = new ItemStackHandler(...) {
    @Override
    protected void onContentsChanged(int slot) {
      super.onContentsChanged(slot);
      setChanged();
    }
  }

  // ...
}
```

## 与客户端同步数据
默认情况下，能力数据不会发送到客户端。为了改变这种情况，模组必须使用数据包来管理自己的同步代码。

有三种不同的情况可能需要发送同步数据包，所有这些都是可选的：
1. 当实体在世界中生成或方块被放置时，你可能想与客户端共享初始化时分配的值。
2. 当存储的数据发生变化时，你可能想通知部分或所有正在观察的客户端。
3. 当新的客户端开始查看实体或方块时，你可能想通知它现有数据。

有关实现网络数据包的更多信息，请参考[网络][network]页面。

## 在玩家死亡后持久化
默认情况下，能力数据在玩家死亡后不会持久化。为了改变这种情况，必须在玩家重生过程中克隆实体时手动复制数据。

这可以通过 `PlayerEvent$Clone` 事件来完成，从原始实体读取数据并将其分配给新实体。在这个事件中，可以使用 `#isWasDeath` 方法来区分死亡后重生和从末地返回的情况。这很重要，因为从末地返回时数据已经存在，所以在这种情况下要注意不要复制值。

[expose]: #exposing-a-capability
[handled]: ../concepts/events.md#creating-an-event-handler
[network]: ../networking/index.md
