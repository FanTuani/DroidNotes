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

## 标准函数和静态方法

**let**：配合`?.`操作符辅助判空。

```kotlin
fun doStudy(study: Study?) { 
    study?.let { // it == study，仅 study 非空时执行
        it.readBooks() 
        it.doHomework() 
    } 
} 
```

**with**：简化连续调用同一个对象的多个方法。

```kotlin
val result = with(obj) { 
    // 这里是obj的上下文 
    "value" // with函数的返回值 
} 
val result = with(StringBuilder()) { 
    append("Start eating fruits.\n") // 代替 builder.append().append()..
    for (fruit in list) { 
        append(fruit).append("\n") 
    } 
    append("Ate all fruits.") 
    toString() 
}
```

**run**：只接收一个 Lambda 参数，要在某个对象的基础上调用的 with：

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape") 
val result = StringBuilder().run { 
    append("Start eating fruits.\n") 
    for (fruit in list) {
        append(fruit).append("\n") 
    } 
    append("Ate all fruits.") 
    toString() 
} 
```

**apply**：不能指定返回值的 run：

```kotlin
val result = StringBuilder().apply { 
    append("Start eating fruits.\n") 
    for (fruit in list) { 
        append(fruit).append("\n") 
    } 
    append("Ate all fruits.") 
} 
println(result.toString()) 
```

### 静态方法

纯静态类可以用单例类代替，若希望普通类中存在静态方法，可以使用`companion object`，其原理不同于真正的静态方法。若要使用真正的静态方法，使用顶层方法，可以在任意位置直接调用。

## 延迟初始化

`::adapter.isInitialized`可用于判断 adapter 变量是否已经初始化。

## 密封类

通过`sealed class`声明，此后使用`when()`的时候，必须判断所有子类才能通过编译：

```kotlin
when (holder) { 
    is LeftViewHolder -> holder.leftMsg.text = msg.content 
    is RightViewHolder -> holder.rightMsg.text = msg.content 
} 
```

## 扩展函数

kotlin 允许将对象的函数进行扩展。

## 重载运算符

kotlin 允许重载对象之间的运算符，也允许重载不同对象之间的运算符。

```kotlin
class Money(val value: Int) { 
    operator fun plus(money: Money): Money { 
        val sum = value + money.value 
        return Money(sum) 
    } 
    operator fun plus(newValue: Int): Money { 
        val sum = value + newValue 
        return Money(sum) 
    } 
} 
```

结合以上两种特性，可以实现：

```kotlin
operator fun String.times(n: Int): String { 
    val builder = StringBuilder() 
    repeat(n) { 
        builder.append(this) 
    } 
    return builder.toString() 
} 
```

# Kotlin 高级用法

## 高阶函数

如果一个函数**接收另一个函数作为参数**，或者**返回值的类型是另一个函数**，那么该函数就称为**高阶函数**。

基本语法：

```kotlin
(String, Int) -> Unit 
// 参数 -> 返回值（Unit 即 Void）
fun foo(func: (String, Int) -> Unit) {
    func("hello", 123)
}
fun main() {
    foo(::bar)
}
```

使用 Lambda 表达调用：

```kotlin
fun calc(a: Int, b: Int, func: (Int, Int) -> Int): Int {
    return func(a, b)
}
fun main() {
    calc(2, 3) { a, b ->
        a + b
    }
}
```

在函数类型的前面加上 ClassName. 表示定义在哪个类中，传入的 Lambda 表达式自动拥有其上下文：

```kotlin
fun StringBuilder.build(block: StringBuilder.() -> Unit): StringBuilder {
    block()
    return this
}
fun main() {
    val result = StringBuilder().build {
        append("something")
        append("something")
        append("something")
    }
}
```

## 内联函数

TODO
