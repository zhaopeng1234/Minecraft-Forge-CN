开始使用Forge
==========================
如果您以前从未开发过Forge mod，本页告诉您如何开始设置一个最基础的Forge开发环境。（译者注：其实我更推荐你用idea的[一个插件](https://mcdev.io/)，能使开发过程更加简单。以下仍然使用原始教程）


预先准备
-------------

* 安装JDK**17**的**64**位版本。Forge官方推荐并支持[Eclipse Temurin][jdk]。请确保您使用的是64位JVM。检查方法是在命令行中执行`java -version`命令。使用32位JVM会在使用[ForgeGradle]时遇到一些问题
* 熟悉一种集成开发环境（IDE）
  * 建议使用集成了Gradle的IDE。

从零开始开发模组
--------------------

1. 从[Forge][files]网站上下载模组开发包（MDK）。在左侧选择对应的Minecraft版本，然后点击页面上的'Mdk'按钮。在接下来的页面上等待几秒钟，然后点击右上角的'Skip'按钮。建议尽可能下载最新版本的 Forge。  
1. 将下载的MDK解压到一个空目录里，这将是你的mod目录，包含一些Gradle文件和一个包含示例mod的`/src`子目录。一些文件可以在不同mod之间重复使用，包括：  
* `/gradle` 子目录  
* `build.gradle`  
* `gradlew`  
* `gradlew.bat`  
* `settings.gradle`
`/src` 子目录不需要在不同的工作空间之间复制;然而，在稍后创建java (`src/main/java`)和resource (`src/main/resources`)目录后，你也许需要刷新Gradle项目。

1. 打开你选择的IDE:  
    * Forge仅明确支持在Eclipse和IntelliJ IDEA上的开发, 但是仍然有可供Visual Studio Code使用的额外运行配置。如果你愿意的话，也可以使用Apache NetBeans或者Vim / Emacs等开发环境。  
    * Eclipse和IntelliJ IDEA都默认集成有Gradle，会在你导入或打开项目后处理剩余的工作空间配置工作。包括从Mojang，MinecraftForge，以及其他地方下载必要的包。而Visual Studio Code需要下载'Gradle for Java'插件。  
    * 在几乎所有相关文件改动后（例如`build.gradle`，`settings.gradle`，等等），需要调用Gradle用于重新评估项目。某些IDE会有'刷新' 按钮; 或者，也可以在命令行中使用`gradlew`命令。

1. 运行对应的Gradle任务，为你选择的IDE生成运行配置:
    * **Eclipse**: 运行`genEclipseRuns`任务。
    * **IntelliJ IDEA**: 运行`genIntellijRuns`任务。如果出现"module not specified"的报错, 设置[`ideaModule`属性][config]的值为你的'main'模块(一般是`${project.name}.main`)。
    * **Visual Studio Code**: 运行`genVSCodeRuns`任务。
    * **其他IDE**: 你可以直接运行`gradle run*`任务，例如`runClient`, `runServer`, `runData`, `runGameTestServer`等。  

自定义你的mod信息
--------------------------------

Edit the `build.gradle` file to customize how your mod is built (e.g., file name, artifact version, etc.).

!!! important
    Do **not** edit the `settings.gradle` unless you know what you are doing. The file specifies the repository that [ForgeGradle] is uploaded to.

### Recommended `build.gradle` Customizations

#### Mod Id Replacement

Replace all occurrences of `examplemod`, including [`mods.toml` and the main mod file][modfiles] with the mod id of your mod. This also includes changing the name of the file you build by setting `base.archivesName` (this is typically set to your mod id).

```gradle
// In some build.gradle
base.archivesName = 'mymod'
```

#### Group Id

The `group` property should be set to your [top-level package][packaging], which should either be a domain you own or your email address:

Type      | Value             | Top-Level Package
:---:     | :---:             | :---
Domain    | example.com       | `com.example`
Subdomain | example.github.io | `io.github.example`
Email     | example@gmail.com | `com.gmail.example`

```gradle
// In some build.gradle
group = 'com.example'
```

The packages within your java source (`src/main/java`) should also now conform to this structure, with an inner package representing the mod id:

```text
com
- example (top-level package specified in group property)
  - mymod (the mod id)
    - MyMod.java (renamed ExampleMod.java)
```

#### Version

Set the `version` property to the current version of your mod. We recommend using a [variation of Maven versioning][mvnver].

```gradle
// In some build.gradle
version = '1.20-1.0.0.0'
```

### Additional Configurations

Additional configurations can be found on the [ForgeGradle] docs.

Building and Testing Your Mod
-----------------------------

1. To build your mod, run `gradlew build`. This will output a file in `build/libs` with the name `[archivesBaseName]-[version].jar`, by default. This file can be placed in the `mods` folder of a Forge-enabled Minecraft setup or distributed.
1. To run your mod in a test environment, you can either use the generated run configurations or use the associated tasks (e.g. `gradlew runClient`). This will launch Minecraft from the run directory (default 'run') along with any source sets specified. The default MDK includes the `main` source set, so any code written in `src/main/java` will be applied.
1. If you are running a dedicated server, whether through the run configuration or `gradlew runServer`, the server will initially shut down immediately. You will need to accept the Minecraft EULA by editing the `eula.txt` file in the run directory. Once accepted, the server will load, which can then be accessed via a direct connect to `localhost`.

!!! note
    You should always test your mod in a dedicated server environment. This includes [client-only mods][client] as they should not do anything when loaded on the server.

[jdk]: https://adoptium.net/temurin/releases?version=17 "Eclipse Temurin 17 Prebuilt Binaries"
[ForgeGradle]: https://docs.minecraftforge.net/en/fg-6.x

[files]: https://files.minecraftforge.net "Forge Files distribution site"
[config]: https://docs.minecraftforge.net/en/fg-6.x/configuration/runs/

[modfiles]: ./modfiles.md
[packaging]: ./structuring.md#packaging
[mvnver]: ./versioning.md
[client]: ../concepts/sides.md#writing-one-sided-mods
