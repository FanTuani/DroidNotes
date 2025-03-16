# Fragment

## 动态添加 Fragment

过程分为五步：

1. 创建待添加 Fragment 的实例。
2. 获取 FragmentManager，在 Activity 中可以直接调用`getSupportFragmentManager()`
   方法获取。
3. 开启一个事务，通过调用`beginTransaction()`方法开启。
4. 向容器内添加或替换 Fragment，一般使用`replace()`方法实现，需要传入容器的 id 和待添加的 Fragment 实例。
5. 提交事务，调用`commit()`方法来完成。

```kotlin
private fun replaceFragment(fragment: Fragment) { 
    val fragmentManager = supportFragmentManager 
    val transaction = fragmentManager.beginTransaction() 
    transaction.replace(R.id.rightLayout, fragment)
    addToBackStack(null) // 添加到返回栈
    transaction.commit() 
} 
```

```kotlin
class MyFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_main, container, false)
    }
}
```

## Fragment 与 Activity 交互

从布局文件中获取 Fragment 实例：

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.leftFrag) 
                as LeftFragment 
```

从 Fragment 中获取父 Activity 实例：

```kotlin
if (activity != null) { // 需要
    val mainActivity = activity as MainActivity 
}
```

此处的 activity 同样也是一个 Context 实例。

## Fragment 的状态与声明周期

分为**运行状态**、**暂停状态**、**停止状态**、**销毁状态**。

四种状态均与所关联的 Activity 同步。其中：调用 FragmentTransaction 的 remove()、replace() 方法会使**没有被加入返回栈**的 Fragment 进入**销毁状态**，使**被加入返回栈**的 Fragment 进入**停止状态**。

Fragment 类中存在与 Activity 相同的回调方法，也有附加的额外回调方法：

- `onAttach()`：当Fragment和Activity建立关联时调用。

- `onCreateView()`：为Fragment创建视图（加载布局）时调用。

- `onActivityCreated()`：确保与Fragment相关联的Activity已经创建完毕时调用。

- `onDestroyView()`：当与Fragment关联的视图被移除时调用。

- `onDetach()`：当Fragment和Activity解除关联时调用。

## 平板适配（限定符）

新建 layout-large 文件夹，新建 activity_main.xml 布局（注意此处新建布局应与原先的 activity_main.xml 布局拥有相同的id），屏幕被认为是 large 的设备会自动加载 layout-large 文件夹下的布局。

通过新建 layout-sw600dp 文件夹，可以指定屏幕宽度大于等于 600dp 的设备使用该文件夹下的布局。

## 为 Fragment 启用 ViewBinding

```kotlin
class NewsContentFrag : Fragment() {
    private var _binding: NewsContentFragBinding? = null
    private val binding get() = _binding!! // 只读访问
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = NewsContentFragBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null // 防止内存泄漏
    }
}
```
