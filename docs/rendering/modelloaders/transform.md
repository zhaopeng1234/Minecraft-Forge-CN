## 变换
当一个 [`BakedModel`][bakedmodel] 作为物品进行渲染时，它可以根据渲染时所处的变换类型进行特殊处理。“变换” 指的是模型渲染的上下文。可能的变换类型在代码中由 `ItemDisplayContext` 枚举表示。处理变换有两种系统：已弃用的原版系统，由 `BakedModel#getTransforms`、`ItemTransforms` 和 `ItemTransform` 组成；以及 Forge 系统，由 `IForgeBakedModel#applyTransform` 方法体现。原版代码已被修改，尽可能优先使用 `applyTransform` 而非原版系统。

### `ItemDisplayContext`
- `NONE`：当未设置上下文时，默认用于显示实体；当一个 `Block` 的 `RenderShape` 设置为 `#ENTITYBLOCK_ANIMATED` 时，Forge 也会使用此值。
- `THIRD_PERSON_LEFT_HAND`/`THIRD_PERSON_RIGHT_HAND`/`FIRST_PERSON_LEFT_HAND`/`FIRST_PERSON_RIGHT_HAND`：第一人称的值表示玩家自己手持物品的情况。第三人称的值表示另一个玩家手持物品，而客户端以第三人称视角观察他们的情况。左右手的含义很明确。
- `HEAD`：表示任何玩家将物品戴在头盔槽时的情况（例如南瓜）。
- `GUI`：表示物品在 `Screen` 中渲染的情况。
- `GROUND`：表示物品作为 `ItemEntity` 在游戏世界中渲染的情况。
- `FIXED`：用于物品展示框。

### 原版方式
原版处理变换的方式是通过 `BakedModel#getTransforms` 方法。该方法返回一个 `ItemTransforms` 对象，这是一个简单的对象，其 `public final` 字段包含各种 `ItemTransform`。`ItemTransform` 表示要应用于模型的旋转、平移和缩放。`ItemTransforms` 是这些变换的容器，除 `NONE` 外，为每个 `ItemDisplayContext` 都持有一个变换。在原版实现中，为 `NONE` 调用 `#getTransform` 会得到默认变换 `ItemTransform#NO_TRANSFORM`。

Forge 已弃用整个原版的变换处理系统，大多数 `BakedModel` 的实现应该从 `BakedModel#getTransforms` 方法中简单地 `return ItemTransforms#NO_TRANSFORMS`（这是默认实现）。相反，应该实现 `#applyTransform` 方法。

### Forge 方式
Forge 处理变换的方式是使用 `#applyTransform` 方法，这是一个被添加到 `BakedModel` 中的方法。它取代了 `#getTransforms` 方法。

#### `BakedModel#applyTransform`
给定一个 `ItemDisplayContext`、一个 `PoseStack` 和一个布尔值（用于确定是否应用左手的变换），此方法会生成一个要渲染的 `BakedModel`。由于返回的 `BakedModel` 可以是一个全新的模型，因此该方法比原版方法更灵活（例如，一张纸在手中看起来是平的，但在地上是皱的）。

[bakedmodel]: ./bakedmodel.md
