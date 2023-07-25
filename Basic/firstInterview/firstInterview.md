```java
/**
 * @author andy_ruohan
 * @description 自增测试
 * @date 2023/7/25 22:55
 */
public class AutoIncrementDemo {
    public static void main (String[] args) {
        int i = 1;
        i = i++;
        int j = i++;
        int k = i + ++i * i++;
        System.out.println("i=" + i);
        System.out.println("j=" + j);
        System.out.println("k=" + k);
    }
}
```

可通过如下命令查看字节码class文件
>lijunxin@lijunxins-Air firstInterview % javap -c AutoIncrementDemo.class

```java
Compiled from "AutoIncrementDemo.java"
public class basic.AutoIncrementDemo {
  public basic.AutoIncrementDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: istore_1
       2: iload_1
       3: iinc          1, 1
       6: istore_1
       7: iload_1
       8: iinc          1, 1
      11: istore_2
      12: iload_1
      13: iinc          1, 1
      16: iload_1
      17: iload_1
      18: iinc          1, 1
      21: imul
      22: iadd
      23: istore_3
      24: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      27: new           #3                  // class java/lang/StringBuilder
      30: dup
      31: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      34: ldc           #5                  // String i=
      36: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      39: iload_1
      40: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      43: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      46: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      49: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      52: new           #3                  // class java/lang/StringBuilder
      55: dup
      56: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      59: ldc           #10                 // String j=
      61: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      64: iload_2
      65: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      68: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      71: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      74: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      77: new           #3                  // class java/lang/StringBuilder
      80: dup
      81: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      84: ldc           #11                 // String k=
      86: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      89: iload_3
      90: invokevirtual #7                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      93: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      96: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      99: return
}

```

