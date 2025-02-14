### 按键映射概述
按键映射（Key Mapping），也称为按键绑定，用于定义特定的操作与输入（如鼠标点击、按键按下等）之间的关联。每个由按键映射定义的操作都可以在客户端能够接收输入时进行检查。此外，每个按键映射都可以通过 [控制选项菜单][controls] 分配给任何输入。

### 注册 `KeyMapping`
要注册 `KeyMapping`，需要在物理客户端的 [**模组事件总线**][modbus] 上监听 `RegisterKeyMappingsEvent` 事件，并调用 `#register` 方法。

```java
// 在仅存在于物理客户端的类中

// 按键映射采用懒加载初始化，直到注册时才会创建实例
public static final Lazy<KeyMapping> EXAMPLE_MAPPING = Lazy.of(() -> /*...*/);

// 该事件仅在物理客户端的模组事件总线上监听
@SubscribeEvent
public void registerBindings(RegisterKeyMappingsEvent event) {
  event.register(EXAMPLE_MAPPING.get());
}
```

### 创建 `KeyMapping`
可以使用 `KeyMapping` 的构造函数来创建实例。构造函数需要传入一个 [翻译键][tk] 来定义映射的名称、映射的默认输入，以及另一个 [翻译键][tk] 来定义该映射在 [控制选项菜单][controls] 中所属的类别。

#### 默认输入
每个按键映射都有一个关联的默认输入，通过 `InputConstants$Key` 提供。每个输入由 `InputConstants$Type`（定义提供输入的设备）和一个整数（定义设备上输入的关联标识符）组成。

Minecraft 原版提供了三种输入类型：
- `KEYSYM`：通过 `GLFW` 键码定义键盘输入。
- `SCANCODE`：通过特定平台的扫描码定义键盘输入。
- `MOUSE`：定义鼠标输入。

!!! 注意
    强烈建议在键盘输入中使用 `KEYSYM` 而非 `SCANCODE`，因为 `GLFW` 键码不依赖于特定系统。可以在 [GLFW 文档][keyinput] 中了解更多信息。

整数部分取决于所提供的输入类型。所有输入代码都在 `GLFW` 中定义：`KEYSYM` 键码以 `GLFW_KEY_*` 为前缀，`MOUSE` 代码以 `GLFW_MOUSE_*` 为前缀。

```java
new KeyMapping(
  "key.examplemod.example1", // 使用此翻译键进行本地化
  InputConstants.Type.KEYSYM, // 默认映射到键盘
  GLFW.GLFW_KEY_P, // 默认按键为 P
  "key.categories.misc" // 映射将位于杂项类别中
)
```

!!! 注意
    如果按键映射不应设置默认输入，应将输入设置为 `InputConstants#UNKNOWN`。原版构造函数需要通过 `InputConstants$Key#getValue` 提取输入代码，而 Forge 构造函数可以直接传入原始输入字段。

#### `IKeyConflictContext`
并非所有映射都适用于所有上下文。有些映射仅在 GUI 中使用，而有些仅在游戏中使用。为避免相同按键在不同上下文中的映射冲突，可以分配一个 `IKeyConflictContext`。

每个冲突上下文包含两个方法：
- `#isActive`：定义映射在当前游戏状态下是否可用。
- `#conflicts`：定义映射是否与相同或不同冲突上下文中的按键冲突。

目前，Forge 通过 `KeyConflictContext` 定义了三种基本上下文：
- `UNIVERSAL`：默认值，表示按键可在所有上下文中使用。
- `GUI`：表示映射仅在 `Screen` 打开时可用。
- `IN_GAME`：表示映射仅在 `Screen` 未打开时可用。

也可以通过实现 `IKeyConflictContext` 来创建新的冲突上下文。

```java
new KeyMapping(
  "key.examplemod.example2",
  KeyConflictContext.GUI, // 映射仅在屏幕打开时可用
  InputConstants.Type.MOUSE, // 默认映射到鼠标
  GLFW.GLFW_MOUSE_BUTTON_LEFT, // 默认鼠标输入为左键
  "key.categories.examplemod.examplecategory" // 映射将位于新的示例类别中
)
```

#### `KeyModifier`
模组开发者可能希望在按下修饰键时，按键映射具有不同的行为（例如，`G` 与 `CTRL + G`）。为了解决这个问题，Forge 在构造函数中添加了一个额外的参数，用于接收 `KeyModifier`，可以应用控制键（`KeyModifier#CONTROL`）、Shift 键（`KeyModifier#SHIFT`）或 Alt 键（`KeyModifier#ALT`）到任何输入。`KeyModifier#NONE` 是默认值，表示不应用任何修饰键。

可以在 [控制选项菜单][controls] 中通过按住修饰键和关联输入来添加修饰符。

```java
new KeyMapping(
  "key.examplemod.example3",
  KeyConflictContext.UNIVERSAL,
  KeyModifier.SHIFT, // 默认映射需要按住 Shift 键
  InputConstants.Type.KEYSYM, // 默认映射到键盘
  GLFW.GLFW_KEY_G, // 默认按键为 G
  "key.categories.misc"
)
```

### 检查 `KeyMapping`
可以检查 `KeyMapping` 是否被点击，并根据情况在条件语句中应用相关逻辑。

#### 在游戏中
在游戏中，应在 [**Forge 事件总线**][forgebus] 上监听 `ClientTickEvent` 事件，并在 `while` 循环中检查 `KeyMapping#consumeClick` 方法。`#consumeClick` 仅在输入被执行且尚未被处理的次数内返回 `true`，因此不会导致游戏无限卡顿。

```java
// 该事件仅在物理客户端的 Forge 事件总线上监听
public void onClientTick(ClientTickEvent event) {
  if (event.phase == TickEvent.Phase.END) { // 每个 tick 事件会被调用两次，仅在结束阶段执行代码
    while (EXAMPLE_MAPPING.get().consumeClick()) {
      // 在此处执行点击时要执行的逻辑
    }
  }
}
```

!!! 警告
    不要使用 `InputEvent` 替代 `ClientTickEvent`，因为它们分别针对键盘和鼠标输入有单独的事件，无法处理其他额外输入。

#### 在 GUI 中
在 GUI 中，可以在 `GuiEventListener` 的方法中使用 `IForgeKeyMapping#isActiveAndMatches` 方法检查映射。最常用的检查方法是 `#keyPressed` 和 `#mouseClicked`。

`#keyPressed` 方法接收 `GLFW` 键码、特定平台的扫描码和一个表示按住的修饰键的位字段。可以使用 `InputConstants#getKey` 创建输入来检查按键是否匹配映射。映射方法本身会检查修饰键。

```java
// 在某个 Screen 子类中
@Override
public boolean keyPressed(int key, int scancode, int mods) {
  if (EXAMPLE_MAPPING.get().isActiveAndMatches(InputConstants.getKey(key, scancode))) {
    // 在此处执行按键按下时要执行的逻辑
    return true;
  }
  return super.keyPressed(x, y, button);
} 
```

!!! 注意
    如果要检查按键的屏幕不是自己创建的，可以在 [**Forge 事件总线**][forgebus] 上监听 `ScreenEvent$KeyPressed` 的 `Pre` 或 `Post` 事件。

`#mouseClicked` 方法接收鼠标的 x 坐标、y 坐标和点击的按钮。可以使用 `InputConstants$Type#getOrCreate` 方法并传入 `MOUSE` 输入类型来创建输入，以检查鼠标按钮是否匹配映射。

```java
// 在某个 Screen 子类中
@Override
public boolean mouseClicked(double x, double y, int button) {
  if (EXAMPLE_MAPPING.get().isActiveAndMatches(InputConstants.TYPE.MOUSE.getOrCreate(button))) {
    // 在此处执行鼠标点击时要执行的逻辑
    return true;
  }
  return super.mouseClicked(x, y, button);
} 
```

!!! 注意
    如果要检查鼠标的屏幕不是自己创建的，可以在 [**Forge 事件总线**][forgebus] 上监听 `ScreenEvent$MouseButtonPressed` 的 `Pre` 或 `Post` 事件。

[modbus]: ../concepts/events.md#mod-event-bus
[controls]: https://minecraft.wiki/w/Options#Controls
[tk]: ../concepts/internationalization.md#translatablecontents
[keyinput]: https://www.glfw.org/docs/3.3/input_guide.html#input_key
[forgebus]: ../concepts/events.md#creating-an-event-handler
