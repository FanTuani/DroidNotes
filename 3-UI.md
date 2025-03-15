# UI

## ViewBinding

在 app/build.gradle.kts 中，添加：

```kotlin
android {
    ...
    viewBinding {
        enable = true
    }
}
```

为某个模块启用视图绑定功能后，系统会为该模块中包含的每个 XML 布局文件生成一个绑定类。每个绑定类均包含对**根视图**以及**具有 ID 的所有视图**的引用。系统会通过以下方式生成绑定类名称：**将 XML 文件的名称转换为 Pascal 大小写形式，并在末尾添加“Binding”一词**。

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: MainLayoutBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = MainLayoutBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

## 控件

### 使用实现接口方式注册监听器

```kotlin
class MainActivity : AppCompatActivity(), View.OnClickListener { 
    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        setContentView(R.layout.activity_main) 
        button.setOnClickListener(this) 
    } 
    override fun onClick(v: View?) { 
        when (v?.id) { 
            R.id.button -> { 
                // 在此处添加逻辑 
            } 
        } 
    } 
} 
```

### ImageView

图片通常是放在 drawable 下的 drawable-xxhdpi 目录。

使用`android:src="@drawable/img_1"`指定图片。

### 显示/隐藏控件

使用`android:visibility`进行指定，其中指定为`gone`时控件将不再占用屏幕空间。例如`progressBar.visibility = View.GONE`。

### AlertDialog

```kotlin
AlertDialog.Builder(this).apply { 
    setTitle("This is Dialog") 
    setMessage("Something important.") 
    setCancelable(false) 
    setPositiveButton("OK") { dialog, which -> } 
    setNegativeButton("Cancel") { dialog, which -> } 
    show() 
} 
```

## 布局

### LinearLayout

`android:gravity`指定文字**在控件中**的对齐方式，而`android:layout_gravity`指定控件**在布局中**的对齐方式。

当 LinearLayout 的排列方向是 horizontal 时，只有垂直方向上的对齐方式才会生效。因为此时水平方向上的长度是不固定的，每添加一个控件，水平方向上的长度都会改变，因而无法指定该方向上的对齐方式。同样的道理，当 LinearLayout 的排列方向是 vertical 时，只有水平方向上的对齐方式才会生效。

**android:layout_weight**

使用后应将原宽度/高度指定为 0dp。实际宽度按 weight 总和占比计算。

### RelativeLayout

通过`android:layout_alignParentLeft`等属性相对于父布局定位。

通过`android:layout_toLeftOf="@id/button3"`等属性相对于其它空间定位。

### 引入布局

通过`<include layout="@layout/title"/>`引用。

### 创建自定义控件

```kotlin
class TitleLayout(context:Context, attrs: AttributeSet) : 
        LinearLayout(context, attrs) {
    init {
        LayoutInflater.from(context).inflate(R.layout.title, this)
        // this 为父布局
        // 设置控件监听器
    }
}
```

在 main_layout.xml 中这样使用：

```xml
<top.ricequakes.myapplication.TitleLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

## ListView

**使用样例**

数据需要借助 adapter 传递给 ListView：

```kotlin
val adapter = ArrayAdapter<String>
              (this, android.R.layout.simple_list_item_1, data)
binding.listView.adapter = adapter
```

这里使用了 android.R.layout.simple_list_item_1 作为 ListView 子项布局的 id，其内部只有一个 TextView。

**自定义 ListView 界面**

1. **定义子项实体类：**

```kotlin
class Fruit(val name: String, val info: String)
```

2. **为子项指定布局（新建 xml 文件）**

3. **创建自定义 adapter：**

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) :
    ArrayAdapter<Fruit>(activity, resourceId, data) {
    @SuppressLint("ViewHolder")
    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view = LayoutInflater.from(context).inflate(resourceId, parent, false)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
        val fruitButton: Button = view.findViewById(R.id.fruitButton)
        val fruit = getItem(position) // 获取当前项的Fruit实例
        if (fruit != null) {
            fruitButton.text = fruit.info
            fruitName.text = fruit.name
        }
        return view
    }
}
```

其中，`getView()`方法会在子项滚动到屏幕内的时候调用。

adapter 的逻辑：`数据 (List<Fruit>)  →  适配器 (Adapter)  →  生成 View  →  UI 组件 (ListView / RecyclerView)`

adapter 需要给 ListView 提供：

- **数据源**（如 `List<Fruit>`）
- **View 布局**（如 `fruit_item.xml`）
- **数据绑定逻辑**（`getView()` 方法）
4. **创建 ListView**

```kotlin
val adapter = FruitAdapter(this, R.layout.fruit_layout, data)
binding.listView.adapter = adapter
```

**提升 ListView 运行效率**

由于`getView()`方法每次都将布局重新加载，ListView 的快速滚动性能会很差。

convertView 参数用于将先前加载好的布局缓存，可以用于性能优化。

```kotlin
val view = convertView ?: LayoutInflater.from(context).inflate(resourceId, parent, false)
```

为了进一步优化`findViewById()`带来的性能损耗，可以借助一个 **ViewHolder** 进行优化。

```kotlin
inner class ViewHolder(val fruitNameView: TextView, val fruitButton: Button)

override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view: View
        val viewHolder: ViewHolder
        if (convertView == null) {
            view = LayoutInflater.from(context).inflate(resourceId, parent, false)
            viewHolder = ViewHolder(
                view.findViewById(R.id.fruitName),
                view.findViewById(R.id.fruitButton)
            )
            view.tag = viewHolder
        } else {
            view = convertView
            viewHolder = view.tag as ViewHolder
        }
        val fruit = getItem(position) // 获取当前项的Fruit实例
        if (fruit != null) {
            viewHolder.fruitButton.text = fruit.info
            viewHolder.fruitNameView.text = fruit.name
        }
        return view
    }
```

**设置子项点击事件**

```kotlin
binding.listView.setOnItemClickListener { parent, view, position, id ->
    val fruit = data[position]
    Toast.makeText(this, fruit.name, Toast.LENGTH_SHORT).show()
}
```

> 当一个控件（如 `Button`、`EditText`）被 **设置为可聚焦** 时，它可以：
> 
> - **响应键盘输入**（如 `EditText`）。
> - **通过键盘、方向键、Tab 切换选中**（如 `Button`）。
> - **阻止 `ListView` 等父级控件接收点击事件**（如果 `Button` 可聚焦，`ListView` 的 `onItemClick` 可能不会触发）。

## RecyclerView

创建 adapter：

```kotlin
class FruitAdapter(private val data: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {

    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val fruitNameView: TextView = view.findViewById(R.id.fruitName)
        val fruitButton: Button = view.findViewById(R.id.fruitButton)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.fruit_layout, parent, false)
        return ViewHolder(view)
    }

    override fun getItemCount() = data.size

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = data[position]
        holder.fruitNameView.text = fruit.name
        holder.fruitButton.text = "info"
    }
}
```

其中，需要重写三个方法：

- `onCreateViewHolder()`用于加载布局并创建 ViewHolder 实例

- `getItemCount()`用于获取列表总长度

- `onBindViewHolder()`用于子项进入屏幕时绑定数据

另外，使用 RecyclerView 时还应指定 LayoutManager：

```kotlin
binding.recyclerView.layoutManager = LinearLayoutManager(this)
val adapter = FruitAdapter(data)
binding.recyclerView.adapter = adapter
```

> ？ViewHolder 的复用是如何实现的？

三列瀑布流布局

```kotlin
binding.recyclerView.layoutManager =
    StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL)
```
