## 自定义模型加载器
“模型” 简单来说就是一种形状。它可以是一个简单的立方体，也可以是多个立方体，可以是一个截角二十面体，或者介于两者之间的任何形状。你看到的大多数模型都是原版的 JSON 格式。其他格式的模型在运行时会由 `IGeometryLoader` 加载到 `IUnbakedGeometry` 中。Forge 为 WaveFront OBJ 文件、桶、复合模型、不同渲染层的模型以及对原版 `builtin/generated` 物品模型的重新实现提供了默认实现。大多数情况下，代码并不关心模型是由什么加载的或者是什么格式，因为它们最终在代码中都由 `BakedModel` 表示。

!!! 警告
    在模型 JSON 文件的顶级通过 `loader` 条目指定自定义模型加载器时，`elements` 条目将被忽略，除非自定义加载器对其进行处理。所有其他原版条目仍会被加载，并且在未烘焙的 `BlockModel` 表示中可用，也可能会在自定义加载器之外被使用。

### WaveFront OBJ 模型
Forge 为 `.obj` 文件格式添加了一个加载器。要使用这些模型，JSON 文件必须引用 `forge:obj` 加载器。这个加载器接受任何在已注册命名空间中且路径以 `.obj` 结尾的模型位置。`.mtl` 文件应与 `.obj` 文件放在同一位置且名称相同，这样就会被自动使用。可能需要手动编辑 `.mtl` 文件，以更改其中指向 JSON 中定义的纹理的路径。此外，根据创建模型的外部程序，纹理的 V 轴可能会翻转（例如，V = 0 可能是底边，而不是顶边）。这可以在建模程序中进行修正，也可以在模型 JSON 中这样处理：

```js
{
  // 在 'model' 声明的同一层级添加以下行
  "loader": "forge:obj",
  "flip_v": true,
  "model": "examplemod:models/block/model.obj",
  "textures": {
    // 可以在 .mtl 中使用 #texture0 引用
    "texture0": "minecraft:block/dirt",
    "particle": "minecraft:block/dirt"
  }
}
```
