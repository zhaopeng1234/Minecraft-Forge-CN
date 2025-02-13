### 语言文件生成
通过继承 `LanguageProvider` 并实现 `#addTranslations` 方法，可以为模组生成[语言文件][lang]。创建的每个 `LanguageProvider` 子类代表一个单独的[语言环境]（例如 `en_us` 代表美式英语，`es_es` 代表西班牙语等）。实现之后，必须将该提供者 [添加][datagen] 到 `DataGenerator` 中。

```java
// 在 MOD 事件总线上
@SubscribeEvent
public void gatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        // 告诉生成器仅在生成客户端资源时运行
        event.includeClient(),
        // 美式英语的本地化
        output -> new MyLanguageProvider(output, MOD_ID, "en_us")
    );
}
```

### `LanguageProvider`
每个语言提供者本质上是一个字符串映射，其中每个翻译键都映射到一个本地化名称。可以使用 `#add` 方法添加翻译键映射。此外，还有一些方法可使用 `Block`、`Item`、`ItemStack`、`Enchantment`、`MobEffect` 和 `EntityType` 的翻译键。

```java
// 在 LanguageProvider#addTranslations 方法中
this.addBlock(EXAMPLE_BLOCK, "Example Block");
this.add("object.examplemod.example_object", "Example Object");
```

!!! 提示
    包含非美式英语字母数字值的本地化名称可以直接提供。提供者会自动将这些字符转换为其对应的 Unicode 形式，以便游戏读取。

    ```java
    // 编码为 'Example with a d\u00EDacritic'
    this.addItem("example.diacritic", "Example with a díacritic");
    ```

[lang]: ../../concepts/internationalization.md
[locale]: https://minecraft.wiki/w/Language#Languages
[datagen]: ../index.md#data-providers
