### 开始贡献
如果你决定为 Forge 做出贡献，需要采取一些特殊步骤来开始开发工作。简单的模组开发环境不足以直接处理 Forge 的代码库。可以参考以下指南来完成设置并开始改进 Forge。

### 分叉和克隆仓库
和大多数主流开源项目一样，Forge 托管在 [GitHub][github] 上。如果你之前为其他项目做过贡献，应该已经熟悉这个流程，可以直接跳到下一部分。

对于初次通过 Git 进行协作的新手，下面有两个简单步骤来开启你的贡献之旅。

!!! 注意
    本指南假设你已经设置好了 GitHub 账户。如果没有，请访问他们的 [注册页面][register] 创建一个账户。此外，本指南不是关于 Git 使用的教程。如果你在使用 Git 时遇到困难，请先参考其他资料。

#### 分叉
首先，你需要通过点击右上角的“Fork”按钮来“分叉” [MinecraftForge 仓库][forgerepo]。如果你属于某个组织，可以选择要托管分叉仓库的账户。

分叉仓库是必要的，因为并非每个 GitHub 用户都能免费访问每个仓库。相反，你需要创建原始仓库的副本，以便稍后通过所谓的拉取请求（Pull Request）贡献你的更改，后面会详细介绍。

#### 克隆
分叉仓库后，就可以在本地进行实际的更改了。为此，你需要将仓库克隆到本地机器上。

使用你喜欢的 Git 客户端，将你的分叉仓库克隆到你选择的目录。作为一个通用示例，下面是一个命令行片段，适用于所有配置正确的系统，它会将仓库克隆到当前目录下名为“MinecraftForge”的目录中（注意，你需要将 `<User>` 替换为你的用户名）：

```git clone https://github.com/<User>/MinecraftForge```

### 检出正确的分支
分叉和克隆仓库是为 Forge 开发所必需的步骤。不过，为了方便创建拉取请求，最好使用分支进行开发。

建议为你计划提交的每个 PR 创建并检出一个分支。这样，在处理旧补丁的同时，你始终可以保留 Forge 的最新更改以用于新的 PR。

完成此步骤后，你就可以设置开发环境了。

### 设置环境
根据你喜欢的集成开发环境（IDE），需要遵循不同的推荐步骤来成功设置开发环境。

#### Eclipse
由于 Eclipse 工作区的工作方式，ForgeGradle 可以完成大部分设置 Forge 工作区的工作。
1. 打开终端或命令提示符，导航到你克隆的分叉仓库目录。
2. 输入 `./gradlew setup` 并按回车键。等待 ForgeGradle 完成操作。
3. 输入 `./gradlew genEclipseRuns` 并按回车键。再次等待 ForgeGradle 完成操作。
4. 打开你的 Eclipse 工作区，选择 `File -> Import -> General -> Existing Gradle Project`。
5. 在打开的对话框中，为“Project root directory”选项浏览到仓库目录。
6. 点击“Finish”按钮完成导入。

这就是在 Eclipse 中设置并运行所需的全部步骤。运行测试模组不需要额外的步骤。只需像在其他项目中一样点击“Run”并选择合适的运行配置即可。

#### IntelliJ IDEA
JetBrains 的旗舰 IDE 对 [Gradle][gradle] 提供了很好的集成支持，而 Gradle 是 Forge 首选的构建系统。然而，由于 Minecraft 模组开发的一些特殊性，要使一切正常工作还需要额外的步骤。

##### IDEA 2021 及以后版本
1. 启动 IntelliJ IDEA 2021。
    - 如果你已经打开了另一个项目，可以通过 `File -> Close project` 选项关闭该项目。
2. 在“Welcome to IntelliJ IDEA”窗口的项目标签中，点击右上角的“Open”按钮，选择你之前克隆的 MinecraftForge 文件夹。
3. 如果提示“Trust Project”，请点击“Trust Project”。
4. IDEA 完成项目导入和文件索引后，运行 Gradle 设置任务。你可以通过以下方式进行操作：
    - 打开屏幕右侧的 Gradle 侧边栏，然后展开 forge 项目树，选择 `Tasks`，再选择 `other`，双击 `setup` 任务（可能也显示为 `MinecraftForge[Setup]`），该任务位于 `Forge -> Tasks -> other -> setup`。
5. 生成运行配置：
    - 打开屏幕右侧的 Gradle 侧边栏，然后展开 forge 项目树，选择 `Tasks`，再选择 `other`，双击 `genIntellijRuns` 任务（可能也显示为 `MinecraftForge[genIntellijRuns]`），该任务位于 `Forge -> Tasks -> forgegradle runs -> genIntellijRuns`。
- 如果你在进行任何更改之前的构建过程中遇到许可错误，运行 `updateLicenses` 任务可能会有所帮助。该任务也位于 `Forge -> Tasks -> other`。

##### IDEA 2019 - 2020 版本
在设置方面，IDEA 2019 - 2020 版本与 2021 版本有一些细微的差异。
1. 将 Forge 的 `build.gradle` 作为 IDEA 项目导入。为此，只需在 `Welcome to IntelliJ IDEA` 启动屏幕上点击 `Import Project`，然后选择 `build.gradle` 文件。
2. IDEA 完成项目导入和文件索引后，运行 Gradle 设置任务。可以通过以下两种方式之一进行操作：
    1. 打开屏幕右侧的 Gradle 侧边栏，然后展开 `forge` 项目树，选择 `Tasks`，再选择 `other`，双击 `setup` 任务（可能也显示为 `MinecraftForge[Setup]`）。或者：
    1. 按两次 CTRL 键，在弹出的 `Run` 命令窗口中输入 `gradle setup`。

然后，你可以使用 `forge_client` Gradle 任务（`Tasks -> fg_runs -> forge_client`）运行 Forge：右键单击该任务，根据需要选择 `Run` 或 `Debug`。

现在，你应该能够使用对 Forge 和原版代码库所做的更改来开发你的模组了。

### 进行更改并提交拉取请求
设置好开发环境后，就可以对 Forge 的代码库进行更改了。不过，在编辑项目代码时，需要避免一些陷阱。

最重要的是，如果你想编辑 Minecraft 源代码，必须只在“Forge”子项目中进行。在“Clean”项目中进行的任何更改都会干扰 ForgeGradle 和补丁生成。这可能会导致灾难性的后果，甚至使你的开发环境完全无法使用。如果你想获得完美的开发体验，请确保只在“Forge”项目中编辑代码！

#### 生成补丁
在对代码库进行更改并彻底测试之后，就可以生成补丁了。只有在处理 Minecraft 代码库（即在“Forge”项目中）时才需要执行此步骤，但这一步对于让你的更改在其他地方生效至关重要。Forge 通过将更改内容注入原版 Minecraft 来工作，因此需要以适当的格式提供这些更改。幸运的是，ForgeGradle 能够为你生成更改集以便提交。

要启动补丁生成，只需从你的 IDE 或命令行运行 `genPatches` Gradle 任务。完成后，你可以提交所有更改（确保不添加任何不必要的文件）并提交拉取请求！

#### 拉取请求
在你的贡献被合并到 Forge 之前的最后一步是提交拉取请求（简称 PR）。这是一个正式请求，用于将你分叉仓库中的更改合并到主代码库中。创建 PR 很简单。只需访问 [这个 GitHub 页面][submitpr] 并按照提示步骤操作即可。此时，良好的分支设置就会发挥作用，因为你能够精确选择要提交的更改。

!!! 注意
    拉取请求有一定的规则，并非每个请求都会被盲目接受。请参考 [这份文档][contribute] 获取更多信息，以确保你的 PR 质量最佳！如果你想最大程度提高 PR 被接受的机会，请遵循这些 [PR 指南][guidelines]！

[github]: https://www.github.com
[register]: https://www.github.com/join
[forgerepo]: https://www.github.com/MinecraftForge/MinecraftForge
[gradle]: https://www.gradle.org
[submitpr]: https://github.com/MinecraftForge/MinecraftForge/compare
[contribute]: https://github.com/MinecraftForge/MinecraftForge/blob/1.13.x/CONTRIBUTING.md
[guidelines]: ./prguidelines.md
