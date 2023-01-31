面试高频考点：
1) JVM/GC
2) JUC(java.util.concurrent)
3) Collection

volatitle是Java虚拟机提供的轻量级的同步机制
1) 保证可见性
    >可见性：一个线程修改了主内容的值，其它线程马上获得相应通知。
2) 禁止指令重排
3) <font color = 'red'>不保证原子性</font>

![](JMM%E7%AE%80%E4%BB%8B.png)ß
![](JMM访问过程.png)

volatile不保证原子性问题解决：
1. synchronized（杀鸡用牛刀，不推荐）
2. AtomicInteger等（推荐）