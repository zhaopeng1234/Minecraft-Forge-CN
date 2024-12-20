
Forge 入门
==========================

如果您以前从未使用Forge制作过Mod，本节将提供设置Forge开发环境所需的最基本信息。 接下来将介绍具体的步骤。

准备环境
-------------

* 安装`JDK17`和**64位**JVM。Forge官方支持并推荐[Eclipse Temurin][jdk]。

    !!! 警告
    确保您使用的是 64 位 JVM。检查方法是在终端中运行`java -version`。使用 [ForgeGradle] 时，使用 32 位 JVM 会导致一些问题。

* 你熟悉的集成开发环境 (IDE).
    * 建议使用集成了 Gradle 的集成开发环境（译者注：本人后续的示例均为在IDEA 2023.1版本下操作）。

从零开始制作Mod
--------------------

1.  从[Forge 文件站点][files]下载 Mod 开发包 (MDK)，点击"Mdk"按钮，在页面上等待一段时间后点击右上角的  "Skip "按钮。 建议尽可能下载最新版本的 Forge。

2.  将下载的 MDK 解压缩到一个空目录中。 这将是您的 mod 目录，其中包含一些 gradle 文件和一个包含示例 mod 的 `src` 子目录。
!!! 注意 以下文件可以在不同的 Mod 中重复使用：
    * `gradle` 子目录
    * `build.gradle`
    * `gradlew`
    * `gradlew.bat`
    * `settings.gradle`<br/>
`src`子目录无需在workspace之间复制； 但是，如果稍后创建了 java (`src/main/java`) 和resource目录 (`src/main/resources`)，则可能需要刷新 Gradle 项目。

3. 在你的 IDE中打开项目:
    * Forge 仅明确支持在 Eclipse 和 IntelliJ IDEA 上进行开发，但也有在 Visual Studio Code 上使用的运行配置。 另外，从 Apache NetBeans 到 Vim / Emacs 等任何环境都可以使用。
    * Eclipse 和 IntelliJ IDEA 的 Gradle 集成（均已安装并默认启用）将在导入或打开项目时自动处理其余的初始工作区设置。 包括从 Mojang、MinecraftForge 等下载必要的软件包。Visual Studio Code 需要 "Gradle for Java "插件来完成同样的工作。
    * 在IDE完成对Gradle相关的文件（例如，`build.gradle`、`settings.gradle`等）的加载后，需要调用Gradle以重新加载项目。 一些IDE带有 "刷新 "按钮来完成此操作；但也可以在终端中使用`gradlew`命令来完成此操作。
    * 译者注：国内开发者可能需要使用[本地代理][proxy]或者是添加[镜像][mirror]以进行依赖包的下载，推荐使用前一种方式。
4. 在所选 IDE中为项目生成运行配置：
    * **Eclipse**: 运行`genEclipseRuns`任务。
    * **IntelliJ IDEA**: 运行 `genIntellijRuns` 任务。 如果出现 "module not specified"错误，请将[`ideaModule`属性][config]的值设置为你的主模块名称（通常为`${project.name}.main`）。
    * **Visual Studio Code**: 运行`genVSCodeRuns`任务。
    * **Other IDEs**: 您可以使用 `gradle run*` 直接运行配置（例如，`runClient`）、 `runClient`、`runServer`、 `runData`、`runGameTestServer`）。 这些命令也可以在受支持的IDE中使用。

自定义您的Mod信息
--------------------------------

编辑`build.gradle`文件以自定义构建 MOD 的方式（例如文件名、工件版本等）。

!!! 重要
请**不要**编辑`settings.gradle`文件，除非您明确知道自己在做什么。 该文件指定了 [ForgeGradle] 使用的版本库。
   

### 推荐的`build.gradle`自定义选项。

#### Mod ID

将gradle.properties中的 `mod_id`属性设置为您的 mod id。另外，还可以在build.gradle文件中设置`base.archivesName`属性来更改您创建的文件的名称（默认为您的mod id）。

#### Group ID

将gradle.properties中的`mod_group_id` 属性设置为您的[顶层软件包名][packaging]，这应该是您拥有的域名或您的电子邮件地址：

类型      | 值             | 顶层软件包
:---:     | :---:             | :---
域名    | example.com       | `com.example`
子域名  | example.github.io | `io.github.example`
电子邮箱 | example@gmail.com | `com.gmail.example`

您的 java 源代码（`src/main/java`）中的包名也应符合此结构，内部的包名与mod id相同：
 `src/main/java`：
```text
com
- example (mod_group_id属性中指定的顶层软件包)
  - mymod (mod id)
    - MyMod.java (重命名自ExampleMod.java)
```

#### Version

将gradle.properties中的`mod_version`属性设置为 MOD 的当前版本。 我们建议使用 [Maven 版本控制][mvnver]。


#### 其他配置

其他配置可参见 [ForgeGradle] 文档。

构建和测试您的模块
-----------------------------

1. 要编译您的 MOD，请运行build任务（或`gradlew build`命令） 。 这将在`build/libs`中输出一个文件，默认名称为`[archivesBaseName]-[version].jar`。 此文件可放置在使用 Forge 的 Minecraft 客户端的`mods` 文件夹中或在网上进行分发。
2. 要在测试环境中运行您的 MOD，可以使用自动生成的运行配置或运行相关任务（如 gradlew runClient）。 这将在运行目录下（默认为 "`/run`"）加载指定的资源启动 Minecraft。默认情况下MDK会加载`main`目录下的源代码，因此在`src/main/java`中编写的任何代码都将被应用。
3. 如果您通过运行配置或 `gradlew runServer` 第一次运行专用服务器，服务器将立即关闭。 您需要通过编辑运行目录中的`eula.txt`文件来接受 Minecraft EULA。 接受后，服务器将加载，然后可以通过直连 `localhost`访问服务器。

!!! 注意
    您应始终在专用服务器环境中测试您的 Mod。 这也包括[客户端专用mod][客户端]，因为它们在服务器上加载时不应执行任何操作。

[jdk]: https://adoptium.net/temurin/releases?version=17 "Eclipse Temurin 17 Prebuilt Binaries"
[ForgeGradle]: https://docs.minecraftforge.net/en/fg-6.x

[files]: https://files.minecraftforge.net "Forge Files distribution site"
[config]: https://docs.minecraftforge.net/en/fg-6.x/configuration/runs/

[modfiles]: ./modfiles.md
[packaging]: ./structuring.md#packaging
[mvnver]: ./versioning.md
[client]: ../concepts/sides.md#writing-one-sided-mods

[proxy]: https://jingyan.baidu.com/article/75ab0bcbbcac3197874db240.html
[mirror]: https://www.cnblogs.com/amadeuslee/p/17953008
