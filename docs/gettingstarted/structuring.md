结构化你的Mod
====================

结构化的mod有利于维护、贡献和更清晰地了解底层代码库。 下面列出了 Java、Minecraft 和 Forge 的一些建议。

!!! 注意
    您并不一定要遵循以下建议；您可以以任何您认为合适的方式构建您的Mod。 不过，我们还是强烈建议您使用以下做法。

软件包管理
---------

在构建您的Mod时，请选择一个独一无二的顶层软件包结构。 许多程序员会为不同的类、接口等使用相同的包、名称。 Java 允许类具有相同的名称，只要它们在不同的包中。 因此，如果两个类有相同的包名和类名，则只会加载其中一个，很可能导致游戏崩溃。

```
a.jar
  - com.example.ExampleClass
b.jar
  - com.example.ExampleClass // 该类不会被正常加载
```

这一点在加载Mod的子模块时更为重要。 如果在两个不同模块中的两个包有相同的包名，这将导致模块加载器**在启动时崩溃**，因为模块会被导出到游戏和其他Mod。
```
module A
  - package X
    - class I
    - class J
module B
  - package X // 这个软件包会导致模块加载器崩溃，因为已经有一个 X 软件包的模块被导出了
    - class R
    - class S
    - class T
```

因此，您的顶级软件包名应该是您自己的东西：域名、电子邮件地址、网站的子域名等。甚至可以是你的名字或用户名，只要你能保证它在预期目标中是唯一的。

Type      | Value             | Top-Level Package
:---:     | :---:             | :---
域名| example.com       | `com.example`
网站的子域名| example.github.io | `io.github.example`
电子邮件地址     | example@gmail.com | `com.gmail.example`

下一级软件包应该是您的modId（例如，`com.example.examplemod`，其中`examplemod`是modId）。 这将确保，除非您有两个具有相同 ID 的Mod（绝对不会出现这种情况），否则您的软件包在加载时不会出现任何问题。

您可以在[Oracle 教程页面][naming].中找到一些其他命名规范。

### 子软件包管理

除了顶层软件包外，我们还强烈建议将Mod的类拆分到子软件包中。 有两种主要的拆分方法：

* **按功能分组**： 为具有共同目的的类创建子包。 例如，块可以放在 `block` 或 `blocks` 下、 实体可以放在`entity` 或 `entities` 下等。 Mojang 使用单数形式命名这些包。
* **按逻辑分组**： 为具有共同逻辑的类创建子包。 例如，如果您要创建一种新的合成台，您可以将它的方块、菜单、物品等放在 `feature.crafting_table`. 下。

#### 客户端、服务器和数据包

一般来说，仅用于特定端或运行环境的代码应与其他类隔离，放在单独的子包中。 例如，与 [数据生成][datagen] 相关的代码应放在 `data` 包中，而仅用于专用服务器的代码应放在 `server` 包中。
然而，强烈建议[客户端专用代码][sides]应隔离在`client`子包中。 这是因为Minecraft 中，服务器无法访问任何 客户端专用软件包。因此，使用专门的软件包将提供一个适当的检查，以验证您是否在您的Mod中进行了跨端操作。

类命名规则
--------------------

通用的类命名方案可以更容易地解读类的目的或找到特定的类。

例如，类通常以其类型作为后缀：

* 一个叫`PowerRing`的`Item` -> `PowerRingItem`.
* 一个叫`NotDirt`的`Block` -> `NotDirtBlock`.
* `Oven`的菜单 -> `OvenMenu`.

!!! 注意
    除实体外，Mojang 的所有类名通常都采用类似的结构。实体只用名称表示（例如：`Pig`、`Zombie`等）。

从多种方法中选择一种
---------------------------

执行特定任务（注册对象、监听事件等）的方法有很多。 一般建议使用单一方法完成特定任务，以保持一致性。 这样做不仅能改善代码格式，还能避免可能出现的任何奇怪的交互或冗余（例如，事件监听器重复执行）。

[naming]: https://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html
[datagen]: ../datagen/index.md
[sides]: ../concepts/sides.md
