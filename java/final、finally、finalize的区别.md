<center><h3>final、finally、finalize的区别

#### final

 final可以用来修饰类、方法、变量，分别有不同的意义。

- **修饰类**：这个类不能被继承，所有的方法不能被重写，但是里面的成员变量并不是不可变得，如果需要让它们不可变则需要用final去修饰变量。

- **修饰方法**：不可以被重写，但是子类可以去使用这个方法。

- **修饰变量**：不可修改，可以用于保护只读数据，尤其是在并发中，有利于减少额外的同步开销，也可以省去一些防御性的拷贝。

final指的是**引用不可变性**，也就是说他只能指向初始化的那个对象，而不关心指向对象内容的变化。所以被final修饰的变量必须被初始化，来做一道题开心一下~~

```java
public class Test {
    public static void main(String[] args) {
       final StringBuilder stringBuilder=new StringBuilder("Hello ");
       stringBuilder.append("World");
        System.out.println(stringBuilder);
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
       final StringBuilder stringBuilder=new StringBuilder("Hello");
       stringBuilder=new StringBuilder("World!");
    }
}
```

上面这两条程序执行的结果是什么？

- A：Hello World ，Hello World！
- B：编译报错，Hello World!
- C：Hello World，编译报错
- D：编译报错，编译报错

~~猜一猜~~，看一看这道题的答案是什么？

#### finally

finally是Java保证重点代码一定被执行的一种机制，例如 try-finally或者try-catch-finally来进行关闭连接，例如关闭JDBC的连接，unlock操作，因为finally一次只能抛出一种异常，其他的异常会被抑止，导致异常丢失，于是在Java7出现了一种新的特性——**try-with-resources**

只要资源实现了AutoCloseable或者Closeable接口，那么try-with-resources就可以将其自动关闭。

```java
public class Test {
    public static void main(String[] args) {
        try(Connection connection=new Connection() ){
            connection.commit();
        }
        catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

这其实就是编译器在编译的时候给自动给我们加了finally块，并添加了资源的close()，所以close才会在运行的时候去执行，当然try-with-resource还支持声明多个资源，不过每个资源需要用‘;’隔开。这样的话Java平台能够更好的处理异常情况，编码量也会少很多。

在上面说过，finally是Java保证重点代码一定被执行的一种机制，**但是有一种情况是一个例外**

```java
public class Test {
    public static void main(String[] args) {
        try {
            //do something
            System.exit(1);
        }
        finally {
            System.out.println("Print from finally");
        }
    }
}
```

因为System.exit()是将JVM中的所有内容都关闭了，如果参数为0则表示正常退出，如果是1则表示非正常退出。所以，这里的finally并不会被执行。

#### finalize

finalize是Object类的一个方法，目的是保证对象在被垃圾收集完成特定的资源回收，但是不被推荐使用。因为你无法保证finalize什么时候执行，执行的是否和你的预期相同，使用的时候会不会影响程序性能导致程序死锁。如果你想去回收资源，可以去运用上面说的try-with-resources或者try-finally去回收资源，那是一种比较好的回收资源的方法。

不过它已经被废弃了，因为**Cleaner**已经替换了它，它利用了幻影引用。