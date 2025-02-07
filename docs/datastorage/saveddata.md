# 保存数据

保存数据（SD）系统是一种替代世界能力的方式，它可以为每个世界附加数据。

## 声明

每个 SD 实现都必须继承 `SavedData` 类。需要注意两个重要的方法：

- `save`：允许实现将 NBT 数据写入世界。
- `setDirty`：在数据更改后必须调用的方法，用于通知游戏有需要写入的更改。如果不调用，`#save` 方法将不会被调用，现有数据将保持不变。

## 附加到世界

任何 `SavedData` 都会动态地加载和/或附加到一个世界。因此，如果一个世界中从未创建过 `SavedData`，那么它就不会存在。

`SavedData` 是从 `DimensionDataStorage` 中创建和加载的，可以通过 `ServerChunkCache#getDataStorage` 或 `ServerLevel#getDataStorage` 来访问它。从那里，你可以通过调用 `DimensionDataStorage#computeIfAbsent` 来获取或创建你的 SD 实例。这将尝试获取当前的 SD 实例（如果存在），或者创建一个新实例并加载所有可用的数据。

`DimensionDataStorage#computeIfAbsent` 接受三个参数：一个将 NBT 数据加载到 SD 并返回它的函数、一个构造 SD 新实例的提供者，以及为实现的世界存储在 `data` 文件夹中的 `.dat` 文件的名称。

例如，如果一个 SD 在地狱中被命名为 “example”，那么将在 `./<世界文件夹>/DIM - 1/data/example.dat` 创建一个文件，并按如下方式实现：

```java
// 在某个类中
public ExampleSavedData create() {
  return new ExampleSavedData();
}

public ExampleSavedData load(CompoundTag tag) {
  ExampleSavedData data = this.create();
  // 加载保存的数据
  return data;
}

// 在类中的某个方法中
netherDataStorage.computeIfAbsent(this::load, this::create, "example");
```

为了使 SD 在多个世界中持久化，应该将 SD 附加到主世界，主世界可以从 `MinecraftServer#overworld` 获取。主世界是唯一一个永远不会完全卸载的维度，因此非常适合存储跨世界的数据。
