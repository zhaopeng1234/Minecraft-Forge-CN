## 渲染类型

在 JSON 文件的顶级添加 `render_type` 条目，可向加载器表明该模型应使用的渲染类型。如果未指定，加载器将自行选择使用的渲染类型，通常会回退到 `ItemBlockRenderTypes#getRenderLayers()` 返回的渲染类型。

自定义模型加载器可以完全忽略此字段。

!!! 注意
    从 1.19 版本开始，对于方块，推荐使用此方法，而不是通过 `ItemBlockRenderTypes#setRenderLayer()` 设置适用渲染类型的已弃用方法。

以下是一个使用玻璃纹理的镂空方块模型示例：

```js
{
  "render_type": "minecraft:cutout",
  "parent": "block/cube_all",
  "textures": {
    "all": "block/glass"
  }
}
```

### 原版值
Forge 提供了以下带有相应区块和实体渲染类型的选项（`NamedRenderTypeManager#preRegisterVanillaRenderTypes()`）：

- `minecraft:solid`
    - 区块渲染类型：`RenderType#solid()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_SOLID`
    - 用于完全实心的方块（例如：石头）
- `minecraft:cutout`
    - 区块渲染类型：`RenderType#cutout()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_CUTOUT`
    - 用于任何给定像素要么完全透明要么完全不透明的方块（例如：玻璃方块）
- `minecraft:cutout_mipped`
    - 区块渲染类型：`RenderType#cutoutMipped()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_CUTOUT`
    - 由于实体渲染类型上的 mipmap 会使物品看起来怪异，因此区块和实体渲染类型不同
    - 用于任何给定像素要么完全透明要么完全不透明，并且纹理在较大距离时应缩小（[mipmapping]）以避免视觉瑕疵的方块（例如：树叶）
- `minecraft:cutout_mipped_all`
    - 区块渲染类型：`RenderType#cutoutMipped()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_CUTOUT_MIPPED`
    - 用于与 `minecraft:cutout_mipped` 类似的情况，当物品表示也应应用 mipmapping 时
- `minecraft:translucent`
    - 区块渲染类型：`RenderType#translucent()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_TRANSLUCENT`
    - 用于任何给定像素可能部分透明的方块（例如：染色玻璃）
- `minecraft:tripwire`
    - 区块渲染类型：`RenderType#tripwire()`
    - 实体渲染类型：`ForgeRenderTypes#ITEM_LAYERED_TRANSLUCENT`
    - 由于绊线渲染类型不能作为实体渲染类型使用，因此区块和实体渲染类型不同
    - 用于有特殊要求，需要渲染到天气渲染目标的方块（例如：绊线）

### 自定义值
可以在 `RegisterNamedRenderTypesEvent` 中注册要在模型中指定的自定义命名渲染类型。此事件在模组事件总线上触发。

自定义命名渲染类型由两到三个组件组成：
- 一个区块渲染类型 - 可以使用 `RenderType.chunkBufferLayers()` 返回列表中的任何类型
- 一个使用 `DefaultVertexFormat.NEW_ENTITY` 顶点格式的渲染类型（“实体渲染类型”）
- 一个在选择 *华丽！* 图形模式时使用的、使用 `DefaultVertexFormat.NEW_ENTITY` 顶点格式的渲染类型（可选）

当使用此命名渲染类型的方块作为区块几何的一部分进行渲染时，将使用区块渲染类型。
当使用此命名渲染类型的物品在快速和精美图形模式下（如在物品栏、地面、物品展示框等中）渲染时，将使用必需的实体渲染类型。
当选择 *华丽！* 图形模式时，可选的实体渲染类型的使用方式与必需的实体渲染类型相同。当必需的实体渲染类型在 *华丽！* 图形模式下不起作用时（通常仅适用于半透明渲染类型），则需要此渲染类型。

```java
public static void onRegisterNamedRenderTypes(RegisterNamedRenderTypesEvent event)
{
  event.register("special_cutout", RenderType.cutout(), Sheets.cutoutBlockSheet());
  event.register("special_translucent", RenderType.translucent(), Sheets.translucentCullBlockSheet(), Sheets.translucentItemSheet());
}
```

然后可以在 JSON 中通过 `<你的模组 ID>:special_cutout` 和 `<你的模组 ID>:special_translucent` 来引用这些自定义渲染类型。

[mipmapping]: https://en.wikipedia.org/wiki/Mipmap
