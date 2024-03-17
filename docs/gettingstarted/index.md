开始使用Forge
==========================
如果您以前从未开发过Forge mod，本页告诉您如何开始设置一个最基础的Forge开发环境。（译者注：其实我更推荐你用idea的[这个](https://mcdev.io/)插件，能使开发过程更加简单。以下仍然使用原始教程）  

预先准备
-------------
* 安装JDK**17**的**64**位版本。Forge官方推荐并支持[Eclipse Temurin][jdk]。请确保您使用的是64位JVM，检查方法是在命令行中执行`java -version`命令。使用32位JVM会在使用[ForgeGradle]时遇到一些问题。  
* 熟悉一种集成开发环境（IDE）  
  * 建议使用内置了Gradle的IDE。

从零开始开发模组
--------------------
1. 从[Forge][files]网站上下载模组开发包（MDK）。在左侧选择对应的Minecraft版本，然后点击页面上的'Mdk'按钮。在接下来的页面上等待几秒钟，然后点击右上角的'Skip'按钮。建议尽可能下载最新版本的 Forge。  
2. 将下载的MDK解压到一个空目录里，这将是你的mod目录，包含一些Gradle文件和一个包含示例mod的`src`子目录。一些文件可以在不同mod之间重复使用，包括：
  * `gradle` 子目录  
  * `build.gradle`  
  * `gradlew`  
  * `gradlew.bat`  
  * `settings.gradle`  
`src` 子目录不需要在不同的工作空间之间复制。然而，在稍后创建java (`src/main/java`)和resource (`src/main/resources`)目录后，你需要刷新Gradle项目。  
3. 打开你选择的IDE:  
    * Forge仅明确支持在Eclipse和IntelliJ IDEA上的开发，但是仍然有可供Visual Studio Code使用的额外运行配置。如果你愿意的话，也可以使用Apache NetBeans或者Vim / Emacs等开发环境。  
    * Eclipse和IntelliJ IDEA都默认集成有Gradle，会在你导入或打开项目后处理剩余的工作空间配置工作。包括从Mojang，MinecraftForge，以及其他地方下载必要的包。而Visual Studio Code需要下载'Gradle for Java'插件。  
    * 在几乎所有相关文件改动后（例如`build.gradle`，`settings.gradle`，等等），需要调用Gradle用于重新评估项目。某些IDE会有'刷新' 按钮；或者，也可以在命令行中使用`gradlew`命令。  
4. 运行对应的Gradle任务，为你选择的IDE生成运行配置：
    * **Eclipse**: 运行`genEclipseRuns`任务。  
    * **IntelliJ IDEA**: 运行`genIntellijRuns`任务。如果出现"module not specified"的报错, 设置[`ideaModule`][config]属性的值为你的'main'模块(一般是`${project.name}.main`)。  
    * **Visual Studio Code**: 运行`genVSCodeRuns`任务。
    * **其他IDE**: 你可以直接运行`gradle run*`任务，例如`runClient`, `runServer`, `runData`, `runGameTestServer`等。  

自定义你的mod信息
--------------------------------
编辑build.gradle文件以定制你的mod的构建方式，例如文件名、版本号等。    
!!! 重要
**不要**编辑`settings.gradle`文件，除非你明确知道你在做什么。该文件指定了[ForgeGradle]所使用的版本库。

### 建议自定义的 `build.gradle` 配置

#### 修改mod Id
替换所有的`examplemod`字符串，包括[`mods.toml`][modfiles]和[mod主文件][modfiles]。另外还要设置`base.archivesName`的值以指定编译生成的文件名。（译者注：你可以看完下一章了解了文件结构后再回来修改）  

```gradle
// 在build.gradle文件中
base {
    archivesName = 'mymod'
}
```

#### 修改Group Id
`group`属性应该被设置为你的[顶级包名][packaging]，对应你拥有的一个域名或者你的电子邮箱地址：  

类型      | 值             | 顶级包名
:---:     | :---:             | :---
域名    | example.com       | `com.example`
子域名 | example.github.io | `io.github.example`
电子邮箱     | example@gmail.com | `com.gmail.example`

```gradle
// 在build.gradle文件中
group = 'com.example'
```

你的java源文件目录（`src/main/java`）也需要符合这个结构，然后使用你的mod id作为下一级目录：

```text
com
- example (你在group属性中设置的顶级包名)
  - mymod (mod id)
    - MyMod.java (重命名ExampleMod.java)
```

#### 版本号

设置`version`属性为你的mod的当前版本号。推荐使用[Maven式版本号][mvnver]。

```gradle
// 在build.gradle文件中
version = '1.20-1.0.0.0'
```

### 其他配置

其他的配置可以在[ForgeGradle]文档中找到。  

构建和测试你的mod
-----------------------------
1. 要构建你的mod，运行`gradlew build`。这会在`build/libs`目录中输出一个以`[archivesBaseName]-[version].jar`命名的文件。这个文件可以放在支持Forge的Minecraft客户端的mods文件夹中加载或分发。  
2. 要在测试环境中运行你的mod，你可以使用初始化项目时自动生成的运行配置。或者运行对应的任务例如`gradlew runClient`。这将从运行目录（默认`run`）启动Minecraft和你的mod代码。mod源代码的位置可以被指定，默认情况下，MDK会使用src/main/java文件夹下的源代码。  
3. 如果你要运行一个专用服务器，可以使用运行配置或使用`gradlew runServer`命令。在第一次运行时，服务器将立即关闭，你需要接受Minecraft EULA条款并编辑运行目录下的`eula.txt`。接受EULA之后，服务器将被加载，并可以通过localhost直连访问。  

!!! 注意
    你应该始终在专用服务器环境中测试你的mod。包括[仅在客户端生效的mod][client]，因为它们在服务器上加载时不应该做任何事情。  


[jdk]: https://adoptium.net/temurin/releases?version=17 "Eclipse Temurin 17 Prebuilt Binaries"
[ForgeGradle]: https://docs.minecraftforge.net/en/fg-6.x

[files]: https://files.minecraftforge.net "Forge Files distribution site"
[config]: https://docs.minecraftforge.net/en/fg-6.x/configuration/runs/

[modfiles]: ./modfiles.md
[packaging]: ./structuring.md#packaging
[mvnver]: ./versioning.md
[client]: ../concepts/sides.md#writing-one-sided-mods
