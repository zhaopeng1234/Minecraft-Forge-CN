### 访问转换器概述
访问转换器（Access Transformers，简称 ATs）允许扩大类、方法和字段的可见性，并修改其 `final` 标志。这使得模组开发者能够访问和修改那些原本无法访问的类成员。

### 添加访问转换器
将访问转换器添加到你的模组项目中非常简单，只需在 `build.gradle` 文件中添加一行代码：
```groovy
// 此代码块也是指定映射版本的地方
minecraft {
  accessTransformer = file('src/main/resources/META-INF/accesstransformer.cfg')
}
```
添加或修改访问转换器后，需要刷新 Gradle 项目，转换才能生效。

在开发过程中，AT 文件可以位于上述代码指定的任何位置。但在非开发环境中加载时，Forge 只会在 JAR 文件的 `META-INF/accesstransformer.cfg` 路径下搜索该文件。

### 注释
从 `#` 到行尾的所有文本都将被视为注释，不会被解析。

### 访问修饰符
访问修饰符指定了给定目标将被转换为的新成员可见性。按可见性从高到低的顺序排列如下：
- `public`：对其包内外的所有类可见。
- `protected`：仅对包内的类和子类可见。
- `default`：仅对包内的类可见。
- `private`：仅对类内部可见。

可以在上述修饰符后附加特殊修饰符 `+f` 或 `-f`，分别用于添加或移除 `final` 修饰符。`final` 修饰符用于防止子类化、方法重写或字段修改。

!!! 警告
    指令仅修改它们直接引用的方法；任何重写的方法不会被进行访问转换。建议确保被转换的方法没有限制可见性的未转换重写方法，否则 JVM 会抛出错误。

    可以安全转换的方法示例包括 `private` 方法、`final` 方法（或 `final` 类中的方法）和 `static` 方法。

### 目标和指令
!!! 重要
    对 Minecraft 类使用访问转换器时，字段和方法必须使用 SRG 名称。

#### 类
要针对类，可以使用以下格式：
```
<访问修饰符> <完全限定类名>
```
内部类通过使用 `$` 作为分隔符，将外部类的完全限定名和内部类的名称组合起来表示。

#### 字段
要针对字段，可以使用以下格式：
```
<访问修饰符> <完全限定类名> <字段名>
```

#### 方法
针对方法需要使用特殊的语法来表示方法参数和返回类型：
```
<访问修饰符> <完全限定类名> <方法名>(<参数类型>)<返回类型>
```

##### 指定类型
这也被称为“描述符”，更多技术细节可参考 [Java 虚拟机规范，SE 8，第 4.3.2 和 4.3.3 节][jvmdescriptors]。
- `B`：`byte`，有符号字节。
- `C`：`char`，UTF - 16 编码的 Unicode 字符代码点。
- `D`：`double`，双精度浮点数。
- `F`：`float`，单精度浮点数。
- `I`：`integer`，32 位整数。
- `J`：`long`，64 位整数。
- `S`：`short`，有符号短整数。
- `Z`：`boolean`，`true` 或 `false` 值。
- `[`：表示数组的一个维度。
  - 示例：`[[S` 表示 `short[][]`。
- `L<类名>;`：表示引用类型。
  - 示例：`Ljava/lang/String;` 表示 `java.lang.String` 引用类型（注意使用斜杠而不是句点）。
- `(`：表示方法描述符，这里应提供参数，如果没有参数则留空。
  - 示例：`<方法>(I)Z` 表示一个需要整数参数并返回布尔值的方法。
- `V`：表示方法不返回值，只能用在方法描述符的末尾。
  - 示例：`<方法>()V` 表示一个没有参数且不返回任何值的方法。

### 示例
```
# 使 Crypt 类中的 ByteArrayToKeyFunction 接口变为 public
public net.minecraft.util.Crypt$ByteArrayToKeyFunction

# 使 MinecraftServer 类中的 'random' 字段变为 protected 并移除 final 修饰符
protected-f net.minecraft.server.MinecraftServer f_129758_ #random

# 使 Util 类中的 'makeExecutor' 方法变为 public，该方法接受一个 String 类型的参数并返回一个 ExecutorService
public net.minecraft.Util m_137477_(Ljava/lang/String;)Ljava/util/concurrent/ExecutorService; #makeExecutor

# 使 UUIDUtil 类中的 'leastMostToIntArray' 方法变为 public，该方法接受两个 long 类型的参数并返回一个 int[]
public net.minecraft.core.UUIDUtil m_235872_(JJ)[I #leastMostToIntArray
```

[specs]: https://github.com/MinecraftForge/AccessTransformers/blob/master/FMLAT.md
[jvmdescriptors]: https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3.2
