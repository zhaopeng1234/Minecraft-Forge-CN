Mod生命周期
==============

在mod加载过程中，在mod专用事件总线上触发各种生命周期事件。在这些事件期间执行许多操作，例如[注册对象][registering]、准备[数据生成][datagen根]或[与其他mod通信][imc]。

事件侦听器应该使用`@EventBusSubscriber(bus = Bus.MOD)`或在mod构造函数中注册：

```Java
@Mod.EventBusSubscriber(modid = "mymod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class MyModEventSubscriber {
  @SubscribeEvent
  static void onCommonSetup(FMLCommonSetupEvent event) { ... }
}

@Mod("mymod")
public class MyMod {
  public MyMod() {
    FMLModLoadingContext.get().getModEventBus().addListener(this::onCommonSetup);
  } 

  private void onCommonSetup(FMLCommonSetupEvent event) { ... }
}
```

!!! 警告
    大多数生命周期事件是并行触发的：所有mod将同时接收相同的事件。Mod*必须*注意线程安全，例如调用其他mod的API或访问原版系统。通过`ParallelDispatchEvent#enqueueWork`延迟代码执行。
    

注册表事件
---------------
注册表事件在mod实例构造后触发。有三个：`NewRegistryEvent`、`DataPackRegistryEvent$NewRegistry`和`RegisterEvent`。这些事件在mod加载期间同步触发。

`NewRegistryEvent`允许开发者使用`RegistryBuilder`类注册自己的自定义注册表。

`DataPackRegistryEvent$NewRegistry`允许开发者通过提供`编解码器`来对JSON对象进行编码和解码，从而注册自定义数据包注册表。

`RegisterEvent`用于[注册对象][registering]到注册表中。每个注册表都会触发此事件。

数据生成
---------------

如果游戏设置为运行[数据生成器][datagen]，则`GatherDataEvent`将是最后触发的事件。此事件用于将mod的数据提供者注册到其关联的数据生成器。此事件触发也是同步的。


通用设置
------------

`FMLCommonSetupEvent`用于物理客户端和物理服务端共有的操作，例如注册[能力][capabilities]。


分端设置
-----------
分端设置的事件在各自对应的[物理端][sides]上触发：物理客户端上的`FMLClientSetupEvent`，专用服务端上的`FMLDedicatedServerSetupEvent`。这是物理端初始化应该发生的地方，例如注册客户端键绑定。


跨Mod通信InterModComms
-------------

这是消息可以发送到mod以实现跨mod兼容性的地方。有两个事件：`InterModEnqueueEvent`和`InterModProcessEvent`。

`InterModComms`是负责保存mod消息的类。在生命周期事件期间调用这些方法是安全的，因为它由`ConcurrentMap`支持。

在`InterModEnqueueEvent`期间，使用`InterModComms#sendTo`向不同的mod发送消息。这些方法接收将收到消息的mod id、与消息数据关联的key以及持有消息数据的supplier 。此外，还可以指定消息的发送者，但默认情况下它将是调用者的mod id。

然后在`InterModProcessEvent`期间，使用`InterModComms#getMessages`获取所有接收到的消息流。方法参数中的的mod id一般是发送消息的mod的mod id。此外，可以指定predicate来过滤消息的key。`#getMessages`方法将返回`IMCMessage`的流，其中保存数据的发送者、数据的接收者、数据key和提供的数据本身。


!!! 注意
	还有另外两个生命周期事件：`FMLConstructModEvent`，在mod实例构建之后但在`RegisterEvent`之前触发；`FMLLoadCompleteEvent`，在`InterModComms`事件之后，在于mod加载完成时触发。


[registering]: ./registries.md#注册的方法
[capabilities]: ../datastorage/capabilities.md
[datagen]: ../datagen/index.md
[imc]: ./lifecycle.md#跨Mod通信InterModComms
[sides]: ./sides.md
