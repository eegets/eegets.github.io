---
layout: post
title:  "深入理解Java|Kotlin泛型"
categories: Kotlin Java 泛型
---

Java泛型是JDK1.5引入的一个新特性，是一种参数化类型。参数化类型就是在不创建新类型的情况下，通过泛型指定的泛泛类型控制形参限制的类型。允许在编译期检测非法类型。

# 泛型特点
* 类型安全。使用泛型定义的参数进行，在编译期可以对一个类型进行验证，从而更快的暴露问题
* 消除强制类型转换。
* 避免了不必要的[装箱、拆箱](https://www.cnblogs.com/dolphin0520/p/3780005.html)操作，提高程序性能
* 提高代码的重用性

# 命名类型参数
* E - 元素，主要由Java集合(Collections)框架使用。
* K - 键，主要用于表示映射中的键的参数类型。
* V - 值，主要用于表示映射中的值的参数类型。
* N - 数字，主要用于表示数字。
* T - 类型，主要用于表示第一类通用型参数。
* S - 类型，主要用于表示第二类通用类型参数。
* U - 类型，主要用于表示第三类通用类型参数。
* V - 类型，主要用于表示第四个通用类型参数。


# 泛型定义

## 泛型类

泛型类的声明和普通类声明类似，除了在类名后添加类型参数声明。

定义
```java
修饰符 class 类名<声明自定义泛型> {
    ...
}
```

实例

```java
public static void main(String[] args) {
    Container<String ,String> c1 = new Container<>("name", "kevin");
    Container<String, Integer> c2 = new Container<>("age", 29);
    Container<Double, Integer> c3 = new Container<>(1.0, 29);
}
public static class Container<K, V> {
    K key;
    V value;
    Container(K k, V v) {
        key = k;
        value = v;
    }
}
```

## 泛型接口
定义
```java
修饰符 interface 接口名<声明自定义泛型>{

}
```
实例
```java
public interface Generator<T> {
    T init();
}

public class GeneratorClass1 implements Generator<String> {
    @Override
    public String init() {
        return "test1";
    }
}

public class GeneratorClass2 implements Generator<Integer> {
    @Override
    public Integer init() {
        return 0;
    }
}
```

## 泛型方法
定义
```java
修饰符 返回值类型 接口名<声明自定义泛型>{

}
```
实例
```java
public interface Generator<T> {
    T init();
}

public class GeneratorClass1 implements Generator<String> {
    @Override
    public String init() {
        return "test1";
    }
}

public class GeneratorClass2 implements Generator<Integer> {
    @Override
    public Integer init() {
        return 0;
    }
}
```


# 通配符

## 通配符的产生
任何使用父类的地方可以被它的子类替换，我们在使用类和对象时经常会接触到里式替换原则，其实在数组中一样也符合这种原则
如下：
```java
  class Fruit {}
    class Apple extends Fruit {}
    class Jonathan extends Apple {}
    class Orange extends Fruit {}

    public class CovariantArrays {
        public  void main(String[] args) {
            Fruit[] fruit = new Apple[10]; // OK
            List<Fruit> fruits=new ArrayList<Apple>();//error
            fruit[0] = new Apple(); // OK
            fruit[1] = new Jonathan(); // OK
            // Runtime type is Apple[], not Fruit[] or Orange[]:
            try {
                // Compiler allows you to add Fruit:
                fruit[0] = new Fruit(); // ArrayStoreException
            } catch(Exception e) { System.out.println(e); }
            try {
                // Compiler allows you to add Oranges:
                fruit[0] = new Orange(); // ArrayStoreException
            } catch(Exception e) { System.out.println(e); }
        }
    }
```
数组中的这种向上转变称为数组协变，而泛型是不支持的，如下代码
```java
 List<Fruit> fruits=new ArrayList<Apple>();//error
```
如上代码会产生编译时错误，之所以这么设计是因为数组支持运行时检查而集合不支持运行时检查。

Java的泛型的这种特性对于有需要向上转型的需求时就无能为力，所以 Java 为了满足这种需求设计出了通配符.

## 上边界限定通配符[Java]/协变[Kotln]
Java

Java语言利用 `<? extends T>` 形式的通配符可以实现泛型的向上转型：
```java
 static void ccc() {
    Apple apple = new Apple();
    apple.name = "Apple";
    ArrayList<? extends Fruit> fruits = new ArrayList<>();
    fruits.add(apple); //Error

    for (int i = 0; i < fruits.size(); i++) {
        Fruit fruit = (Fruit) fruits.get(i);
        System.out.println("println---" + fruit.name);
    }
}
```

Kotlin

Kotlin语言利用`<out T>`形式的通配符实现泛型的向上转型：

```kotlin
 fun ccc() {
    val apple = Apple()
    apple.name = "Apple"
    val fruits: ArrayList<out Fruit> = ArrayList()
    fruits.add(apple) //Error
    for (i in fruits.indices) {
        println("println---" + fruits[i].name)
    }
}
```
使用上通配符后编译器为了保证运行时的安全，`会限定对其写的操作，开放读的操作`，因为编译器只能保证 fruits 集合中存在的是 Fruits 及它的子类，并不知道具体的类型，所以上述代码`fruits.add(apple)`会报错


## 下边界限定通配符[Java]/逆变[Kotln]
Java

Java语言利用 `<? super T>` 形式的通配符可以实现泛型的向上转型：
```java
static void ccc() {
    Apple apple = new Apple();
    apple.name = "Apple";
    ArrayList<? super Apple> fruits = new ArrayList<>();
    fruits.add(apple);//OK
    fruits.add(new Fruits()); //Error
    for (int i = 0; i < fruits.size(); i++) {
        Apple fruit = fruits.get(i);//Error
        System.out.println("println---" + fruit.name);
    }
}
```
Kotlin

Kotlin语言利用`<in T>`形式的通配符实现泛型的向上转型：

```kotlin
fun ccc() {
    val apple = Apple()
    apple.name = "Apple"
    val fruits: ArrayList<in Apple> = ArrayList()
    fruits.add(apple) //OK
    fruits.add(new Fruits()) //Error
    for (i in fruits.indices) {
        println("println---" + fruits[i].name) //error
    }
}
```
 与上边界通配符相反，下边界通配符通常限定读的操作，开放写的操作，对于如上代码，它标示某种类型的List，这个类型是Apple的基础类型。也就是说，我们实际上并不知道类型是什么，但是这个类型肯定是Apple的父类型。因此，我们知道向这个List添加一个Apple对象或者其子类型对象是安全的，这些对象都可以向上转型为Apple。但是我们不知道加入Fruit对象是否安全，

## 无边界通配符[Java]/星投影[Kotln]

还有一种通配符是无边界通配符，它的使用形式是一个单独的问号：List<?>，也就是没有任何限定

Java

```java
<?> 
 ```

kotlin

```kotlin
<*>
```
无边界通配符或星投影是没有任何限定的，正是由于其没任何限定，所以我们并不能确定参数是哪种类型，此时我们也是不可以往其中添加对象的。

`MutableList<?>`和`MutableList`有什么区别呢？

如下代码

Java

```java
  List<?> list1 = new ArrayList<>();
  aaa.add(""); //Error

  List list2 = new ArrayList();
  aaa1.add(""); //Ok
```
Kotlin
```kotlin

  val list1: MutableList<*> = mutableListOf<Any>()
  fruits.add(Fruit()) //Error

  val list2: MutableList<Any> = mutableListOf<Any>()
  fruits.add(Fruit()) //OK

```

> MutableList<*> list 表示 list 是持有某种特定类型的 MutableList，但是不知道具体是哪种类型。那么我们可以向其中添加对象吗？当然不可以，因为并不知道实际是哪种类型，所以不能添加任何类型，这是不安全的。而 MutableList<Any> list ，也就是没有传入泛型参数，表示这个 list 持有的元素的类型是 Any也就是Object，因此可以添加任何类型的对象，只不过编译器会有警告信息。

# 泛型参数约束

## 单个泛型参数约束[Java]/[Kotln]

单个泛型约束还是很简单的，Java中我们使用`extends`进行约束，Kotlin中使用：

我们想实现两个泛型参数的比较，使用Comparable进行比较，代码如下：

Java
```Java
public <T extends Comparable<T>> T maxOf(T params1, T params2) {
    if (params1.compareTo(params2) > 0) {
        return params1;
    } else {
        return params2;
    }
}
```

Kotlin
```Kotlin
fun <T: Comparable<T>> maxOf(params1: T, params2: T): T {
    return if (params1 > params2) {
        params1
    } else{
        params2
    }
}
```

单个约束我们在之前已经了解情况了，那如果我们想实现多个约束该怎么办呢？

## 多个泛型参数约束[Java]/[Kotlin]

Java中实现多个泛型参数使用`& Supplier`，而Kotlin中使用`where`

还是如上代码，我们改造一下：

Java

```java
<T extends Comparable<T> & Supplier<R>, R extends String> R maxOf(T params1, T params2) {
    if (params1.compareTo(params2) > 0) {
        return params1.get();
    } else {
        return params2.get();
    }
}
```

Kotlin

```kotlin
fun <T, R> maxOf(params1: T, params2: T): R where T: Comparable<T>, T:() ->R {
    if (params1 > params2) {
        return params1.invoke()
    } else {
        return params2.invoke()
    }
}
```

# 类型擦除

我们都知道，Java的泛型是伪泛型，这是因为Java在编译期间，所有的泛型信息都会被擦掉，正确理解泛型概念的首要前提是理解类型擦除。Java的泛型基本上都是在编译器这个层次上实现的，在生成的字节码中是不包含泛型中的类型信息的，使用泛型的时候加上类型参数，在编译器编译的时候会去掉，这个过程成为类型擦除。

例如定义List<Object>和List<String>等类型，在编译后都会变成List，JVM看到的只是List，而由泛型附加的类型信息对JVM是看不到的

* 1：原始类型相等
```java
public class Test {
    public static void main(String[] args) {
        ArrayList<String> list1 = new ArrayList<String>();
        list1.add("abc");

        ArrayList<Integer> list2 = new ArrayList<Integer>();
        list2.add(123);
        System.out.println(list1.getClass() == list2.getClass());
    }
}
```
在这个例子中，我们定义了两个ArrayList数组，不过一个是ArrayList<String>泛型类型的，只能存储字符串；一个是ArrayList<Integer>泛型类型的，只能存储整数，最后，我们通过list1对象和list2对象的getClass()方法获取他们的类的信息，最后发现结果为true。说明泛型类型String和Integer都被擦除掉了，只剩下原始类型。

* 2：通过反射添加其它类型元素

```java
public class Test {
    public static void main(String[] args) throws Exception {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(1);  //这样调用 add 方法只能存储整形，因为泛型类型的实例为 Integer
        list.getClass().getMethod("add", Object.class).invoke(list, "asd");
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}
```

在程序中定义了一个ArrayList泛型类型实例化为Integer对象，如果直接调用add()方法，那么只能存储整数数据，不过当我们利用反射调用add()方法的时候，却可以存储字符串，这说明了Integer泛型实例在编译之后被擦除掉了，只保留了原始类型。

* 3：类型擦除后保留的原始类型

原始类型 就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型，无论何时定义一个泛型，相应的原始类型都会被自动提供，类型变量擦除，并使用其限定类型（无限定的变量用Object）替换。

```java
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T  value) {  duyou
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}
```
因为在Pair<T>中，T 是一个无限定的类型变量，所以用Object替换，其结果就是一个普通的类，如同泛型加入Java语言之前的已经实现的样子。在程序中可以包含不同类型的Pair，如Pair<String>或Pair<Integer>，但是擦除类型后他们的就成为原始的Pair类型了，原始类型都是Object。

[关于类型擦除的问题，查看大神文章即可](https://www.cnblogs.com/wuqinglong/p/9456193.html)

# 类型擦除引起的问题和解决办法
因为种种原因[部分原因是因为java要兼容所有用户，在Jdk1.5之前是没有泛型的，但是这个量级又比较庞大，所以Sun公司不得已才使用伪泛型]，Java不能实现真正的泛型，只能使用类型擦除的伪泛型，但是这也引发了一些新的问题。

**1：先检查、再编译，以及检查编译的对象和引用传递问题**
先来看一段代码
```java
ArrayList<String> list1 = new ArrayList<>();
list.add(1)  //编译错误
list.add("1") //编译通过 
```
以上代码，我们定义了一个可读可写的list，在使用add传入一个Integer对象时，程序立马报错。这就说明程序在编译之前先进行类型检查。

那么这个类型检查到底是针对谁的呢？再看代码

```java
ArrayList<String> list1 = new ArrayList<>();
list1.add(1); //编译错误
list1.add(""); //编译通过


ArrayList list2 = new ArrayList<String>();
list2.add(1); //编译通过 
list2.add(""); //编译通过 
```

可以看到，我们使用list2创建对象时，程序是无错误的，意思是我们可以传入任何对象，那原因是什么呢？

主要原因是`new ArrayList()`只是在内存中开辟了一个存储控件，可以存储任何类型对象，而真正涉及类型检测的是它的引用，因为我们的list1引用是`ArrayList<String>`，所以能完成泛型类型的检测，而list2引用的是`ArrayList`，没有使用泛型，所以是不行的。

举个更全面的例子：
```java
public static void main(String[] args) {  
    ArrayList<String> arrayList1=new ArrayList();  
    arrayList1.add("1");//编译通过  
    arrayList1.add(1);//编译错误  
    String str1=arrayList1.get(0);//返回类型就是String  
        
    ArrayList arrayList2=new ArrayList<String>();  
    arrayList2.add("1");//编译通过  
    arrayList2.add(1);//编译通过  
    Object object=arrayList2.get(0);//返回类型就是Object  
        
    new ArrayList<String>().add("11");//编译通过  
    new ArrayList<String>().add(22);//编译错误  
    String string=new ArrayList<String>().get(0);//返回类型就是String  
}  
```

**2：自动类型转换**

因为类型擦除的问题，所以所有的泛型类型变量最后都会被替换为原始类型，这样就有一个疑问。既然都被替换为原始类型，那么为什么我们在获取的时候，不需要进行强制类型转换呢？看下ArrayList的get方法：

```java
public E get(int index) {  
    RangeCheck(index);  
    return (E) elementData[index];  
}  
```

可以看到，在return之前，会根据泛型变量进行强转。假设泛型类型变量为Date，虽然泛型信息会被擦除掉，但是会将(E) elementData[index]，编译为(Date)elementData[index]。所以我们不用自己进行强转。

**3：类型擦除引起和多态的冲突**

现在有这样一个泛型类：

```java
class Pair<T> {  
    private T value;  
    public T getValue() {  
        return value;  
    }  
    public void setValue(T value) {  
        this.value = value;  
    }  
}
```
  

然后我们想要一个子类继承它
```java
class DateInter extends Pair<Date> {  
    @Override  
    public void setValue(Date value) {  
        super.setValue(value);  
    }  
    @Override  
    public Date getValue() {  
        return super.getValue();  
    }  
}
```  
在这个子类中，我们设定父类的泛型类型为Pair<Date>，在子类中，我们覆盖了父类的两个方法，我们的原意是这样的：
将父类的泛型类型限定为Date，那么父类里面的两个方法的参数都为Date类型：“

```java
public Date getValue() {  
    return value;  
}  
public void setValue(Date value) {  
    this.value = value;  
}  
```
 
所以，我们在子类中重写这两个方法一点问题也没有，实际上，从他们的@Override标签中也可以看到，一点问题也没有，实际上是这样的吗？

分析：

实际上，类型擦除后，父类的的泛型类型全部变为了原始类型Object，所以父类编译之后会变成下面的样子：

```java
class Pair {  
    private Object value;  
    public Object getValue() {  
        return value;  
    }  
    public void setValue(Object  value) {  
        this.value = value;  
    }  
}
```
  
再看子类的两个重写的方法的类型：
```java
@Override  
public void setValue(Date value) {  
    super.setValue(value);  
}  
@Override  
public Date getValue() {  
    return super.getValue();  
}
```
  
先来分析setValue方法，父类的类型是Object，而子类的类型是Date，参数类型不一样，这如果实在普通的继承关系中，根本就不会是重写，而是重载。
我们在一个main方法测试一下：

```java
public static void main(String[] args) throws ClassNotFoundException {  
    DateInter dateInter=new DateInter();  
    dateInter.setValue(new Date());                  
    dateInter.setValue(new Object());//编译错误  
}  
```
如果是重载，那么子类中两个setValue方法，一个是参数Object类型，一个是Date类型，可是我们发现，根本就没有这样的一个子类继承自父类的Object类型参数的方法。所以说，却是是重写了，而不是重载了。

为什么会这样呢？

原因是这样的，我们传入父类的泛型类型是Date，Pair<Date>，我们的本意是将泛型类变为如下：

```java
class Pair {  
    private Date value;  
    public Date getValue() {  
        return value;  
    }  
    public void setValue(Date value) {  
        this.value = value;  
    }  
}  

```
然后再子类中重写参数类型为Date的那两个方法，实现继承中的多态。
可是由于种种原因，虚拟机并不能将泛型类型变为Date，只能将类型擦除掉，变为原始类型Object。这样，我们的本意是进行重写，实现多态。可是类型擦除后，只能变为了重载。这样，类型擦除就和多态有了冲突。JVM知道你的本意吗？知道！！！可是它能直接实现吗，不能！！！如果真的不能的话，那我们怎么去重写我们想要的Date类型参数的方法啊。

于是JVM采用了一个特殊的方法，来完成这项功能，那就是桥方法。

首先，我们用javap -c className的方式反编译下DateInter子类的字节码，结果如下：
```java
class com.tao.test.DateInter extends com.tao.test.Pair<java.util.Date> {  
  com.tao.test.DateInter();  
    Code:  
       0: aload_0  
       1: invokespecial #8                  // Method com/tao/test/Pair."<init>"  
:()V  
       4: return  
  
  public void setValue(java.util.Date);  //我们重写的setValue方法  
    Code:  
       0: aload_0  
       1: aload_1  
       2: invokespecial #16                 // Method com/tao/test/Pair.setValue  
:(Ljava/lang/Object;)V  
       5: return  
  
  public java.util.Date getValue();    //我们重写的getValue方法  
    Code:  
       0: aload_0  
       1: invokespecial #23                 // Method com/tao/test/Pair.getValue  
:()Ljava/lang/Object;  
       4: checkcast     #26                 // class java/util/Date  
       7: areturn  
  
  public java.lang.Object getValue();     //编译时由编译器生成的巧方法  
    Code:  
       0: aload_0  
       1: invokevirtual #28                 // Method getValue:()Ljava/util/Date 去调用我们重写的getValue方法  
;  
       4: areturn  
  
  public void setValue(java.lang.Object);   //编译时由编译器生成的巧方法  
    Code:  
       0: aload_0  
       1: aload_1  
       2: checkcast     #26                 // class java/util/Date  
       5: invokevirtual #30                 // Method setValue:(Ljava/util/Date;   去调用我们重写的setValue方法  
)V  
       8: return  
}  
```
从编译的结果来看，我们本意重写setValue和getValue方法的子类，竟然有4个方法，其实不用惊奇，最后的两个方法，就是编译器自己生成的桥方法。可以看到桥方法的参数类型都是Object，也就是说，子类中真正覆盖父类两个方法的就是这两个我们看不到的`桥方法`。而打在我们自己定义的setvalue和getValue方法上面的@Oveerride只不过是假象。而桥方法的内部实现，就只是去调用我们自己重写的那两个方法。
所以，虚拟机巧妙的使用了`桥方法`，来解决了类型擦除和多态的冲突。

# 泛型内联特化Reified [Kotlin独有]
我们在之前已经了解了泛型的类型擦除，类型擦除大致会带来一些问题，
比如Kotlin中常用的`转换操作符 as`

```kotlin
fun <T> Any.asAny(): T? {
    return this as? T
}
```
上述代码在进行类型转换时，没有进行检查，可能会出现因为类型不一致而出现的运行时崩溃

例如下面的代码
```kotlin
fun <T> Any.asAny(): T? {
    return this as? T
}

fun main() {
    val res = 1.asAny<String>()?.substring(1)
    println(res)
}
```
输出结果
```kotlin
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
	at com.eegets.javademo.generic.ExtKt.main(Ext.kt:24)
	at com.eegets.javademo.generic.ExtKt.main(Ext.kt)
```
可以看到出现了ClassCastException异常，是因为我们没有进行类型检查，所以为了安全获取数据一般需要显示传递转换结果的class信息
```kotlin
fun <T> Any.asAny(clazz: Class<T>): T? {
    return if (clazz.isInstance(this)) {
        this as? T
    } else {
        null
    }
}

fun main() {
    val res = 1.asAny<String>(String::class.java)?.substring(1)
    println(res)
}
```
输出结果
```kotlin
null
```
这样是能解决问题，但是需要传递class方式，这种方式比较笨重，尤其是参数过多时。

那有没有可以排除这种传递参数之外更好的实现呢？

## Reified内联特化关键字

好在Kotlin有更好的应对方案，Java没有，这就是Reified[具体化也可以叫特化]关键字

Reified使用非常简单，主要分两步（都是必须要加的）：
* 1：在泛型类型前面增加reified修饰符
* 2：在方法前增加inline内联

我们可以改进一下上述代码
```kotlin
inline fun <reified T> Any.asAny(clazz: Class<T>): T? {
    return if (this is T) {
        this
    } else {
        null
    }
}

fun main() {
    val res = 1.asAny<String>(String::class.java)?.substring(1)
    println(res)
}
```

这时候输出就正常了

```java
public static final void main() {
      Integer $this$asAny$iv = 1;  //待转换的值
      Class clazz$iv = String.class;
      int $i$f$asAny = false;
      String var10000 = (String)($this$asAny$iv instanceof String ? $this$asAny$iv : null);　　//通过Java的instanceof对`$this$asAny$iv`常量验证是否是类型`String`
      if ((String)($this$asAny$iv instanceof String ? $this$asAny$iv : null) != null) { //使用内联进行代码替换，并通过instanceof进行常量验证
         String var4 = var10000;
         byte var6 = 1;
         $i$f$asAny = false;
         if (var4 == null) {
            throw new TypeCastException("null cannot be cast to non-null type java.lang.String");
         }

         var10000 = var4.substring(var6);
         Intrinsics.checkExpressionValueIsNotNull(var10000, "(this as java.lang.String).substring(startIndex)");
      } else {
         var10000 = null;
      }

      String res = var10000;
      boolean var5 = false;
      System.out.println(res);
   }
```

如上生成的Java源码可以看到，首先通过`reified`关键字会通过Java关键字`instanceof`验证常量`$this$asAny$iv`是否是类型String,然后再通过内联进行代码替换，另外也可以看出，通过inline内联函数修饰之后，制定的类型是不被擦除的，因为inline函数在编译期会将字节码copy到调用的方法里，所以编译器在执行此代码时是知道具体的类型的，然后把泛型替换为具体类型，从而达到不擦除类型的目的。



# [参考自泛型类相关文章]

* [java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一](https://blog.csdn.net/s10461/article/details/53941091)

* [深入理解Java和Kotlin中的泛型](https://ethanhua.github.io/2018/01/09/genericity/)

* [深入理解 Java 泛型](https://dunwu.github.io/javacore/basics/java-generic.html#_1-%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81%E6%B3%9B%E5%9E%8B)


* [Java泛型类型擦除以及类型擦除带来的问题](https://www.cnblogs.com/wuqinglong/p/9456193.html)

* [java泛型 泛型的内部原理：类型擦除以及类型擦除带来的问题](https://blog.csdn.net/wisgood/article/details/11762427)

* [Reified Types in Kotlin: how to use the type within a function (KAD 14)](https://antonioleiva.com/reified-types-kotlin/)
