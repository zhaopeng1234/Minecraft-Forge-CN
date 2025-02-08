## 面数据
在原版的 “元素（elements）” 模型中，关于元素面的额外数据可以在元素层级或面层级进行指定。未指定自身面数据的面将回退到元素的面数据，如果元素层级也未指定面数据，则使用默认值。

要在生成的物品模型中使用此扩展，由于原版物品模型生成器未扩展以读取这些额外数据，模型必须通过 `forge:item_layers` 模型加载器进行加载。

面数据的所有值都是可选的。

### 元素模型
在原版的 “元素” 模型中，面数据适用于指定该数据的面，或者适用于指定该数据的元素中所有没有自身面数据的面。

!!! 注意
    如果在一个面上指定了 `forge_data`，它将不会从元素层级的 `forge_data` 声明中继承任何参数。

额外数据可以通过以下示例中的两种方式进行指定：
```js
{
  "elements": [
    {
      "forge_data": {
        "color": "0xFFFF0000",
        "block_light": 15,
        "sky_light": 15,
        "ambient_occlusion": false
      },
      "faces": {
        "north": {
          "forge_data": {
            "color": "0xFFFF0000",
            "block_light": 15,
            "sky_light": 15,
            "ambient_occlusion": false
          },
          // ...
        },
        // ...
      },
      // ...
    }
  ]
}
```

### 生成的物品模型
在使用 `forge:item_layers` 加载器生成的物品模型中，面数据是为每个纹理层指定的，并适用于所有几何图形（正面/背面四边形和边缘四边形）。

`forge_data` 字段必须位于模型 JSON 的顶级，每个键值对将一个面数据对象与一个图层索引关联起来。

在以下示例中，图层 1 将被染成红色并以全亮度发光：
```js
{
  "textures": {
    "layer0": "minecraft:item/stick",
    "layer1": "minecraft:item/glowstone_dust"
  },
  "forge_data": {
    "1": {
      "color": "0xFFFF0000",
      "block_light": 15,
      "sky_light": 15,
      "ambient_occlusion": false
    }
  }
}
```

### 参数

#### 颜色
使用 `color` 条目指定颜色值将把该颜色作为色调应用到四边形上。默认值为 `0xFFFFFFFF`（白色，完全不透明）。颜色必须采用 `ARGB` 格式并打包成 32 位整数，可以指定为十六进制字符串（`"0xAARRGGBB"`）或十进制整数字面量（JSON 不支持十六进制整数字面量）。

!!! 警告
    四个颜色分量会与纹理的像素相乘。省略 alpha 分量相当于将其设为 0，这会使几何图形完全透明。

如果颜色值是固定的，这可以替代使用 [`BlockColor` 和 `ItemColor`][tinting] 进行染色。

#### 方块光照和天空光照
分别使用 `block_light` 和 `sky_light` 条目指定方块光照和/或天空光照值将覆盖四边形的相应光照值。两个值的默认值均为 0。这些值必须在 0 - 15（包含）的范围内，并且在渲染面时被视为相应光照类型的最小值，这意味着世界中相应光照类型的更高值将覆盖指定的值。

指定的光照值纯粹是客户端的，既不会影响服务器的光照等级，也不会影响周围方块的亮度。

#### 环境光遮蔽
指定 `ambient_occlusion` 标志将为四边形配置 [环境光遮蔽（AO）]。默认值为 `true`。此标志的行为等同于原版格式的顶级 `ambientocclusion` 标志。

![环境光遮蔽效果][ao_img]  
*左侧启用了环境光遮蔽，右侧禁用了环境光遮蔽，通过平滑光照图形设置进行演示*

!!! 注意
    如果顶级 AO 标志设置为 `false`，在元素或面上将此标志指定为 `true` 无法覆盖顶级标志。
    ```js
    {
      "ambientocclusion": false,
      "elements": [
        {
          "forge_data": {
            "ambient_occlusion": true // 无效
          }
        }
      ]
    }
    ```

[tinting]: ../../resources/client/models/tinting.md
[AO]: https://en.wikipedia.org/wiki/Ambient_occlusion
[ao_img]: ./ambientocclusion_annotated.png
