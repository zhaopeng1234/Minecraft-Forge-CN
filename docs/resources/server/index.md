## 数据包

在1.13版本中，Mojang将[数据包][datapack]添加到了基础游戏中。它们允许通过`data`目录修改逻辑服务器的文件。这涵盖了进度、战利品表、结构、配方、标签等等。Forge以及你的模组同样可以拥有数据包。因此，任何用户都能够修改此目录中定义的所有配方、战利品表以及其他数据。

### 创建数据包
数据包存储在项目资源中的`data`目录内。
由于你可以添加或修改已有的数据包，如原版、Forge或其他模组的数据包，所以你的模组可以拥有多个数据域。
接着，你可以按照[此处][createdatapack]的步骤来创建任何数据包。

更多阅读：[资源位置][resourcelocation]

[datapack]: https://minecraft.wiki/w/Data_pack
[createdatapack]: https://minecraft.wiki/w/Tutorials/Creating_a_data_pack
[resourcelocation]: ../../concepts/resources.md#ResourceLocation
