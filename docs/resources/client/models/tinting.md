## 纹理着色
在原版游戏中，许多方块和物品会根据它们所处的位置或自身属性改变纹理颜色，比如草方块。模型支持在面上指定 “着色索引”，这些索引是整数，随后可由 `BlockColor` 和 `ItemColor` 处理。关于在原版模型中如何定义着色索引的信息，请参考[维基百科][wiki]。

### `BlockColor`/`ItemColor`
这两个都是单方法接口。`BlockColor` 接收一个 `BlockState`、一个（可为空的）`BlockAndTintGetter` 以及一个（可为空的）`BlockPos`。`ItemColor` 接收一个 `ItemStack`。它们都接收一个 `int` 类型的参数 `tintIndex`，即要着色面的着色索引。它们都返回一个 `int` 类型的颜色乘数。这个 `int` 被视为 4 个无符号字节，按顺序为透明度（alpha）、红色、绿色和蓝色，从最高有效字节到最低有效字节。对于着色面上的每个像素，每个颜色通道的值为 `(int)((float) base * multiplier / 255.0)`，其中 `base` 是该通道的原始值，`multiplier` 是颜色乘数中对应的字节值。请注意，方块不使用透明度通道。例如，未着色的草纹理看起来是白色和灰色的。草的 `BlockColor` 和 `ItemColor` 返回的颜色乘数中红色和蓝色分量较低，但透明度和绿色分量较高（至少在温暖的生物群系中是这样），所以在进行乘法运算时，绿色会更突出，而红色/蓝色则会减弱。

如果一个物品继承自 `builtin/generated` 模型，那么每个图层（“layer0”、“layer1” 等）都有一个与其图层索引相对应的着色索引。

### 创建颜色处理器
`BlockColor` 需要注册到游戏的 `BlockColors` 实例中。可以通过 `RegisterColorHandlersEvent$Block` 获取 `BlockColors`，并通过 `#register` 方法注册一个 `BlockColor`。请注意，这不会导致给定方块的 `BlockItem` 被着色。`BlockItem` 属于物品，需要使用 `ItemColor` 进行着色。

```java
@SubscribeEvent
public void registerBlockColors(RegisterColorHandlersEvent.Block event){
  event.register(myBlockColor, coloredBlock1, coloredBlock2,...);
}
```

`ItemColor` 需要注册到游戏的 `ItemColors` 实例中。可以通过 `RegisterColorHandlersEvent$Item` 获取 `ItemColors`，并通过 `#register` 方法注册一个 `ItemColor`。此方法有重载形式，也可以接收 `Block`，这只是为 `Block#asItem`（即方块的 `BlockItem`）注册颜色处理器。

```java
@SubscribeEvent
public void registerItemColors(RegisterColorHandlersEvent.Item event){
  event.register(myItemColor, coloredItem1, coloredItem2,...);
}
```

[wiki]: https://minecraft.wiki/w/Tutorials/Models#Block_models
