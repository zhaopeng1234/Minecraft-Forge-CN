构建你的mod
====================

让我们看看如何在不同的文件中编写你的mod，以及这些文件应该做什么。   

软件包
---------

选择一个独特的软件包名称。如果你拥有一个与你的项目相关的URL，你可以使用它作为你的顶级包的名称。例如，如果你拥有 "example.com"，你可以使用`com.example`作为你的顶级包的名称。

**如果你不拥有某个域名，不要用它来做你的顶级包名。你可以使用你的电子邮件，你托管网站的一个子域，或者你的名字/用户名，只要它能够是独一无二的。**   

在顶级包名之后，为你的mod附加一个独特的名字，比如`examplemod`。在这个例子中，最终会是`com.example.examplemod`。

`mods.toml` 文件
-------------------

**mods.toml中的许可证是必需的。如果没有，会发生错误。请在https://choosealicense.com/ 查看您的选择。**

这个文件定义了你的MOD的元数据。玩家可以在游戏的主屏幕上通过 "Mods"按钮查看其信息。一个信息文件可以描述多个MOD。  

`mods.toml`文件的格式为 [TOML][], MDK中的`mods.toml`文件示例包含了该文件内容的解释。它应该被保存在`src/main/resources/META-INF/mods.toml`。一个基本的`mods.toml`，描述了一个mod，可能看起来像这样：
```toml
# 要加载的mod loader类型 - 对于常规的FML @Mod mods，它应该是javafml。
modLoader="javafml"
# 一个与上述mod加载器相匹配的版本范围--对于常规的FML @Mod来说，它将是forge版本。
# minecraft 1.18对应的Forge版本是38
loaderVersion="[38,)"
# 你的mod的许可证。这是强制性的，可以使你的再分发策略更容易被理解。
# 在https://choosealicense.com/查看你的选择。保留所有权利(All Rights Reserved)是默认的许可证，也是这里的默认值。
license="All Rights Reserved"
# 当这个mod出现问题时，玩家应该访问的网址
issueTrackerURL="github.com/MinecraftForge/MinecraftForge/issues"
# 在这个文件中定义的mod是否应该显示为独立的资源包
showAsResourcePack=false

[[mods]]
  modId="examplemod"
  version="1.0.0.0"
  displayName="Example Mod"
  updateJSONURL="minecraftforge.net/versions.json"
  displayURL="minecraftforge.net"
  logoFile="logo.png"
  credits="I'd like to thank my mother and father."
  authors="Author"
  description='''
  Lets you craft dirt into diamonds. This is a traditional mod that has existed for eons. It is ancient. The holy Notch created it. Jeb rainbowfied it. Dinnerbone made it upside down. Etc.
  '''

  [[dependencies.examplemod]]
    modId="forge"
    mandatory=true
    versionRange="[38,)"
    ordering="NONE"
    side="BOTH"

  [[dependencies.examplemod]]
    modId="minecraft"
    mandatory=true
    versionRange="[1.18,1.19)"
    ordering="NONE"
    side="BOTH"
```

如果任何字符串的值被设为`${file.jarVersion}`，Forge将在运行时用你jar manifest中指定的**Implementation Version**字符串替换。由于用户开发环境无法获取jar清单，它将被`NONE`代替。因此，我们通常建议不使用`version`字段。下面是一个可能被赋予mod的属性表，其中`mandatory`意味着是必要字段，没有该属性会导致错误。

|     Property |   Type   | Default  | Description |
|-------------:|:--------:|:--------:|:------------|
|        modid |  string  | mandatory | 这个文件所链接的modid。 |
|      version |  string  | mandatory | MOD的版本。它应该只是由点分隔的数字，最好符合Forge的[Semantic Versioning][versioning]结构。 |
|  displayName |  string  | mandatory | 这个MOD对玩家显示的名称。 |
| updateJSONURL |  string  |   `""`   | mod的[version JSON][updatechecker]地址，用于检查更新。 |
|   displayURL |  string  |   `""`   | MOD主页的链接。 |
|     logoFile |  string  |   `""`   | MOD的logo的文件名。它必须放在资源文件夹的根目录而不是子文件夹中。 |
|      credits |  string  |   `""`   | 一个包含任何你想提及的致谢人的字符串。 |
|      authors |  string  |   `""`   | 这个mod的作者们。 |
|  description |  string  | mandatory | 这个mod的描述 |
| dependencies | [list] |   `[]`   | 这个mod的相关依赖 |

\* 所有的版本范围描述都使用 [Maven Version Range Specification][mvr].

mod文件
------------

一般来说，我们将从你的软件包根目录的一个和mod名称相同的java文件开始。这是你的mod的*入口点*，并且将用一些特殊的标识来标记它。

什么是 `@Mod`?
-------------

这个注解向Forge Mod Loader表明该类是一个Mod入口点。`@Mod`注解的值应该与`src/main/resources/META-INF/mods.toml`文件中的mod id匹配。

使用子软件包保持你的代码清洁
------------------------------------------

与其把所有的东西都塞进一个类和包里，建议你把你的mod分割成多个子包。

一个常见的子包策略分为`common`和`client`代码包，它们的含义是这些代码可以在服务器/客户端运行，或者只在客户端运行。在`common`包内会有Item、Blocks和Block Entities等东西（它们也可以成为另一个子包）。像界面和渲染器这样的东西会被放在`client`包里。

!!! 注意
    这种软件包命名风格只是一种建议，尽管它是一种常用的风格。你可以自由使用你自己的软件包命名风格。

通过将你的代码保持在干净的子包中，你可以更方便地扩展你的mod。

类的命名方式
--------------------

一个通用的类的命名方案可以更容易地解读一个类是什么，它也使和你一起开发mod的人更容易找到东西。

比如：

* 一个叫做`PowerRing`的`Item`放在一个`item`包中、类的名称为`PowerRingItem`。
* 一个叫做`NotDirt`的`Block`放在一个`block`包中、类的名称为`NotDirtBlock`。
* 最后、对于`SuperChewer`方块的一个`BlockEntity`来说，放在一个`block.entity`或`blockentity`包中，类的名称是`SuperChewerBlockEntity`。

在你的类名后面加上它们的*种类*后缀，使你更容易弄清一个类是什么或猜测一个物品的类名。

[TOML]: https://github.com/toml-lang/toml
[versioning]: ./versioning.md
[updatechecker]: ../misc/updatechecker.md
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
