## `BakedModel`

`BakedModel` 是调用原版模型加载器的 `UnbakedModel#bake` 方法，或者自定义模型加载器的 `IUnbakedGeometry#bake` 方法的结果。与 `UnbakedModel` 或 `IUnbakedGeometry` 不同，后两者纯粹表示一种形状，不涉及物品或方块的概念，而 `BakedModel` 没那么抽象。它表示已经过优化并简化为（几乎）可以直接送往 GPU 处理的几何图形。它还可以处理物品或方块的状态以改变模型。

在大多数情况下，并不需要手动实现这个接口。可以选择使用现有的实现之一。

### `getOverrides`

返回此模型要使用的 [`ItemOverrides`][overrides]。只有当此模型作为物品进行渲染时，才会用到这个方法。

### `useAmbientOcclusion`

如果该模型在游戏世界中作为方块进行渲染，且相关方块不发光，同时环境光遮蔽功能已启用，那么这个模型会以 [环境光遮蔽](ambocc) 的效果进行渲染。

### `isGui3d`

如果该模型作为物品在物品栏、作为实体在地面上、在物品展示框等场景中渲染，此方法返回的结果会使模型看起来 “扁平”。在图形用户界面（GUI）中，这也会禁用光照效果。

### `isCustomRenderer`

!!! 重要
    除非你清楚自己在做什么，否则直接让此方法 `return false` 并继续即可。

当将此模型作为物品进行渲染时，如果返回 `true`，模型将不会被渲染，而是会转而使用 `BlockEntityWithoutLevelRenderer#renderByItem` 方法。对于某些原版物品，如箱子和旗帜，此方法会被硬编码为将物品的数据复制到一个 `BlockEntity` 中，然后使用 `BlockEntityRenderer` 来渲染该方块实体，以此替代物品的渲染。对于其他所有物品，它将使用 `IClientItemExtensions#getCustomRenderer` 提供的 `BlockEntityWithoutLevelRenderer` 实例。更多信息请参考 [BlockEntityWithoutLevelRenderer][bewlr] 页面。

### `getParticleIcon`

该方法用于指定粒子应使用的纹理。对于方块来说，当实体落在上面、方块被破坏等情况时会显示此纹理。对于物品来说，在物品被破坏或被食用时会显示。

!!! 重要
    没有参数的原版方法已被弃用，建议使用 `#getParticleIcon(ModelData)`，因为模型数据可能会影响特定模型的渲染方式。

### <s>`getTransforms`</s>

此方法已被弃用，建议实现 `#applyTransform` 方法。如果实现了 `#applyTransform`，默认实现就可以正常使用。请参考 [变换][transform] 部分。

### `applyTransform`

请参考 [变换][transform] 部分。

### `getQuads`

这是 `BakedModel` 的主要方法。它返回一个 `BakedQuad` 列表，这些对象包含用于渲染模型的底层顶点数据。如果模型作为方块进行渲染，传入的 `BlockState` 不为 `null`。如果模型作为物品进行渲染，`#getOverrides` 返回的 `ItemOverrides` 负责处理物品的状态，此时 `BlockState` 参数将为 `null`。

!!! 注意
    `BakedQuad` 中顶点的原点是底部、西北角落。顶点坐标值小于 0 或大于 1 会使顶点位于方块之外。为避免光照问题，请按逆时针顺序提供顶点。

传入的 `Direction` 参数用于面剔除。如果正在渲染的方块与另一个方块相邻的那一侧是不透明的，那么与该侧相关的面将不会被渲染。如果该参数为 `null`，则返回所有与侧面无关的面（这些面永远不会被剔除）。

`rand` 参数是一个 `Random` 实例。

该方法还接受一个非空的 `ModelData` 实例。这可用于通过 `ModelProperty` 在渲染特定模型时定义额外的数据。例如，`CompositeModel$Data` 就是这样一种属性，它用于为使用 `forge:composite` 模型加载器的模型存储任何额外的子模型数据。

请注意，这个方法会被频繁调用：在游戏世界中，每个方块的每个未被剔除的面与每个支持的方块渲染层的组合都会调用一次（调用次数在 0 到 28 次之间）。这个方法应该尽可能快，并且可能需要大量使用缓存。

[overrides]: ./itemoverrides.md
[ambocc]: https://en.wikipedia.org/wiki/Ambient_occlusion
[bewlr]: ../../items/bewlr.md
[transform]: ./transform.md
