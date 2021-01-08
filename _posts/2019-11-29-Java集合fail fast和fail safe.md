
在我们讨论fail fast 和fail safe两种机制的区别时,我们先了解一下什么是并发修改

### 什么是并发修改

当一个或者多个线程正在遍历一个集合Collection时,此时另外一个线程修改了此集合中的内容(添加,删除或修改).这就是并发修改.

### 什么是fail fast

我们先看看如下ArrayList中对于fail-fast的注解

```java
<p><a name="fail-fast">
 * The iterators returned by this class's {@link #iterator() iterator} and
 * {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:</a>
 * if the list is structurally modified at any time after the iterator is
 * created, in any way except through the iterator's own
 * {@link ListIterator#remove() remove} or
 * {@link ListIterator#add(Object) add} methods, the iterator will throw a
 * {@link ConcurrentModificationException}.  Thus, in the face of
 * concurrent modification, the iterator fails quickly and cleanly, rather
 * than risking arbitrary, non-deterministic behavior at an undetermined
 * time in the future.
```

大概意思是：当Iterator迭代器被创建后,除了迭代器本身的方法`remove` `add ` 可以改变集合的结构外,其他的因素如若改变了集合的结构,都将被抛出`ConcurrentModificationException`异常,因此,面对有并发修改时,迭代器会快速而干净的报出fails,而不是操作带有不确定性的行为。

请继续看ArrayList官方的注解

```java
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:  <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
```

大概意思是：迭代器的快速失败行为是不一定能够得到保证的，一般来说，存在非同步的并发修改时，不可能做出任何坚决的保证的。但是快速失败迭代器会做出最大的努力来抛出`ConcurrentModificationException`。因此，编写依赖于此异常的程序的做法是不正确的。正确的做法应该是：迭代器的快速失败行为应该仅用于检测程序中的bug。

#### 总结一下就是: fail-fast,即快速失败机制,它是java集合中的一种错误检测机制, 当单个或多个线程在结构上对集合进行改变时,就有可能产生fail-fast机制.

我们再iterator执行next时操作remove,此时会报`ConcurrentModificationException`,如下:

```java
 public static void main(String[] args) {
        method();
    }
   static void method(){
       final ArrayList<Integer> list = new ArrayList<>();
       list.add(1);
       list.add(2);
       list.add(3);
       list.add(4);
       list.add(5);
       Iterator iterator = list.iterator();
       while(iterator.hasNext()){
           System.out.println("while==="+iterator.next());
           list.add(6);
       }
    }
```

日志输出

```java
while===1
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.eegets.rxjava.JavaDemo.method(JavaDemo.java:40)
	at com.eegets.rxjava.JavaDemo.main(JavaDemo.java:11)

Process finished with exit code 1

```



我们通过多线程操作List的数据结构时, 同样也会报出`ConcurrentModificationException`,如下:

```java
 public static void main(String[] args) {
        method();
    }
   static void method(){
       final ArrayList<Integer> list = new ArrayList<>();
       new Thread(new Runnable() {
           @Override
           public void run() {
               list.add(1);
               list.add(2);
               list.add(3);
           }
       }).start();
       new Thread(new Runnable() {
           @Override
           public void run() {
               list.add(99);
           }
       }).start();
       Iterator iterator = list.iterator();
       while(iterator.hasNext()){
           System.out.println("while==="+iterator.next());
       }
    }
```

同样我们看看日志输出

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.eegets.rxjava.JavaIteratorDemo.method(JavaIteratorDemo.java:38)
	at com.eegets.rxjava.JavaIteratorDemo.main(JavaIteratorDemo.java:11)

Process finished with exit code 1
```

如上两种操作我们得出结论:

> 当我们更改迭代器的结构时都会报出异常

为什么会这样,我们可以看看fail-fast的工作原理

```java
private class Itr implements Iterator<E> {
        // Android-changed: Add "limit" field to detect end of iteration.
        // The "limit" of this iterator. This is the size of the list at the time the
        // iterator was created. Adding & removing elements will invalidate the iteration
        // anyway (and cause next() to throw) so saving this value will guarantee that the
        // value of hasNext() remains stable and won't flap between true and false when elements
        // are added and removed from the list.
        protected int limit = ArrayList.this.size;

        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor < limit;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            int i = cursor;
            if (i >= limit)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
                limit--;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

```

我们分析一下如上代码:

如上代码我们可以看出有两处相同的地方,而且这两处地方都报的是`ConcurrentModificationException`

```java
  if (modCount != expectedModCount)
                throw new ConcurrentModificationException();

```

通过代码得知,`expectedModCount` 这个值再对象创建时就被赋予了一个固定值`modCont`, 所以`expectedModCount` 的值肯定是固定的,那问题就显而易见了,当迭代器遍历元素时, 如果modCount这个值发生了改变,那么就会报出`ConcurrentModificationException`异常.

那么什么时候modCount会发生改变呢?

```java
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        checkForComodification();
        l.add(index+offset, element);
        this.modCount = l.modCount;
        size++;
    }

    public E remove(int index) {
        rangeCheck(index);
        checkForComodification();
        E result = l.remove(index+offset);
        this.modCount = l.modCount;
        size--;
        return result;
    }
```

我们可以看到当执行`add`, `remove`以及 `addAll` 时都会让modCount的值发生变化.

### 什么是fail safe

fail safe机制的原理是会复制原集合的一份数据出来,然后操作复制后的数据,避免操作原数据.

使用  `CopyOnWriteArrayList`和`ConcurrentHashMap`无论改变值还是结构都不会报出`ConcurrentModificationException`

我们用该方式修改一下上述代码再运行看看

```java
 public static void main(String[] args) {
        method();
    }
   static void method(){
       final CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
       new Thread(new Runnable() {
           @Override
           public void run() {
               list.add(1);
               list.add(2);
               list.add(3);
           }
       }).start();
       new Thread(new Runnable() {
           @Override
           public void run() {
               list.add(99);
           }
       }).start();
       Iterator iterator = list.iterator();
       while(iterator.hasNext()){
           System.out.println("while==="+iterator.next());
       }
    }
```

日志输出

```java
while===1
while===2
while===3
while===4
while===5
while===99

Process finished with exit code 0
```

如上输出验证了我们刚才的说法.

虽然fail safe避免了抛出异常,但是存在以下缺点:

* 复制时需要额外的空间以及时间上的开销
* 不能保证遍历的是最新的内容

### 总结一下

*  在操作数据变化时尽量少的使用while循环或者foreach循环(`增强for循环内部也是通过迭代器处理`)遍历迭代器
* 如果非要使用while或foreach循环,那么我们可以使用 `CopyOnWriteArrayList`和`ConcurrentHashMap`
* 使用`普通for循环`遍历数据
* 在使用iterator迭代的时候使用synchronized或者Lock进行同步

引用: 
https://blog.csdn.net/ch717828/article/details/46892051
https://medium.com/@mr.anmolsehgal/fail-fast-and-fail-safe-iterations-in-java-collections-11ce8ca4180e