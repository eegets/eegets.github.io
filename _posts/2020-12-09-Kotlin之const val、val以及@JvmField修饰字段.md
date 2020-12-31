我们先看看kotlin官方文档的解释

* val 声明一个`只读属性(也称作只读变量)`或`局部变量`

	> * 如果被声明的变量是方法内部的局部变量，可以称为`常量`
	> * 如果被声明变量是顶级变量，称为`只读变量`

* const 将属性标记为编译期常量`(编译期常量是在程序编译期会替换成字面量)`
	
	> * 位于顶层或者是 `object` 声明 或 `companion object` 的一个成员
	> * 以 String 或原生类型值初始化
        > * 没有自定义 `getter`


我们通过代码一一理解一下

* 局部变量

```kotlin
fun main() {
	val params = 0
	params
	println("params: $params")
}
```
局部变量中，我们无法实现`params`的 `get()`方法， `const`要求也不满足，所以无法被`const`标记，也无法重新分配值，此时我们可以把`val params = 0`看做是常量

那是不是所有被`val`修饰的就是常量呢？这就引出我们的重点`只读变量了`，我们在一开始说过`被val声明变量是顶级变量，称为只读变量`


* 只读变量

```kotlin
val params
    get() = Math.random()

fun main() {
	println("params - $params")
}
```
如上代码，我们每运行一次，得到的结果值都不一样，原因就在这个get，我们可以通过get()赋值，此时他就不是一个常量，而是一个只能读取不能写入的变量了

那kotlin的常量如何定义呢？

kotlin中声明常量有两种方式: `const val `以及 `@JvmFiled val` 

* const
```kotlin
const val params = 0
fun main() {
	println("params - $params")
}
```
我们看看生成的java代码如下
```java
public final class AAAAAKt {
   public static final int params = 0;

   public static final void main() {
      String var0 = "params - 0"; //变动１处
      boolean var1 = false;
      System.out.println(var0);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```
const修饰的常量必须是top-level(`top-level是属于文件内部，不在类中定义`)以及object 声明 或 companion object 的一个成员

* @JvmField
```kotlin
@JvmField val params = 0
fun main() {
    println("params - $params")
}
```
同样看看生成的Java代码如下
```java
public final class AAAAAKt {
   @JvmField
   public static final int params = 0;

   public static final void main() {
      String var0 = "params - " + params;　//变动１处
      boolean var1 = false;
      System.out.println(var0);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```
从上述两处Java源码可以看出，`const`和`@JvmField`还是有区别的，具体如下改动
```kotlin
// const val
public static final int params = 0;

public static final void main() {
      String var0 = "params - 0";//［１］
      boolean var1 = false;
      System.out.println(var0);
 }

// @JvmField val
@JvmField
public static final int params = 0;

public static final void main() {
      String var0 = "params - " + params;//［２］
      boolean var1 = false;
      System.out.println(var0);
}

```
上述两处代码比较可以看出，主要区别是被`const`标示的常量值`［１］`实现了内联，而`@JvmField`注释的常量`［２］`则没有

## 总结：
* 1: val修饰局部变量可以被当做是`常量`
* 2: val修饰`静态类或伴生对象的字段`时只能称作`只读变量`
* 3: const val 修饰静态类或伴生对象的字段时时被称为`常量`
* 4: ＠JvmField修饰静态类或伴生对象的字段时被称为`常量`
* 5: const val修饰的字段实现了内联（也就是copy代码块带相应位置）而@JvmField修饰的没有


### refrence: 

https://www.kotlincn.net/docs/reference/properties.html


[Where Should I Keep My Constants in Kotlin?](https://blog.egorand.me/where-do-i-put-my-constants-in-kotlin/)