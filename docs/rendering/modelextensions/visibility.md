## 部件可见性

在模型 JSON 文件的顶级添加 `visibility` 条目，可以控制模型不同部分的可见性，从而决定它们是否应该被烘焙到最终的 [`BakedModel`][bakedmodel] 中。“部件” 的定义取决于加载此模型的模型加载器，自定义模型加载器可以完全忽略此条目。在 Forge 提供的模型加载器中，只有 [复合模型加载器][composite] 和 [OBJ 模型加载器][obj] 使用了此功能。可见性条目以 `"部件名称": 布尔值` 的形式指定。

以下是一个复合模型的示例，该模型有两个部件，其中第二个部件不会被烘焙到最终模型中，还有两个子模型覆盖了此可见性设置，分别使第一个部件可见和两个部件都可见：

```js
// mycompositemodel.json
{
  "loader": "forge:composite",
  "children": {
    "part_one": {
      "parent": "mymod:mypartmodel_one"
    },
    "part_two": {
      "parent": "mymod:mypartmodel_two"
    }
  },
  "visibility": {
    "part_two": false
  }
}

// mycompositechild_one.json
{
  "parent": "mymod:mycompositemodel",
  "visibility": {
    "part_one": false,
    "part_two": true
  }
}

// mycompositechild_two.json
{
  "parent": "mymod:mycompositemodel",
  "visibility": {
    "part_two": true
  }
}
```

给定部件的可见性通过以下方式确定：首先检查模型是否为此部件指定了可见性，如果没有，则递归检查模型的父模型，直到找到相关条目或没有更多父模型可检查，在没有找到条目的情况下，默认值为 `true`。

这允许进行如下设置：多个模型使用单个复合模型的不同部件。

1. 一个复合模型指定多个组件。
2. 多个模型将此复合模型指定为其父模型。
3. 这些子模型分别为部件指定不同的可见性。

[bakedmodel]: ../modelloaders/bakedmodel.md
[composite]: ../modelloaders/index.md/#composite-models
[obj]: ../modelloaders/index.md/#wavefront-obj-models
