##### 字符串常量加载
```java
public class StringPoolDemo {
    public static void main(String[] args) {
        String str1 = new StringBuilder("re").append("dius").toString();
        System.out.println(str1);
        System.out.println(str1.intern());
        System.out.println(str1 == str1.intern());
        
        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2);
        System.out.println(str2.intern());
        System.out.println(str2 == str2.intern());
    }
}
```
上例运行结果为：
```
redius
redius
true
java
java
false
```
第一次 str1 == str1.intern() 结果为true，第二次 str1 == str1.intern() 的结果为false，原因在于：有一个初始化的java字符串（会在JDK类库的初始化过程中被加载并初始化），在加载sun.misc.Version这个类的时候进入常量池。
![](System类中加载sun.misc.Version.png)
![](Version类中的java常量.png)
>可参考文献：《深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）》代码清单2-8

##### synchronized底层解析
```java
/**
 * @author andy_ruohan
 * @description 用于synchronized字节码解析样例
 * @date 2023/5/10 21:32
 */
public class SyncDemo {
    private final Object objectLock = new Object();

    public void m1() {
        synchronized (objectLock) {
            System.out.println("---hello synchronized code block---");
        }
    }
}
```

针对以上代码，执行以下命令：
```
lijunxin@lijunxins-MacBook-Air reentrantLock % javac SyncDemo.java  
lijunxin@lijunxins-MacBook-Air reentrantLock % javap -c SyncDemo.class
```

运行结果：
```java
public class jvm.lock.reentrantLock.SyncDemo {
  public jvm.lock.reentrantLock.SyncDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: new           #2                  // class java/lang/Object
       8: dup
       9: invokespecial #1                  // Method java/lang/Object."<init>":()V
      12: putfield      #7                  // Field objectLock:Ljava/lang/Object;
      15: return

  public void m1();
    Code:
       0: aload_0
       1: getfield      #7                  // Field objectLock:Ljava/lang/Object;
       4: dup
       5: astore_1
       6: monitorenter
       7: getstatic     #13                 // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc           #19                 // String ---hello synchronized code block---
      12: invokevirtual #21                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: aload_1
      16: monitorexit
      17: goto          25
      20: astore_2
      21: aload_1
      22: monitorexit
      23: aload_2
      24: athrow
      25: return
    Exception table:
       from    to  target type
           7    17    20   any
          20    23    20   any
}

```
注意：`6: monitorenter`和`16: monitorexit`分别对应加锁、解锁。其中`22: monitorexit`是为了异常情况下解锁。

####synchronized可重入锁原理：  
<font color = 'red'>每个锁对象拥有一个锁计数器和一个指面持有该锁的线程的指针。</font>当执行monitorenter时，如果目标锁对象的计数器为零，那么说明它没有被其他线程所持有，Java虚拟机会将该锁对象的持有线程设置为当前线程，并且将其计数器加1。  

在目标锁对象的计数器不为零的情況下，如果锁对象的持有线程是当前线程，那么Java虚拟机可以将其计数器加1，否则需要等待，直至持有线程释放该锁。当执行monitorexit时，Java虚拟机则需将锁对象的计数器减1。计数器为零代表锁已被释放。