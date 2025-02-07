# 编解码器

编解码器是 Mojang 的 [DataFixerUpper] 中的一种序列化工具，用于描述对象如何在不同格式之间转换，例如用于 JSON 的 `JsonElement` 和用于 NBT 的 `Tag`。

## 使用编解码器

编解码器主要用于将 Java 对象编码（序列化）为某种数据格式类型，以及将格式化的数据对象解码（反序列化）回其关联的 Java 类型。这通常分别通过 `Codec#encodeStart` 和 `Codec#parse` 来实现。

### DynamicOps

为了确定要编码和解码的中间文件格式，`#encodeStart` 和 `#parse` 都需要一个 `DynamicOps` 实例来定义该格式内的数据。

[DataFixerUpper] 库包含 `JsonOps`，用于对存储在 [`Gson` 的][gson] `JsonElement` 实例中的 JSON 数据进行编解码。`JsonOps` 支持两种版本的 `JsonElement` 序列化：`JsonOps#INSTANCE` 定义标准的 JSON 文件，`JsonOps#COMPRESSED` 允许将数据压缩为单个字符串。

```java
// 假设 exampleCodec 表示一个 Codec<ExampleJavaObject>
// 假设 exampleObject 是一个 ExampleJavaObject
// 假设 exampleJson 是一个 JsonElement

// 将 Java 对象编码为常规的 JsonElement
exampleCodec.encodeStart(JsonOps.INSTANCE, exampleObject);

// 将 Java 对象编码为压缩的 JsonElement
exampleCodec.encodeStart(JsonOps.COMPRESSED, exampleObject);

// 将 JsonElement 解码为 Java 对象
// 假设 JsonElement 是正常解析的
exampleCodec.parse(JsonOps.INSTANCE, exampleJson);
```

Minecraft 还提供了 `NbtOps`，用于对存储在 `Tag` 实例中的 NBT 数据进行编解码。可以使用 `NbtOps#INSTANCE` 来引用它。

```java
// 假设 exampleCodec 表示一个 Codec<ExampleJavaObject>
// 假设 exampleObject 是一个 ExampleJavaObject
// 假设 exampleNbt 是一个 Tag

// 将 Java 对象编码为 Tag
exampleCodec.encodeStart(JsonOps.INSTANCE, exampleObject);

// 将 Tag 解码为 Java 对象
exampleCodec.parse(JsonOps.INSTANCE, exampleNbt);
```

#### 格式转换

`DynamicOps` 也可以单独用于在两种不同的编码格式之间进行转换。这可以通过 `#convertTo` 并提供要转换的 `DynamicOps` 格式和编码对象来完成。

```java
// 将 Tag 转换为 JsonElement
// 假设 exampleTag 是一个 Tag
JsonElement convertedJson = NbtOps.INSTANCE.convertTo(JsonOps.INSTANCE, exampleTag);
```

### DataResult

使用编解码器进行编码或解码的数据会返回一个 `DataResult`，它根据转换是否成功持有转换后的实例或一些错误数据。当转换成功时，`#result` 提供的 `Optional` 将包含成功转换的对象。如果转换失败，`#error` 提供的 `Optional` 将包含 `PartialResult`，它根据编解码器持有错误消息和部分转换的对象。

此外，`DataResult` 上有许多方法可用于将结果或错误转换为所需的格式。例如，`#resultOrPartial` 将在成功时返回一个包含结果的 `Optional`，在失败时返回部分转换的对象。该方法接受一个字符串消费者，以确定如何报告错误消息（如果存在）。

```java
// 假设 exampleCodec 表示一个 Codec<ExampleJavaObject>
// 假设 exampleJson 是一个 JsonElement

// 将 JsonElement 解码为 Java 对象
DataResult<ExampleJavaObject> result = exampleCodec.parse(JsonOps.INSTANCE, exampleJson);

result
  // 获取结果或错误时的部分结果，报告错误消息
  .resultOrPartial(errorMessage -> /* 处理错误消息 */)
  // 如果结果或部分结果存在，进行相应操作
  .ifPresent(decodedObject -> /* 处理解码后的对象 */);
```

## 现有编解码器

### 基本类型

`Codec` 类包含某些定义的基本类型的编解码器的静态实例。

| 编解码器 | Java 类型 |
| :---: | :--- |
| `BOOL` | `Boolean` |
| `BYTE` | `Byte` |
| `SHORT` | `Short` |
| `INT` | `Integer` |
| `LONG` | `Long` |
| `FLOAT` | `Float` |
| `DOUBLE` | `Double` |
| `STRING` | `String` |
| `BYTE_BUFFER` | `ByteBuffer` |
| `INT_STREAM` | `IntStream` |
| `LONG_STREAM` | `LongStream` |
| `PASSTHROUGH` | `Dynamic<?>`* |
| `EMPTY` | `Unit`** |

* `Dynamic` 是一个持有以支持的 `DynamicOps` 格式编码的值的对象。这些通常用于将编码的对象格式转换为其他编码的对象格式。

** `Unit` 是一个用于表示 `null` 对象的对象。

### 原版和 Forge

Minecraft 和 Forge 为经常进行编码和解码的对象定义了许多编解码器。一些示例包括用于 `ResourceLocation` 的 `ResourceLocation#CODEC`、用于 `Instant` 且格式为 `DateTimeFormatter#ISO_INSTANT` 的 `ExtraCodecs#INSTANT_ISO8601`，以及用于 `CompoundTag` 的 `CompoundTag#CODEC`。

!!! 警告
    `CompoundTag` 不能使用 `JsonOps` 从 JSON 中解码数字列表。`JsonOps` 在转换时会将数字设置为最窄的类型。`ListTag` 会强制其数据为特定类型，因此不同类型的数字（例如 `64` 会是 `byte`，`384` 会是 `short`）在转换时会抛出错误。

原版和 Forge 的注册表也有针对注册表包含的对象类型的编解码器（例如 `Registry#BLOCK` 或 `ForgeRegistries#BLOCKS` 有一个 `Codec<Block>`）。`Registry#byNameCodec` 和 `IForgeRegistry#getCodec` 会将注册表对象编码为它们的注册表名称，如果是压缩形式则编码为整数标识符。原版注册表还有一个 `Registry#holderByNameCodec`，它将其编码为注册表名称，并解码为包装在 `Holder` 中的注册表对象。

## 创建编解码器

可以为任何对象创建用于编码和解码的编解码器。为了便于理解，将展示等效的编码后的 JSON。

### 记录类型

编解码器可以通过使用记录类型来定义对象。每个记录类型编解码器使用显式命名的字段来定义任何对象。有多种方法可以创建记录类型编解码器，但最简单的方法是通过 `RecordCodecBuilder#create`。

`RecordCodecBuilder#create` 接受一个函数，该函数定义一个 `Instance` 并返回对象的应用（`App`）。这可以类比为创建类的*实例*以及用于将类*应用*到构造对象的构造函数。

```java
// 要为其创建编解码器的某个对象
public class SomeObject {

  public SomeObject(String s, int i, boolean b) { /* ... */ }

  public String s() { /* ... */ }

  public int i() { /* ... */ }

  public boolean b() { /* ... */ }
}
```

#### 字段

一个 `Instance` 可以使用 `#group` 定义最多 16 个字段。每个字段必须是一个应用，用于定义正在创建对象的实例和对象的类型。满足此要求的最简单方法是获取一个 `Codec`，设置要从其解码的字段名称，并设置用于编码该字段的 getter。

如果字段是必需的，可以使用 `#fieldOf` 从 `Codec` 创建一个字段；如果字段被包装在 `Optional` 中或有默认值，则可以使用 `#optionalFieldOf`。这两种方法都需要一个包含编码对象中字段名称的字符串。然后可以使用 `#forGetter` 设置用于编码该字段的 getter，它接受一个函数，该函数给定对象后返回字段数据。

从那里，可以通过 `#apply` 应用结果产品，以定义实例应如何为应用构造对象。为了方便起见，分组的字段应按它们在构造函数中出现的相同顺序列出，这样函数可以简单地是构造函数方法引用。

```java
public static final Codec<SomeObject> RECORD_CODEC = RecordCodecBuilder.create(instance -> // 给定一个实例
  instance.group( // 定义实例中的字段
    Codec.STRING.fieldOf("s").forGetter(SomeObject::s), // 字符串
    Codec.INT.optionalFieldOf("i", 0).forGetter(SomeObject::i), // 整数，如果字段不存在则默认为 0
    Codec.BOOL.fieldOf("b").forGetter(SomeObject::b) // 布尔值
  ).apply(instance, SomeObject::new) // 定义如何创建对象
);
```

```js
// 编码后的 SomeObject
{
  "s": "value",
  "i": 5,
  "b": false
}

// 另一个编码后的 SomeObject
{
  "s": "value2",
  // i 被省略，默认为 0
  "b": true
}
```

### 转换器

编解码器可以通过映射方法转换为等效或部分等效的表示形式。每个映射方法接受两个函数：一个用于将当前类型转换为新类型，另一个用于将新类型转换回当前类型。这通过 `#xmap` 函数完成。

```java
// 一个类
public class ClassA {

  public ClassB toB() { /* ... */ }
}

// 另一个等效的类
public class ClassB {

  public ClassA toA() { /* ... */ }
}

// 假设存在某个编解码器 A_CODEC
public static final Codec<ClassB> B_CODEC = A_CODEC.xmap(ClassA::toB, ClassB::toA);
```

如果一种类型是部分等效的，即转换过程中有一些限制，则有一些映射函数会返回一个 `DataResult`，可用于在遇到异常或无效状态时返回错误状态。

| A 是否完全等效于 B | B 是否完全等效于 A | 转换方法 |
| :---: | :---: | :--- |
| 是 | 是 | `#xmap` |
| 是 | 否 | `#flatComapMap` |
| 否 | 是 | `#comapFlatMap` |
| 否 | 否 | `#flatXMap` |

```java
// 给定一个将字符串编解码器转换为整数的编解码器
// 并非所有字符串都可以转换为整数（A 并不完全等效于 B）
// 所有整数都可以转换为字符串（B 完全等效于 A）
public static final Codec<Integer> INT_CODEC = Codec.STRING.comapFlatMap(
  s -> { // 在失败时返回包含错误的数据结果
    try {
      return DataResult.success(Integer.valueOf(s));
    } catch (NumberFormatException e) {
      return DataResult.error(s + " 不是一个整数。");
    }
  },
  Integer::toString // 常规函数
);
```

```js
// 将返回 5
"5"

// 将报错，不是一个整数
"value"
```

#### 范围编解码器

范围编解码器是 `#flatXMap` 的一种实现，如果值不在设定的最小值和最大值（包含）之间，则返回一个错误 `DataResult`。如果值超出范围，仍会将其作为部分结果提供。分别有针对整数、浮点数和双精度数的实现，通过 `#intRange`、`#floatRange` 和 `#doubleRange`。

```java
public static final Codec<Integer> RANGE_CODEC = Codec.intRange(0, 4); 
```

```js
// 有效，在 [0, 4] 范围内
4

// 将报错，不在 [0, 4] 范围内
5
```

### 默认值

如果编码或解码的结果失败，可以通过 `Codec#orElse` 或 `Codec#orElseGet` 提供一个默认值。

```java
public static final Codec<Integer> DEFAULT_CODEC = Codec.INT.orElse(0); // 也可以通过 #orElseGet 提供一个值
```

```js
// 不是一个整数，默认为 0
"value"
```

### 单位

一个提供代码内值且编码为空的编解码器可以使用 `Codec#unit` 表示。如果编解码器在数据对象中使用了不可编码的条目，这很有用。

```java
public static final Codec<IForgeRegistry<Block>> UNIT_CODEC = Codec.unit(
  () -> ForgeRegistries.BLOCKS // 也可以是一个原始值
);
```

```js
// 这里为空，将返回方块注册表编解码器
```

### 列表

可以通过对象编解码器通过 `Codec#listOf` 生成对象列表的编解码器。

```java
// BlockPos#CODEC 是一个 Codec<BlockPos>
public static final Codec<List<BlockPos>> LIST_CODEC = BlockPos.CODEC.listOf();
```

```js
// 编码后的 List<BlockPos>
[
  [1, 2, 3], // BlockPos(1, 2, 3)
  [4, 5, 6], // BlockPos(4, 5, 6)
  [7, 8, 9]  // BlockPos(7, 8, 9)
]
```

使用列表编解码器解码的列表对象存储在一个**不可变**列表中。如果需要可变列表，应该对列表编解码器应用一个[转换器]。

### 映射

可以通过两个编解码器通过 `Codec#unboundedMap` 生成键和值对象的映射的编解码器。无界映射可以指定任何基于字符串或可转换为字符串的值作为键。

```java
// BlockPos#CODEC 是一个 Codec<BlockPos>
public static final Codec<Map<String, BlockPos>> MAP_CODEC = Codec.unboundedMap(Codec.STRING, BlockPos.CODEC);
```

```js
// 编码后的 Map<String, BlockPos>
{
  "key1": [1, 2, 3], // key1 -> BlockPos(1, 2, 3)
  "key2": [4, 5, 6], // key2 -> BlockPos(4, 5, 6)
  "key3": [7, 8, 9]  // key3 -> BlockPos(7, 8, 9)
}
```

使用无界映射编解码器解码的映射对象存储在一个**不可变**映射中。如果需要可变映射，应该对映射编解码器应用一个[转换器]。

!!! 警告
    无界映射仅支持编码/解码为字符串的键。可以使用键值[对]列表编解码器来绕过此限制。

### 对

可以通过两个编解码器通过 `Codec#pair` 生成对象对的编解码器。

对编解码器通过首先解码对中的左对象，然后取编码对象的剩余部分并从中解码右对象来解码对象。因此，编解码器必须要么在解码后表达关于编码对象的某些信息（例如[记录类型]），要么它们必须被增强为 `MapCodec` 并通过 `#codec` 转换为常规编解码器。这通常可以通过使编解码器成为某个对象的[字段]来完成。

```java
public static final Codec<Pair<Integer, String>> PAIR_CODEC = Codec.pair(
  Codec.INT.fieldOf("left").codec(),
  Codec.STRING.fieldOf("right").codec()
);
```

```js
// 编码后的 Pair<Integer, String>
{
  "left": 5,       // fieldOf 查找 'left' 键以获取左对象
  "right": "value" // fieldOf 查找 'right' 键以获取右对象
}
```

!!! 提示
    可以使用应用了[转换器]的键值对列表对具有非字符串键的映射编解码器进行编码/解码。

### 二者选一

可以通过两个编解码器通过 `Codec#either` 生成对某个对象数据进行两种不同编码/解码方法的编解码器。

二者选一编解码器首先尝试使用第一个编解码器解码对象。如果失败，它尝试使用第二个编解码器进行解码。如果第二个也失败，那么 `DataResult` 将只包含第二个编解码器失败的错误。

```java
public static final Codec<Either<Integer, String>> EITHER_CODEC = Codec.either(
  Codec.INT,
  Codec.STRING
);
```

```js
// 编码后的 Either$Left<Integer, String>
5

// 编码后的 Either$Right<Integer, String>
"value"
```

!!! 提示
    这可以与[转换器]结合使用，以从两种不同的编码方法中获取特定对象。

### 分发

编解码器可以有子编解码器，通过 `Codec#dispatch` 可以根据指定的类型对特定对象进行解码。这通常用于包含编解码器的注册表，例如规则测试或方块放置器。

分发编解码器首先尝试从某个字符串键（通常是 `type`）获取编码类型。然后，对该类型进行解码，调用用于解码实际对象的特定编解码器的 getter。如果用于解码对象的 `DynamicOps` 压缩其映射，或者对象编解码器本身没有增强为 `MapCodec`（例如记录类型或带字段的基本类型），则对象需要存储在 `value` 键中。否则，对象将在与其他数据相同的级别进行解码。

```java
// 定义我们的对象
public abstract class ExampleObject {

  // 定义用于指定编码对象类型的方法
  public abstract Codec<? extends ExampleObject> type();
}

// 创建存储字符串的简单对象
public class StringObject extends ExampleObject {

  public StringObject(String s) { /* ...
