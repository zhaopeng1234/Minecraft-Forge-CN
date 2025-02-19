# 文档
这里**不是**[MinecraftForge](https://docs.minecraftforge.net)官方文档，仅为本人个人翻译行为。

目的是方便中文开发者能够跨过语言关卡，更方便地学习和开发minecraft的mod。然而，这仍然需要你具有一定的java开发经验，以处理代码相关的问题。

对于一些原文档中没有详细描述的部分，我也会添加一些自己开发过程中的经验。如果翻译或示例有错误，欢迎在讨论区指出。

<br/>

# 目录

* ### 从零开始:
    - [Forge 入门](/docs/gettingstarted/index.md)
    - [Mod文件](/docs/gettingstarted/modfiles.md)
    - [结构化你的mod](/docs/gettingstarted/structuring.md)
    - [版本控制](/docs/gettingstarted/versioning.md)
* ### 核心概念:
    - [注册表](/docs/concepts/registries.md)
    - [端](/docs/concepts/sides.md)
    - [事件](/docs/concepts/events.md)
    - [Mod生命周期](/docs/concepts/lifecycle.md)
    - [资源](/docs/concepts/resources.md)
    - [国际化](/docs/concepts/internationalization.md)
* ### 方块:
    - [简介](/docs/blocks/index.md)
    - [方块状态](/docs/blocks/states.md)
* ### 物品:
    - [简介](/docs/items/index.md)
    - [BEWLR(BlockEntityWithoutLevelRenderer)](/docs/items/bewlr.md)
* ### 网络连接:
    - [介绍](/docs/networking/index.md)
    - [SimpleImpl](/docs/networking/simpleimpl.md)
* ### 方块实体(Block Entities):
    - [介绍](/docs/blockentities/index.md)
    - [BlockEntityRenderer](/docs/blockentities/ber.md)
* ### 游戏效果:
    - [粒子](/docs/gameeffects/particles.md)
    - [声音](/docs/gameeffects/sounds.md)
* ### 数据保存:
    - [能力](/docs/datastorage/capabilities.md)
    - [保存的数据](/docs/datastorage/saveddata.md)
    - [编解码器](/docs/datastorage/codecs.md)
* ### 图形化用户界面(GUI):
    - [菜单](/docs/gui/menus.md)
    - [屏幕](/docs/gui/screens.md)
* ### 渲染:
    * #### 模型扩展:
      - [~~介绍(缺失)~~](/docs/rendering/modelextensions/index.md)
      - [根变换](/docs/rendering/modelextensions/transforms.md)
      - [渲染类型](/docs/rendering/modelextensions/rendertypes.md)
      - [部件可见性](/docs/rendering/modelextensions/visibility.md)
      - [面数据](/docs/rendering/modelextensions/facedata.md)
    * #### 模型加载器:
      - [介绍](/docs/rendering/modelloaders/index.md)
      - [Baked Model](/docs/rendering/modelloaders/bakedmodel.md)
      - [变换](/docs/rendering/modelloaders/transform.md)
      - [Item Overrides](/docs/rendering/modelloaders/itemoverrides.md)
* ### 资源:
    * #### 客户端资产:
      - [介绍](/docs/resources/client/index.md)
      - 模型:
        - [介绍](/docs/resources/client/models/index.md)
        - [纹理着色](/docs/resources/client/models/tinting.md)
        - [物品属性](/docs/resources/client/models/itemproperties.md)
    * #### 服务端数据:
      - [介绍](/docs/resources/server/index.md)
      - 配方:
        - [介绍](/docs/resources/server/recipes/index.md)
        - [自定义配方](/docs/resources/server/recipes/custom.md)
        - [原料](/docs/resources/server/recipes/ingredients.md)
        - [非数据包合成配方](/docs/resources/server/recipes/incode.md)
      - [战利品表](/docs/resources/server/loottables.md)
      - [全局战利品修改器](/docs/resources/server/glm.md)
      - [标签](/docs/resources/server/tags.md)
      - [进度系统](/docs/resources/server/advancements.md)
      - [条件加载数据](/docs/resources/server/conditional.md)
* ### 数据生成器:
    * [介绍](/docs/datagen/index.md)
    * #### 客户端资产:
      - [模型提供者](/docs/datagen/client/modelproviders.md)
      - [语言提供者](/docs/datagen/client/localization.md)
      - [声音提供者](/docs/datagen/client/sounds.md)
    * #### 服务器数据:
      - [配方提供者](/docs/datagen/server/recipes.md)
      - [战利品表表提供者](/docs/datagen/server/loottables.md)
      - [标签提供者](/docs/datagen/server/tags.md)
      - [进度提供者](/docs/datagen/server/advancements.md)
      - [全局战利品修饰符提供者](/docs/datagen/server/glm.md)
      - [数据包注册表对象提供者](/docs/datagen/server/datapackregistries.md)
* ### 其他功能:
    - [配置](/docs/misc/config.md)
    - [按键映射](/docs/misc/keymappings.md)
    - [游戏测试](/docs/misc/gametest.md)
    - [Forge更新检查器](/docs/misc/updatechecker.md)
    - [调试分析器](/docs/misc/debugprofiler.md)
* ### 高级话题:
    - [访问转换器](/docs/advanced/accesstransformers.md)
* ### 给Forge做出贡献:
    - [介绍](/docs/forgedev/index.md)
    - [Pull Request指南](/docs/forgedev/prguidelines.md)
* ### 旧版本:
    - [介绍](/docs/legacy/index.md)
    - [迁移到此版本](/docs/legacy/porting.md)
