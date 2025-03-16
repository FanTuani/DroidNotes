# Broadcast

## 分类

- 标准广播（normal broadcasts）：异步执行，所有 BroadcastReceiver 同时收到。

- 有序广播（ordered broadcasts）：同步执行，有序收到，可以截断。

## 动态注册监听时间变化

```kotlin
class MainActivity : AppCompatActivity() {
    lateinit var timeChangeReceiver: TimeChangeReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val intentFilter = IntentFilter()
        intentFilter.addAction("android.intent.action.TIME_TICK")
        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver, intentFilter)
    }

    inner class TimeChangeReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            Toast.makeText(context, "time", Toast.LENGTH_SHORT).show()
        }
    }
}
```

## 接收静态广播

目前只有少数特殊的系统广播允许静态接收。

静态广播接收器类可以在 AndroidStudio 中直接创建，Exported 属性表示是否允许接收本程序以外的广播。

在 AndroidManifest.xml 中：

```xml
<receiver
    android:name=".BootCompleteReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>ceiver>
```

> BOOT_COMPLETE 已不能这样使用。

## 发送标准广播

```kotlin
val intent = Intent("top.ricequakes.learnbroadcast.MY_BROADCAST")
intent.setPackage(packageName)
sendBroadcast(intent)
```

通过`setPackage()`指定发送给哪个应用程序，使自定义的隐式广播变为显式广播，可以被静态注册的 BroadcastReceiver 接收。

此处的 intent 同样可以携带数据。

## 发送有序广播

```kotlin
..
sendOrderedBroadcast(intent, null) 
```

在 AndroidManifest.xml 中可以设置广播接收的优先级：

```xml
<intent-filter android:priority="100"> 
    <action android:name="com.example.broadcasttest.MY_BROADCAST"/> 
</intent-filter> 
```

在相应 BroadcastReceiver 类中可以通过`abortBroadcast()`截断广播。

## 强制下线功能

```kotlin
open class BaseActivity : AppCompatActivity() {
    lateinit var receiver: ForceOfflineReceiver
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ActivityCollector.addActivity(this)
    }
    override fun onResume() {
        super.onResume()
        val intentFilter = IntentFilter()
        intentFilter.addAction("top.ricequakes.FORCE_OFFLINE")
        receiver = ForceOfflineReceiver()
        registerReceiver(receiver, intentFilter, Context.RECEIVER_NOT_EXPORTED)
    }
    override fun onPause() {
        super.onPause()
        unregisterReceiver(receiver)
    }
    override fun onDestroy() {
        super.onDestroy()
        ActivityCollector.removeActivity(this)
    }
}
```

BaseActivity 作为所有 Activity 的父类，可以用于管理 Activity 的行为，比如此处可以为每个 Activity 注册 BroadcastReceiver。

将接收器在`onResume()`和`onPause()`注册注销，可以实现**仅栈顶**的 Activity 在监听广播。

使用 ActivityCollector 管理所有 Activity 的销毁。
