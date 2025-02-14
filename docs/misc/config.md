### 配置系统概述
配置用于定义可应用于模组实例的设置和用户偏好。Forge 使用基于 [TOML][toml] 文件的配置系统，并通过 [NightConfig][nightconfig] 进行读取。

### 创建配置
可以通过 `IConfigSpec` 的子类型来创建配置。Forge 通过 `ForgeConfigSpec` 实现了该类型，并通过 `ForgeConfigSpec$Builder` 来构建配置。构建器可以使用 `Builder#push` 创建一个配置节，使用 `Builder#pop` 离开一个配置节。之后，可以使用以下两种方法之一构建配置：

| 方法 | 描述 |
| :--- | :--- |
| `build` | 创建 `ForgeConfigSpec`。 |
| `configure` | 创建一个包含配置值的类和 `ForgeConfigSpec` 的对。 |

!!! 注意
    `ForgeConfigSpec$Builder#configure` 通常与 `static` 块和一个将 `ForgeConfigSpec$Builder` 作为构造函数参数的类一起使用，以附加和保存配置值：

    ```java
    // 在某个配置类中
    ExampleConfig(ForgeConfigSpec.Builder builder) {
      // 在 final 字段中定义值
    }

    // 在构造函数可访问的地方
    static {
      Pair<ExampleConfig, ForgeConfigSpec> pair = new ForgeConfigSpec.Builder()
        .configure(ExampleConfig::new);
      // 将对的值存储在某个常量字段中
    }
    ```

每个配置值都可以提供额外的上下文来实现额外的行为。上下文必须在配置值完全构建之前定义：

| 方法 | 描述 |
| :--- | :--- |
| `comment` | 提供配置值功能的描述。可以提供多个字符串以实现多行注释。 |
| `translation` | 为配置值的名称提供翻译键。 |
| `worldRestart` | 更改此配置值前必须重启世界。 |

#### ConfigValue
可以使用任何 `#define` 方法结合已定义的上下文来构建配置值。

所有配置值方法至少接受两个组件：
- 表示变量名称的路径：一个用 `.` 分隔的字符串，表示配置值所在的节。
- 当没有有效配置时的默认值。

`ConfigValue` 特定的方法还接受两个额外的组件：
- 一个验证器，用于确保反序列化的对象有效。
- 一个表示配置值数据类型的类。

```java
// 对于某个 ForgeConfigSpec$Builder builder
ConfigValue<T> value = builder.comment("Comment")
  .define("config_value_name", defaultValue);
```

可以使用 `ConfigValue#get` 获取配置值本身。这些值会被缓存，以避免多次从文件中读取。

#### 其他配置值类型
- **范围值**
    - 描述：值必须在定义的边界之间。
    - 类类型：`Comparable<T>`
    - 方法名称：`#defineInRange`
    - 额外组件：
        - 配置值的最小值和最大值。
        - 一个表示配置值数据类型的类。

!!! 注意
    `DoubleValue`、`IntValue` 和 `LongValue` 是范围值，分别指定类为 `Double`、`Integer` 和 `Long`。

- **白名单值**
    - 描述：值必须在提供的集合中。
    - 类类型：`T`
    - 方法名称：`#defineInList`
    - 额外组件：
        - 一个包含配置允许值的集合。

- **列表值**
    - 描述：值是一个条目列表。
    - 类类型：`List<T>`
    - 方法名称：`#defineList`，如果列表可以为空则使用 `#defineListAllowEmpty`
    - 额外组件：
        - 一个验证器，用于确保从列表中反序列化的元素有效。

- **枚举值**
    - 描述：提供集合中的一个枚举值。
    - 类类型：`Enum<T>`
    - 方法名称：`#defineEnum`
    - 额外组件：
        - 一个将字符串或整数转换为枚举的获取器。
        - 一个包含配置允许值的集合。

- **布尔值**
    - 描述：一个 `boolean` 值。
    - 类类型：`Boolean`
    - 方法名称：`#define`

### 注册配置
一旦构建了 `ForgeConfigSpec`，就必须进行注册，以便 Forge 可以根据需要加载、跟踪和同步配置设置。配置应在模组构造函数中通过 `ModLoadingContext#registerConfig` 进行注册。可以使用给定的类型（表示配置所属的端）、`ForgeConfigSpec` 以及可选的特定配置文件名来注册配置。

```java
// 在模组构造函数中，使用 ForgeConfigSpec CONFIG
ModLoadingContext.get().registerConfig(Type.COMMON, CONFIG);
```

以下是可用的配置类型列表：

| 类型 | 加载位置 | 是否同步到客户端 | 客户端位置 | 服务器位置 | 默认文件后缀 |
| :---: | :---: | :---: | :---: | :---: | :--- |
| CLIENT | 仅客户端 | 否 | `.minecraft/config` | N/A | `-client` |
| COMMON | 两端 | 否 | `.minecraft/config` | `<server_folder>/config` | `-common` |
| SERVER | 仅服务器 | 是 | `.minecraft/saves/<level_name>/serverconfig` | `<server_folder>/world/serverconfig` | `-server` |

!!! 提示
    Forge 在其代码库中记录了 [配置类型][type]。

### 配置事件
可以使用 `ModConfigEvent$Loading` 和 `ModConfigEvent$Reloading` 事件来执行每次加载或重新加载配置时发生的操作。这些事件必须 [注册][events] 到模组事件总线。

!!! 警告
    这些事件会为模组的所有配置调用；应使用提供的 `ModConfig` 对象来表示正在加载或重新加载的配置。

[toml]: https://toml.io/
[nightconfig]: https://github.com/TheElectronWill/night-config
[type]: https://github.com/MinecraftForge/MinecraftForge/blob/c3e0b071a268b02537f9d79ef8e7cd9b100db416/fmlcore/src/main/java/net/minecraftforge/fml/config/ModConfig.java#L108-L136
[events]: ../concepts/events.md#creating-an-event-handler
