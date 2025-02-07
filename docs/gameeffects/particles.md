粒子
=========

粒子是游戏中的一种特效，用于提升沉浸感。由于其创建和引用方式的特殊性，在使用时需要格外谨慎。

创建粒子
-------------------

粒子分为其 [**仅客户端**][sides] 的实现部分，用于显示粒子，以及其通用实现部分，用于引用粒子或从服务器同步数据。

| 类            | 所属端   | 描述 |
| :---             | :---:  |     :---    |
| ParticleType     | 两端   | 粒子类型定义的注册对象，用于在两端引用粒子 |
| ParticleOptions    | 两端   | 一种数据容器，用于从网络或命令向相关客户端同步信息 |
| ParticleProvider | 客户端 | 由 `ParticleType` 注册的工厂，用于根据相关的 `ParticleOptions` 构造一个 `Particle`。
| Particle         | 客户端 | 在相关客户端上显示的可渲染逻辑 |

### ParticleType

`ParticleType` 是一个注册对象，用于定义特定粒子类型，并在两端为特定粒子提供可用的引用。因此，每个 `ParticleType` 都必须[注册][registration]。

每个 `ParticleType` 接受两个参数：一个 `overrideLimiter`，用于确定粒子是否无论距离远近都进行渲染；以及一个 `ParticleOptions$Deserializer`，用于在客户端读取发送的 `ParticleOptions`。由于基础的 `ParticleType` 是抽象类，需要实现一个方法：`#codec`。这表示如何对该类型相关的 `ParticleOptions` 进行编码和解码。

!!! 注意
    `ParticleType#codec` 仅在原版实现的生物群系编解码器中使用。

在大多数情况下，无需向客户端发送任何粒子数据。对于这些情况，创建 `SimpleParticleType` 的新实例会更简单：`SimpleParticleType` 是 `ParticleType` 和 `ParticleOptions` 的一种实现，除了类型之外，不会向客户端发送任何自定义数据。除了用于着色的红石粉以及与方块/物品相关的粒子外，大多数原版实现都使用 `SimpleParticleType`。

!!! 重要
    如果仅在客户端引用粒子，创建粒子时不需要 `ParticleType`。但是，如果要使用 `ParticleEngine` 中的任何预构建逻辑，或者从服务器生成粒子，则必须使用 `ParticleType`。

### ParticleOptions

`ParticleOptions` 表示每个粒子所接受的数据。它还用于从服务器生成的粒子发送数据。所有粒子生成方法都接受一个 `ParticleOptions`，以便了解粒子的类型以及与生成粒子相关的数据。

`ParticleOptions` 分为三个方法：

| 方法         | 描述 |
| :---           | :---        |
| getType        | 获取粒子的类型定义，即 `ParticleType`
| writeToNetwork | 将粒子数据写入服务器上的缓冲区，以便发送到客户端
| writeToString  | 将粒子数据写入字符串

这些对象要么根据需要即时构造，要么由于是 `SimpleParticleType` 而作为单例存在。

#### ParticleOptions$Deserializer

为了在客户端接收 `ParticleOptions`，或者在命令中引用数据，必须通过 `ParticleOptions$Deserializer` 对粒子数据进行反序列化。`ParticleOptions$Deserializer` 中的每个方法在 `ParticleOptions` 中都有一个对应的编码方法：

| 方法      | ParticleOptions 编码器 | 描述 |
| :---        | :---:                 | :---        |
| fromCommand | writeToString         | 从字符串（通常来自命令）解码粒子数据。 |
| fromNetwork | writeToNetwork        | 在客户端从缓冲区解码粒子数据。 |

当需要发送自定义粒子数据时，这个对象会被传入 `ParticleType` 的构造函数中。

### Particle

`Particle` 提供了将数据绘制到屏幕上所需的渲染逻辑。要创建任何 `Particle`，必须实现两个方法：

| 方法        | 描述 |
| :---          | :---        |
| render        | 将粒子渲染到屏幕上。 |
| getRenderType | 获取粒子的渲染类型。 |

`Particle` 的一个常见子类 `TextureSheetParticle` 用于渲染纹理。虽然需要实现 `#getRenderType`，但设置的任何纹理精灵都会在粒子的位置进行渲染。

#### ParticleRenderType

`ParticleRenderType` 是 `RenderType` 的一种变体，它为该类型的每个粒子构建启动和结束阶段，然后通过 `Tesselator` 一次性渲染所有粒子。粒子可以有六种不同的渲染类型。

| 渲染类型                | 描述 |
| :---                       | :---        |
| TERRAIN_SHEET              | 渲染纹理位于可用方块中的粒子。 |
| PARTICLE_SHEET_OPAQUE      | 渲染纹理不透明且位于可用粒子中的粒子。 |
| PARTICLE_SHEET_TRANSLUCENT | 渲染纹理半透明且位于可用粒子中的粒子。 |
| PARTICLE_SHEET_LIT         | 与 `PARTICLE_SHEET_OPAQUE` 相同，只是不使用粒子着色器。 |
| CUSTOM                     | 提供混合和深度掩码的设置，但不提供渲染功能，因为这将在 `Particle#render` 中实现。 |
| NO_RENDER                  | 粒子永远不会渲染。 |

实现自定义渲染类型将留给读者作为练习。

### ParticleProvider

最后，粒子通常通过 `ParticleProvider` 创建。工厂有一个单一方法 `#createParticle`，用于根据粒子数据、客户端世界、位置和移动增量创建粒子。由于 `Particle` 不依赖于任何特定的 `ParticleType`，因此可以根据需要在不同的工厂中重复使用。

必须通过在 **模组事件总线** 上订阅 `RegisterParticleProvidersEvent` 来注册 `ParticleProvider`。在事件中，可以通过将工厂实例提供给 `#registerSpecial` 方法来注册工厂。

!!! 重要
    `RegisterParticleProvidersEvent` 应该仅在客户端调用，因此应在某个独立的客户端类中进行端区分，可通过 `DistExecutor` 或 `@EventBusSubscriber` 进行引用。

#### ParticleDescription、SpriteSet 和 SpriteParticleRegistration

有三种粒子渲染类型不能使用上述注册方法：`PARTICLE_SHEET_OPAQUE`、`PARTICLE_SHEET_TRANSLUCENT` 和 `PARTICLE_SHEET_LIT`。这是因为这三种粒子渲染类型都使用由 `ParticleEngine` 直接加载的精灵集。因此，必须通过不同的方法获取和注册所提供的纹理。这里假设你的粒子是 `TextureSheetParticle` 的子类型，因为这是此逻辑的唯一原版实现。

要为粒子添加纹理，必须在 `assets/<modid>/particles` 中添加一个新的 JSON 文件。这就是所谓的 `ParticleDescription`。此文件的名称将代表工厂所附加的 `ParticleType` 的注册名称。每个粒子 JSON 是一个对象。该对象存储一个单一的键 `textures`，它包含一个 `ResourceLocation` 数组。这里表示的任何 `<modid>:<path>` 纹理都将指向 `assets/<modid>/textures/particle/<path>.png` 中的纹理。

```js
{
  "textures": [
    // 将指向位于
    // assets/mymod/textures/particle/particle_texture.png 的纹理
    "mymod:particle_texture",
    // 纹理应按绘制顺序排列
    // 例如：particle_texture 首先渲染，然后过一段时间后渲染 particle_texture2
    "mymod:particle_texture2"
  ]
}
```

要引用粒子纹理，`TextureSheetParticle` 的子类型应该接受从 `SpriteSet` 获得的 `SpriteSet` 或 `TextureAtlasSprite`。`SpriteSet` 包含一个纹理列表，这些纹理引用了我们的 `ParticleDescription` 中定义的精灵。`SpriteSet` 有两个方法，这两个方法都以不同方式获取 `TextureAtlasSprite`。第一个方法接受两个整数。其底层实现允许精灵随着时间变化而改变纹理。第二个方法接受一个 `Random` 实例，从精灵集中获取一个随机纹理。可以通过使用 `TextureSheetParticle` 中接受 `SpriteSet` 的辅助方法之一来设置精灵：`#pickSprite` 使用随机选择纹理的方法，`#setSpriteFromAge` 使用两个整数的百分比方法来选择纹理。

要注册这些粒子纹理，需要将 `SpriteParticleRegistration` 提供给 `RegisterParticleProvidersEvent#registerSpriteSet` 方法。此方法接受一个 `SpriteSet`，该 `SpriteSet` 包含粒子的相关精灵集，并创建一个 `ParticleProvider` 来创建粒子。最简单的实现方法可以是在某个类上实现 `ParticleProvider`，并让构造函数接受一个 `SpriteSet`。然后可以像往常一样将 `SpriteSet` 传递给粒子。

!!! 注意
    如果你要注册的 `TextureSheetParticle` 子类型仅包含一个纹理，那么你可以将 `ParticleProvider$Sprite` 提供给 `#registerSprite` 方法，它与 `ParticleProvider` 具有基本相同的函数式接口方法。

生成粒子
-------------------

粒子可以从任意世界实例中生成。然而，每一端都有特定的生成粒子的方式。如果在 `ClientLevel` 上，可以调用 `#addParticle` 来生成粒子，或者调用 `#addAlwaysVisibleParticle` 来生成从任何距离都可见的粒子。如果在 `ServerLevel` 上，可以调用 `#sendParticles` 向客户端发送一个数据包以生成粒子。在服务器上调用这两个 `ClientLevel` 方法将不会有任何效果。

[sides]:../concepts/sides.md
[registration]:../concepts/registries.md#methods-for-registering
