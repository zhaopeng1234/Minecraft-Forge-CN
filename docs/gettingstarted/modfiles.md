Mod文件（Mod Files）
=========
mod文件负责决定将哪些mod打包到你的JAR包中，在"Mods"菜单中显示哪些信息，以及如何在游戏中加载你的mod。


mods.toml
---------
`mods.toml`文件定义了你的mod的元数据。它也包含了在"Mods"菜单中显示的额外信息，以及在游戏中加载mod的方式。

该文件使用[TOML][toml]格式。该文件必须存放在你的资源目录（`src/main/resources`）下的`META-INF`文件夹中，`mods.toml`文件看起来是这样的：

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

`mods.toml`文件被分为三部分：非特定mod的属性，这些属性与mod文件相关；mod属性，每个mod各有一段；以及依赖配置，每个mod或mod的依赖各有一段。下面将详细解释`mods.toml`文件中的各个属性，其中标为`强制`的属性表示必须指定一个值，否则将抛出异常。

### 非特定mod的属性（Non-mod-specific properties）

非特定mod的属性是和jar包自身相关的属性，指示mod如何加载，以及一些额外的全局元数据

Property             | Type    | Default       | Description | Example
:---                 | :---:   | :---:         | :---:       | :---
`modLoader`          | string  | **强制** | mod使用的语言加载器，可以用于支持多种语言结构，例如mod主文件使用的Kotlin对象，或者定义了不同入口点的方法，例如接口或方法，forge提供了Java语言加载器`"javafml"`和低代码加载器`"lowcodefml"` | `"javafml"`
`loaderVersion`      | string  | **强制** | 可接受的语言加载器的版本范围，用[Maven版本范围][mvr]表示。对于`javafml`和`lowcodefml`，该版本号是Forge版本的主版本号| `"[46,)"`
`license`            | string  | **强制** | 本jar包中的mod所使用的许可证。建议使用[SPDX标识符][spdx] 和/或 指向你所用的许可证的链接。你可以访问https://choosealicense.com/ 帮助你选择你要使用的许可证 | `"MIT"`
`showAsResourcePack` | boolean | `false`       | 如果值是`true`，多个mod的资源会在“资源包”菜单中被显示为独立的资源包，而不是组合成一个“mod资源”包。 | `true`
`services`           | array   | `[]`          | 你的mod**使用**的服务的数组。在Forge对JAVA模块化系统（JPMS）的实现中，会将这些服务作为为mod模块中的一部分加载。 | `["net.minecraftforge.forgespi.language.IModLanguageProvider"]`
`properties`         | table   | `{}`          | 替换属性表。`StringSubstitutor`使用表中对应的值来替换`${file.<key>}`。这目前只用于替换[mod属性][modsp]中的`version`属性 | `{ "example" = "1.2.3" }` 用于替换 `${file.example}`
`issueTrackerURL`    | string  | *空*     | 一个用于报告和追踪mod问题的网址。 | `"https://forums.minecraftforge.net/"`

!!! 重要
  `services`属性在功能上和使用模块的[`uses`][uses]指令是相同的，这允许程序[*加载*][serviceload]一个指定类型的服务。  

### mod属性（Mod-Specific Properties）

mod属性使用`[[mods]]`标头与指定的模块绑定。这是一个[表数组][array]：直到下一个标头为止的所有键值对，都将作为该mod的属性。


```toml
# examplemod1的属性
[[mods]]
modId = "examplemod1"

# examplemod2的属性
[[mods]]
modId = "examplemod2"
```

Property        | Type    | Default                 | Description | Example
:---            | :---:   | :---:                   | :---:       | :---
`modId`         | string  | **强制**           | 代表mod的唯一标识符，id必须符合`^[a-z][a-z0-9_]{1,63}$`的规则：以小写字母开头，由2至64个小写字母/数字/下划线组成。 | `"examplemod"`
`namespace`     | string  | `modId`属性的值        | 覆盖mod的命名空间。必须符合`^[a-z][a-z0-9_.-]{1,63}$`的规则：以小写字母开头，由2至64个小写字母/数字/下划线/./- 组成。目前没有使用。 | `"example"`
`version`       | string  | `"1"`                   | mod的版本号，推荐使用[Maven式版本号][mvnver]。如果设置为`${file.jarVersion}`，会被替换为JAR包中manifest文件的`Implementation-Version`属性（在开发环境中显示为`0.0NONE`）。 | `"1.20-1.0.0.0"`
`displayName`   | string  | `modId`属性的值        | 更易读的mod名称。用于在mod列表中或者mod不匹配时展示。 | `"Example Mod"`
`description`   | string  | `"MISSING DESCRIPTION"` | 在mod列表页面中展示的mod说明。推荐使用[多行字符串文字][multiline]。 | `"This is an example."`
`logoFile`      | string  | *空*               | 在mod列表中使用的图片的文件名和后缀。logo必须在JAR包或者资源目录（`src/main/resources`）的根目录下。 | `"example_logo.png"`
`logoBlur`      | boolean | `true`                  | 使用`GL_LINEAR*`（true）还是`GL_NEAREST*`（false）渲染`logoFile`。 | `false`
`updateJSONURL` | string  | *nothing*               | 指向一个JSON文件的网址，[更新检查器][update]用该文件确保你正在使用最新版本的mod。 | `"https://files.minecraftforge.net/net/minecraftforge/forge/promotions_slim.json"`
`features`      | table   | `{}`                    | 参照 “[features]” | `{ java_version = "17" }`
`modproperties` | table   | `{}`                    | 与该mod相关的一个键/值对列表。目前Forge没有使用，主要是给mod用。 | `{ example = "value" }` 
`modUrl`        | string  | *空*               | 指向mod下载地址的网址。目前没有使用。 | `"https://files.minecraftforge.net/"`
`credits`       | string  | *空*               | 在mod列表页面显示的功劳和鸣谢 | `"The person over here and there."`
`authors`       | string  | *空*               | 在mod列表页面显示的作者列表 | `"Example Person"`
`displayURL`    | string  | *空*               | 在mod列表页面显示的mod主页 | `"https://minecraftforge.net/"`
`displayTest`   | string  | `"MATCH_VERSION"`       | 参照“[sides]” | `"NONE"`

#### Features

The features system allows mods to demand that certain settings, software, or hardware are available when loading the system. When a feature is not satisfied, mod loading will fail, informing the user about the requirement. Currently, Forge provides the following features:

Feature        | Description | Example
:---:          | :---:       | :---
`java_version` | The acceptable version range of the Java version, expressed as a [Maven Version Range][mvr]. This should be the supported version used by Minecraft. | `"[17,)"`

### Dependency Configurations

Mods can specify their dependencies, which are checked by Forge before loading the mods. These configurations are created using the [array of tables][array] `[[dependencies.<modid>]]` where `modid` is the identifier of the mod the dependency is for.

Property       | Type    | Default       | Description | Example
:---           | :---:   | :---:         | :---:       | :---
`modId`        | string  | **mandatory** | The identifier of the mod added as a dependency. | `"example_library"`
`mandatory`    | boolean | **mandatory** | Whether the game should crash when this dependency is not met. | `true`
`versionRange` | string  | `""`          | The acceptable version range of the language loader, expressed as a [Maven Version Range][mvr]. An empty string matches any version. | `"[1, 2)"`
`ordering`     | string  | `"NONE"`      | Defines if the mod must load before (`"BEFORE"`) or after (`"AFTER"`) this dependency. If the ordering does not matter, return `"NONE"` | `"AFTER"`
`side`         | string  | `"BOTH"`      | The [physical side][dist] the dependency must be present on: `"CLIENT"`, `"SERVER"`, or `"BOTH"`.| `"CLIENT"`
`referralUrl`  | string  | *nothing*     | A URL to the download page of the dependency. Currently unused. | `"https://library.example.com/"`

!!! warning
    The `ordering` of two mods may cause a crash due to a cyclic dependency: for example, mod A must load `"BEFORE"` mod B and mod B `"BEFORE"` mod A.

Mod Entrypoints
---------------

Now that the `mods.toml` is filled out, we need to provide an entrypoint to being programming the mod. Entrypoints are essentially the starting point for executing the mod. The entrypoint itself is determined by the language loader used in the `mods.toml`.

### `javafml` and `@Mod`

`javafml` is a language loader provided by Forge for the Java programming language. The entrypoint is defined using a public class with the `@Mod` annotation. The value of `@Mod` must contain one of the mod ids specified within the `mods.toml`. From there, all initialization logic (e.g., [registering events][events], [adding `DeferredRegister`s][registration]) can be specified within the constructor of the class. The mod bus can be obtained from `FMLJavaModLoadingContext`.

```java
@Mod("examplemod") // Must match mods.toml
public class Example {

  public Example() {
    // Initialize logic here
    var modBus = FMLJavaModLoadingContext.get().getModEventBus();

    // ...
  }
}
```

### `lowcodefml`

`lowcodefml` is a language loader used as a way to distribute datapacks and resource packs as mods without the need of an in-code entrypoint. It is specified as `lowcodefml` rather than `nocodefml` for minor additions in the future that might require minimal coding.

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
