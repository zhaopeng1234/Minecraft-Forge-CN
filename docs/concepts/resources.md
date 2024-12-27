
资源
=========
资源是游戏使用的额外数据，存储在数据文件中，而不是在代码中。
Minecraft有两个主要资源系统：一个在逻辑客户端上，用于视觉效果，如模型、纹理和本地化，称为`资产(assets)`，另一个在逻辑服务器上，用于游戏，如食谱和战利品表，称为`数据(data)`。 [资源包][respack]控制前者，而[数据包][datapack]控制后者。

在默认的mod开发工具包中，资产和数据的目录位于项目的`src/main/resources`目录下。

启用多个资源包或数据包时，它们会被合并。通常，堆栈顶部包中的文件会覆盖下面的文件；但是，对于某些文件，例如本地化文件和标签，数据实际上是按内容合并的。mods在其`resources`目录中定义资源和数据包，但它们被视为“Mod资源”包的子集。Mod资源包不能被禁用，但它们可以被其他资源包覆盖。可以使用原版的`/datapack`命令禁用Mod数据包。

所有资源都应该有蛇形书写的路径和文件名（小写，单词边界使用“_”），这在1.11及更高版本中强制执行。

资源位置resourcelocation
------------------
Minecraft使用`ResourceLocation`来标识资源。`ResourceLocation`包含两个部分：命名空间和路径。它通常指向`assets/<namespace>/<ctx>/<path>`中的资源，其中`ctx`是一个特定于上下文的路径片段，它取决于`ResourceLocation`是如何被使用的。当`ResourceLocation`被写入/读取为来自字符串时，它被视为`<namespace>:<path>`。如果命名空间和冒号被省略，那么当字符串被读入`ResourceLocation`时命名空间将始终默认为`"minecraft"`。mod应该将其资源放入与其mod id同名的命名空间（例如，id为`examplemod`的mod应该将其资源分别放在`assets/examplemod`和`data/examplemod`中，`ResourceLocation`指向这些文件时类似`examplemod:<path>`）。这不是必需的，在某些情况下，可能需要使用不同的（甚至多个）命名空间。`ResourceLocation`也在资源系统之外使用，因为它们碰巧是唯一标识对象的好方法（例如[注册表][registries]）。


[respack]: ../resources/client/index.md
[datapack]: ../resources/server/index.md
[registries]: ./registries.md
