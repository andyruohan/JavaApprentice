## 基础知识
1. Collection和Map是Java集合框架的两大根接口,它们都直接继承自Object类。
2. transient关键字：标记为transient的字段不会被序列化。

## 弱项模拟 
### 泛型
以下泛型代码编译时会报错的是？选项：
   
```java
A.
List<String> list = new ArrayList<>();
list.add("Java");
```

```java
B.
List<?> list = new ArrayList<String>();
list.add("Java"); // 报错行
```

```java
C.
public static <T> T getFirst(List<T> list) {
    return list.get(0);
}
```

```java
D.
List<Integer> list = Arrays.asList(1,2,3);
```
解析：选 B。List<?> 是无界通配符，仅支持读取（不能添加非 null 元素），故 add ("Java") 编译报错。

### 反射（高级：动态代理 + 枚举反射限制）
以下代码运行结果是？
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

interface Service { void execute(); }
enum Status { RUNNING, STOPPED; }

public class ReflectionAdvanced {
    public static void main(String[] args) throws Exception {
        // 动态代理：代理Service接口
        Service proxy = (Service) Proxy.newProxyInstance(
            Service.class.getClassLoader(),
            new Class[]{Service.class},
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.print("Proxy-");
                    return null;
                }
            }
        );
        proxy.execute(); // 语句1

        // 反射创建枚举实例
        Constructor<Status> ctor = Status.class.getDeclaredConstructor(String.class, int.class);
        ctor.setAccessible(true);
        Status custom = ctor.newInstance("CUSTOM", 2); // 语句2
    }
}
```
选项：  
A. 输出 “Proxy-”，并创建 Status.CUSTOM 实例.   
B. 输出 “Proxy-”，语句 2 抛出 IllegalArgumentException.   
C. 语句 1 抛出 ClassCastException，语句 2 执行成功.   
D. 语句 1 编译错误，语句 2 抛出 NoSuchMethodException.   

解析：选 B。  
 - 动态代理基于接口生成代理类，Proxy.newProxyInstance返回的代理对象可强转为 Service 接口，调用execute()会触发 InvocationHandler 的invoke方法，输出 “Proxy-”。
 - 枚举类的构造器由 JVM 隐式定义（参数为String name, int ordinal），但反射无法创建枚举实例：即使设置setAccessible(true)，Constructor.newInstance()会直接抛出IllegalArgumentException（JVM 强制限制）。

### 线程同步（高级：StampedLock+ABA 问题）
以下关于 StampedLock 与 AtomicStampedReference 的描述，正确的是？
```java
import java.util.concurrent.atomic.AtomicStampedReference;
import java.util.concurrent.locks.StampedLock;

public class SyncAdvanced {
    private int value = 100;
    private StampedLock lock = new StampedLock();
    private AtomicStampedReference<Integer> asr = new AtomicStampedReference<>(100, 0);

    // StampedLock乐观读
    public int optimisticRead() {
        long stamp = lock.tryOptimisticRead();
        int current = value;
        if (!lock.validate(stamp)) { // 乐观读验证
            stamp = lock.readLock();
            try {
                current = value;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return current;
    }

    // AtomicStampedReference解决ABA
    public boolean update() {
        int current = asr.getReference();
        int stamp = asr.getStamp();
        return asr.compareAndSet(current, current + 1, stamp, stamp + 1);
    }
}
```

选项：  
A. optimisticRead()中，若乐观读验证失败，会升级为悲观读锁，保证数据一致性.   
B. StampedLock的乐观读无需释放锁，因此性能远高于ReentrantReadWriteLock.  
C. AtomicStampedReference的compareAndSet仅比较引用值，未解决 ABA 问题.   
D. update()中，若其他线程将值从 100→101→100，compareAndSet仍会执行成功.   

解析：选 A。   
- StampedLock的乐观读逻辑：先尝试无锁读取（tryOptimisticRead），若验证（validate）发现数据被修改，则升级为悲观读锁重新读取，保证一致性。
- B 错误：乐观读虽无需释放锁，但验证失败会升级为悲观锁；且高竞争场景下，StampedLock性能不一定优于ReentrantReadWriteLock。
- C 错误：AtomicStampedReference同时比较引用值 + 版本戳，解决了 ABA 问题（普通AtomicReference仅比较引用值）。
- D 错误：若值从 100→101→100，版本戳会从 0→1→2，compareAndSet中旧戳（0）不匹配，执行失败。

### Java 类加载与实例初始化顺序选择题
以下代码执行后，输出结果的顺序是？
```java
class Parent {
	// 父类静态代码块
	static {
		System.out.print("1-父类静态代码块 ");
	}

	// 父类普通代码块
	{
		System.out.print("2-父类普通代码块 ");
	}

	// 父类无参构造器
	public Parent() {
		System.out.print("3-父类构造函数 ");
	}
}

class Child extends Parent {
	// 子类静态代码块
	static {
		System.out.print("4-子类静态代码块 ");
	}

	// 子类普通代码块
	{
		System.out.print("5-子类普通代码块 ");
	}

	// 子类无参构造器
	public Child() {
		System.out.print("6-子类构造函数 ");
	}
}

public class InitOrderTest {
	public static void main(String[] args) {
		// 第一次创建子类实例
		new Child();
		// 第二次创建子类实例
		new Child();
	}
}
```
选项：  

A. 1 - 父类静态代码块 4 - 子类静态代码块 2 - 父类普通代码块 3 - 父类构造函数 5 - 子类普通代码块 6 - 子类构造函数 2 - 父类普通代码块 3 - 父类构造函数 5 - 子类普通代码块 6 - 子类构造函数.   
B. 1 - 父类静态代码块 2 - 父类普通代码块 3 - 父类构造函数 4 - 子类静态代码块 5 - 子类普通代码块 6 - 子类构造函数 2 - 父类普通代码块 3 - 父类构造函数 5 - 子类普通代码块 6 - 子类构造函数.   
C. 1 - 父类静态代码块 4 - 子类静态代码块 2 - 父类普通代码块 5 - 子类普通代码块 3 - 父类构造函数 6 - 子类构造函数 2 - 父类普通代码块 5 - 子类普通代码块 3 - 父类构造函数 6 - 子类构造函数.    
D. 1 - 父类静态代码块 4 - 子类静态代码块 3 - 父类构造函数 2 - 父类普通代码块 6 - 子类构造函数 5 - 子类普通代码块 2 - 父类普通代码块 3 - 父类构造函数 5 - 子类普通代码块 6 - 子类构造函数

解析:选 A。   

**Java 类加载与实例初始化的核心顺序遵循 “静态先行、父类优先” 的原则**，具体拆解如下：
1. 类加载阶段（静态代码块执行规则）
   类加载时静态代码块仅执行一次（无论创建多少实例），且先执行父类静态代码块，再执行子类静态代码块。
   本题中第一次创建Child实例时，JVM 先加载Parent类 → 执行1-父类静态代码块，再加载Child类 → 执行4-子类静态代码块；第二次创建Child实例时，类已加载完成，静态代码块不再执行。
2. 实例初始化阶段（普通代码块 + 构造函数执行规则）
   每次创建实例时，都会执行以下顺序：① 执行父类普通代码块 → ② 执行父类构造函数 → ③ 执行子类普通代码块 → ④ 执行子类构造函数。本质原因：子类构造器默认隐含super()（调用父类无参构造），而父类普通代码块会被编译器插入到父类构造器的最前端，因此顺序为：父类普通代码块 → 父类构造函数 → 子类普通代码块 → 子类构造函数。

### 集合遍历与并发修改
以下代码的输出是？
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
for (int i = 0; i < list.size(); i++) {
    if (list.get(i) % 2 == 0) {
        list.remove(i);
    }
}
System.out.println(list);
```
A. [1, 2, 3, 4, 5]  
B. [1, 3, 5]  
C. [1, 3, 4, 5]  
D. 抛出ConcurrentModificationException     

解析：选B。

陷阱点：ConcurrentModificationException通常在使用迭代器（如Iterator或for-each）时修改集合结构时抛出。但本题使用的是索引遍历（for (int i = 0; ...)），直接通过索引操作列表，不会触发该异常。   
逻辑分析：   
初始列表：[1, 2, 3, 4, 5].  
i=0：元素1（奇数，不删除）.  
i=1：元素2（偶数，删除后列表变为[1, 3, 4, 5]，i递增到2.  
i=2：元素4（偶数，删除后列表变为[1, 3, 5]，i递增到3.  
i=3：此时list.size()为3，i=3不小于3，循环结束.  
最终列表：[1, 3, 5].  

错误点：原答案D（抛出异常）是错误的，因为索引遍历不会触发ConcurrentModificationException。   