Minecraft中的“端”
===================

开发Minecraft的Mod时需要了解的一个非常重要的概念是`端`： *客户端* 和 *服务端*。在理解`端`的概念时有很多常见的误解和错误，这些误解和错误可能会导致一些 BUG，这些 BUG 可能不会让游戏崩溃，但却会产生意想不到的效果。

不同类型的“端”
------------------------

当我们说到 "客户端 "或 "服务端"时，通常都会相当直观地理解我们所说的是游戏的哪个部分。 毕竟，客户端是用户与之交互的地方，而服务端则是用户连接多人游戏的地方。 很简单吧？

事实证明，即使是两个这样的术语，也可能存在一些歧义。 在此，我们将消除 "客户端 "和 "服务端"的四种可能含义：

* 物理客户端 - *物理客户端* 是您从启动器启动 Minecraft 时运行的整个程序。 在游戏的图形化可交互生命周期中运行的所有线程、进程和服务都是物理客户端的一部分。
* 物理服务器 - 通常被称为专用服务器，*物理服务器*是在您启动任何类型的`minecraft_server.jar`时运行的整个程序，它不会显示可播放的图形用户界面。
* 逻辑服务器 - *逻辑服务器*负责运行游戏逻辑：怪物生成、天气、更新库存、血量、怪物AI和所有其他游戏机制。 逻辑服务器存在于物理服务器中，但在单人游戏时它也可以在物理客户端中与逻辑客户端一起运行。 逻辑服务器总是在名为`Server Thread`的线程中运行。
* 逻辑客户端 - *逻辑客户端*接受玩家的输入，并将其转发给逻辑服务器。 此外，它还从逻辑服务器接收信息，并以图形方式提供给玩家。 逻辑客户端在`Render Thread`中运行，但通常会产生几个其他线程来处理音频和块渲染批处理等事务。
在 MinecraftForge 代码库中，物理侧由名为 `Dist` 的枚举表示，而逻辑侧则由名为 `LogicalSide`.
 的枚举表示。
 
执行特定端的操作
-----------------------------------

### `Level`类中的`isClientSide`方法

这个布尔判断将是您最常用的判断边的方法。 在`Level`对象上查询此字段可以确定Level所属的**逻辑**边。 也就是说，如果此字段为`true`，则该Level当前运行在逻辑客户端上。 如果字段为`false`，则该级别在逻辑服务器上运行。 因此，物理服务器将始终在该字段中包含`false`，但我们不能假定`false`意味着物理服务器、 因为当这个字段为`false`时，也可以表示单人游戏中的逻辑服务器。

当您需要确定是否应运行游戏逻辑和其他机制时，请使用此检查。 例如，如果您想在玩家每次点击您的方块时都对其造成伤害，或让您的机器将泥土加工成钻石，那么只有在确保`isClientSide`为`false`后才能这样做。将游戏逻辑应用到逻辑客户端在最好的情况下会导致不同步（幽灵实体、不同步统计等），在最坏的情况下会导致崩溃。

您应将此方法作为默认检查方法。 除了`DistExecutor`之外，您很少需要使用其他方法来确定`端`和调整行为。

### `DistExecutor`

考虑到客户端和服务端Mod使用单一的 "通用 "jar，并将物理端分为两个 jar，我们想到了一个重要的问题： 我们如何使用只存在于一个物理端的代码？ `net.minecraft.client` 中的所有代码仅存在于物理客户端。 如果您编写的任何类以任何方式引用了这些名称，那么当在不存在这些名称的环境中加载相应类时，游戏就会崩溃。 初学者中一个非常常见的错误是在方块或方块实体类中调用 `Minecraft.getInstance()` ，一旦加载该类，任何物理服务器都将崩溃。

我们如何解决这个问题？ 幸运的是，FML 有`DistExecutor`，它提供了在不同物理端运行不同方法，或仅在一端运行单个方法的各种方法。

!!! 注意
    重要的是要了解 FML 基于 ** 物理**端进行检查。 单人游戏世界（逻辑服务器 + 物理客户端中的逻辑客户端）将始终使用 `Dist.CLIENT`!

`DistExecutor`的工作原理是接收supplier提供的执行方法，然后利用[`invokedynamicJVM 指令`][invokedynamic]防止类加载。 被执行的方法应是静态方法，并位于不同的类中。 此外，如果静态方法没有参数，则应使用方法引用而不是执行方法的supplier。

`DistExecutor`中有两个主要方法： `runWhenOn`和`callWhenOn`。 这些方法分别接收执行方法应运行的物理端和提供的执行方法，然后执行方法或返回结果。
这两种方法又进一步细分为`safe*`和`unsafe*`变体。 就其目的而言，安全和不安全变体的名称是错误的。 主要区别在于，在开发环境中，`safe*` 方法将验证提供的执行方法是否是返回另一个类的方法引用的 lambda，否则将抛出错误。 在生产环境中，`safe*` 和 `unsafe*` 在功能上是相同的。

```java
// 在客户端类ExampleClass中：
public static void unsafeRunMethodExample(Object param1, Object param2) {
  // ...
}

public static Object safeCallMethodExample() {
  // ...
}

// 在一些普通类中
DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> ExampleClass.unsafeRunMethodExample(var1, var2));

DistExecutor.safeCallWhenOn(Dist.CLIENT, () -> ExampleClass::safeCallMethodExample);

```

!!! 警告
    由于 `invokedynamic` 在 Java 9+ 中的工作方式发生了变化、 在开发环境中，所有 `safe*` 变体的 `DistExecutor` 方法都会抛出包裹在 `BootstrapMethodError` 中的原始异常。 应使用`unsafe*` 变体或检查 [`FMLEnvironment#dist`][dist] 。

### 线程组

如果`Thread.currentThread().getThreadGroup() == SidedThreadGroups.SERVER` 为真，则当前线程可能位于逻辑服务端上。 否则，当前线程可能位于逻辑客户端。 当您无法访问 `Level` 对象以调用`isClientSide` 时，这对于检索 **逻辑** 端非常有用。 它*通过查看当前运行线程的组来猜测*您处于哪个逻辑端。 由于这是一种猜测，因此只有在用尽其他方法后才能使用这种方法。 几乎在所有情况下，您都应该首选检查 `Level#isClientSide`。

### `FMLEnvironment#dist` 和 `@OnlyIn`

`FMLEnvironment#dist`保存代码运行的**物理**端。 由于它是在启动时确定的，因此不依赖于猜测来返回结果。 不过，这种方法的使用场景有限。

使用`@OnlyIn(Dist)`注解对方法或字段进行注解时，会向加载器表明相应成员应完全从定义中剥离，而不是在指定的**物理**侧。 通常，只有在浏览反编译后的 Minecraft 代码时才能看到这些内容，它们表示 Mojang 混淆器剥离掉的方法。 **没有**直接使用此注解的理由。 请使用 `DistExecutor` 或检查 `FMLEnvironment#dist` 代替。

常见错误
---------------

### 跨越逻辑的端

无论您想从一个逻辑端向另一个逻辑端发送信息，您都必须**始终**使用网络数据包。 在单人游戏场景中，直接从逻辑服务器向逻辑客户端传输数据是非常诱人的。

这实际上经常通过静态字段在不经意间使用。 由于在单人游戏场景中，逻辑客户端和逻辑服务器共享同一个 JVM，因此写入和读取静态字段的两个线程都会导致各种竞争条件以及与线程相关的典型问题。

通过从在逻辑服务器上运行或可以在逻辑服务器上运行的普通代码中访问物理客户端专用类（如`Minecraft`），也可以显式地犯下这一错误。 对于在物理客户端中进行调试的初学者来说，这个错误很容易被忽略。 代码可以在物理客户端上运行，但在物理服务器上会立即崩溃。


编写单端mod
----------------------

在最近的版本中，Minecraft Forge 从 mods.toml 中移除了 "sidedness "属性。 这意味着无论在物理客户端还是物理服务端上加载，您的Mod都能正常工作。 因此，对于单端Mod，您通常会在 `DistExecutor#safeRunWhenOn` 或 `DistExecutor#safeRunWhenOn` 中注册事件处理程序，而不是直接调用Mod构造函数中的相关注册方法。 基本上，如果您的Mod在错误的一端加载，它应该什么也不做，不监听任何事件，等等。 单侧Mod本质上不应该注册方块、物品......，因为它们也需要在另一侧可用。

此外，如果您的 MOD 是单端的，那么它通常不会禁止用户加入缺少该 MOD 的服务器。 因此，您应该在[mods.toml][structuring]中将`displayTest`属性设置为任何必要的值。

```toml
[[mods]]
  # ...

  # MATCH_VERSION 的意思是，如果客户端和服务器端的版本不同，您的 MOD 将导致一个红色的 X。 这是默认行为，如果您的 MOD 同时包含服务端和客户端元素，则应选择此行为。
  # IGNORE_SERVER_VERSION 的意思是，如果您的 MOD 出现在服务器上，但不在客户端上，则不会导致红 X。 如果您的 MOD 仅在服务器上运行，则应使用此功能。
  # IGNORE_ALL_VERSION 表示如果您的 MOD 出现在客户端或服务器上，将不会导致红 X。 这是一种特殊情况，只有当您的 MOD 没有服务器组件时才能使用。
  # NONE 表示未在您的 MOD 上设置显示测试。 您需要自行设置，更多信息请参阅 IExtensionPoint.DisplayTest。 您可以使用此值定义任何您想要的方案。
  # IMPORTANT NOTE: 这不是关于您的模块在哪种环境（客户端或专用服务器）下加载的指示。 您的模块应加载（也许什么也不做！）到它能找到的任何地方。
  displayTest="IGNORE_ALL_VERSION" #  如果没有指定，则默认为MATCH_VERSION(可选的)
```

如果要使用自定义显示测试，则应将`displayTest`选项设置为`NONE`、 并应注册 `IExtensionPoint$DisplayTest` 扩展：

```java
//确保网络另一端不存在 mod 不会导致客户端将服务器显示为不兼容
ModLoadingContext.get().registerExtensionPoint(IExtensionPoint.DisplayTest.class, () -> new IExtensionPoint.DisplayTest(() -> NetworkConstants.IGNORESERVERONLY, (a, b) -> true));
```

这就告诉客户端，它应该忽略服务端Mod版本不存在的情况，而服务端则不应该告诉客户端该Mod应该存在。 因此，该代码段既适用于客户端Mod，也适用于服务器端Mod。


[invokedynamic]: https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-6.html#jvms-6.5.invokedynamic
[dist]: #fmlenvironmentdist-and-onlyin
[structuring]: ../gettingstarted/modfiles.md#modstoml
