## Lambda 编程

# Kotlin

## 逻辑控制

**单行函数简化**

```kotlin
fun largerNumber(num1: Int, num2: Int) = max(num1, num2) 
```

**if - else 返回值**

```kotlin
fun largerNumber(num1: Int, num2: Int) = if (num1 > num2) num1 else num2 
```

**when (switch)**

```kotlin
fun getScore(name: String) = when (name) { 
    // 匹配值 -> { 执行逻辑 } 
    "Tom" -> 86 
    "Jim" -> 77 
    "Jack" -> 95 
    "Lily" -> 100 
    else -> 0 
}
// 类型匹配
fun checkNumber(num: Number) { 
    when (num) { // is == instanceof
        is Int -> println("number is Int") 
        is Double -> println("number is Double") 
        else -> println("number not support") 
    } 
} 
// 无参 when
fun getScore(name: String) = when { 
    name.startsWith("Tom") -> 86 
    name == "Jim" -> 77 
    name == "Jack" -> 95 
    name == "Lily" -> 100 
    else -> 0 
} 
```

**for in**

```kotlin
for (i in 0..10) { // [0, 10] 闭区间
    println(i) 
}
val range = 0 until 10 // [0, 10)
for (i in 0 until 10 step 2) { // i += 2
    println(i) 
} 
for (i in 10 downTo 1) { // 倒序
    println(i) 
} 
```

## 面向对象

**继承**

所有非抽象类默认不可继承，使用`open`关键字声明可继承。

```kotlin
open class Person {}
class Student() : Person() {}
```

**构造函数**

分为**主构造函数**和**次构造函数**。

**主构造函数**：

主构造函数没有函数体，直接定义在类名后面，若要编写逻辑，应写在`init{}`中：

```kotlin
class Student(val sno: String, val grade: Int) : Person() {
    init {
        // do something..
    }
}
```

根据语言规定：**子类中的构造函数必须调用父类中的构造函数**，`Person()`处的括号实际上指明了`Student`类初始化时应调用父类的哪个构造函数。另外，在需要调用父类有参构造函数时，应当将对应字段添加至子类构造函数。`val`或`var`自动推导，无需声明：

```kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int) : 
    Person(name, age) { 
} 
```

**次构造函数**：

一个类只能有一个主构造函数，但是可以有多个次构造函数。次构造函数必须调用主构造函数，可以用于实例化一个类，并拥有函数体：

```kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int) : 
         Person(name, age) { 
    constructor(name: String, age: Int) : this("", 0, name, age) { 
    } 
    constructor() : this("", 0) { 
    } 
} 
```

**特殊情况：**

子类中只有次构造函数，没有主构造函数。在此情况下，子类不能通过主构造函数调用父类主构造函数，应在次构造函数中用`super()`显式调用父类主构造函数。

```kotlin
class Student : Person { 
    constructor(name: String, age: Int) : super(name, age) { 
    } 
} 
```

## 接口

```kotlin
interface Study { 
    fun readBooks() 
    fun doHomework() {..} // 可以存在默认实现
}
// 注意此处如何继承父类和实现接口
class Student(name: String, age: Int) : Person(name, age), Study { 
    override fun readBooks() {..}
    // override fun doHomework() {..} 若使用默认实现则可不实现
}

fun main() { 
    val student = Student("Jack", 19) 
    doStudy(student) 
} 
fun doStudy(study: Study) { // 面向接口编程 & 多态
    study.readBooks() 
    study.doHomework() 
} 
```

## 数据类与单例类

**数据类 data**

自动生成`equals() hashCode() toString()`等方法。

```kotlin
data class Cellphone(val brand: String, val price: Double) 
```

> 当一个类中没有任何代码时，可以将尾部的大括号省略。

**单例类 object**

调用方式类似静态方法，保证全局只会存在一个 Singleton 实例。

```kotlin
object Singleton { 
    fun singletonTest() { 
        println("singletonTest is called.") 
    } 
}
fun main() {
    Singleton.singletonTest() 
}
```

## Lambda 编程

**初始化 list 集合**

```kotlin
// 不可变
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
// 可变
val list = mutableListOf("Apple", "Banana", "Orange", "Pear", "Grape") 
list.add("Watermelon") 
```

**map 集合**

```kotlin
val map = HashMap<String, Int>() 
map["Apple"] = 1 
map["Banana"] = 2 
map["Orange"] = 3
// or
val map = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3) 
// 遍历
for ((fruit, number) in map) { 
    println("fruit is " + fruit + ", number is " + number) 
} 
```

**集合的函数式 API**

```kotlin
// 找到长度最长的
val maxLengthFruit = list.maxBy { it.length } 
```

**Lambda 表达式的语法结构定义**：

`{参数名1: 参数类型, 参数名2: 参数类型 -> 函数体} `

如果有参数传入到 Lambda 表达式，需要声明参数列表，参数列表的结尾使用一个`->`符号表示参数列表的结束以及函数体的开始。

函数体中可以编写代码，并且最后一行代码会作为 Lambda 表达式的返回值。

> ## 推导函数式 API
> 
> ```kotlin
> val list = listOf("Apple", "Banana", "Watermelon") 
> val lambda = { fruit: String -> fruit.length } 
> val maxLengthFruit = list.maxBy(lambda)
> ```
> 
> 可以直接将 Lambda 表达式传入 maxBy 函数
> 
> ```kotlin
> val maxLengthFruit = list.maxBy({ fruit: String -> fruit.length }) 
> ```
> 
> 若 Lambda 参数是函数的最后一个参数，可以将 Lambda 表达式移到函数括号外面
> 
> ```kotlin
> val maxLengthFruit = list.maxBy() { fruit: String -> fruit.length } 
> ```
> 
> 若 Lambda 参数是函数的唯一参数，可以将函数的括号省略
> 
> ```kotlin
> val maxLengthFruit = list.maxBy { fruit: String -> fruit.length } 
> ```
> 
> 借助类型推导
> 
> ```kotlin
> val maxLengthFruit = list.maxBy { fruit -> fruit.length } 
> ```
> 
> 若 Lambda 表达式的参数列表中只有一个参数，可以使用`it`关键字代替
> 
> ```kotlin
> val maxLengthFruit = list.maxBy { it.length } 
> ```

**Kotlin 使用 Java 单抽象方法接口**

Java 单抽象方法接口形如：

```java
public interface Runnable { 
    void run(); 
} 
```

则，形如以下的类：

```java
new Thread(new Runnable() { 
    @Override 
    public void run() { 
        System.out.println("Thread is running"); 
    } 
}).start(); 
```

在 Kotlin 中可简化为：

```kotlin
Thread { 
    println("Thread is running") 
}.start() 
```

## 空指针检查

`?.`**操作符**

```kotlin
if (a != null) a.run()
a?.run() // 等价
```

`?:`操作符

```kotlin
val c = if (a != null) a else b
val c = a ?: b // 等价
```

例如：

```kotlin
fun getTextLength(text: String?) = text?.length ?: 0 
```

`let`**函数**

```kotlin
fun doStudy(study: Study?) { 
    study?.let { // it == study，仅 study 非空时执行
        it.readBooks() 
        it.doHomework() 
    } 
} 
```

## 其它技巧

**字符串内嵌表达式**

若表达式中仅有一个变量，可以将两边大括号省略。

```kotlin
"hello, ${obj.name}. nice to meet you!" 
```

**乱序传参**

配合参数默认值使用，多数情况下可以代替次构造函数。

```kotlin
printParams(str = "world", num = 123) 
```


