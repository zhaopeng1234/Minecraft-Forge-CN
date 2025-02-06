
# 方块实体

“方块实体”（BlockEntities）类似于简化版的“实体”（Entities），它们与方块绑定。
它们用于存储动态数据、执行基于刻的任务以及进行动态渲染。
原版《我的世界》中的一些例子包括处理箱子中的物品栏、熔炉的熔炼逻辑，或者信标的区域效果。
在模组中还有更高级的例子，比如采石场、分类机、管道和显示器。

!!! 注意
    “方块实体”并非万能解决方案，使用不当可能会导致卡顿。
    如有可能，尽量避免使用。

## 注册

方块实体是动态创建和移除的，因此它们本身并非注册对象。

要创建一个“方块实体”，你需要继承 `BlockEntity` 类。因此，需要注册另一个对象，以便轻松创建和引用这个动态对象的“类型”。对于“方块实体”，这些被称为“方块实体类型”（`BlockEntityType`）。

一个“方块实体类型”可以像任何其他注册对象一样[注册][registration]。要构造一个 `BlockEntityType`，可以通过 `BlockEntityType$Builder#of` 使用其构建器形式。这需要两个参数：一个 `BlockEntityType$BlockEntitySupplier`，它接受一个 `BlockPos` 和 `BlockState` 来创建关联的 `BlockEntity` 的新实例，以及一个可变参数列表，列出这个 `BlockEntity` 可以附加到的 `Block`。通过调用 `BlockEntityType$Builder#build` 来构建 `BlockEntityType`。这需要传入一个 `Type`，它表示在 `DataFixer` 中用于引用这个注册对象的类型安全引用。由于 `DataFixer` 是模组可选使用的系统，这个参数可以传 `null`。

```java
// 对于某个 DeferredRegister<BlockEntityType<?>> REGISTER
public static final RegistryObject<BlockEntityType<MyBE>> MY_BE = REGISTER.register("mybe", () -> BlockEntityType.Builder.of(MyBE::new, validBlocks).build(null));

// 在 MyBE 中，这是一个 BlockEntity 子类
public MyBE(BlockPos pos, BlockState state) {
  super(MY_BE.get(), pos, state);
}
```

## 创建一个“方块实体”

要创建一个“方块实体”并将其附加到一个“方块”上，你的“方块”子类必须实现 `EntityBlock` 接口。必须实现 `EntityBlock#newBlockEntity(BlockPos, BlockState)` 方法，并返回你的 `BlockEntity` 的一个新实例。

## 在“方块实体”中存储数据

为了保存数据，需要重写以下两个方法：
```java
BlockEntity#saveAdditional(CompoundTag tag)

BlockEntity#load(CompoundTag tag)
```
每当包含“方块实体”的 `LevelChunk` 从标签加载或保存到标签时，就会调用这些方法。
使用它们来读取和写入你的方块实体类中的字段。

!!! 注意
		每当你的数据发生变化时，你需要调用 `BlockEntity#setChanged`；否则，在保存世界时，包含你的 `BlockEntity` 的 `LevelChunk` 可能会被跳过。

!!! 重要
		调用 `super` 方法非常重要！
标签名 `id`、`x`、`y`、`z`、`ForgeData` 和 `ForgeCaps` 由 `super` 方法保留。

## 使“方块实体”刻动

如果你需要一个刻动的“方块实体”，例如在熔炼过程中跟踪进度，那么必须在 `EntityBlock` 中实现并重写另一个方法：`EntityBlock#getTicker(Level, BlockState, BlockEntityType)`。这可以根据用户所在的逻辑端实现不同的刻动器，或者只实现一个通用的刻动器。无论哪种情况，都必须返回一个 `BlockEntityTicker`。由于这是一个函数式接口，它可以只接受一个表示刻动器的方法：

```java
// 在某个方块子类内部
@Nullable
@Override
public <T extends BlockEntity> BlockEntityTicker<T> getTicker(Level level, BlockState state, BlockEntityType<T> type) {
  return type == MyBlockEntityTypes.MYBE.get()? MyBlockEntity::tick : null;
}

// 在 MyBlockEntity 内部
public static void tick(Level level, BlockPos pos, BlockState state, MyBlockEntity blockEntity) {
  // 执行操作
}
```

!!! 注意
    这个方法每刻都会被调用；因此，你应该避免在这里进行复杂的计算。如果可能，你应该每隔 X 刻进行更复杂的计算。（每秒的刻数可能低于 20（二十），但不会高于 20）

## 将数据同步到客户端

有三种将数据同步到客户端的方法：在区块加载时同步、在方块更新时同步，以及使用自定义网络消息。

### 在 `LevelChunk` 加载时同步

为此，你需要重写：
```java
BlockEntity#getUpdateTag()

IForgeBlockEntity#handleUpdateTag(CompoundTag tag)
```
同样，这相当简单，第一个方法收集应该发送到客户端的数据，
而第二个方法处理这些数据。如果你的 `BlockEntity` 包含的数据不多，你也许可以使用[在“方块实体”中存储数据][storing-data]部分的方法。

!!! 重要
    为方块实体同步过多/无用的数据可能会导致网络拥塞。你应该通过仅在客户端需要时发送客户端所需的信息来优化网络使用。例如，在更新标签中发送方块实体的物品栏通常是不必要的，因为这可以通过其[`AbstractContainerMenu`][menu]进行同步。

### 在方块更新时同步

这种方法稍微复杂一些，但同样你只需要重写两到三个方法。
这里有一个简单的示例实现：
```java
@Override
public CompoundTag getUpdateTag() {
  CompoundTag tag = new CompoundTag();
  // 将你的数据写入标签
  return tag;
}

@Override
public Packet<ClientGamePacketListener> getUpdatePacket() {
  // 将从 #getUpdateTag 获取标签
  return ClientboundBlockEntityDataPacket.create(this);
}

// 可以重写 IForgeBlockEntity#onDataPacket。默认情况下，这将委托给 #load。
```
静态构造函数 `ClientboundBlockEntityDataPacket#create` 接受：

* `BlockEntity`。
* 一个可选的函数，用于从 `BlockEntity` 获取 `CompoundTag`。默认情况下，它使用 `BlockEntity#getUpdateTag`。

现在，要发送数据包，必须在服务器上发出更新通知。
```java
Level#sendBlockUpdated(BlockPos pos, BlockState oldState, BlockState newState, int flags)
```
`pos` 应该是你的 `BlockEntity` 的位置。
对于 `oldState` 和 `newState`，你可以传入该位置当前的 `BlockState`。
`flags` 是一个位掩码，应该包含 `2`，这将把更改同步到客户端。有关更多信息以及其余标志，请参阅 `Block`。标志 `2` 等同于 `Block#UPDATE_CLIENTS`。

### 使用自定义网络消息同步

这种同步方式可能是最复杂的，但通常也是最优化的，
因为你可以确保只有需要同步的数据才会真正被同步。
在尝试这种方法之前，你应该首先查看[“网络”][networking]部分，特别是[“SimpleImpl”][simple_impl]。
一旦你创建了自定义网络消息，你可以使用 `SimpleChannel#send(PacketDistributor$PacketTarget, MSG)` 将其发送给所有加载了该 `BlockEntity` 的用户。

!!! 警告
    进行安全检查很重要，当消息到达玩家时，`BlockEntity` 可能已经被销毁/替换！你还应该检查区块是否已加载（`Level#hasChunkAt(BlockPos)`）。

[registration]:../concepts/registries.md#methods-for-registering
[storing-data]: #在方块实体中存储数据
[menu]:../gui/menus.md
[networking]:../networking/index.md
[simple_impl]:../networking/simpleimpl.md
