1) JVM/GC
2) JUC(java.util.concurrent)
3) Collection

volatitle是Java虚拟机提供的轻量级的同步机制
1) 保证可见性
2) 禁止指令重排
3) <font color = 'red'>不保证原子性</font>

>可见性：一个线程修改了主内容的值，其它线程马上获得相应通知。