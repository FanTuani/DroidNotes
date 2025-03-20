# Service

> Service 是 Android 中实现程序后台运行的解决方案，其运行不依赖任何用户界面。

## Android 多线程编程

使用`thread()`顶层函数：

```kotlin
Thread { 
    // 编写具体的逻辑 
}.start() 

thread { 
    // 编写具体的逻辑 
} 
```

## 在子线程中进行 UI 操作

Android 的 UI 是线程不安全的。更新应用程序里的 UI 元素，必须在主线程中进行，否则会出现异常。

```kotlin
val updateText = 1 
val handler = object : Handler(Looper.getMaininLooper()) {
    override fun handleMessage(msg: Message) {
        // 在这里可以进行UI操作 
        when (msg.what) {
            updateText -> textView.text = "Nice to meet you"
        }
    }
}
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    changeTextBtn.setOnClickListener {
        thread {
            val msg = Message()
            msg.what = updateText
            handler.sendMessage(msg) // 将Message对象发送出去 
        }
    }
} 
```

Android 中异步消息处理主要由 4 个部分组成：**Message**、**Handler**、**MessageQueue**和 **Looper**。

**Message：**

Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间 传递数据。上一小节中我们使用到了Message的what字段，除此之外还可以使用arg1和 arg2字段来携带一些整型数据，使用obj字段携带一个Object对象。 

**Handler：**

Handler顾名思义也就是处理者的意思，它主要是用于发送和处理消息的。发送消息一般 是使用Handler的sendMessage()方法、post()方法等，而发出的消息经过一系列地辗 转处理后，最终会传递到Handler的handleMessage()方法中。 

**MessageQueue：** 

MessageQueue是消息队列的意思，它主要用于存放所有通过Handler发送的消息。这部 分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个MessageQueue对 象。 

**Looper：**

Looper是每个线程中的MessageQueue的管家，调用Looper的loop()方法后，就会进入 一个无限循环当中，然后每当发现MessageQueue中存在一条消息时，就会将它取出，并 传递到Handler的handleMessage()方法中。每个线程中只会有一个Looper对象。

> 首先在主线程当中创建一个Handler对象，并重写handleMessage()方法。然后当子线程中需要进行UI操作时，就创建一个Message对象，并通过Handler将这条消息发送出去。之后这条消息会被添加到MessageQueue的队列中等待被处理，而Looper则会一直尝试从MessageQueue中取出待处理消息，最后分发回Handler的handleMessage()方法中。由于Handler的构造函数中我们传入了Looper.getMainLooper()，所以此时handleMessage()方法中的代码也会在主线程中运行，于是我们在这里就可以安心地进行UI操作了。

## AsyncTask


