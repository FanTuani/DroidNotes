# ContentProvider

> 使用 ContentProvider 是 Android 实现跨程序共享数据的标准方式。

## 前置-运行时权限

在 AndroidManifest.xml 中声明权限：

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

运行时请求权限：

```kotlin
private fun call() {
    try {
        val intent = Intent(Intent.ACTION_CALL)
        intent.data = "tel:10086".toUri()
        startActivity(intent)
    } catch (e: SecurityException) {
        e.printStackTrace()
    }
}
..
binding.button.setOnClickListener {
    if (ContextCompat.checkSelfPermission(this,
            Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this,
            arrayOf(Manifest.permission.CALL_PHONE), 1)
    } else {
        call()
    }
}
```

## 访问其他程序中的数据

借助 ContentResolver 类可以对数据进行增删查改。与 SQLite 不同的是，其 crud 函数的参数为**内容 URI**。其主要由 **authority** 和 **path** 组成。前者用于区分应用程序，后者用于区分表名。格式如下：

```uri
content://com.example.app.provider/table1 
content://com.example.app.provider/table2 
```

**查询：**

```kotlin
val cursor = contentResolver.query( 
    uri, 
    projection, 
    selection, 
    selectionArgs, 
    sortOrder) 
while (cursor.moveToNext()) { 
    val column1 = cursor.getString(cursor.getColumnIndex("column1")) 
    val column2 = cursor.getInt(cursor.getColumnIndex("column2")) 
} 
cursor.close() 
```

其余操作与 SQLite 类似。

## 创建自己的 ContentProvider

```kotlin
class MyProvider : ContentProvider() { 
    override fun onCreate(): Boolean { 
        return false 
    } 
    override fun query(uri: Uri, projection: Array<String>?, selection: String?, 
            selectionArgs: Array<String>?, sortOrder: String?): Cursor? { 
        return null 
    } 
    override fun insert(uri: Uri, values: ContentValues?): Uri? { 
        return null 
    }  
    override fun update(uri: Uri, values: ContentValues?, selection: String?, 
            selectionArgs: Array<String>?): Int { 
        return 0 
    } 
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int { 
        return 0 
    } 
    override fun getType(uri: Uri): String? { 
        return null 
    } 
}
```

Todo
