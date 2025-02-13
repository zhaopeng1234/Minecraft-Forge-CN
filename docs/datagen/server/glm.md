### 全局战利品修饰符生成
通过继承 `GlobalLootModifierProvider` 并实现 `#start` 方法，可以为模组生成[全局战利品修饰符（GLMs）][glm]。可以通过调用 `#add` 方法并指定修饰符的名称和要序列化的[修饰符实例][instance]来生成并添加每个 GLM。实现之后，必须将该提供者 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成服务器数据时运行
        event.includeServer(),
        output -> new MyGlobalLootModifierProvider(output, MOD_ID)
    );
}

// 在某个 GlobalLootModifierProvider#start 方法中
this.add("example_modifier", new ExampleModifier(
  new LootItemCondition[] {
    WeatherCheck.weather().setRaining(true).build() // 在下雨时执行
  },
  "val1",
  10,
  Items.DIRT
));
```

### 代码解释
#### 事件监听与提供者添加
```java
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new MyGlobalLootModifierProvider(output, MOD_ID)
    );
}
```
- `@SubscribeEvent`：这是一个事件注解，用于标记 `gatherData` 方法，使其能够监听 `GatherDataEvent` 事件。当数据生成过程启动时，这个方法会被调用。
- `GatherDataEvent event`：事件对象，包含了数据生成过程的相关信息，如生成器、是否包含客户端或服务器数据等。
- `event.includeServer()`：判断是否是在生成服务器数据，如果是则添加提供者。
- `output -> new MyGlobalLootModifierProvider(output, MOD_ID)`：使用 lambda 表达式创建 `MyGlobalLootModifierProvider` 实例，`output` 是 `PackOutput` 对象，用于指定数据输出的位置，`MOD_ID` 是模组的唯一标识符。

#### 生成并添加全局战利品修饰符
```java
this.add("example_modifier", new ExampleModifier(
  new LootItemCondition[] {
    WeatherCheck.weather().setRaining(true).build()
  },
  "val1",
  10,
  Items.DIRT
));
```
- `this.add`：调用 `GlobalLootModifierProvider` 的 `add` 方法，用于添加一个全局战利品修饰符。
- `"example_modifier"`：修饰符的名称，用于在配置文件或代码中引用该修饰符。
- `new ExampleModifier(...)`：创建一个自定义的 `ExampleModifier` 实例，该实例继承自 `IGlobalLootModifier` 接口，用于定义战利品的修改逻辑。
- `new LootItemCondition[] {...}`：一个战利品条件数组，这里使用 `WeatherCheck` 创建了一个条件，即当正在下雨时执行该修饰符。
- `"val1", 10, Items.DIRT`：这些是传递给 `ExampleModifier` 构造函数的额外参数，具体含义取决于 `ExampleModifier` 的实现。

通过以上步骤，可以方便地为模组生成和添加全局战利品修饰符，从而实现对游戏中战利品生成的定制化修改。

[glm]: ../../resources/server/glm.md
[instance]: ../../resources/server/glm.md#igloballootmodifier
[datagen]: ../index.md#data-providers
