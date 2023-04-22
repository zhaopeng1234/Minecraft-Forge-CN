开始使用Forge
==========================

这是一个简单的指南，可以让您从无到有创建一个基本的模组。接下来是详细的步骤。（译者注：其实我更推荐你用idea的[一个插件](https://mcdev.io/),能够省掉很多事）

从0开始做mod
--------------------

1. 安装jdk**17**的**64**位版本。Minecraft和MinecraftForge都是使用Java 17编译的，因此需要特定版本的java环境。使用32位的JVM会导致在运行后续的gradle任务时出现一些问题。你可以从[Eclipse Adoptium][jdk]获取openjdk（译者注：或者从oracle官网获取标准版的）。  

2. 从[Forge的网站][files]获得Mod Development Kit（MDK），注意对应的minecraft版本。  

3. 将下载的MDK解压到一个空目录中。你应该看到一些文件和`src/main/java`目录中的一个示例mod。这些文件中只有少数是MOD开发必须使用的，你可以在你所有的项目中重复使用这些文件，包括：   
*    `build.gradle`
*    `gradlew.bat`
*    `gradlew`
*    `gradle` 文件夹

4. 将上面列出的文件移到一个新的文件夹。这将是你的mod项目文件夹。   

5. 选择你的IDE:  
*    Forge只明确地支持用Eclipse开发，但有其他的gradle task用于IntelliJ IDEA或Visual Studio Code环境。然而，不管是Netbeans或者vim/emacs环境，都能够设法使用。
*    对于Intellij IDEA和Eclipse来说，它们的Gradle集成将处理其余的初始工作区设置。这包括从Mojang、MinecraftForge和其他一些软件源下载依赖包。
*    对于VSCode，"Gradle Tasks"插件可以用来处理初始工作区的设置。
*    为了使大多数对build.gradle文件的修改生效，需要调用Gradle来重新加载项目。这可以通过前面提到的两个IDE的Gradle面板上的“刷新”按钮来完成。
*    （译者注：gradle的版本不是越新越好，必须低于8.0。另外，由于国内的网络环境，最好配置代理或者阿里云代码仓库）   

6. 生成IDE启动/运行配置:
*    对于Eclipse，在cmd中当前目录下运行`gradlew genEclipseRuns`命令。这将生成启动配置并下载游戏运行所需的任何资源。完成后，刷新你的项目。
*    对于IntelliJ，在cmd中当前目录下运行`gradlew genIntellijRuns`命令。这将生成运行配置并下载游戏运行所需的任何资源。如果你遇到“未指定模块”的错误，你可以编辑配置来选择你的主模块，或者通过`ideaModule`属性指定它。
*    对于VSCode，cmd中当前目录下运行`gradlew genVSCodeRuns`命令。这将生成启动配置并下载游戏运行所需的任何资源。


自定义你的mod信息
--------------------------------

编辑`build.gradle`文件以定制你的mod的构建方式（文件名、版本和其他东西）。

!!! 重要   
    **不要**编辑build.gradle文件中的`buildscript {}`部分，其默认文本对于ForgeGradle的运行是必要的。

几乎任何在`// Only edit below this line, the above code adds and enables the necessary things for Forge to be setup.`标记下面的内容都可以修改。许多东西也可以在那里删除或自定义。

### 简单的 `build.gradle` 自定义

强烈建议所有mod项目都进行这些自定义。 
* 要改变你建立的文件的名称--请编辑`archivesBaseName`的值。
* 要改变项目的 "maven坐标"--请编辑`group`的值。
* 要改变mod的版本号--请编辑`version`的值。
* 要更新运行配置--将所有出现的`examplemod`改为你的mod id。

### 迁移到Mojang的官方映射上

Forge将在可预见的未来使用Mojang的官方映射，或MojMaps。官方映射包括类、方法和字段名，不包含参数和javadocs。目前，我们不能保证这些映射在法律上是安全的；但是，由于Mojang希望我们使用它，Forge决定好意地接受这个提议。你可以参考 [Forge的立场][mojmap].


构建和测试你的模型
-----------------------------

1. 为了构建你的mod，运行`gradlew build`。这将在`build/libs`中输出一个文件，名称为`[archivesBaseName]-[version].jar`。这个文件可以放在支持Forge的Minecraft客户端的`mods`文件夹中设置或分发。    
2. 要测试运行你的mod，最简单的方法是使用你初始化项目时生成的运行配置。或者，你可以运行`gradlew runClient`。这将从`<runDir>`位置启动Minecraft和你的mod代码。mod源代码的位置可以被指定，默认情况下，MDK会导入`src/main/java`文件夹下的源代码。    
3. 你也可以使用服务器运行配置或通过`gradlew runServer`运行一个专用服务器。这将启动带有GUI的Minecraft服务器。在第一次运行后，服务器将立即关闭，直到编辑`run/eula.txt`接受Minecraft EULA。接受EULA之后，服务器将被加载，并可以通过`localhost`直连访问。    

!!! 注意
   如果你的mod打算在服务器环境下运行，建议你始终使用服务器运行配置进行测试。
    
[files]: https://files.minecraftforge.net "Forge Files distribution site"
[jdk]: https://adoptium.net/temurin/releases "Temurin Prebuilt Binaries"
[mojmap]: https://github.com/MinecraftForge/MCPConfig/blob/master/Mojang.md
