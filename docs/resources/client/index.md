## 资源包
[资源包][respack] 允许通过 `assets` 目录对客户端资源进行自定义。这包括纹理、模型、音效、本地化文件等。你的模组（以及 Forge 本身）也可以拥有资源包。因此，任何用户都可以修改此目录中定义的所有纹理、模型和其他资源。

### 创建资源包
资源包存储在项目的资源文件夹中。`assets` 目录包含资源包的内容，而资源包本身由 `assets` 文件夹旁的 `pack.mcmeta` 文件定义。
由于你可以添加或修改已有的资源包，如原版、Forge 或其他模组的资源包，所以你的模组可以有多个资源域。
然后，你可以按照 [Minecraft 维基百科上的步骤][createrespack] 创建任何资源包。

更多阅读：[资源位置][resourcelocation]

[respack]: https://minecraft.wiki/w/Resource_Pack
[createrespack]: https://minecraft.wiki/w/Tutorials/Creating_a_resource_pack
[resourcelocation]: ../../concepts/resources.md#资源位置
