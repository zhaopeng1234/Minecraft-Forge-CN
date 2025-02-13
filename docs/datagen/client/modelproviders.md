### 模型生成
默认情况下，可以为模型或方块状态生成[模型]。每个都提供了一种生成必要 JSON 文件的方法（模型使用 `ModelBuilder#toJson`，方块状态使用 `IGeneratedBlockState#toJson`）。实现之后，必须将[相关的提供者][provider] [添加][datagen]到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    DataGenerator gen = event.getGenerator();
    ExistingFileHelper efh = event.getExistingFileHelper();

    gen.addProvider(
        // 告诉生成器仅在生成客户端资源时运行
        event.includeClient(),
        output -> new MyItemModelProvider(output, MOD_ID, efh)
    );
    gen.addProvider(
        event.includeClient(),
        output -> new MyBlockStateProvider(output, MOD_ID, efh)
    );
}
```

### 模型文件
`ModelFile` 是所有由提供者引用或生成的模型的基础。每个模型文件存储相对于 `models` 子目录的位置，并可以断言该文件是否存在。

#### 现有模型文件
`ExistingModelFile` 是 `ModelFile` 的子类，它通过 [`ExistingFileHelper#exists`][efh] 检查模型是否已经存在于 `models` 子目录中。所有非生成的模型通常通过 `ExistingModelFile` 引用。

#### 未检查的模型文件
`UncheckedModelFile` 是 `ModelFile` 的子类，它假设指定的模型存在于某个位置。

!!! 注意
    不应该使用 `UncheckedModelFile` 来引用模型。如果使用了，那么相关资源没有被 `ExistingFileHelper` 正确跟踪。

### 模型构建器
`ModelBuilder` 表示一个待生成的 `ModelFile`。它包含模型的所有数据：其父模型、面、纹理、变换、光照和[加载器]。

!!! 提示
    虽然可以生成复杂的模型，但建议事先使用建模软件构建这些模型。然后，数据提供者可以通过父复杂模型中定义的引用，为具有特定纹理的子模型进行生成。

构建器的父模型（通过 `ModelBuilder#parent`）可以是任何 `ModelFile`：已生成的或现有的。生成的文件会在构建器创建后立即添加到 `ModelProvider` 中。构建器本身可以作为父模型传入，也可以传入 `ResourceLocation`。

!!! 警告
    如果在传入 `ResourceLocation` 时，父模型在子模型之前未生成，则会抛出异常。

模型中的每个元素（通过 `ModelBuilder#element`）定义为一个立方体，使用两个三维点（分别通过 `ElementBuilder#from` 和 `#to`），每个轴的值限制在 `[-16, 32]`（包含 -16 和 32）之间。立方体的每个面（`ElementBuilder#face`）可以指定面何时被剔除（`FaceBuilder#cullface`）、[着色索引][color]（`FaceBuilder#tintindex`）、来自 `textures` 键的纹理引用（`FaceBuilder#texture`）、纹理上的 UV 坐标（`FaceBuilder#uvs`）以及 90 度间隔的旋转（`FaceBuilder#rotation`）。

!!! 注意
    建议任何轴上元素超出 `[0, 16]` 边界的方块模型拆分为多个方块，例如对于多方块结构，以避免光照和剔除问题。

每个立方体还可以围绕指定点（`RotationBuilder#origin`）在给定轴（`RotationBuilder#axis`）上以 22.5 度间隔旋转（`ElementBuilder#rotation`）。立方体也可以相对于整个模型缩放所有面（`RotationBuilder#rescale`）。立方体还可以确定是否渲染其阴影（`ElementBuilder#shade`）。

每个模型定义了一个纹理键列表（`ModelBuilder#texture`），这些键指向一个位置或引用。然后，任何元素都可以通过前缀 `#` 引用这些键（例如，纹理键 `example` 可以在元素中使用 `#example` 引用）。位置指定纹理在 `assets/<命名空间>/textures/<路径>.png` 中的位置。引用由当前模型的父模型用作键，以便稍后定义纹理。

模型还可以针对任何定义的视角（在第一人称左手、在 GUI 中、在地面上等）进行变换（`ModelBuilder#transforms`）。对于任何视角（`TransformsBuilder#transform`），可以设置旋转（`TransformVecBuilder#rotation`）、平移（`TransformVecBuilder#translation`）和缩放（`TransformVecBuilder#scale`）。

最后，模型可以设置是否在游戏中使用环境光遮蔽（`ModelBuilder#ao`），以及从 `ModelBuilder#guiLight` 指定的位置对模型进行光照和着色。

#### `BlockModelBuilder`
`BlockModelBuilder` 表示一个待生成的方块模型。除了 `ModelBuilder` 的功能外，还可以为整个模型生成一个变换（`BlockModelBuilder#rootTransform`）。根可以围绕某个原点（`RootTransformBuilder#origin`）进行平移（`RootTransformBuilder#transform`）、旋转（`RootTransformBuilder#rotation`、`RootTransformBuilder#postRotation`）和缩放（`RootTransformBuilder#scale`），可以单独进行，也可以在一次变换中完成（`RootTransformBuilder#transform`）。

#### `ItemModelBuilder`
`ItemModelBuilder` 表示一个待生成的物品模型。除了 `ModelBuilder` 的功能外，还可以生成[覆盖物]（`OverrideBuilder#override`）。应用于模型的每个覆盖物可以应用条件，表示给定属性必须高于指定值（`OverrideBuilder#predicate`）。如果条件满足，则将渲染指定的模型（`OverrideBuilder#model`）而不是此模型。

### 模型提供者
`ModelProvider` 的子类负责生成构造好的 `ModelBuilder`。提供者接受生成器、模组 ID、`models` 文件夹中要生成的子目录、`ModelBuilder` 工厂和现有文件帮助器。每个提供者子类必须实现 `#registerModels` 方法。

提供者包含一些基本方法，用于创建 `ModelBuilder` 或方便获取纹理或模型引用：

| 方法 | 描述 |
| :---: | :--- |
| `getBuilder` | 在提供者的子目录中为给定的模组 ID 创建一个新的 `ModelBuilder`。 |
| `withExistingParent` | 为给定的父模型创建一个新的 `ModelBuilder`。当父模型不是由构建器生成时应使用此方法。 |
| `mcLoc` | 为 `minecraft` 命名空间中的路径创建一个 `ResourceLocation`。 |
| `modLoc` | 为给定模组 ID 命名空间中的路径创建一个 `ResourceLocation`。 |

此外，还有一些辅助方法可用于使用原版模板轻松生成常见模型。大多数是用于方块模型，只有少数是通用的。

!!! 注意
    虽然模型位于特定的子目录中，但这并不意味着该模型不能被其他子目录中的模型引用。通常，这表明该模型用于该类型的对象。

#### `BlockModelProvider`
`BlockModelProvider` 用于在 `block` 文件夹中通过 `BlockModelBuilder` 生成方块模型。方块模型通常应继承 `minecraft:block/block` 或其某个子模型，以便与物品模型一起使用。

!!! 注意
    方块模型及其对应的物品模型通常不是通过 `BlockModelProvider` 和 `ItemModelProvider` 的直接子类生成的，而是通过 [`BlockStateProvider`][blockstateprovider] 生成的。

#### `ItemModelProvider`
`ItemModelProvider` 用于在 `item` 文件夹中通过 `ItemModelBuilder` 生成物品模型。大多数物品模型继承 `item/generated` 并使用 `layer0` 来指定其纹理，可以使用 `#singleTexture` 方法实现。

!!! 注意
    `item/generated` 可以支持五个纹理层相互叠加：`layer0`、`layer1`、`layer2`、`layer3` 和 `layer4`。

```java
// 在某个 ItemModelProvider#registerModels 方法中

// 将生成 'assets/<modid>/models/item/example_item.json'
// 父模型将是 'minecraft:item/generated'
// 对于纹理键 'layer0'
//  它将位于 'assets/<modid>/textures/item/example_item.png'
this.basicItem(EXAMPLE_ITEM.get());
```

!!! 注意
    方块的物品模型通常应继承现有的方块模型，而不是为物品生成单独的模型。

### 方块状态提供者
`BlockStateProvider` 负责为方块在 `blockstates` 中生成[方块状态 JSON 文件][blockstate]，在 `models/block` 中生成方块模型，在 `models/item` 中生成物品模型。提供者接受数据生成器、模组 ID 和现有文件帮助器。每个 `BlockStateProvider` 子类必须实现 `#registerStatesAndModels` 方法。

提供者包含用于生成方块状态 JSON 文件和方块模型的基本方法。物品模型必须单独生成，因为方块状态 JSON 文件可能定义在不同上下文中使用的多个模型。然而，在处理更复杂的任务时，模组开发者应该了解一些常见的方法：

| 方法 | 描述 |
| :---: | :--- |
| `models` | 获取用于生成方块物品模型的 [`BlockModelProvider`][blockmodels]。 |
| `itemModels` | 获取用于生成方块物品模型的 [`ItemModelProvider`][itemmodels]。 |
| `modLoc` | 为给定模组 ID 命名空间中的路径创建一个 `ResourceLocation`。 |
| `mcLoc` | 为 `minecraft` 命名空间中的路径创建一个 `ResourceLocation`。 |
| `blockTexture` | 引用 `textures/block` 中与方块同名的纹理。 |
| `simpleBlockItem` | 为给定关联模型文件的方块创建一个物品模型。 |
| `simpleBlockWithItem` | 为方块模型创建一个单一方块状态，并使用方块模型作为其父模型创建一个物品模型。 |

方块状态 JSON 文件由变体或条件组成。每个变体或条件引用一个 `ConfiguredModelList`：一个 `ConfiguredModel` 列表。每个配置模型包含模型文件（通过 `ConfiguredModel$Builder#modelFile`）、90 度间隔的 X 和 Y 旋转（分别通过 `#rotationX` 和 `rotationY`）、模型在方块状态 JSON 文件旋转时纹理是否可以旋转（通过 `#uvLock`）以及与列表中其他模型相比该模型出现的权重（通过 `#weight`）。

构建器（`ConfiguredModel#builder`）还可以通过使用 `#nextModel` 创建下一个模型并重复设置，直到调用 `#build` 来创建一个 `ConfiguredModel` 数组。

### `VariantBlockStateBuilder`
可以使用 `BlockStateProvider#getVariantBuilder` 生成变体。每个变体指定一个[属性]列表（`PartialBlockstate`），当这些属性与游戏世界中的 `BlockState` 匹配时，将从相应的模型列表中选择一个模型进行显示。如果存在未被任何定义的变体覆盖的 `BlockState`，则会抛出异常。对于任何 `BlockState`，只能有一个变体为真。

`PartialBlockstate` 通常使用以下三种方法之一定义：

| 方法 | 描述 |
| :---: | --- |
| `partialState` | 创建一个待定义的 `PartialBlockstate`。 |
| `forAllStates` | 定义一个函数，其中给定的 `BlockState` 可以由一个 `ConfiguredModel` 数组表示。 |
| `forAllStatesExcept` | 定义一个类似于 `#forAllStates` 的函数；不过，它还会指定哪些属性不影响渲染的模型。 |

对于 `PartialBlockstate`，可以指定已定义的属性（`#with`）。可以设置配置好的模型（`#setModels`），将其追加到现有模型中（`#addModels`），或者进行构建（先使用 `#modelForState`，完成后使用 `ConfiguredModel$Builder#addModel` 而不是 `#ConfiguredModel$Builder#build`）。

```java
// 在某个 BlockStateProvider#registerStatesAndModels 方法中

// EXAMPLE_BLOCK_1: 具有属性 BlockStateProperties#AXIS
this.getVariantBuilder(EXAMPLE_BLOCK_1) // 获取变体构建器
  .partialState() // 构建部分状态
  .with(AXIS, Axis.Y) // 当 BlockState 的 AXIS 属性为 Y 时
    .modelForState() // 设置当 AXIS 为 Y 时的模型
    .modelFile(yModelFile1) // 可以显示 'yModelFile1'
    .nextModel() // 添加另一个当 AXIS 为 Y 时的模型
    .modelFile(yModelFile2) // 可以显示 'yModelFile2'
    .weight(2) // 'yModelFile2' 显示的概率为 2/3
    .addModel() // 完成当 AXIS 为 Y 时的模型设置
  .with(AXIS, Axis.Z) // 当 BlockState 的 AXIS 属性为 Z 时
    .modelForState() // 设置当 AXIS 为 Z 时的模型
    .modelFile(hModelFile) // 可以显示 'hModelFile'
    .addModel() // 完成当 AXIS 为 Z 时的模型设置
  .with(AXIS, Axis.X)  // 当 BlockState 的 AXIS 属性为 X 时
    .modelForState() // 设置当 AXIS 为 X 时的模型
    .modelFile(hModelFile) // 可以显示 'hModelFile'
    .rotationY(90) // 将 'hModelFile' 绕 Y 轴旋转 90 度
    .addModel(); // 完成当 AXIS 为 X 时的模型设置

// EXAMPLE_BLOCK_2: 具有属性 BlockStateProperties#HORIZONTAL_FACING
this.getVariantBuilder(EXAMPLE_BLOCK_2) // 获取变体构建器
  .forAllStates(state -> // 对于所有可能的状态
    ConfiguredModel.builder() // 创建配置模型构建器
      .modelFile(modelFile) // 可以显示 'modelFile'
      .rotationY((int) state.getValue(HORIZONTAL_FACING).toYRot()) // 根据属性将 'modelFile' 绕 Y 轴旋转
      .build() // 创建配置模型数组
  );

// EXAMPLE_BLOCK_3: 具有属性 BlockStateProperties#HORIZONTAL_FACING、BlockStateProperties#WATERLOGGED
this.getVariantBuilder(EXAMPLE_BLOCK_3) // 获取变体构建器
  .forAllStatesExcept(state -> // 对于所有 HORIZONTAL_FACING 状态
    ConfiguredModel.builder() // 创建配置模型构建器
      .modelFile(modelFile) // 可以显示 'modelFile'
      .rotationY((int) state.getValue(HORIZONTAL_FACING).toYRot()) // 根据属性将 'modelFile' 绕 Y 轴旋转
      .build(), // 创建配置模型数组
  WATERLOGGED); // 忽略 WATERLOGGED 属性
```

### `MultiPartBlockStateBuilder`
可以使用 `BlockStateProvider#getMultipartBuilder` 生成多部分方块状态。每个部分（`MultiPartBlockStateBuilder#part`）指定一组属性条件，当这些条件与游戏世界中的 `BlockState` 匹配时，将从模型列表中选择一个模型进行显示。所有与 `BlockState` 匹配的条件组所对应的模型将相互叠加显示。

对于任何部分（通过 `ConfiguredModel$Builder#addModel` 获取），可以添加一个条件（通过 `#condition`），即当某个属性为指定值之一时满足条件。条件必须全部满足；或者，当设置了 `#useOr` 时，至少有一个条件必须满足。条件可以进行分组（通过 `#nestedGroup`），只要当前分组只包含其他组而不包含单个条件。可以使用 `#endNestedGroup` 退出条件组，并使用 `#end` 完成一个部分的设置。

```java
// 在某个 BlockStateProvider#registerStatesAndModels 方法中

// 红石线
this.getMultipartBuilder(REDSTONE) // 获取多部分构建器
  .part() // 创建部分
    .modelFile(redstoneDot) // 可以显示 'redstoneDot'
    .addModel() // 'redstoneDot' 在以下情况显示...
    .useOr() // 这些条件中至少有一个为真
    .nestedGroup() // 当所有分组条件都为真时
      .condition(WEST_REDSTONE, NONE) // 当 WEST_REDSTONE 为 NONE 时为真
      .condition(EAST_REDSTONE, NONE) // 当 EAST_REDSTONE 为 NONE 时为真
      .condition(SOUTH_REDSTONE, NONE) // 当 SOUTH_REDSTONE 为 NONE 时为真
      .condition(NORTH_REDSTONE, NONE) // 当 NORTH_REDSTONE 为 NONE 时为真
    .endNestedGroup() // 结束分组
    .nestedGroup() // 当所有分组条件都为真时
      .condition(EAST_REDSTONE, SIDE, UP) // 当 EAST_REDSTONE 为 SIDE 或 UP 时为真
      .condition(NORTH_REDSTONE, SIDE, UP) // 当 NORTH_REDSTONE 为 SIDE 或 UP 时为真
    .endNestedGroup() // 结束分组
    .nestedGroup() // 当所有分组条件都为真时
      .condition(EAST_REDSTONE, SIDE, UP) // 当 EAST_REDSTONE 为 SIDE 或 UP 时为真
      .condition(SOUTH_REDSTONE, SIDE, UP) // 当 SOUTH_REDSTONE 为 SIDE 或 UP 时为真
    .endNestedGroup() // 结束分组
    .nestedGroup() // 当所有分组条件都为真时
      .condition(WEST_REDSTONE, SIDE, UP) // 当 WEST_REDSTONE 为 SIDE 或 UP 时为真
      .condition(SOUTH_REDSTONE, SIDE, UP) // 当 SOUTH_REDSTONE 为 SIDE 或 UP 时为真
    .endNestedGroup() // 结束分组
    .nestedGroup() // 当所有分组条件都为真时
      .condition(WEST_REDSTONE, SIDE, UP) // 当 WEST_REDSTONE 为 SIDE 或 UP 时为真
      .condition(NORTH_REDSTONE, SIDE, UP) // 当 NORTH_REDSTONE 为 SIDE 或 UP 时为真
    .endNestedGroup() // 结束分组
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneSide0) // 可以显示 'redstoneSide0'
    .addModel() // 'redstoneSide0' 在以下情况显示...
    .condition(NORTH_REDSTONE, SIDE, UP) // 当 NORTH_REDSTONE 为 SIDE 或 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneSideAlt0) // 可以显示 'redstoneSideAlt0'
    .addModel() // 'redstoneSideAlt0' 在以下情况显示...
    .condition(SOUTH_REDSTONE, SIDE, UP) // 当 SOUTH_REDSTONE 为 SIDE 或 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneSideAlt1) // 可以显示 'redstoneSideAlt1'
    .rotationY(270) // 将 'redstoneSideAlt1' 绕 Y 轴旋转 270 度
    .addModel() // 'redstoneSideAlt1' 在以下情况显示...
    .condition(EAST_REDSTONE, SIDE, UP) // 当 EAST_REDSTONE 为 SIDE 或 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneSide1) // 可以显示 'redstoneSide1'
    .rotationY(270) // 将 'redstoneSide1' 绕 Y 轴旋转 270 度
    .addModel() // 'redstoneSide1' 在以下情况显示...
    .condition(WEST_REDSTONE, SIDE, UP) // 当 WEST_REDSTONE 为 SIDE 或 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneUp) // 可以显示 'redstoneUp'
    .addModel() // 'redstoneUp' 在以下情况显示...
    .condition(NORTH_REDSTONE, UP) // 当 NORTH_REDSTONE 为 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneUp) // 可以显示 'redstoneUp'
    .rotationY(90) // 将 'redstoneUp' 绕 Y 轴旋转 90 度
    .addModel() // 'redstoneUp' 在以下情况显示...
    .condition(EAST_REDSTONE, UP) // 当 EAST_REDSTONE 为 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneUp) // 可以显示 'redstoneUp'
    .rotationY(180) // 将 'redstoneUp' 绕 Y 轴旋转 180 度
    .addModel() // 'redstoneUp' 在以下情况显示...
    .condition(SOUTH_REDSTONE, UP) // 当 SOUTH_REDSTONE 为 UP 时
    .end() // 完成此部分
  .part() // 创建部分
    .modelFile(redstoneUp) // 可以显示 'redstoneUp'
    .rotationY(270) // 将 'redstoneUp' 绕 Y 轴旋转 270 度
    .addModel() // 'redstoneUp' 在以下情况显示...
    .condition(WEST_REDSTONE, UP) // 当 WEST_REDSTONE 为 UP 时
    .end(); // 完成此部分
```

### 模型加载器构建器
也可以为给定的 `ModelBuilder` 生成自定义模型加载器。自定义模型加载器继承自 `CustomLoaderBuilder`，可以通过 `#customLoader` 应用到 `ModelBuilder` 上。传入的工厂方法会创建一个新的加载器构建器，可对其进行配置。完成所有更改后，自定义加载器可以通过 `CustomLoaderBuilder#end` 返回到 `ModelBuilder`。

| 模型构建器 | 工厂方法 | 描述 |
| :---: | :---: | --- |
| `DynamicFluidContainerModelBuilder` | `#begin` | 为指定的流体生成一个桶模型。 |
| `CompositeModelBuilder` | `#begin` | 生成一个由多个模型组成的模型。 |
| `ItemLayersModelBuilder` | `#begin` | 生成一个 Forge 实现的 `item/generated` 模型。 |
| `SeparateTransformsModelBuilder` | `#begin` | 生成一个根据指定[变换]而变化的模型。 |
| `ObjModelBuilder` | `#begin` | 生成一个 [OBJ 模型][obj]。 |

```java
// 对于某个 BlockModelBuilder 构建器
builder.customLoader(ObjModelBuilder::begin) // 自定义加载器 'forge:obj'
  .modelLocation(modLoc("models/block/model.obj")) // 设置 OBJ 模型的位置
  .flipV(true) // 翻转提供的 .mtl 纹理中的 V 坐标
  .end() // 完成自定义加载器配置
.texture("particle", mcLoc("block/dirt")) // 将粒子纹理设置为泥土
.texture("texture0", mcLoc("block/dirt")); // 将 'texture0' 纹理设置为泥土
```

### 自定义模型加载器构建器
可以通过扩展 `CustomLoaderBuilder` 来创建自定义加载器构建器。构造函数可以设置为 `protected` 可见性，并将 `ResourceLocation` 硬编码为通过 `ModelEvent$RegisterGeometryLoaders#register` 注册的加载器 ID。然后，可以通过静态工厂方法或如果构造函数设为 `public` 则直接使用构造函数来初始化构建器。

```java
public class ExampleLoaderBuilder<T extends ModelBuilder<T>> extends CustomLoaderBuilder<T> {
  public static <T extends ModelBuilder<T>> ExampleLoaderBuilder<T> begin(T parent, ExistingFileHelper existingFileHelper) {
    return new ExampleLoaderBuilder<>(parent, existingFileHelper);
  }

  protected ExampleLoaderBuilder(T parent, ExistingFileHelper existingFileHelper) {
    super(new ResourceLocation(MOD_ID, "example_loader"), parent, existingFileHelper);
  }
}
```

之后，加载器指定的任何配置都应作为可链式调用的方法添加。

```java
// 在 ExampleLoaderBuilder 中
public ExampleLoaderBuilder<T> exampleInt(int example) {
  // 设置整数
  return this;
}

public ExampleLoaderBuilder<T> exampleString(String example) {
  // 设置字符串
  return this;
}
```

如果指定了任何额外的配置，应该重写 `#toJson` 方法来写入额外的属性。

```java
// 在 ExampleLoaderBuilder 中
@Override
public JsonObject toJson(JsonObject json) {
  json = super.toJson(json); // 处理基本加载器属性
  // 编码自定义加载器属性
  return json;
}
```

### 自定义模型提供者
自定义模型提供者需要一个 `ModelBuilder` 子类（用于定义要生成的模型的基础）和一个 `ModelProvider` 子类（用于生成模型）。

`ModelBuilder` 子类包含可以专门应用于那些类型模型的任何特殊属性（例如物品模型可以有覆盖物）。如果添加了任何额外的属性，需要重写 `#toJson` 方法来写入额外的信息。

```java
public class ExampleModelBuilder extends ModelBuilder<ExampleModelBuilder> {
  // ...
}
```

`ModelProvider` 子类不需要特殊的逻辑。构造函数应该硬编码 `models` 文件夹中的子目录以及用于表示待生成模型的 `ModelBuilder`。

```java
public class ExampleModelProvider extends ModelProvider<ExampleModelBuilder> {

  public ExampleModelProvider(PackOutput output, String modid, ExistingFileHelper existingFileHelper) {
    // 如果在 '#getBuilder' 中未指定 'modid'，模型将生成到 'assets/<modid>/models/example'
    super(output, modid, "example", ExampleModelBuilder::new, existingFileHelper);
  }
}
```

### 自定义模型消费者
像 [`BlockStateProvider`][blockstateprovider] 这样的自定义模型消费者可以通过手动生成模型来创建。应该指定并提供用于生成模型的 `ModelProvider` 子类。

```java
public class ExampleModelConsumerProvider implements IDataProvider {

  public ExampleModelConsumerProvider(PackOutput output, String modid, ExistingFileHelper existingFileHelper) {
    this.example = new ExampleModelProvider(output, modid, existingFileHelper);
  }
}
```

一旦数据提供者开始运行，可以使用 `ModelProvider#generateAll` 来生成 `ModelProvider` 子类中的模型。

```java
// 在 ExampleModelConsumerProvider 中
@Override
public CompletableFuture<?> run(CachedOutput cache) {
  // 填充模型提供者
  CompletableFuture<?> exampleFutures = this.example.generateAll(cache); // 生成模型

  // 运行逻辑并为写入文件创建 CompletableFuture(s)
  // ...

  // 假设我们有一个新的 CompletableFuture providerFuture
  return CompletableFuture.allOf(exampleFutures, providerFuture);
}
```

[provider]: #model-providers
[models]: ../../resources/client/models/index.md
[datagen]: ../index.md#data-providers
[efh]: ../index.md#existing-files
[loader]: #custom-model-loader-builders
[color]: ../../resources/client/models/tinting.md#blockcoloritemcolor
[overrides]: ../../resources/client/models/itemproperties.md
[blockstateprovider]: #block-state-provider
[blockstate]: https://minecraft.wiki/w/Tutorials/Models#Block_states
[blockmodels]: #blockmodelprovider
[itemmodels]: #itemmodelprovider
[properties]: ../../blocks/states.md#implementing-block-states
[transform]: ../../rendering/modelloaders/transform.md
[obj]: ../../rendering/modelloaders/index.md#wavefront-obj-models
