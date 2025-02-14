### 游戏测试概述
游戏测试（Game Tests）是一种在游戏内运行单元测试的方式。该系统设计为可扩展且能并行运行，以便高效地执行大量不同的测试。测试对象交互和行为只是这个框架众多应用中的一部分。

### 创建游戏测试
一个标准的游戏测试遵循三个基本步骤：
1. 加载一个结构或模板，该模板包含用于测试交互或行为的场景。
2. 编写一个方法，该方法包含在场景上执行的逻辑。
3. 执行方法逻辑。如果达到成功状态，则测试通过；否则，测试失败，结果会存储在场景相邻的讲台中。

因此，要创建一个游戏测试，必须有一个包含场景初始状态的现有模板，以及一个提供执行逻辑的方法。

#### 测试方法
游戏测试方法是一个 `Consumer<GameTestHelper>` 引用，这意味着它接受一个 `GameTestHelper` 参数且不返回任何值。要使游戏测试方法被识别，必须使用 `@GameTest` 注解：

```java
public class ExampleGameTests {
  @GameTest
  public static void exampleTest(GameTestHelper helper) {
    // 执行操作
  }
}
```

`@GameTest` 注解还包含一些成员，用于配置游戏测试的运行方式：

```java
// 在某个类中
@GameTest(
  setupTicks = 20L, // 测试花费 20 刻来进行执行前的准备
  required = false // 失败会被记录，但不影响批次的执行
)
public static void exampleConfiguredTest(GameTestHelper helper) {
  // 执行操作
}
```

##### 相对定位
所有 `GameTestHelper` 方法都会使用结构方块的当前位置，将结构模板场景内的相对坐标转换为绝对坐标。为了方便在相对位置和绝对位置之间进行转换，可以分别使用 `GameTestHelper#absolutePos` 和 `GameTestHelper#relativePos` 方法。

可以在游戏中通过 [测试命令][test] 加载结构模板，将玩家放置在所需位置，然后运行 `/test pos` 命令来获取结构模板的相对位置。该命令会获取玩家相对于距离玩家 200 格内最近结构的坐标，并将相对位置作为可复制的文本组件输出到聊天框中，可将其用作最终的局部变量。

!!! 提示
    `/test pos` 命令生成的局部变量可以通过在命令末尾追加引用名称来指定：
    ```bash
    /test pos <var> # 输出 'final BlockPos <var> = new BlockPos(...);'
    ```

##### 成功完成
游戏测试方法的主要职责是在有效完成时标记测试成功。如果在达到超时时间（由 `GameTest#timeoutTicks` 定义）之前没有达到成功状态，则测试自动失败。

`GameTestHelper` 中有许多抽象方法可用于定义成功状态，但有四个方法非常重要，需要了解：

| 方法 | 描述 |
| :---: | --- |
| `#succeed` | 将测试标记为成功。 |
| `#succeedIf` | 立即测试提供的 `Runnable`，如果没有抛出 `GameTestAssertException` 则测试成功。如果测试在当前刻没有成功，则标记为失败。 |
| `#succeedWhen` | 每刻测试提供的 `Runnable`，直到超时。如果在某一刻的检查中没有抛出 `GameTestAssertException`，则测试成功。 |
| `#succeedOnTickWhen` | 在指定的刻测试提供的 `Runnable`，如果没有抛出 `GameTestAssertException` 则测试成功。如果 `Runnable` 在其他刻成功，则标记为失败。 |

!!! 重要
    游戏测试会每刻执行，直到测试被标记为成功。因此，在给定刻安排成功的方法必须确保在之前的刻总是失败。

##### 调度操作
并非所有操作都会在测试开始时立即执行。可以安排操作在特定时间或间隔执行：

| 方法 | 描述 |
| :---: | --- |
| `#runAtTickTime` | 在指定的刻执行操作。 |
| `#runAfterDelay` | 在当前刻之后的 `x` 刻执行操作。 |
| `#onEachTick` | 每刻执行操作。 |

##### 断言
在游戏测试的任何时候，都可以进行断言以检查给定条件是否为真。`GameTestHelper` 中有许多断言方法，简单来说，就是在未满足适当状态时抛出 `GameTestAssertException`。

#### 生成测试方法
如果需要动态生成游戏测试方法，可以创建一个测试方法生成器。这些方法不接受参数，并返回一个 `TestFunction` 集合。要使测试方法生成器被识别，必须使用 `@GameTestGenerator` 注解：

```java
public class ExampleGameTests {
  @GameTestGenerator
  public static Collection<TestFunction> exampleTests() {
    // 返回一个 TestFunction 集合
  }
}
```

##### TestFunction
`TestFunction` 是 `@GameTest` 注解和运行测试的方法所包含的封装信息。

!!! 提示
    任何使用 `@GameTest` 注解的方法都会使用 `GameTestRegistry#turnMethodIntoTestFunction` 转换为 `TestFunction`。该方法可以作为创建 `TestFunction` 而不使用注解的参考。

#### 批量处理
游戏测试可以按批次执行，而不是按注册顺序执行。可以通过为测试提供相同的 `GameTest#batch` 字符串，将测试添加到一个批次中。

单独的批量处理本身并没有太大作用，但可以用于在测试运行的当前关卡中执行设置和清理状态。这可以通过使用 `@BeforeBatch` 注解标记一个方法进行设置，或使用 `@AfterBatch` 注解标记一个方法进行清理。`#batch` 方法必须与游戏测试中提供的字符串匹配。

批量方法是 `Consumer<ServerLevel>` 引用，这意味着它们接受一个 `ServerLevel` 参数且不返回任何值：

```java
public class ExampleGameTests {
  @BeforeBatch(batch = "firstBatch")
  public static void beforeTest(ServerLevel level) {
    // 执行设置操作
  }

  @GameTest(batch = "firstBatch")
  public static void exampleTest2(GameTestHelper helper) {
    // 执行操作
  }
}
```

### 注册游戏测试
游戏测试必须进行注册才能在游戏中运行。有两种注册方法：使用 `@GameTestHolder` 注解或 `RegisterGameTestsEvent`。两种注册方法都要求测试方法使用 `@GameTest`、`@GameTestGenerator`、`@BeforeBatch` 或 `@AfterBatch` 进行注解。

#### GameTestHolder
`@GameTestHolder` 注解会注册类型（类、接口、枚举或记录）内的任何测试方法。`@GameTestHolder` 包含一个方法，有多种用途。在这种情况下，提供的 `#value` 必须是模组的模组 ID，否则测试在默认配置下将不会运行。

```java
@GameTestHolder(MODID)
public class ExampleGameTests {
  // ...
}
```

#### RegisterGameTestsEvent
`RegisterGameTestsEvent` 也可以使用 `#register` 方法注册类或方法。事件监听器必须 [添加][event] 到模组事件总线。通过这种方式注册的测试方法必须在每个使用 `@GameTest` 注解的方法中为 `GameTest#templateNamespace` 提供模组 ID。

```java
// 在某个类中
public void registerTests(RegisterGameTestsEvent event) {
  event.register(ExampleGameTests.class);
}

// 在 ExampleGameTests 类中
@GameTest(templateNamespace = MODID)
public static void exampleTest3(GameTestHelper helper) {
  // 执行设置操作
}
```

!!! 注意
    提供给 `GameTestHolder#value` 和 `GameTest#templateNamespace` 的值可以与当前模组 ID 不同，但需要更改 [构建脚本][namespaces] 中的配置。

### 结构模板
游戏测试在由结构或模板加载的场景中执行。所有模板定义了场景的尺寸以及将加载的初始数据（方块和实体）。模板必须作为 `.nbt` 文件存储在 `data/<namespace>/structures` 目录中。

!!! 提示
    可以使用结构方块创建和保存结构模板。

模板的位置由以下几个因素决定：
- 模板的命名空间是否指定。
- 类名是否应前缀到模板名称。
- 模板名称是否指定。

模板的命名空间由 `GameTest#templateNamespace` 决定，如果未指定，则由 `GameTestHolder#value` 决定，如果两者都未指定，则为 `minecraft`。

如果 `@PrefixGameTestTemplate` 应用于带有测试注解的类或方法并设置为 `false`，则简单类名不会前缀到模板名称。否则，简单类名会转换为小写并前缀到模板名称之前，后面跟一个点。

模板名称由 `GameTest#template` 决定。如果未指定，则使用方法的小写名称。

```java
// 所有结构的模组 ID 为 MODID
@GameTestHolder(MODID)
public class ExampleGameTests {

  // 类名前缀，模板名称未指定
  // 模板位置为 'modid:examplegametests.exampletest'
  @GameTest
  public static void exampleTest(GameTestHelper helper) { /*...*/ }

  // 类名不前缀，模板名称未指定
  // 模板位置为 'modid:exampletest2'
  @PrefixGameTestTemplate(false)
  @GameTest
  public static void exampleTest2(GameTestHelper helper) { /*...*/ }

  // 类名前缀，模板名称指定
  // 模板位置为 'modid:examplegametests.test_template'
  @GameTest(template = "test_template")
  public static void exampleTest3(GameTestHelper helper) { /*...*/ }

  // 类名不前缀，模板名称指定
  // 模板位置为 'modid:test_template2'
  @PrefixGameTestTemplate(false)
  @GameTest(template = "test_template2")
  public static void exampleTest4(GameTestHelper helper) { /*...*/ }
}
```

### 运行游戏测试
可以使用 `/test` 命令运行游戏测试。`test` 命令具有高度可配置性，但只有几个子命令对于运行测试很重要：

| 子命令 | 描述 |
| :---: | --- |
| `run` | 运行指定的测试：`run <test_name>`。 |
| `runall` | 运行所有可用的测试。 |
| `runthis` | 运行距离玩家 15 格内最近的测试。 |
| `runthese` | 运行距离玩家 200 格内的测试。 |
| `runfailed` | 运行上一次运行中失败的所有测试。 |

!!! 注意
    子命令跟随 `test` 命令：`/test <subcommand>`。

### 构建脚本配置
游戏测试在构建脚本（`build.gradle` 文件）中提供了额外的配置设置，以便在不同的设置中运行和集成。

#### 启用其他命名空间
如果按照 [推荐方式设置构建脚本][buildscript]，则只有当前模组 ID 下的游戏测试会被启用。要启用从其他命名空间加载游戏测试，运行配置必须将属性 `forge.enabledGameTestNamespaces` 设置为一个字符串，其中每个命名空间用逗号分隔。如果该属性为空或未设置，则将加载所有命名空间。

```gradle
// 在运行配置内部
property 'forge.enabledGameTestNamespaces', 'modid1,modid2,modid3'
```

!!! 警告
    命名空间之间不能有空格，否则命名空间将无法正确加载。

#### 游戏测试服务器运行配置
游戏测试服务器是一种特殊的配置，用于运行一个构建服务器。构建服务器返回所需的、失败的游戏测试的数量作为退出代码。所有失败的测试（无论是必需的还是可选的）都会被记录。可以使用 `gradlew runGameTestServer` 运行此服务器。

#### 在其他运行配置中启用游戏测试
默认情况下，只有 `client`、`server` 和 `gameTestServer` 运行配置启用了游戏测试。如果其他运行配置也应该运行游戏测试，则必须将 `forge.enableGameTest` 属性设置为 `true`。

```gradle
// 在运行配置内部
property 'forge.enableGameTest', 'true'
```

[test]: #running-game-tests
[namespaces]: #enabling-other-namespaces
[event]: ../concepts/events.md#creating-an-event-handler
[buildscript]: ../gettingstarted/index.md#simple-buildgradle-customizations
