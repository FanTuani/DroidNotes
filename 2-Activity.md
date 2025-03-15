# Activity

## Toast

```kotlin
Toast.makeText(context, "title", Toast.LENGTH_SHORT).show()
```

## 启动 Activity

**使用显式 Intent**

```kotlin
val intent = Intent(context, SecondActivity::class.java)
startActivity(intent)
```

**使用隐式 Intent**

编辑 AndroidManifest.xml，在`<activity>`标签下配置`<intent-filter>`的内容，可以指定当前 Activity 能够响应的 action 和 category。二者同时匹配则可以响应。action 必须唯一，但 category 可以有多个。

```xml
<intent-filter> 
    <action android:name="com.example.activitytest.ACTION_START" /> 
    <category android:name="android.intent.category.DEFAULT" /> 
</intent-filter> 
```

## Activity 生命周期

### 返回栈

Android 使用任务（task）来管理Activity，一个任务就是一组存放在栈里的 Activity 的集合，这个栈也被称作返回栈（back stack）。

### Activity 状态

每个 Activity 在其生命周期中最多可能会有4种状态：

**1. 运行状态**

**位于栈顶**的 Activity 处于运行状态。一般不被系统回收。

**2. 暂停状态**

**不位于栈顶**但**仍然可见**的 Activity 处于暂停状态（如上层 Activity 未占满屏幕）。内存极低情况下会被系统回收。

**3. 停止状态**

**不位于栈顶**且**完全不可见**的 Activity 处于停止状态。系统会为这种 Activity 保存相应的状态和成员变量，但并不可靠，可能被系统回收。

**4. 销毁状态**

**从返回栈中移除后**的 Activity 处于销毁状态。系统最倾向于回收处于这种状态的 Activity。

### Activity 生存期

- `onCreate()`。在Activity第一次被创建的时候调用。应该在这个方法中完成Activity的初始化操作，比如加载布局、绑定事件等。（进入完整生存期）

- `onStart()`。在Activity由不可见变为可见的时候调用。（进入可见生存期）

- `onResume()`。在Activity准备好和用户进行交互的时候调用。此时的Activity一定位于返回栈的栈顶，并且处于运行状态。（进入前台生存期）

- `onPause()`。在系统准备启动或恢复另一个Activity的时候调用。（退出前台生存期）

- `onStop()`。在Activity完全不可见的时候调用。它和onPause()方法的主要区别在于，如果启动的新Activity是一个对话框式的Activity，那么onPause()方法会得到执行，而onStop()方法并不会执行。（退出可见生存期）

- `onDestroy()`在Activity被销毁之前调用，之后Activity的状态将变为销毁状态。（退出完整生存期）

- `onRestart()`。在Activity由停止状态变为运行状态之前调用，也就是Activity被重新启动了。

### Activity 被回收时保存数据

重写`onSaveInstanceState(outState: Bundle)`函数：

```kotlin
override fun onSaveInstanceState(outState: Bundle) { 
    super.onSaveInstanceState(outState) 
    val tempData = "Something you just typed" 
    outState.putString("data_key", tempData) 
} 
override fun onCreate(savedInstanceState: Bundle?) // 取出
```

## Activity 四种启动模式

- standard

- singleTop

- singleTask

- singleInstance

## Activity 最佳实践

### 找到当前所处的 Activity

新建 BaseActivity 类继承 AppCompatActivity，并使其成为其它 Activity 类的父类，在其 onCreate 方法中打印类名即可。

### 随时随地退出程序

使用单例类管理全部现存 Activity：

```kotlin
object ActivityCollector { 
    private val activities = ArrayList<Activity>() 
    fun addActivity(activity: Activity) { 
        activities.add(activity) 
    } 
    fun removeActivity(activity: Activity) { 
        activities.remove(activity) 
    } 
    fun finishAll() { 
        for (activity in activities) { 
            if (!activity.isFinishing) { 
                activity.finish() 
            } 
        } 
        activities.clear() 
    } 
} 
```

配合刚才的 BaseActivity：

```kotlin
open class BaseActivity : AppCompatActivity() { 
    override fun onCreate(savedInstanceState: Bundle?) { 
        super.onCreate(savedInstanceState) 
        Log.d("BaseActivity", javaClass.simpleName) 
        ActivityCollector.addActivity(this) 
    } 
    override fun onDestroy() { 
        super.onDestroy() 
        ActivityCollector.removeActivity(this) 
    } 
} 
```

通过`ActivityCollector.finishAll()`退出程序。

### 启动 Activity 的最佳写法

被启动的 Activity 应当这样声明自己所需要的启动参数：

```kotlin
class Activity: BaseActivity() { 
    companion object { // 可以静态调用
        fun actionStart(context: Context, data1: String, data2: String) { 
            val intent = Intent(context, Activity::class.java).apply { 
                putExtra("param1", "data1") 
                putExtra("param2", "data2") 
            } 
            context.startActivity(intent) 
        } 
    } 
} 
```

通过`SecondActivity.actionStart(this, "data1", "data2")`启动。

# 
