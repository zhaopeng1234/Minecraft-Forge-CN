
SimpleImpl
==========

SimpleImpl是围绕`SimpleChannel`类构建的数据包系统的名称。到目前为止，使用这个系统是在客户端和服务器之间发送自定义数据最简单的方法。

入门
---------------

首先，你需要创建`SimpleChannel`对象。我们建议你在一个单独的类中完成这个操作，比如`ModidPacketHandler`。在这个类中，将`SimpleChannel`创建为一个静态字段，如下所示：

```java
private static final String PROTOCOL_VERSION = "1";
public static final SimpleChannel INSTANCE = NetworkRegistry.newSimpleChannel(
  new ResourceLocation("mymodid", "main"),
  () -> PROTOCOL_VERSION,
  PROTOCOL_VERSION::equals,
  PROTOCOL_VERSION::equals
);
```

第一个参数是通道的名称。第二个参数是一个`Supplier<String>`，用于返回当前的网络协议版本。第三个和第四个参数分别是`Predicate<String>`，用于检查传入连接的协议版本是否分别与客户端或服务器在网络上兼容。
在这里，我们直接将其与`PROTOCOL_VERSION`字段进行比较，这意味着客户端和服务器的`PROTOCOL_VERSION`必须始终匹配，否则FML将拒绝登录。

版本检查器
-------------------

如果你的模组不要求对方有特定的网络通道，或者根本不要求对方是Forge实例，你应该注意正确定义你的版本兼容性检查器（`Predicate<String>`参数），以处理版本检查器可能接收到的额外“元版本”（在`NetworkRegistry`中定义）。这些元版本包括：

* `ABSENT` - 如果此通道在另一端点缺失。请注意，在这种情况下，该端点仍然是一个Forge端点，并且可能有其他模组。
* `ACCEPTVANILLA` - 如果该端点是一个原版（或非Forge）端点。

如果对这两个元版本都返回`false`，则意味着此通道必须在另一端点存在。如果你只是复制上面的代码，它就是这么工作的。请注意，这些值也用于列表ping兼容性检查，该检查负责在多人游戏服务器选择屏幕中显示绿色对勾/红色叉号。

注册数据包
-------------------

接下来，我们必须声明我们想要发送和接收的消息类型。这可以通过`INSTANCE#registerMessage`来完成，它接受5个参数：

- 第一个参数是数据包的鉴别器。这是每个通道内数据包的唯一ID。我们建议你使用一个局部变量来保存这个ID，然后使用`id++`来调用`registerMessage`。这样可以保证ID的100%唯一性。
- 第二个参数是实际的数据包类`MSG`。
- 第三个参数是一个`BiConsumer<MSG, FriendlyByteBuf>`，负责将消息编码到提供的`FriendlyByteBuf`中。
- 第四个参数是一个`Function<FriendlyByteBuf, MSG>`，负责从提供的`FriendlyByteBuf`中解码消息。
- 最后一个参数是一个`BiConsumer<MSG, Supplier<NetworkEvent.Context>>`，负责处理消息本身。

最后三个参数可以是对Java中静态或实例方法的方法引用。请记住，实例方法`MSG#encode(FriendlyByteBuf)`仍然满足`BiConsumer<MSG, FriendlyByteBuf>`；`MSG`只是成为隐式的第一个参数。

处理数据包
----------------

在数据包处理程序中有几个要点需要强调。数据包处理程序可以同时访问消息对象和网络上下文。上下文允许你访问发送数据包的玩家（如果在服务器上），并提供了一种将线程安全的工作加入队列的方法。

```java
public static void handle(MyMessage msg, Supplier<NetworkEvent.Context> ctx) {
  ctx.get().enqueueWork(() -> {
    // 需要线程安全的工作（大多数工作）
    ServerPlayer sender = ctx.get().getSender(); // 发送此数据包的客户端
    // 执行操作
  });
  ctx.get().setPacketHandled(true);
}
```

从服务器发送到客户端的数据包应该在另一个类中处理，并通过`DistExecutor#unsafeRunWhenOn`进行包装。

```java
// 在Packet类中
public static void handle(MyClientMessage msg, Supplier<NetworkEvent.Context> ctx) {
  ctx.get().enqueueWork(() ->
    // 确保仅在物理客户端上执行
    DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> () -> ClientPacketHandlerClass.handlePacket(msg, ctx))
  );
  ctx.get().setPacketHandled(true);
}

// 在ClientPacketHandlerClass中
public static void handlePacket(MyClientMessage msg, Supplier<NetworkEvent.Context> ctx) {
  // 执行操作
}
```

注意`#setPacketHandled`的存在，它用于告诉网络系统该数据包已成功处理完毕。

!!! warning
    从Minecraft 1.8开始，数据包默认在网络线程上处理。

   这意味着你的处理程序不能直接与大多数游戏对象进行交互。Forge提供了一种便捷的方法，通过提供的`NetworkEvent$Context`让你的代码在主线程上执行。只需调用`NetworkEvent$Context#enqueueWork(Runnable)`，它将在下次有机会时在主线程上调用给定的`Runnable`。

!!! warning
    在服务器上处理数据包时要保持警惕。客户端可能会尝试通过发送意外数据来利用数据包处理过程。

   一个常见的问题是容易受到**任意区块生成**的影响。这通常发生在服务器信任客户端发送的方块位置来访问方块和方块实体时。当在未加载的关卡区域访问方块和方块实体时，服务器将从磁盘生成或加载该区域，然后迅速将其写回磁盘。这可能会被利用，对服务器的性能和存储空间造成**灾难性破坏**而不留痕迹。

    为避免这个问题，一个通用的经验法则是仅当`Level#hasChunkAt`为`true`时才访问方块和方块实体。

发送数据包
---------------

### 发送到服务器

将数据包发送到服务器只有一种方法。这是因为客户端一次只能连接到*一个*服务器。为此，我们必须再次使用前面定义的`SimpleChannel`。只需调用`INSTANCE.sendToServer(new MyMessage())`。如果存在该类型的处理程序，消息将被发送到相应的处理程序。

### 发送到客户端

可以使用`SimpleChannel`将数据包直接发送到客户端：`HANDLER.sendTo(new MyClientMessage(), serverPlayer.connection.getConnection(), NetworkDirection.PLAY_TO_CLIENT)`。然而，这可能相当不方便。Forge提供了一些便捷函数可供使用：

```java
// 发送给一个玩家
INSTANCE.send(PacketDistributor.PLAYER.with(serverPlayer), new MyMessage());

// 发送给追踪此关卡区块的所有玩家
INSTANCE.send(PacketDistributor.TRACKING_CHUNK.with(levelChunk), new MyMessage());

// 发送给所有已连接的玩家
INSTANCE.send(PacketDistributor.ALL.noArg(), new MyMessage());
```

还有其他可用的`PacketDistributor`类型；有关更多详细信息，请查看`PacketDistributor`类的文档。
