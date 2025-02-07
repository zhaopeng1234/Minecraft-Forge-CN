# 屏幕

屏幕通常是Minecraft中所有图形用户界面（GUI）的基础：接收用户输入，在服务器上进行验证，并将最终的操作结果同步回客户端。它们可以与[菜单]结合，为类似物品栏的视图创建通信网络；或者也可以独立存在，由模组开发者通过他们自己的[网络]实现来处理。

屏幕由众多部分组成，这使得要完全理解在Minecraft中 “屏幕” 实际上是什么变得困难。因此，本文档将在讨论屏幕本身之前，先介绍屏幕的各个组件及其应用方式。

## 相对坐标

每当渲染任何内容时，都需要有某种标识符来指定其显示位置。由于存在众多抽象概念，Minecraft的大多数渲染调用都在坐标平面中使用x、y和z值。x值从左到右递增，y值从上到下递增，z值从远到近递增。然而，这些坐标并非固定在特定范围内。它们会根据屏幕大小以及选项中指定的缩放比例而变化。因此，在渲染时必须格外小心，以确保坐标值能够根据可变的屏幕大小进行适当缩放。

有关如何使坐标相对化的信息将在[屏幕]部分介绍。

!!! 重要
    如果你选择使用固定坐标或对屏幕进行错误缩放，渲染的对象可能会看起来怪异或位置错误。检查坐标是否正确相对化的一个简单方法是点击视频设置中的 “Gui Scale” 按钮。在确定GUI的渲染缩放比例时，该值将作为显示器宽度和高度的除数。

## GUI图形

Minecraft渲染的任何GUI通常都使用`GuiGraphics`。`GuiGraphics`几乎是所有渲染方法的第一个参数；它包含用于渲染常用对象的基本方法。这些方法可分为五类：彩色矩形、字符串、纹理、物品和工具提示。此外，还有一个用于渲染组件片段的方法（`#enableScissor` / `#disableScissor`）。`GuiGraphics`还公开了`PoseStack`，用于应用使组件正确渲染到指定位置所需的变换。另外，颜色采用[ARGB][argb]格式。

### 彩色矩形

彩色矩形通过位置颜色着色器绘制。有三种类型的彩色矩形可供绘制。

首先，有彩色的水平和垂直单像素宽线条，分别是`#hLine`和`#vLine`。`#hLine`接受定义左右边界（包含）的两个x坐标、顶部y坐标以及颜色。`#vLine`接受左边的x坐标、定义顶部和底部边界（包含）的两个y坐标以及颜色。

其次，是`#fill`方法，用于在屏幕上绘制矩形。上述线条绘制方法在内部会调用此方法。它接受左边的x坐标、顶部y坐标、右边的x坐标、底部y坐标以及颜色。

最后，是`#fillGradient`方法，用于绘制带有垂直渐变的矩形。它接受右边的x坐标、底部y坐标、左边的x坐标、顶部y坐标、z坐标以及底部和顶部颜色。

### 字符串

字符串通过其`Font`绘制，通常有用于正常、透明和偏移模式的各自着色器。有两种可渲染的字符串对齐方式，每种都带有阴影：左对齐字符串（`#drawString`）和居中对齐字符串（`#drawCenteredString`）。这两个方法都接受字符串将使用的字体、要绘制的字符串、分别代表字符串左边或中心的x坐标、顶部y坐标以及颜色。

!!! 注意
    字符串通常应以[`Component`s][component]的形式传入，因为它们可以处理各种用例，包括该方法的另外两个重载。

### 纹理

纹理通过位块传输（blitting）绘制，因此方法名为`#blit`，其作用是复制图像的位并将它们直接绘制到屏幕上。这些通过位置纹理着色器绘制。虽然有许多不同的`#blit`重载，但我们只讨论两个静态的`#blit`方法。

第一个静态`#blit`接受六个整数，并假定要渲染的纹理位于256 x 256的PNG文件上。它接受屏幕的左边x和顶部y坐标、PNG文件内的左边x和顶部y坐标以及要渲染图像的宽度和高度。

!!! 注意
    必须指定PNG文件的大小，以便对坐标进行归一化处理，从而获得相关的UV值。

第一个静态`#blit`调用的另一个静态`#blit`将参数扩展到九个整数，只假定图像位于PNG文件上。它接受屏幕的左边x和顶部y坐标、z坐标（称为位块传输偏移）、PNG文件内的左边x和顶部y坐标、要渲染图像的宽度和高度以及PNG文件的宽度和高度。

#### 位块传输偏移

渲染纹理时，z坐标通常设置为位块传输偏移。该偏移负责在查看屏幕时正确分层渲染内容。z坐标较小的渲染内容在背景中绘制，反之，z坐标较大的渲染内容在前景中绘制。可以通过`#translate`直接在`PoseStack`本身上设置z偏移。`GuiGraphics`的某些方法（例如物品渲染）在内部应用了一些基本的偏移逻辑。

!!! 重要
    设置位块传输偏移后，在渲染完对象后必须重置它。否则，屏幕内的其他对象可能会在错误的层中渲染，从而导致图形问题。建议在平移之前推送当前姿态，然后在完成所有偏移渲染后弹出。

## 可渲染对象

`Renderable`本质上是要渲染的对象。这些对象包括屏幕、按钮、聊天框、列表等。`Renderable`只有一个方法：`#render`。该方法接受用于将内容渲染到屏幕上的`GuiGraphics`、根据相对屏幕大小缩放后的鼠标x和y位置，以及刻增量（自上一帧以来经过了多少刻）。

一些常见的可渲染对象是屏幕和 “小部件”：通常在屏幕上渲染的可交互元素，例如`Button`及其子类型`ImageButton`，以及用于在屏幕上输入文本的`EditBox`。

## Gui事件监听器

Minecraft中渲染的任何屏幕都实现了`GuiEventListener`。`GuiEventListener`负责处理用户与屏幕的交互。这些交互包括来自鼠标的输入（移动、点击、释放、拖动、滚动、悬停）和来自键盘的输入（按下、释放、打字）。每个方法返回相关操作是否成功影响了屏幕。像按钮、聊天框、列表等小部件也实现了这个接口。

### 容器事件处理程序

与`GuiEventListener`几乎同义的是它们的子类型：`ContainerEventHandler`。这些负责处理包含小部件的屏幕上的用户交互，管理当前聚焦的内容以及相关交互的应用方式。`ContainerEventHandler`添加了三个额外功能：可交互子元素、拖动和聚焦。

事件处理程序包含子元素，用于确定元素的交互顺序。在鼠标事件处理程序（不包括拖动）中，鼠标悬停的列表中的第一个子元素将执行其逻辑。

通过`#mouseClicked`和`#mouseReleased`实现的用鼠标拖动元素的操作，提供了更精确执行的逻辑。

聚焦允许在事件执行期间首先检查并处理特定的子元素，例如在键盘事件或拖动鼠标期间。焦点通常通过`#setFocused`设置。此外，可以使用`#nextFocusPath`循环切换可交互子元素，根据传入的`FocusNavigationEvent`选择子元素。

!!! 注意
    屏幕通过`AbstractContainerEventHandler`实现`ContainerEventHandler`，它添加了用于拖动和聚焦子元素的设置器和获取器逻辑。

## 可叙述项

`NarratableEntry`是可以通过Minecraft的辅助功能叙述功能进行讲述的元素。每个元素可以根据悬停或选择的内容提供不同的叙述，通常按焦点、悬停，然后是所有其他情况的优先级进行。

`NarratableEntry`有三个方法：一个用于确定元素的优先级（`#narrationPriority`），一个用于确定是否讲述叙述内容（`#isActive`），最后一个用于为其相关输出（语音或阅读）提供叙述内容（`#updateNarration`）。

!!! 注意
    Minecraft中的所有小部件都是`NarratableEntry`，因此如果使用可用的子类型，通常无需手动实现。

## 屏幕子类型

有了上述所有知识，就可以构建一个基本的屏幕。为了更易于理解，屏幕的组件将按照通常遇到的顺序进行介绍。

首先，所有屏幕都接受一个`Component`，它代表屏幕的标题。这个组件通常由其某个子类型绘制到屏幕上。它仅在基础屏幕中用于叙述消息。

```java
// 在某个屏幕子类中
public MyScreen(Component title) {
    super(title);
}
```

### 初始化

一旦屏幕初始化完成，就会调用 `#init` 方法。`#init` 方法会设置屏幕内的初始设置，从 `ItemRenderer` 和 `Minecraft` 实例到根据游戏缩放的相对宽度和高度。任何设置操作，如添加小部件或预计算相对坐标，都应该在这个方法中完成。如果游戏窗口大小改变，屏幕将通过调用 `#init` 方法重新初始化。

有三种向屏幕添加小部件的方法，每种方法都有不同的用途：

| 方法                 | 描述                                                         |
| :-------------------: | :----------------------------------------------------------- |
| `#addWidget`           | 添加一个可交互且可进行语音叙述，但不进行渲染的小部件。         |
| `#addRenderableOnly`   | 添加一个仅进行渲染的小部件；它不可交互也不可进行语音叙述。   |
| `#addRenderableWidget` | 添加一个可交互、可进行语音叙述且可进行渲染的小部件。         |

通常，`#addRenderableWidget` 是最常用的方法。

```java
// 在某个 Screen 子类中
@Override
protected void init() {
    super.init();

    // 添加小部件和预计算值
    this.addRenderableWidget(new EditBox(/* ... */));
}
```

### 屏幕更新

屏幕还会使用 `#tick` 方法进行更新，以执行某种程度的客户端渲染逻辑。最常见的例子就是 `EditBox` 的闪烁光标。

```java
// 在某个 Screen 子类中
@Override
public void tick() {
    super.tick();

    // 为 editBox 中的 EditBox 添加更新逻辑
    this.editBox.tick();
}
```

### 输入处理

由于屏幕是 `GuiEventListener` 的子类，输入处理方法也可以被重写，例如用于处理特定 [按键][keymapping] 的逻辑。

### 屏幕渲染

最后，屏幕通过作为 `Renderable` 子类提供的 `#render` 方法进行渲染。如前所述，`#render` 方法会在每一帧绘制屏幕需要渲染的所有内容，如背景、小部件、工具提示等。默认情况下，`#render` 方法只会将小部件渲染到屏幕上。

在屏幕内渲染的、通常不由子类处理的两个最常见的内容是背景和工具提示。

可以使用 `#renderBackground` 方法渲染背景，其中一个方法会在渲染屏幕且其背后的世界无法渲染时，接受选项背景的垂直偏移量。

工具提示通过 `GuiGraphics#renderTooltip` 或 `GuiGraphics#renderComponentTooltip` 方法进行渲染，这些方法可以接受要渲染的文本组件、一个可选的自定义工具提示组件，以及工具提示应在屏幕上渲染位置的相对 x/y 坐标。

```java
// 在某个 Screen 子类中

// mouseX 和 mouseY 表示光标在屏幕上的缩放坐标位置
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    // 通常先渲染背景
    this.renderBackground(graphics);

    // 在渲染小部件之前渲染一些内容（背景纹理）

    // 如果这是 Screen 的直接子类，则渲染小部件
    super.render(graphics, mouseX, mouseY, partialTick);

    // 在渲染小部件之后渲染一些内容（工具提示）
}
```

### 关闭屏幕

当屏幕关闭时，有两个方法负责清理工作：`#onClose` 和 `#removed`。

每当用户输入关闭当前屏幕的指令时，就会调用 `#onClose` 方法。这个方法通常用作回调，用于销毁并保存屏幕自身的任何内部进程。这包括向服务器发送数据包。

`#removed` 方法会在屏幕即将切换并被释放给垃圾回收器之前调用。它会处理在屏幕打开之前未重置回初始状态的任何内容。

```java
// 在某个 Screen 子类中

@Override
public void onClose() {
    // 在此处停止任何处理程序

    // 最后调用，以防它干扰重写的逻辑
    super.onClose();
}

@Override
public void removed() {
    // 在此处重置初始状态

    // 最后调用，以防它干扰重写的逻辑
    super.removed();
}
```

## `AbstractContainerScreen`

如果屏幕直接与一个[菜单][menus]关联，那么应该继承 `AbstractContainerScreen`。`AbstractContainerScreen` 充当菜单的渲染器和输入处理程序，并包含与物品槽同步和交互的逻辑。因此，通常只需要重写或实现两个方法，就可以让一个容器屏幕正常工作。同样，为了便于理解，下面将按照通常遇到的顺序介绍容器屏幕的组件。

`AbstractContainerScreen` 通常需要三个参数：正在打开的容器菜单（由泛型 `T` 表示）、玩家的物品栏（仅用于显示名称）以及屏幕本身的标题。在这里，可以设置一些定位字段：

| 字段             | 描述                                                         |
| :--------------: | :----------------------------------------------------------- |
| `imageWidth`      | 用于背景的纹理宽度。通常位于一个 256 x 256 的 PNG 文件中，默认值为 176。 |
| `imageHeight`     | 用于背景的纹理高度。通常位于一个 256 x 256 的 PNG 文件中，默认值为 166。 |
| `titleLabelX`     | 屏幕标题将被渲染位置的相对 x 坐标。                         |
| `titleLabelY`     | 屏幕标题将被渲染位置的相对 y 坐标。                         |
| `inventoryLabelX` | 玩家物品栏名称将被渲染位置的相对 x 坐标。                   |
| `inventoryLabelY` | 玩家物品栏名称将被渲染位置的相对 y 坐标。                   |

!!! 重要
    在前面的部分提到，预计算的相对坐标应该在 `#init` 方法中设置。这一点仍然适用，因为这里提到的值不是预计算的坐标，而是静态值和相对坐标。

    图像的宽度和高度值是静态且不变的，因为它们代表背景纹理的大小。为了在渲染时更方便，在 `#init` 方法中会预计算两个额外的值（`leftPos` 和 `topPos`），它们标记了背景将被渲染的左上角位置。标签坐标是相对于这些值的。

    `leftPos` 和 `topPos` 也可以方便地用于渲染背景，因为它们已经代表了要传递给 `#blit` 方法的位置。

```java
// 在某个 AbstractContainerScreen 子类中
public MyContainerScreen(MyMenu menu, Inventory playerInventory, Component title) {
    super(menu, playerInventory, title);

    this.titleLabelX = 10;
    this.inventoryLabelX = 10;

    /*
     * 如果 'imageHeight' 发生变化，'inventoryLabelY' 也必须改变，
     * 因为该值取决于 'imageHeight' 的值。
     */
}
```

### 菜单访问

由于菜单被传递到屏幕中，菜单内通过物品槽、数据槽或自定义系统同步的任何值，现在都可以通过 `menu` 字段进行访问。

### 容器更新

当玩家存活并正在查看屏幕时，容器屏幕会在 `#tick` 方法中通过 `#containerTick` 方法进行更新。这实际上取代了容器屏幕中的 `#tick` 方法，其最常见的用途是更新配方书。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
protected void containerTick() {
    super.containerTick();

    // 在此处进行更新操作
}
```

### 渲染容器屏幕

容器屏幕通过三个方法进行渲染：`#renderBg` 用于渲染背景纹理，`#renderLabels` 用于在背景之上渲染任何文本，`#render` 方法包含了前两个方法，此外还提供了一个灰色背景和工具提示。

从 `#render` 方法开始，最常见的重写方式（通常也是唯一的情况）是添加背景，调用父类方法渲染容器屏幕，最后在其上方渲染工具提示。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    this.renderBackground(graphics);
    super.render(graphics, mouseX, mouseY, partialTick);

    /*
     * 这个方法是由容器屏幕添加的，用于渲染
     * 鼠标悬停物品槽的工具提示。
     */
    this.renderTooltip(graphics, mouseX, mouseY);
}
```

在父类的 `#render` 方法中，会调用 `#renderBg` 方法来渲染屏幕的背景。最标准的实现方式使用三个方法调用：两个用于设置，一个用于绘制背景纹理。

```java
// 在某个 AbstractContainerScreen 子类中

// 背景纹理的位置（assets/<命名空间>/<路径>）
private static final ResourceLocation BACKGROUND_LOCATION = new ResourceLocation(MOD_ID, "textures/gui/container/my_container_screen.png");

@Override
protected void renderBg(GuiGraphics graphics, float partialTick, int mouseX, int mouseY) {
    /*
     * 将背景纹理渲染到屏幕上。'leftPos' 和 'topPos' 应该已经代表了
     * 纹理应该渲染的左上角位置，因为它们是从 'imageWidth' 和 'imageHeight' 预计算得到的。
     * 两个零代表 256 x 256 PNG 文件内的整数 u/v 坐标。
     */
    graphics.blit(BACKGROUND_LOCATION, this.leftPos, this.topPos, 0, 0, this.imageWidth, this.imageHeight);
}
```

最后，`#renderLabels` 方法会被调用，用于在背景上方但工具提示下方渲染任何文本。这只是简单地使用字体来绘制相关的组件。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
protected void renderLabels(GuiGraphics graphics, int mouseX, int mouseY) {
    super.renderLabels(graphics, mouseX, mouseY);

    // 假设我们有某个 Component 'label'
    // 'label' 将在 'labelX' 和 'labelY' 位置绘制
    graphics.drawString(this.font, this.label, this.labelX, this.labelY, 0x404040);
}
```

!!! 注意
    在渲染标签时，**不需要**指定 `leftPos` 和 `topPos` 偏移量。这些偏移量已经在 `PoseStack` 中进行了转换，因此这个方法内的所有内容都是相对于这些坐标进行绘制的。

## 注册 AbstractContainerScreen

要将 `AbstractContainerScreen` 与菜单一起使用，需要对其进行注册。可以在 [**模组事件总线**][modbus] 上的 `FMLClientSetupEvent` 事件中调用 `MenuScreens#register` 方法来完成注册。

```java
// 在模组事件总线上监听事件
private void clientSetup(FMLClientSetupEvent event) {
    event.enqueueWork(
        // 假设 RegistryObject<MenuType<MyMenu>> MY_MENU
        // 假设 MyContainerScreen<MyMenu> 接受三个参数
        () -> MenuScreens.register(MY_MENU.get(), MyContainerScreen::new)
    );
}
```

!!! 警告
    `MenuScreens#register` 方法不是线程安全的，因此需要在并行调度事件提供的 `#enqueueWork` 方法内部调用。

[menus]: ./menus.md
[network]: ../networking/index.md
[screen]: #the-screen-subtype
[argb]: https://en.wikipedia.org/wiki/RGBA_color_model#ARGB32
[component]: ../concepts/internationalization.md#translatablecontents
[keymapping]: ../misc/keymappings.md#inside-a-gui
[modbus]: ../concepts/events.md#mod-event-bus
