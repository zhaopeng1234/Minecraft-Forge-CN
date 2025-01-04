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
    - [BlockEntityWithoutLevelRenderer](/docs/items/bewlr.md)
* ### Networking:
    - [Introduction](/docs/networking/index.md)
    - [SimpleImpl](/docs/networking/simpleimpl.md)
* ### Block Entities:
    - [Introduction](/docs/blockentities/index.md)
    - [BlockEntityRenderer](/docs/blockentities/ber.md)
* ### Game Effects:
    - [Particles](/docs/gameeffects/particles.md)
    - [Sounds](/docs/gameeffects/sounds.md)
* ### Data Storage:
    - [Capabilities](/docs/datastorage/capabilities.md)
    - [Saved Data](/docs/datastorage/saveddata.md)
    - [Codecs](/docs/datastorage/codecs.md)
* ### Graphical User Interfaces:
    - [Menus](/docs/gui/menus.md)
    - [Screens](/docs/gui/screens.md)
* ### Rendering:
    * #### Model Extensions:
      - [Introduction](/docs/rendering/modelextensions/index.md)
      - [Root Transforms](/docs/rendering/modelextensions/transforms.md)
      - [Render Types](/docs/rendering/modelextensions/rendertypes.md)
      - [Part Visibility](/docs/rendering/modelextensions/visibility.md)
      - [Face Data](/docs/rendering/modelextensions/facedata.md)
    * #### Model Loaders:
      - [Introduction](/docs/rendering/modelloaders/index.md)
      - [Baked Model](/docs/rendering/modelloaders/bakedmodel.md)
      - [Transform](/docs/rendering/modelloaders/transform.md)
      - [Item Overrides](/docs/rendering/modelloaders/itemoverrides.md)
* ### Resources:
    * #### Client Assets:
      - [Introduction](/docs/resources/client/index.md)
      - Models:
        - [Introduction](/docs/resources/client/models/index.md)
        - [Texture Tinting](/docs/resources/client/models/tinting.md)
        - [Item Properties](/docs/resources/client/models/itemproperties.md)
    * #### Server Data:
      - [Introduction](/docs/resources/server/index.md)
      - Recipes:
        - [Introduction](/docs/resources/server/recipes/index.md)
        - [Custom Recipes](/docs/resources/server/recipes/custom.md)
        - [Ingredients](/docs/resources/server/recipes/ingredients.md)
        - [Non-Datapack Recipes](/docs/resources/server/recipes/incode.md)
      - [Loot Tables](/docs/resources/server/loottables.md)
      - [Global Loot Modifiers](/docs/resources/server/glm.md)
      - [Tags](/docs/resources/server/tags.md)
      - [Advancements](/docs/resources/server/advancements.md)
      - [Conditionally-Loaded Data](/docs/resources/server/conditional.md)
* ### Data Generation:
    * [Introduction](/docs/datagen/index.md)
    * #### Client Assets:
      - [Model Providers](/docs/datagen/client/modelproviders.md)
      - [Language Providers](/docs/datagen/client/localization.md)
      - [Sound Providers](/docs/datagen/client/sounds.md)
    * #### Server Data:
      - [Recipe Providers](/docs/datagen/server/recipes.md)
      - [Loot Table Providers](/docs/datagen/server/loottables.md)
      - [Tag Providers](/docs/datagen/server/tags.md)
      - [Advancement Providers](/docs/datagen/server/advancements.md)
      - [Global Loot Modifier Providers](/docs/datagen/server/glm.md)
      - [Datapack Registry Object Providers](/docs/datagen/server/datapackregistries.md)
* ### Miscellaneous Features:
    - [Configuration](/docs/misc/config.md)
    - [Key Mappings](/docs/misc/keymappings.md)
    - [Game Tests](/docs/misc/gametest.md)
    - [Forge Update Checker](/docs/misc/updatechecker.md)
    - [Debug Profiler](/docs/misc/debugprofiler.md)
* ### Advanced Topics:
    - [Access Transformers](/docs/advanced/accesstransformers.md)
* ### Contributing to Forge:
    - [Introduction](/docs/forgedev/index.md)
    - [Pull Request Guidelines](/docs/forgedev/prguidelines.md)
* ### Legacy Versions:
    - [Introduction](/docs/legacy/index.md)
    - [Porting to This Version](/docs/legacy/porting.md)
