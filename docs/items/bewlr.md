
BlockEntityWithoutLevelRenderer
=======================
`BlockEntityWithoutLevelRenderer`是一种处理物品动态渲染的方法，这个系统比旧的`ItemStack`系统简单得多，后者需要`BlockEntity`，并且不允许访问`ItemStack`。


使用BEWLR(BlockEntityWithoutLevelRenderer)
--------------------------
BlockEntityWithoutLevelRenderer允许您使用`public void renderByItem(ItemStack itemStack, ItemDisplayContext ctx, PoseStack poseStack, MultiBufferSource bufferSource, int combinedLight, int combinedOverlay)``渲染您的物品。

为了使用BEWLR，`Item`必须首先满足模型的`BakedModel#isCustomRenderer`返回true。如果没有，它将使用默认`IItemRenderer#getBlockEntityRenderer`。一旦返回true，物品的BEWLR将被访问进行渲染。

!!! 注意
    `Block`也可以使用BEWLR进行渲染，只要`Block#getRenderShape`被设置为`RenderShape#ENTITYBLOCK_ANIMATED`。

要设置物品的BEWLR，必须在`Item#initializeClient`方法中消费`IClientItemExtensions`的匿名实例。在匿名实例中，应该重写`IClientItemExtensions#getCustomRenderer`以返回BEWLR的实例：


```java
// 在物品的类中
@Override
public void initializeClient(Consumer<IClientItemExtensions> consumer) {
  consumer.accept(new IClientItemExtensions() {

    @Override
    public BlockEntityWithoutLevelRenderer getCustomRenderer() {
      return myBEWLRInstance;
    }
  });
}
```

!!! 重要
   每个mod应该只有一个自定义BEWLR的实例。

使用BEWLR不需要额外的设置。
