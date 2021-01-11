---
layout: post
title:  "Kotlin默认参数以及@JvmOverloads"
categories: @JvmOverloads
---


kotlin默认参数方法以及@JvmOverloads

我们先来看看java中的构造方法

我们都知道`java`方法中的参数是必须要传递的

示例代码

```java
 public class TestJava {
    public static void main(String[] args) {
        new Person("");
    }

    static class Person{
        Person(String msg){
            System.out.println("调用了无参的构造函数" + msg);
        }
    }
}
```

如上代码构造方法需要传递一个参数，那么我们在实例化的时候需要传入

但是kotlin中的参数是可以设置默认值的，设置默认值的参数，调用时如果没有传递就使用设定的默认值

我们首先来看只有一个参数，并且设置了默认的情况


```kotlin
class TestParameter(private val str1: String? = "defaultMsg1"){
    fun printMsg(){
        println("print-->str1---$str1")
    }
}

```

kotlin调用

```kotlin
   TestParameter().printMsg()
```

java调用
```java
   new TestParameter().printMsg();
```

我们通过Kotlin Bttecode看看为什么设置默认参数后我们可以不需要传参

```java
public final class TestParameter {
   private final String str1;

   public final void printMsg() {
      String var1 = "print-->str1---" + this.str1;
      boolean var2 = false;
      System.out.println(var1);
   }

   public TestParameter(@Nullable String str1) {
      this.str1 = str1;
   }

   public TestParameter() {
      this((String)null, 1, (DefaultConstructorMarker)null);
   }
}
```

看反编译代码后我们不难发现

我们的构造方法被创建了两个，一个是不带参数的，一个是带参数的

我们通过`kotlin`或`java`调用时，如没传递参数，其实调用的是`TestParameter()`无参构造


接着看两个或两个以上的参数，并且有一个被设置默认参数的情况

```kotlin
class TestOneNULLParameter(private val str1: String?, private val str2: String? = "defaultMsg2"){
    fun printMsg(){
        println("print-->str1---$str1---str2---$str2")
    }
}
```

kotlin调用

```kotlin
 	TestOneNULLParameter("msg1").printMsg()
```

java调用

```java
	new TestOneNULLParameter("msg1","").printMsg();
```
可以看出，当我们设置了`str=“msg2”`时，kotlin调用时我们只传递了`msg1`参数，第二个参数没有传递
java调用时，我们必须要传递两个参数，如果第二个参数没有传递，程序会立马报错，这是为什么呢？我们还是看反编译代码找原因

```kotlin
public final class TestOneNULLParameter {
   private final String str1;
   private final String str2;

   public final void printMsg() {
      String var1 = "print-->str1---" + this.str1 + "---str2---" + this.str2;
      boolean var2 = false;
      System.out.println(var1);
   }

   public TestOneNULLParameter(@Nullable String str1, @Nullable String str2) {
      this.str1 = str1;
      this.str2 = str2;
   }
}
```

如上代码我们看出，此时我们反编译的代码构造方法只有一个，并且参数是两个`str1`和‘str2’，那也要就说当我们实例化`TestOneNULLParameter`时，
我们必须要传递两个参数，`Java`调用的方式我们理解，`Kotlin`为什么就只传一个参数也是可以的？我们继续反编译`kotlin`调用处代码

```java
	(new TestOneNULLParameter("msg1", (String)null, 2, (DefaultConstructorMarker)null)).printMsg();
```
根据上述反编译调用代码我们知道了原因，原来kotlin在调用时，其实也是传递两个参数的，只不过`kotlin`默认给我们赋值了一个参数为‘null’

那另外一个问题来了，我们能不能也让用`Java`调用时默认赋值的参数可以不传递吗？，答案也是可以的，这就得用kotlin的关键字 `@JvmOverloads`

### JvmOverloads

> @JvmOverloads注解把多个构造函数合并成一个构造函数
我们用代码解释一下这句话，我们修改一下`TestOneNULLParameter`代码

```kotlin
	class TestOneNULLJVMOverloadsParameter @JvmOverloads constructor(private val str1: String?, private val str2: String? = "defaultMsg2"){
    fun printMsg(){
        println("print-->str1---$str1---str2---$str2")
    }
}
```

kotlin调用

```kotlin
	 TestOneNULLJVMOverloadsParameter( "msg1").printMsg()
```
java调用

```java
	new TestOneNULLJVMOverloadsParameter("msg1").printMsg();
```

这个时候我们发现`kotlin`和‘java’都可以只传递一个参数了，带有默认参数都被忽略传递了，原因看反编译源码

```java
public final class TestOneNULLJVMOverloadsParameter {
   private final String str1;
   private final String str2;

   public final void printMsg() {
      String var1 = "print-->str1---" + this.str1 + "---str2---" + this.str2;
      boolean var2 = false;
      System.out.println(var1);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1, @Nullable String str2) {
      this.str1 = str1;
      this.str2 = str2;
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1) {
      this(str1, (String)null, 2, (DefaultConstructorMarker)null);
   }
}

```

如上源码可以清晰的看到，反编译后的代码为我们生成了两个构造，一个参数的就是我们想要的结果，我们现在就可以理解那句话了
`@JvmOverloads注解把多个构造函数合并成一个构造函数`
还是不太明白，我们让`TestOneNULLJVMOverloadsParameter`构造方法传递n个参数（都带默认参数），看看会不会生成对应的n+1个对应的构造方法

```kotlin
	class TestOneNULLJVMOverloadsParameter @JvmOverloads constructor(private val str1: String?,private val str2: String?,private val str3: String?, private val str4: String? = "defaultMsg2"){
    fun printMsg(){
        println("print-->str1---$str1---str2---$str2")
    }
}

```

反编译后代码

```java
	public final class TestOneNULLJVMOverloadsParameter {
   private final String str1;
   private final String str2;
   private final String str3;
   private final String str4;
   private final String str5;

   public final void printMsg() {
      String var1 = "print-->str1---" + this.str1 + "---str2---" + this.str2;
      boolean var2 = false;
      System.out.println(var1);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1, @Nullable String str2, @Nullable String str3, @Nullable String str4, @Nullable String str5) {
      this.str1 = str1;
      this.str2 = str2;
      this.str3 = str3;
      this.str4 = str4;
      this.str5 = str5;
   }
   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1, @Nullable String str2, @Nullable String str3, @Nullable String str4) {
      this(str1, str2, str3, str4, (String)null, 16, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1, @Nullable String str2, @Nullable String str3) {
      this(str1, str2, str3, (String)null, (String)null, 24, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1, @Nullable String str2) {
      this(str1, str2, (String)null, (String)null, (String)null, 28, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter(@Nullable String str1) {
      this(str1, (String)null, (String)null, (String)null, (String)null, 30, (DefaultConstructorMarker)null);
   }

   @JvmOverloads
   public TestOneNULLJVMOverloadsParameter() {
      this((String)null, (String)null, (String)null, (String)null, (String)null, 31, (DefaultConstructorMarker)null);
   }
}
```
如上代码验证了我们的想法

------------------------------------------------------------------------------------------------------------------------------------

我们上边的示例代码给构造方法添加` @JvmOverloads `
如果要给函数添加，其实很简单，我们这里都做一下记录

### 函数方法重载

```kotlin
  @JvmOverloads
  fun printMsg(){
      println("print-->str1---$str1---str2---$str2")
  }
```

### 构造方法重载

```kotlin
  class ClassName @JvmOverloads constructor(private val str1: String?, private val str2: String?){
    fun printMsg(){
        println("print-->str1---$str1---str2---$str2")
    }
}
```

总结一下：

@JvmOverloads是兼容Java调用Kotlin有默认参数方法的，相对java还是挺方便的