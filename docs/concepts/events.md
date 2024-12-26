事件
======

Forge使用事件总线，允许mod拦截来自各种原版和mod行为的事件。

示例：当右键单击原版木棒时，可以使用事件来执行操作。

用于大多数事件的主事件总线位于`MinecraftForge#EVENT_BUS`。还有另一个mod专用事件总线位于`FMLJavaModLoadingContext#getModEventBus`您应该只在特定情况下使用。有关此总线的更多信息可以在下面找到。


每个事件都在这些总线上触发：大多数事件在forge主事件总线上触发，但有些事件在mod专用事件总线上触发。

事件处理器是已注册到事件总线的一系列方法。

创建事件处理器(Event Handler)
-------------------------

事件处理器方法只有一个参数，不返回结果。该方法可以是静态的，也可以是实例的，具体取决于实现。

事件处理器可以使用`IEventBus#addListener`或用于泛型事件的`IEventBus#addGenericListener`（由继承`GenericEvent<T>`的子类表示）注册。两种添加事件处理器的方法都接受表示方法引用的consumer。泛型事件处理程序需要指定泛型的类。事件处理器必须在mod主类的构造函数中注册。

```java
// 在ExampleMod的主mod类中

// 此事件在mod总线上
private void modEventHandler(RegisterEvent event) {
	// ...
}

// 这个事件在Forge总线上
private static void forgeEventHandler(AttachCapabilitiesEvent<Entity> event) {
	// ...
}

// 在mod构造函数中
modEventBus.addListener(this::modEventHandler);
forgeEventBus.addGenericListener(Entity.class, ExampleMod::forgeEventHandler);
```

### 实例中的事件处理器注解

此事件处理器侦听`EntityItemPickupEvent`，正如名称所述，每当`实体`拾取物品时，该事件就会发布到事件总线。

```java
public class MyForgeEventHandler {
	@SubscribeEvent
	public void pickupItem(EntityItemPickupEvent event) {
		System.out.println("Item picked up!");
	}
}
```

要注册此事件处理器，请使用`MinecraftForge.EVENT_BUS.Register(…)`并将事件处理器所在类的实例传递给它。如果要将此处理程序注册到mod专用事件总线，则应使用`FMLJavaModLoadingContext.get().getModEventBus().Register(...)`。

### 静态类中的事件处理器注解

事件处理器也可能是静态的。处理方法仍然用`@SubscribeEvent`注释。与实例处理器的唯一区别是它被标记`static`。为了注册静态事件处理器，不能使用类的实例。`类`本身必须传入。一个例子：

```java
public class MyStaticForgeEventHandler {
	@SubscribeEvent
	public static void arrowNocked(ArrowNockEvent event) {
		System.out.println("Arrow nocked!");
	}
}
```

必须像这样注册：`MinecraftForge.EVENT_BUS.register(MyStaticForgeEventHandler.class)`.

### 自动注册静态事件处理程序

一个类可以使用`@Mod$EventBusSubscriber`注解。当`@Mod`类本身构造时，这样的类会自动注册到`MinecraftForge#EVENT_BUS`。这本质上相当于添加`MinecraftForge.EVENT_BUS.Register(AnnotatedClass.class);`到`@Mod`类构造函数的末尾。

您可以将要监听的总线传给`@Mod$EventBusSubscriber`注解。建议您指定modid，因为注解处理器可能无法弄清楚。以及您注册的总线，因为它可以提醒您确保您在正确的总线上。您还可以指定`Dist`或物理端来加载此事件订阅者。这可用于在专用服务器上不加载客户端事件订阅者。

静态事件侦听器侦听`RenderLevelStageEvent`的示例，它只会在客户端上调用：

```java
@Mod.EventBusSubscriber(modid = "mymod", bus = Bus.FORGE, value = Dist.CLIENT)
public class MyStaticClientOnlyEventHandler {
	@SubscribeEvent
	public static void drawLast(RenderLevelStageEvent event) {
		System.out.println("Drawing!");
	}
}
```

!!! 注意
    这不会注册类的实例；它注册类本身（即事件处理方法必须是静态的）。

取消
---------
如果一个事件可以被取消，它将被标记为`@Cancelable`注解，并且`Event#isCancelable()`方法将返回`true`。可以通过调用`Event#setCanceled(boolean canceled)`来修改可取消事件的取消状态，其中传递布尔值`true`被解释为取消事件，传递布尔值`false`被解释为“取消”事件。但是，如果事件不能被取消（如事件`#isCancelable()`）所定义的，无论传递的布尔值如何，都会抛出`UnsupportedOperationException`，因为不可取消事件事件的取消状态被认为是不可变的。

!!! 重要
    并非所有事件都可以取消！尝试取消不可取消的事件将导致抛出未经检查的`UnsupportedOperationException`，这将导致游戏崩溃！在尝试取消之前，请始终使用`Event#isCancelable()`检查是否可以取消事件！

结果
-------

有些事件有`Event$Result`。结果可以是以下三种情况之一：`DENY`停止事件，`DEFAULT`使用原版行为，`ALLOW`强制动作发生，无论它原本是否会发生。事件的结果可以通过在事件上使用`Event$Result`调用`#setResult`来设置。并非所有事件都有结果；带有结果的事件将使用`@HasResult`注释。

!!! 重要
    不同的事件可能以不同的方式使用Result，在使用Result之前参考事件的JavaDoc。
    
优先级
--------

事件处理器方法（使用`@SubscribeEvent`标记的）具有优先级。您可以通过设置注解的`priority`值来设置事件处理器方法的优先级。优先级可以是`EventPriority`枚举的任何值（`HIGHEST`、`HIGH`、`NORMAL`、`LOW`和`LOWEST`）。具有优先级`HIGHEST`的事件处理器首先执行，然后按降序执行，直到`LOWEST`事件。


子事件
----------

许多事件都有自己的不同变体。它们可以不同，但都基于一个公共因子（例如`PlayerEvent`），也可以是具有多个阶段的事件（例如`PotionBrewEvent`）。请注意，如果您侦听父事件类，您将收到对*所有*子类的方法的调用。

Mod事件总线
-------------

mod事件总线主要用于监听mod初始化的生命周期事件。mod总线上的每个事件都需要实现`IModBusEvent`。其中许多事件也同时进行，因此mod可以同时初始化。这确实意味着您不能在这些事件中直接执行来自其他mod的代码。为此您应该使用`InterModComms`系统。
这些是在mod事件总线上的mod初始化期间调用的四个最常用的生命周期事件：

* `FMLCommonSetupEvent`
* `FMLClientSetupEvent` & `FMLDedicatedServerSetupEvent`
* `InterModEnqueueEvent`
* `InterModProcessEvent`

!!! 注意
    `FMLClientSetupEvent`和`FMLDedicatedServerSetupEvent`仅在各自的端上调用。

这四个生命周期事件都是并行运行的，因为它们都是`ParallelDispatchEvent`的子类。如果您想在`ParallelDispatchEvent`执行期间在主线程上运行代码，可以使用`#enqueueWork`。

在生命周期事件旁边，还有一些在mod事件总线上触发的杂项事件，您可以在其中注册、设置或初始化各种事物。与生命周期事件相比，这些事件中的大多数不是并行运行的。几个例子：

* `RegisterColorHandlersEvent`
* `ModelEvent$BakingCompleted`
* `TextureStitchEvent`
* `RegisterEvent`

一个很好的经验法则：如果在mod初始化期间应该处理事件时，该事件就会在mod事件总线上触发。
