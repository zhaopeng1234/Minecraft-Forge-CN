
国际化和本地化
=====================================
国际化，或i18n，是一种设计代码的方式，使其不需要更改即可适应各种语言。本地化是使显示的文本适应用户语言的过程。

I18n是使用_翻译键translation keys_实现的。翻译键是一个字符串，用于标识一段没有特定语言的可显示文本。例如，`block.minecraft.dirt`是引用土方块名称的翻译键。这样，可以引用可显示文本而不关心特定语言。代码不需要更改即可适应新语言。

本地化将在游戏的语言环境中进行。在Minecraft客户端中，语言环境由语言设置指定。在专用服务器上，唯一支持的语言环境是`en_us`。可以在[Minecraft Wiki][langs]上找到可用语言环境的列表。


语言文件Language files
--------------
语言文件位于`assets/[namespace]/lang/[locale].json`（例如`examplemod` 提供的所有美式英语的翻译位于`assets/examplemod/lang/en_us.json`文件中）。文件格式只是简单的从翻译键到值的json map。文件必须以UTF-8编码。旧的.lang文件可以使用[转换器][converter]转换为json。

```js
{
  "item.examplemod.example_item": "Example Item Name",
  "block.examplemod.example_block": "Example Block Name",
  "commands.examplemod.examplecommand.error": "Example Command Errored!"
}
```

方块和物品的使用
---------------------------
Block、Item 和其他一些Minecraft类具有用于显示其名称的内置翻译键。这些翻译键是通过重载`#getDescriptionId`指定的。Item还有`#getDescriptionId(ItemStack)`可以重载此方法并根据ItemStack的NBT提供不同的翻译键。

默认情况下，`#getDescriptionId`将返回`block.`或`item.`添加到方块或物品的注册表名称前，并将`:`替换为`.`。`BlockItem`重载此方法以获取其对应的`block`的翻译key默认值。例如，ID为`examplemod:example_item`的物品在语言文件中实际上需要以下行：

```js
{
  "item.examplemod.example_item": "Example Item Name"
}
```

!!! 注意
    翻译键的唯一目的是国际化。不要将它们用于逻辑，应该使用注册表名称。


本地化方法
--------------------

!!! 警告
   一个常见的问题是让服务器为客户端本地化。服务器只能在自己的语言环境中本地化，这不一定与连接的客户端的语言环境匹配。
    为了尊重客户端的语言设置，服务器应该让客户端使用`TranslatableComponent`或其他确保翻译键与语言无关的方法将文本本地化为自己的语言。
    

### `net.minecraft.client.resources.language.I18n` (仅客户端)

**这个I18n类只能在Minecraft客户端上找到！** 它旨在供仅在客户端上运行的代码使用。尝试在服务器上使用它会引发异常并崩溃。

- `get(String, Object...)` 通过格式化字符串在客户端的语言环境中本地化。第一个参数是翻译键，其余参数是`String.format(String, Object...)`的格式化参数。

### 可翻译内容`TranslatableContents`

`TranslatableContents`是一个`ComponentContents`，它被延迟本地化和格式化。向玩家发送消息时非常有用，因为它会自动在他们自己的语言环境中本地化。

`TranslatableContents(String, Object...)`构造函数的第一个参数是翻译键，其余的参数被用于格式化。唯一支持的格式说明符是`%s`和`%1$s`、`%2$s`、`%3$s`等。格式化参数可以是`Component`，它们将被插入到生成的格式化文本中，并保留所有属性。

`MutableComponent`可以使用`Component#translatable`传入`TranslatableContents`的参数创建。它也可以使用`MutableComponent#create`传入`ComponentContents`创建。

### 文本组件助手`TextComponentHelper`
- `createComponentTranslation(CommandSource, String, Object...)`根据接收者创建本地化和格式化的`MutableComponent`。如果接收者是原版的客户端，则急切地进行本地化和格式化。如果不是，则使用包含`TranslatableContents`的`Component`进行本地化和格式化。这仅在服务器允许原版的客户端连接时有用。


[langs]: https://minecraft.wiki/w/Language#Languages
[converter]: https://tterrag.com/lang2json/
