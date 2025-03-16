# Storage

## 文件存储

所有的文件都默认存储到`/data/data/<package name>/files/`目录下。

- **MODE_PRIVATE**：覆盖

- **MODE_APPEND**：追加

```kotlin
fun save(inputText: String) { 
    val output = openFileOutput("data", Context.MODE_PRIVATE) 
    val writer = BufferedWriter(OutputStreamWriter(output)) 
    writer.use { 
        it.write(inputText) 
    } 
} 
```

`use`函数保证在 Lambda 表达式中的代码全部执行完之后自动将外层的流关闭。

```kotlin
fun load(): String {
    val content = StringBuilder()
    val input = openFileInput("data")
    val reader = BufferedReader(InputStreamReader(input))
    reader.use {
        reader.forEachLine {
            content.append(it)
        }
    }
    return content.toString()
}
```

## SharedPreferences

SharedPreferences 文件存放在`/data/data/<packagename>/shared_prefs/`目录下。

要想使用 SharedPreferences 存储数据，首先需要获取 SharedPreferences 对象。

- Context 类中的`getSharedPreferences()`方法：传入文件名和操作模式（唯一）

- Activity 类中的`getPreferences()`方法：传入操作模式，文件名默认类名

存储数据：

1. 调用 SharedPreferences 对象的`edit()`方法获取 SharedPreferences.Editor 对象。

2. 向 SharedPreferences.Editor 对象中添加数据，`putXxx()`。

3. 调用`apply()`方法将添加的数据提交，从而完成数据存储操作。
