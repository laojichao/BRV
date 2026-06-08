# BRV - Android RecyclerView 快速开发框架

## 项目概述

BRV 是一个功能强大的 Android RecyclerView 封装框架，通过简洁的 API 快速构建各类列表，支持多类型列表、分组展开折叠、拖拽排序、侧滑删除、分页加载等特性。

## 技术栈

- **语言**: Kotlin 1.6.10
- **最低 SDK**: 19 (brv) / 21 (sample)
- **目标 SDK**: 30 (brv) / 33 (sample)
- **构建工具**: Gradle 7.1.3
- **关键依赖**:
  - `androidx.recyclerview:recyclerview:1.2.0` - RecyclerView 核心
  - `io.github.scwang90:refresh-layout-kernel:2.0.5` - SmartRefreshLayout 下拉刷新/上拉加载
  - `com.github.liangjingkanji:StateLayout:1.4.2` - 缺省页状态管理
  - DataBinding / ViewBinding 支持

## 项目结构

```
BRV/
├── brv/                          # 核心库模块
│   └── src/main/java/com/drake/brv/
│       ├── BindingAdapter.kt     # 核心适配器，多类型列表、事件、选择、分组
│       ├── PageRefreshLayout.kt  # 分页刷新布局，扩展 SmartRefreshLayout
│       ├── DefaultDecoration.kt  # 分割线工具
│       ├── item/                 # 条目接口（通过实现接口扩展功能）
│       │   ├── ItemBind.kt       # 绑定数据回调
│       │   ├── ItemExpand.kt     # 可展开/折叠分组
│       │   ├── ItemHover.kt      # 粘性头部
│       │   ├── ItemDrag.kt       # 可拖拽
│       │   ├── ItemSwipe.kt      # 可侧滑删除
│       │   ├── ItemPosition.kt   # 获取条目位置
│       │   ├── ItemStableId.kt   # 稳定 ID
│       │   └── ItemDepth.kt      # 分组深度
│       ├── animation/            # 列表动画（Alpha/Scale/Slide）
│       ├── annotaion/            # 注解（AnimationType/ItemOrientation/DividerOrientation）
│       ├── binding/              # Observable 实现
│       ├── layoutmanager/        # 支持悬停的 LayoutManager（Linear/Grid/Staggered）
│       ├── listener/             # 监听器（拖拽/差异对比/点击防抖）
│       ├── reflect/              # 反射工具
│       └── utils/                # 工具类（BRV 全局配置/RecyclerView 扩展）
├── sample/                       # 示例应用
│   └── src/main/java/com/drake/brv/sample/
│       ├── model/                # 数据模型示例
│       ├── ui/                   # Activity/Fragment 示例
│       └── vm/                   # ViewModel 示例
├── docs/                         # 文档资源
└── build.gradle                  # 根构建配置
```

## 构建与运行

```bash
# 构建 brv 库
./gradlew :brv:assembleRelease

# 构建并运行示例应用
./gradlew :sample:installDebug

# 生成 API 文档
./gradlew :brv:dokkaHtml
```

## 核心 API 用法

### 1. 初始化全局配置（Application 中）

```kotlin
BRV.modelId = BR.m  // DataBinding 变量 ID
BRV.debounceClickInterval = 500  // 防抖点击间隔（毫秒）
```

### 2. 快速创建列表

```kotlin
// 使用扩展函数 setup
recyclerView.linear().setup {
    // 添加多类型
    addType<TypeA>(R.layout.item_a)
    addType<TypeB>(R.layout.item_b)

    // 点击事件（自带防抖）
    onClick(R.id.tv_title) {
        val model = getModel<TypeA>()
    }

    // 绑定数据
    onBind {
        // 通用绑定逻辑
    }
}

// 赋值数据
recyclerView.models = listOf(TypeA(), TypeB())
```

### 3. 数据模型实现 ItemBind 接口

```kotlin
class MyModel(var name: String) : ItemBind {
    override fun onBind(vh: BindingAdapter.BindingViewHolder) {
        vh.findView<TextView>(R.id.tv_name).text = name
    }
}
```

### 4. 分组展开/折叠

```kotlin
// 数据模型实现 ItemExpand 接口
class GroupModel : ItemExpand {
    override var itemGroupPosition: Int = 0
    override var itemExpand: Boolean = false
    override fun getItemSublist(): List<Any?>? = children
}

// 展开/折叠操作
adapter.expandOrCollapse(position, scrollTop = true, depth = -1)
```

### 5. 拖拽排序

```kotlin
// 数据模型实现 ItemDrag 接口
data class DragModel(override var itemOrientationDrag: Int = ItemOrientation.ALL) : ItemDrag
```

### 6. 侧滑删除

```kotlin
// 数据模型实现 ItemSwipe 接口
data class SwipeModel(override var itemOrientationSwipe: Int = ItemOrientation.ALL) : ItemSwipe
```

### 7. 分页加载（PageRefreshLayout）

```xml
<com.drake.brv.PageRefreshLayout
    android:id="@+id/pager"
    app:rv_id="@+id/rv">
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv" />
</com.drake.brv.PageRefreshLayout>
```

```kotlin
pager.onRefresh {
    // 请求第一页数据
}.onLoadMore {
    // 请求下一页数据
}

// 添加数据（自动处理分页）
pager.addData(dataList) { it.isNotEmpty() }
```

### 8. 分割线

```kotlin
recyclerView.divider {
    setColor(Color.GRAY)
    setDivider(1, dp = true)
    setMargin(16, dp = true)
}
```

### 9. 列表动画

```kotlin
adapter.setAnimation(AnimationType.SLIDE_BOTTOM)
adapter.animationRepeat = false  // 默认每个 item 只播放一次
```

### 10. 选择模式

```kotlin
adapter.onChecked { position, checked, allChecked -> }
adapter.setChecked(position, true)
adapter.checkedAll(true)
adapter.checkedReverse()
adapter.getCheckedModels<MyModel>()
```

## 接口扩展机制

BRV 通过实现接口来扩展条目功能，所有接口定义在 `com.drake.brv.item` 包下：

| 接口 | 功能 |
|------|------|
| `ItemBind` | 自定义绑定逻辑 |
| `ItemExpand` | 分组展开/折叠 |
| `ItemHover` | 粘性头部 |
| `ItemDrag` | 拖拽排序 |
| `ItemSwipe` | 侧滑删除 |
| `ItemPosition` | 获取条目位置 |
| `ItemStableId` | 稳定 ID（DiffUtil） |
| `ItemDepth` | 分组深度 |
| `ItemAttached` | View attach/detach 回调 |

## RecyclerView 扩展函数

定义在 `com.drake.brv.utils.RecyclerUtils.kt`：

- `linear()` / `grid()` / `staggered()` - 快速设置 LayoutManager
- `setup {}` - 配置 BindingAdapter
- `models` / `mutable` - 读写数据
- `addModels()` - 追加数据
- `setDifferModels()` - 差异对比更新
- `divider {}` / `dividerSpace()` - 添加分割线

## 注意事项

- DataBinding 变量名需在 `BRV.modelId` 中配置（如 `BR.m`）
- `addType` 泛型为接口类型时自动等效于 `addInterfaceType`，其子类都会匹配
- `models` 赋值 `List` 会被自动转为 `MutableList`，引用会改变
- 分组展开使用 `expandOrCollapse` 方法，`depth = -1` 表示递归展开全部
- 防抖点击默认间隔 500ms，可通过 `BRV.debounceClickInterval` 全局修改
