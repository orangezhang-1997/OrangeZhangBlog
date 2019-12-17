<center><h3>String、StringBuilder与StringBuffer的区别</h3>></center>>

#### String

String被声明为final class,所有属性为final的，因为是被final修饰保证了基础线程安全，因为你没办法对里面的数据结构进行改变，在并发情况下一个对象被多次访问，可以省略同步时间和锁的等待时间可以大幅度的提高性能。也是因为不变，所以每次裁剪、拼接字符串等操作都会产生新的String对象，我们拼接或者裁剪过程中会产生许多中间对象，占据堆空间导致GC发生的比较频繁。

#### StringBuffer

StringBuffer是为了减少拼接或裁减中产生的太多中间对象而提供了一个类，可以用append()或者add()，把字符串添加到已有序列的末尾或者指定位置，StringBuffer本质上就是一个线程安全的可修改字符串序列，保证了线程安全，它的线程安全是用synchronized关键字去实现。如果你点进StringBuffer的源码中去看的话就会发现它的方法都是加了synchronized关键字来保证线程安全，所以说在没必要的情况下就别使用StringBuffer了，毕竟它的速度是比较慢的，推荐使用下面的StringBuilder

```
 @Override
    public synchronized StringBuffer append(Object obj) {
        toStringCache = null;
        super.append(String.valueOf(obj));
        return this;
    }

   @Override
    public synchronized StringBuffer delete(int start, int end) {
        toStringCache = null;
        super.delete(start, end);
        return this;
    }
     @Override
    public synchronized StringBuffer deleteCharAt(int index) {
        toStringCache = null;
        super.deleteCharAt(index);
        return this;
    }
```

#### StringBuilder

StringBuilder是Java1.5之后出来的，在能力和StringBuffer没有本质区别，但是它去掉了线程安全的部分，有效减少开销。

#### StringBuffer与StringBuilder的区别

StringBuffer与StringBuilder的底层数据结构与String一样，在JDK9之前的基本数据结构就是可修改的char数组，在JDK9的时候则是byte数组，与String的区别在于StringBuilder与StringBuffer的值可以改变，值改变后对象的引用不会发生改变。按照默认大小创建一个字符数组，但是问题来了，这个数组应该创建多大？如果太小，拼接的时候就需要更大的数组，如果太大又浪费了空间，现在字符串的默认初始长度是16，然后再不断加入数据，如果超过了默认大小，就会创建一个新的数组，将原来的数组复制过去然后再将旧的数组丢弃。

两者的不同则是方法有没有加了synchronized，StringBuffer加了synchronized而StringBuilder并没有加，所以前者是**线程安全**的而后者是线程不安全的，但是**后者比前者的速度快**。一般在对安全方面不作要求的话就基本用StringBuilder。

```
public class Test {
    public static void main(String[] args) {
        String s="Hello"+"World"+"!!";
    }
}
```

上面这段操作在JVM中优化后其实就是`StringBuilder s=new StringBuilder().append("Hello").append("World").append("!!");  

#### String面试题 

我们在面试过程中经常会出现下面这一道题

```
public class Test {
    public static void main(String[] args) {
        String string="123";
        String string1=new String("123");
        String string2="123";
        String string3=new String("123");
        System.out.println(string==string1);//false
        System.out.println(string==string2);//true
        System.out.println(string2==string3);//false
        System.out.println(string1==string3);//false
   　　　System.out.println(string1.intern()==string);//true
    }
}
```

面试官会让你说出下面几个都会输出什么，或者会问你有几个对象。这里考的是什么？考察的其实就是面试者对字符串常量池的理解，还有ｉｎｔｅｒｎ（）方法的理解。

**为了避免产生大量的字符串对象，JVM中有一个常量池叫做字符串常量池，我们创建的字符串大多就放在常量池中。当我们创建一个字符串时，先去检查池中的字符串有没有与这个字符串相等的字符串，如果有的话就不需要创建，直接从池中刚查找到的对象引用；如果没有找到，那就新建一个字符串并将这个字符串放入字符串常量池中。然而，new一个对象则不一样，new一个对象是不在常量池中找，直接在堆里创建一个对象，也不会将对象放入池中。**

首先在字符串常量池中创建了一个字符串“abc”，因为这时候字符串常量池并没有“abc”这个字符串数组， 所以会在字符串常量池中就会去创建一个字符串"abc"；接下来因为有了一个**new**关键字,所以这里就会直接在堆里去创建这个实例对象，因为string1与string前者在堆中，后者在字符串常量池中，所以两个==为false，然后给string2赋值一个"123"；这时候string2首先在字符串常量池中寻找，发现常量池中有这个字符串，所以就将对象引用戳在这个常量池的字符串中。string3再去new一个实例对象，这时候因为是重新创建了一个对象，所以string3与string1的hashcode值并不相同。最后调用了String的一个intern()方法，这个intern()方法的作用就相当于将这个堆里的字符串加入到了字符串常量池中，如果字符串常量池中并没有这个字符串，则会新创建一个字符串，如果字符串常量池中有这个字符串，则会返回字符串常量池中的那个字符串。 