## 模型
[模型系统][models] 是Minecraft赋予方块和物品形状的方式。通过模型系统，方块和物品被映射到它们各自的模型上，这些模型定义了它们的外观。模型系统的主要目标之一，不仅是允许通过资源包更改纹理，还能改变方块或物品的整体形状。实际上，任何添加物品或方块的模组，也都包含一个针对其方块和物品的迷你资源包。

### 模型文件
模型和纹理通过[`ResourceLocation`][resloc]相互关联，但在`ModelManager`中使用`ModelResourceLocation`进行存储。模型会根据其是引用[方块状态模型][statemodel]还是[物品模型][itemmodels]，通过方块或物品的注册名，在不同位置被引用。方块的`ModelResourceLocation`会表示其注册名以及当前[`BlockState`][state]的字符串化版本，而物品则使用其注册名加上`inventory`。

!!! 注意
    JSON模型仅支持长方体元素；无法表示三角楔体或类似的形状。若要创建更复杂的模型，必须使用其他格式。

### 纹理
与模型一样，纹理也包含在资源包中，并通过`ResourceLocation`进行引用。在Minecraft中，[UV坐标][uv]（0, 0）表示**左上角**。UV坐标*始终*在0到16之间。如果纹理更大或更小，坐标会按比例缩放以适配。纹理也应该是正方形，并且纹理的边长应为2的幂次方，否则会破坏mipmapping（例如，1x1、2x2、8x8、16x16和128x128是合适的。5x5和30x30不建议使用，因为它们不是2的幂次方。5x10和4x8完全不可用，因为它们不是正方形）。仅当纹理是[动画纹理][animated]时，才可以不是正方形。

[models]: https://minecraft.wiki/w/Tutorials/Models#File_path
[resloc]: ../../../concepts/resources.md#resourcelocation
[statemodel]: https://minecraft.wiki/w/Tutorials/Models#Block_states
[itemmodels]: https://minecraft.wiki/w/Tutorials/Models#Item_models
[state]: ../../../blocks/states.md
[uv]: https://en.wikipedia.org/wiki/UV_mapping
[animated]: https://minecraft.wiki/w/Resource_Pack?so=search#Animation
