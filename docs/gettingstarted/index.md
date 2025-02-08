Forge入门
==========================

如果您以前从未制作过基于Forge的Mod，本页面将提供准备Forge开发环境所需的基本信息。 以下是具体步骤：

准备环境
-------------
* 安装`JDK21`和**64位**JVM。Forge官方推荐并支持[Eclipse Temurin][jdk]。(译者注：官方文档中写的JDK版本是17，但是如果你是为MC 1.20.5+ 开发mod,需要将jdk版本设置为21)
    !!! 警告
    确保您使用的是 64 位 JVM。检查方法是在终端中运行`java -version`。使用 [ForgeGradle] 时，32 位 JVM 会导致一些问题。

* 安装你熟悉的IDE，建议使用集成了Gradle的IDE。(译者注：本人后续的示例均为在IDEA 2024.3.1.1版本, gradle8.1.1下操作,较低的idea版本对高版本的jdk支持有问题)

从零开始制作Mod
--------------------
1. 从[Forge文件站][files]下载 Mod 开发包 (MDK)，点击页面上的"Mdk"按钮，等待一段时间后点击页面右上角的  "Skip"按钮。建议尽可能下载最新版本的 Forge。(译者注：本人使用`1.20.6-50.1.0`版本)

2.  将下载的MDK解压缩到一个空目录中。这将是您的 mod 目录，其中包含一些 gradle 文件和一个包含示例 mod代码的 `/src` 子目录。
 !!! 注意 以下文件可以在不同的 Mod 中重复使用：
    * `gradle` 子目录
    * `build.gradle`
    * `gradlew`
    * `gradlew.bat`
    * `settings.gradle`<br/>
	其中，`/src`子目录无需在不同的工作区之间复制；然而在创建了java (`src/main/java`)和resource (`src/main/resources`)目录之后，需要刷新Gradle项目。

3. 在你的 IDE中打开项目:
    * Forge仅提供了在 Eclipse 和 IntelliJ IDEA 上的官方支持，但也为 VS Code 准备了运行配置。另外，也可以使用其它的IDE，例如 Apache NetBeans，Vim / Emacs 等。
    * Eclipse 和 IntelliJ IDEA 默认包含的 Gradle 集成，将在导入或打开项目时自动处理初始的工作区设置。包括从 Mojang、MinecraftForge 等网站下载必要的依赖包。VS Code 则需要 "Gradle for Java"插件来完成该工作。
    * 在IDE完成对Gradle相关文件（例如`build.gradle`、`settings.gradle`等）的加载后，需要调用Gradle以重新加载项目。一些IDE带有"刷新"按钮来完成此操作；也可以在终端中使用`gradlew`命令来完成此操作。
    * 译者注：国内开发者可能需要使用[本地代理][proxy]或者是添加[镜像][mirror]以进行依赖包的下载，推荐使用前一种方式。
4. 在所选 IDE中为项目生成运行配置：
    * **Eclipse**: 运行`genEclipseRuns`任务。
    * **IntelliJ IDEA**: 运行 `genIntellijRuns` 任务。 如果出现 "module not specified"错误，请将[`ideaModule`属性][config]的值设置为你的主模块名称（通常为`${project.name}.main`）。
    * **Visual Studio Code**: 运行`genVSCodeRuns`任务。
    * **其它 IDE**: 可以使用 `gradle run*` 直接运行任务(例如，`runClient`、`runClient`、`runServer`、 `runData`、`runGameTestServer`)。在上面的IDE中也可以使用这些命令。

自定义您的Mod信息
--------------------------------
编辑`build.gradle`文件以自定义构建 MOD 的方式(例如文件名、工件版本等)。中文开发者在使用jdk18以上版本时，会遇到控制台输出中文显示为乱码的情况，可以在build.gradle中加入以下代码解决：
```
tasks.withType(JavaExec).configureEach {
    jvmArgs += '-Dstdout.encoding=UTF-8'
    jvmArgs += '-Dstderr.encoding=UTF-8'
}
```

!!! 重要
除非您明确知道自己在做什么，否则请**不要**编辑`settings.gradle`文件。 该文件指定了 [ForgeGradle] 使用的版本库。
   

### 推荐的`build.gradle`自定义选项。

#### Mod ID

将gradle.properties中的 `mod_id`属性设置为您的 mod id，这将自动同步到大部分的配置文件中。同时还需要更改mod主文件`ExampleMod.class`中的MODID常量。另外，还可以在build.gradle文件中设置`base.archivesName`属性(默认为您的mod id)，更改打包后的mod文件的名称。


#### Group ID

将gradle.properties中的`mod_group_id` 属性设置为您的[顶层软件包名][packaging]，这应该是您拥有的域名或您的电子邮件地址，如下所示：

类型      | 值             | 顶层软件包名
:---:     | :---:             | :---
域名    | <span>example.com</span>       | `com.example`
子域名  | <span>example.github.io</span> | `io.github.example`
电子邮箱 | example<span>@gmail.com</span> | `com.gmail.example`

您的 java 源代码(`src/main/java`)中的包名也应符合此结构，内部的包名应该与mod id相同。
 `src/main/java`文件夹中应该如下所示：
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

1. 要编译您的 MOD，请运行build任务(或使用`gradlew build`命令) 。这将在`build/libs`文件夹中生成一个文件，默认名称为`[archivesBaseName]-[version].jar`。 该文件可放置在使用 Forge 的 Minecraft 客户端的`mods` 文件夹中使用或在网上进行分发。

2. 要在测试环境中运行您的 MOD，可以使用自动生成的任务配置或使用相关命令(如gradlew runClient)。这将在运行目录下（默认为 "`/run`"）启动 Minecraft，加载指定的其它资源文件。默认情况下，MDK会加载`main`目录下的资源，因此在`src/main/java`中编写的任何代码都将被加载。

3. 如果您通过运行任务配置或使用 `gradlew runServer`命令，第一次启动专用服务器，服务器将在启动后立即关闭。 您需要通过编辑运行目录中的`eula.txt`文件来接受 Minecraft EULA。之后，再次启动服务器，服务器将正常启动，之后可以通过直连 `localhost`访问服务器。

!!! 注意
    您应始终在专用服务器环境中测试您的 Mod。 这也包括[客户端专用mod][client]，因为它们在服务器上加载时不应执行任何操作。

[jdk]: https://adoptium.net/temurin/releases?version=21 "Eclipse Temurin 21 Prebuilt Binaries"
[ForgeGradle]: https://docs.minecraftforge.net/en/fg-6.x
[files]: https://files.minecraftforge.net "Forge Files distribution site"
[config]: https://docs.minecraftforge.net/en/fg-6.x/configuration/runs/
[modfiles]: ./modfiles.md
[packaging]: ./structuring.md#软件包管理
[mvnver]: ./versioning.md
[client]: ../concepts/sides.md#编写单端mod
[proxy]: https://jingyan.baidu.com/article/75ab0bcbbcac3197874db240.html
[mirror]: https://www.cnblogs.com/amadeuslee/p/17953008
