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

**存储数据**：

1. 调用 SharedPreferences 对象的`edit()`方法获取 SharedPreferences.Editor 对象。

2. 向 SharedPreferences.Editor 对象中添加数据，`putXxx()`。

3. 调用`apply()`方法将添加的数据提交，从而完成数据存储操作。

```kotlin
getSharedPreferences("data", MODE_PRIVATE).edit {
    putString("name", "Tom")
    putInt("age", 28)
    putBoolean("married", false)
}
```

**读取数据**：

```kotlin
val prefs = getSharedPreferences("data", Context.MODE_PRIVATE) 
val name = prefs.getString("name", "") 
```

## SQLite

### 基本使用

SQLiteOpenHelper 是一个抽象类，要使用必须创建帮助类并继承，然后重写`onCreate()`和`onUpgrade()`。

`getReadableDatabase()`和`getWritableDatabase()`可用于创建或打开数据库，当磁盘空间已满数据库不可写入的时候，后者将出现异常。

数据库文件会存放在`/data/data//databases/`目录下。

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) : 
        SQLiteOpenHelper(context, name, null, version) { 
    private val createBook = "create table Book (" + 
            " id integer primary key autoincrement," + 
            "author text," + 
            "price real," + 
            "pages integer," + 
            "name text)" 
    override fun onCreate(db: SQLiteDatabase) { 
        db.execSQL(createBook) 
    } 
    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) { 
    } 
} 
```

调用`DBHelper(this, "BookStore.db", 1).writableDatabase`，若数据库不存在，会调用`onCreate()`。若数据库已存在，则不会。

通过在创建 DBHelper 时更新版本号来调用`onUpgrade()`。

### 操作数据

**添加：**

```kotlin
val db = dbHelper.writableDatabase
val values = ContentValues().apply {
    put("name", "The Da Vinci Code")
    put("author", "Dan Brown")
    put("pages", 454)
    put("price", 16.96)
}
db.insert("Book", null, values) 
```

**更新：**

```kotlin
val db = dbHelper.writableDatabase
val values = ContentValues()
values.put("price", 10.99)
db.update("Book", values, "name = ?", arrayOf("The Da Vinci Code")) 
```

**删除：**

```kotlin
val db = dbHelper.writableDatabase 
db.delete("Book", "pages > ?", arrayOf("500")) 
```

**查询：**

```kotlin
val db = dbHelper.writableDatabase
// 查询Book表中所有的数据 
val cursor = db.query("Book", null, null, null, null, null, null)
if (cursor.moveToFirst()) {
    do {
        // 遍历Cursor对象，取出数据并打印 
        val name = cursor.getString(cursor.getColumnIndex("name"))
        val author = cursor.getString(cursor.getColumnIndex("author"))
        ..
    } while (cursor.moveToNext())
}
cursor.close() 
```

**直接使用 SQL：**

```kotlin
db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)", 
    arrayOf("The Da Vinci Code", "Dan Brown", "454", "16.96") 
) 
```

其中`?`为占位符，会被最后一个数组型参数替换。

### 最佳实践-事务

事务保证一系列操作要么全部完成，要么一个都不完成。

```kotlin
val db = dbHelper.writableDatabase 
db.beginTransaction()
try {
    db.xxx() 
    ..
} finally {
    db.endTransaction()
}
```

### 最佳实践-升级数据库

维护每一个版本的数据库变动：

```kotlin
override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
    if (oldVersion <= 1) {
        db.execSQL(createCategory)
    }
    if (oldVersion <= 2) {
        db.execSQL("alter table Book add column category_id integer")
    }
} 
```

### Room
