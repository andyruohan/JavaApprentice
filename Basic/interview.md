面试高频考点：
1) JVM/GC
2) JUC(java.util.concurrent)
3) Collection

## JMM 
### JMM 模型的定义
JMM（Java Memory Model，即Java内存模型）本身是一种抽象的概念并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段、静态字段和构成数组对象的元素）的访问方式。

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而主内存是共享内存区域，所有线程都可以访问。
><font color = 'red'>但线程对变量的操作（读取赋值等）必须在工作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存</font>，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的`变量副本拷贝`，因此不同的线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成，其简要访问过程如下图：
![](JMM访问过程.png)

### JMM 模型的特性
- **原子性（Atomicity）**：对基本数据类型的读取和赋值操作具有原子性，但复合操作（例如 i++）不保证原子性。  
- **可见性（Visibility）**：保证一个线程修改的变量对其他线程是可见的。使用volatile关键字或锁可以确保可见性。  
- **有序性（Ordering）**：程序执行的顺序可能会受到编译器优化、处理器优化以及指令重排序的影响。通过volatile、synchronized等关键字可以确保指令执行的顺序性。

JMM关于同步的规定：
1) 线程解锁前，必须把共享变量的值刷新回主内存
2) 线程加锁前，必须读取主内存的最新值到自己的工作内存
3) 加锁解锁是同一把锁


## volatile
volatile是Java虚拟机提供的轻量级的同步机制
1) 保证可见性
    >可见性：一个线程修改了主内容的值，其它线程马上获得相应通知。
2) 禁止指令重排
3) <font color = 'red'>不保证原子性</font>

volatile不保证原子性问题解决：
1. synchronized（杀鸡用牛刀，不推荐）
2. AtomicInteger等（推荐）


#### synchronized线程不安全举例（配合volatile解决）
```java {.line-numbers}
public class SingleInstance{

    // 使用 volatile 禁止 new 对象时进行指令重排
    // new 对象，有三条指令完成
    // 1 JVM为对象分配一块内存M在堆区
    // 2 在内存M上为对象进行初始化
    // 3 将内存M的地址复制给instance变量
    private volatile static SingleInstance instance;

    private SingleInstance(){
        if(instance != null){
            throw new RuntimeException("single instance existed");
        }
    }

    public SingaleInstance getInstance(){
        // 减少锁竞争，避免过多的线程进入同步队列，进入 blocking 状态
        if(instance == null){
            // 此处 synchronized 可以保证原子性和可见性
            // 而有序性，指的是保证线程串行进入 synchronized代码块内
            // 所以，此有序性无法保证 synchronized 代码块内部的有序性
            synchronized(SingleInstance.class){
                // 避免重复创建对象
                if(instance == null){
                    instance = new SingleInstance();
                }
            }
        }
        return instance;
    }    
}
```
>new对象的三个步骤(简化）: 
① JVM为对象分配一块内存M。
② 在内存M上为对象进行初始化。
③ 将内存M的地址复制给singleton变量。


假设有两个线程ThreadA 和ThreadB同时来请求SingleInstance.getInstance方法，正常情况按照 ①②③ 的顺序来执行。但由于多线程<font color='red'>指令重排</font>的存在，对象初始化的时候可能按照 ①③② 的顺序，即：**ThreadB是可以拿到一个引用已经有了但是内存资源还没有分配的对象**。此时在变量 instance 前加入 volatitle 禁止指令重排，便可以解决。

```java
/**
 * Description: ABA问题的产生
 **/
public class ABADemo {
    private static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    private static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {
        System.out.println("以下是ABA问题的产生");

        new Thread(() -> {
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "t1").start();

        new Thread(() -> {
            //先暂停1秒 保证完成ABA
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicReference.compareAndSet(100, 2019) + "\t" + atomicReference.get());
        }, "t2").start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
>以下是ABA问题的产生
true	2019

```java
public class ABADemo {
    private static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    private static AtomicStampedReference<Integer> stampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {
        System.out.println("以下是ABA问题的解决");

        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第1次版本号" + stamp + "\t值是" + stampedReference.getReference());
            //暂停1秒钟t3线程
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            stampedReference.compareAndSet(100, 101, stampedReference.getStamp(), stampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t 第2次版本号" + stampedReference.getStamp() + "\t值是" + stampedReference.getReference());
            stampedReference.compareAndSet(101, 100, stampedReference.getStamp(), stampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t 第3次版本号" + stampedReference.getStamp() + "\t值是" + stampedReference.getReference());
        }, "t3").start();

        new Thread(() -> {
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第1次版本号" + stamp + "\t值是" + stampedReference.getReference());
            //保证线程3完成1次ABA
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = stampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName() + "\t 修改成功否" + result + "\t最新版本号" + stampedReference.getStamp());
            System.out.println("最新的值\t" + stampedReference.getReference());
        }, "t4").start();
    }
}
```
>以下是ABA问题的解决  
t3	 第1次版本号1	值是100  
t4	 第1次版本号1	值是100  
t3	 第2次版本号2	值是101  
t3	 第3次版本号3	值是100  
t4	 修改成功否false	最新版本号3  
最新的值	100  

