
Mod文件
=========

Mod文件决定了哪些Mod会被打包到最终的jar包中、在"Mod"菜单中显示哪些信息，以及如何在游戏中加载您的Mod。

mods.toml
---------

`mods.toml`文件中定义了您的Mod的元数据。 它还包含 "Mods "菜单中显示的附加信息，以及如何将您的 MOD 载入游戏。

该文件使用 [Tom's Obvious Minimal Language，即 TOML][toml] 格式。 该文件必须存储在您源代码的资源目录下的 `META-INF` 文件夹中（`src/main/resources/META-INF/mods.`）。 
`mods.toml`文件内容大致如下所示：

```toml
modLoader="javafml"
loaderVersion="[46,)"

license="All Rights Reserved"
issueTrackerURL="https://github.com/MinecraftForge/MinecraftForge/issues"
showAsResourcePack=false

[[mods]]
  modId="examplemod"
  version="1.0.0.0"
  displayName="Example Mod"
  updateJSONURL="https://files.minecraftforge.net/net/minecraftforge/forge/promotions_slim.json"
  displayURL="https://minecraftforge.net"
  logoFile="logo.png"
  credits="I'd like to thank my mother and father."
  authors="Author"
  description='''
  Lets you craft dirt into diamonds. This is a traditional mod that has existed for eons. It is ancient. The holy Notch created it. Jeb rainbowfied it. Dinnerbone made it upside down. Etc.
  '''
  displayTest="MATCH_VERSION"

[[dependencies.examplemod]]
  modId="forge"
  mandatory=true
  versionRange="[46,)"
  ordering="NONE"
  side="BOTH"

[[dependencies.examplemod]]
  modId="minecraft"
  mandatory=true
  versionRange="[1.20]"
  ordering="NONE"
  side="BOTH"
```

`mods.toml`文件分为三个部分：非Mod特有的属性，这些属性与mod文件相关；Mod特有的属性，每个mod都有各自的一段属性；以及依赖配置，每个mod或mod的依赖都有一段配置。 下面将分别解释`mods.toml` 中的每个属性，其中`mandatory`表示必须指定一个值，否则将出现异常。

### 非Mod特有的属性

非Mod特有的属性是与jar包本身相关的属性，用于配置Mod的加载方式和其他全局元数据。

Property             | Type    | Default       | Description | Example
:---                 | :---:   | :---:         | :---:       | :---
`modLoader`          | string  | **mandatory** | Mod使用的语言加载器。可用于支持其他语言结构，如用于主文件的 Kotlin 对象，或者确定入口点的不同方法，如某个接口或方法。 Forge 提供 Java 加载器`"javafml"`和低代码/无代码加载器`"lowcodefml"`。 | `"javafml"`
`loaderVersion`      | string  | **mandatory** | 语言加载器的可接受版本范围，以[Maven 版本范围][mvr]表示。 对于`javafml`和`lowcodefml`，版本指的是是 Forge 的主要版本。 | `"[46,)"`
`license`            | string  | **mandatory** | 此jar包中的MOD所使用的许可证。 建议将其设置为您正在使用的[SPDX 标识符][spdx]和/或许可证链接。 您可以访问 https://choosealicense.com/ 以选择要使用的许可证。 | `"MIT"`
`showAsResourcePack` | boolean | `false`       | 当`true`时，mod 的资源将作为单独的资源包显示在游戏中的"资源包"菜单上，而不会与"mod资源包"合并。 | `true`
`services`           | array   | `[]`          | 您的模块使用的**服务**列表。这将作为Mod模块的一部分被Forge的Java模块化系统使用。 | `["net.minecraftforge.forgespi.language.IModLanguageProvider"]`
`properties`         | table   | `{}`          | 替换属性表。 `StringSubstitutor`使用该表将 `${file.<key>}` 替换为相应的值。 目前仅用于替换[mod 特有属性][modsp]中的`version`属性。 | `{ "example" = "1.2.3" }` referenced by `${file.example}`
`issueTrackerURL`    | string  | *nothing*     | 报告和跟踪Mod问题的 URL。 | `"https://forums.minecraftforge.net/"`

!!! 重要
   `services`属性在功能上等同于在模块中使用[`uses指令`][uses]，它允许[*加载*][serviceload]给定类型的服务。
  
### Mod特有的属性

Mod特有的属性使用 `[[mods]]` 标头与指定的Mod绑定。 这是一个[表数组][array]；所有键/值属性都将附加到该Mod，直到下一个标头。

```toml
#examplemod1 的属性
[[mods]]
modId = "examplemod1"

#examplemod2 的属性
[[mods]]
modId = "examplemod2"
```

Property        | Type    | Default                 | Description | Example
:---            | :---:   | :---:                   | :---:       | :---
`modId`         | string  | **mandatory**           | 代表此Mod的唯一标识符。 id必须与`^[a-z][a-z0-9_]{1,63}$` 匹配（2-64 个字符的字符串；以小写字母开头；由小写字母、数字或下划线组成）。 | `"examplemod"`
`namespace`     | string  | value of `modId`        | mod 的覆盖命名空间。 命名空间必须与 `^[a-z][a-z0-9_.-]{1,63}$` （2-64 个字符的字符串；以小写字母开头；由小写字母、数字、下划线、点或破折号组成）相匹配。 目前未使用。 | `"example"`
`version`       | string  | `"1"`                   | Mod 的版本，最好使用 [Maven 版本控制的变体][mvnver]。 当设置为`${file.jarVersion}`时，它将被替换为 JAR 清单中`Implementation-Version`属性的值（在开发环境中显示为`0.0NONE`）。 | `"1.20-1.0.0.0"`
`displayName`   | string  | value of `modId`        | Mod的名称。 用于在各个屏幕上显示Mod（例如Mod列表、Mod不匹配）。 | `"Example Mod"`
`description`   | string  | `"MISSING DESCRIPTION"` | Mod列表屏幕中显示的Mod描述。 建议使用[多行字面量字符串][multiline].。 | `"This is an example."`
`logoFile`      | string  | *nothing*               | Mod列表中使用的图像文件的名称和扩展名。Logo必须位于JAR 的根目录下或位于源代码的资源根目录下（`src/main/resources`）。 | `"example_logo.png"`
`logoBlur`      | boolean | `true`                  | 是否使用 `GL_LINEAR*` (true) 或 `GL_NEAREST*` (false) 来显示`logoFile`。 | `false`
`updateJSONURL` | string  | *nothing*               | 指向一个JSON值的 URL， [update checker][update] 使用该 JSON检查您正在玩的 MOD 是否是最新版本。 | `"https://files.minecraftforge.net/net/minecraftforge/forge/promotions_slim.json"`
`features`      | table   | `{}`                    | 请参阅'[特性][features]'. | `{ java_version = "17" }`
`modproperties` | table   | `{}`                    |与此Mod相关的键/值表。目前 Forge 尚未使用，主要供 mod 使用。 | `{ example = "value" }` 
`modUrl`        | string  | *nothing*               | 指向 Mod 下载页面的 URL。 目前未使用。 | `"https://files.minecraftforge.net/"`
`credits`       | string  | *nothing*               | 在Mod列表中显示的Mod的荣誉和鸣谢。 | `"The person over here and there."`
`authors`       | string  | *nothing*               | Mod列表中显示的Mod作者。 | `"Example Person"`
`displayURL`    | string  | *nothing*               | Mod列表中显示的Mod展示页面的 URL。 | `"https://minecraftforge.net/"`
`displayTest`   | string  | `"MATCH_VERSION"`       | 请参阅'[端][sides]'. | `"NONE"`

#### 特性

特性系统允许 mod 在加载时要求某些设置、软件或硬件可用。 当某项功能不满足要求时，mod 载入会失败，并告知用户相关要求。 目前，Forge 提供以下功能：

Feature        | Description | Example
:---:          | :---:       | :---
`java_version` | 可接受的 Java 版本范围，以[Maven 版本范围][mvr]表示。这应该是 Minecraft支持使用的版本。 | `"[17,)"`

### 依赖配置

Mod 可以指定其依赖项，Forge 会在加载 Mod 之前检查这些依赖项。 这些配置使用 [ 表数组][array] `[[dependencies.<modid>]]`其中`modid`是依赖项所依赖的Mod的标识符。

Property       | Type    | Default       | Description | Example
:---           | :---:   | :---:         | :---:       | :---
`modId`        | string  | **mandatory** | 作为依赖项添加的Mod的标识符。 | `"example_library"`
`mandatory`    | boolean | **mandatory** | 当不满足此依赖条件时，游戏是否应该崩溃。 | `true`
`versionRange` | string  | `""`          | 语言加载器可接受的版本范围，以[Maven 版本范围][mvr]表示。 空字符串匹配任何版本。 | `"[1, 2)"`
`ordering`     | string  | `"NONE"`      | 定义mod是否必须在此依赖之前 (`"BEFORE"`) 或之后 (`"AFTER"`) 加载。 如果排序无关紧要，则返回 `"NONE"` | `"AFTER"`
`side`         | string  | `"BOTH"`      | 依赖项必须生效的[物理侧][dist]：`"CLIENT"`、`"SERVER"`、或`"BOTH"`.。| `"CLIENT"`
`referralUrl`  | string  | *nothing*     | 指向依赖项下载页面的 URL。 目前未使用。 | `"https://library.example.com/"`

!!! 警告
    两个模块的`ordering`可能会由于循环依赖关系而导致崩溃： 例如，模式 A 必须加载 `"BEFORE"` 模式 B 和模式 B `"BEFORE"` 模式 A。

Mod 入口点
---------------

现在，`mods.toml`已经填写完毕，我们需要提供一个入口点，以便对 mod 进行编程。 入口点是执行 MOD 的起点。 入口点本身由`mods.toml`中使用的语言加载器决定。

### `javafml` 和 `@Mod`

`javafml` 是 Forge 为 Java 编程语言提供的语言加载器。入口点使用带有 `@Mod` 注解的public类来定义。 `@Mod` 的值必须等于 `mods.toml` 中指定的modid。 从这里开始，所有初始化逻辑（例如，[注册事件][events]，[添加延迟加载][registration]）都可以在类的构造函数中指定。 Mod 总线可从 `FMLJavaModLoadingContext`.获取。

```java
@Mod("examplemod") // 必须和mods.toml中的modId相同
public class Example {

  public Example() {
    // 初始化逻辑
    var modBus = FMLJavaModLoadingContext.get().getModEventBus();

    // ...
  }
}
```

### `lowcodefml`

`lowcodefml` 是一种语言加载器，用于将数据包和资源包作为mod 发布，而无需代码内的入口点。 它被指定为`lowcodefml`，而不是`nocodefml`，是因为未来仍然可能需要少量编码以添加一个版本。

[toml]: https://toml.io/
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
[spdx]: https://spdx.org/licenses/
[modsp]: #mod-specific-properties
[uses]: https://docs.oracle.com/javase/specs/jls/se17/html/jls-7.html#jls-7.7.3
[serviceload]: https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ServiceLoader.html#load(java.lang.Class)
[array]: https://toml.io/en/v1.0.0#array-of-tables
[mvnver]: ./versioning.md
[multiline]: https://toml.io/en/v1.0.0#string
[update]: ../misc/updatechecker.md
[features]: #features
[sides]: ../concepts/sides.md#writing-one-sided-mods
[dist]: ../concepts/sides.md#different-kinds-of-sides
[events]: ../concepts/events.md
[registration]: ../concepts/registries.md#deferredregister
