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