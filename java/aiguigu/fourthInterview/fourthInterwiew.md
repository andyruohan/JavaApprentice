### 服务可用性几个9是什么意思
![](pictures/服务可用性.png)
>互联网大厂一般为4个9。

### Arrays.asList不支持add
#### 问题现象
```java
/*
 * @author andy_ruohan
 * @description Arrays不支持add
 * @date 2024/8/3 15:18
 */
public class ArraysBugDemo
{
	public static void main(String[] args)
	{
		List<Integer> list = Arrays.asList(1,2);
		list.add(3);
		list.forEach(System.out::println);
	}
}
```
上述代码运行结果为：
```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at test.Arrays_BugDemo.main(Arrays_BugDemo.java:16)
```

#### 问题根因
原因为Arrays.asList方法的回参虽然是ArrayList，但却是Arrays的一个内部类，而非真正的ArrayList。二者之所以都能给List变量赋值，是因为它们都继承自AbstractList，而Arrays的内部类ArrayList并没有实现add方法，故会抛出UnsupportedOperationException。
![](pictures/Arrays的asList方法源码.png)

#### 解决方法
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2));
```

### 遍历集合时禁止使用集合的remove方法
#### 问题现象
```java
/**
 * @author andy_ruohan
 * @description 遍历集合时禁止使用list.remove
 * @date 2024/8/3 16:26
 */
public class IteratorRemoveDemo {
	public static void main(String[] args) {
		List<Integer> list = new ArrayList<>();
		list.add(11);
		list.add(12);
		list.add(13);
		list.add(14);
		list.add(15);

		Iterator<Integer> iterator = list.iterator();
		while(iterator.hasNext()) {
			Integer value = iterator.next();
			if(value == 12) {
				list.remove(value);
			}
		}

		list.forEach(System.out::println);
	}
}
```
上述代码运行结果为：
```
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:911)
	at java.util.ArrayList$Itr.next(ArrayList.java:861)
	at test.IteratorRemoveDemo.main(IteratorRemoveDemo.java:25)
```

#### 问题根因
1) 第一次遇到 value == 12 时，通过 list.remove(value) 直接修改了集合。这导致集合的 modCount 被改变，而迭代器的 expectedModCount 没有相应更新。
2) 在下一次循环中，当 iterator.next() 被调用时，迭代器会检查当前的 modCount 是否与 expectedModCount 一致。此时会检测到二者不一致，并抛出 ConcurrentModificationException。
```java 
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
```

```java
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```


#### 解决方法
不使用 list.remove(value) ，使用迭代器的 iterator.remove()。
```java
       if (value.equals(12)) {
	        // 使用 Iterator 的 remove 方法
	        iterator.remove();
       }
```

### Hash 冲突
#### 写一下 Hash 冲突的代码案例
```java
/**
 * @author andy_ruohan
 * @description Hash冲突两种模拟case
 * @date 2024/8/7 22:22
 */
public class HashConflictDemo {
	public static void main(String[] args) {
		hashConflictCase1();
		hashConflictCase2();
	}

	private static void hashConflictCase1() {
        // 不会产生hash冲突
		System.out.println("AA".hashCode());
		System.out.println("BB".hashCode());
		System.out.println();

		// 以下是一些典型的hash冲突案例
		System.out.println("Aa".hashCode());
		System.out.println("BB".hashCode());
		System.out.println();
		System.out.println("柳柴".hashCode());
		System.out.println("柴柕".hashCode());
	}

	private static void hashConflictCase2() {
		// 以下是通过重复new产生hash冲突案例
		HashSet<Integer> hashSet = new HashSet<>();
		for (int i = 1; i <= 15 * 10000; i++) {
			int bookHashCode = new Object().hashCode();
			if (!hashSet.contains(bookHashCode)) {
				hashSet.add(bookHashCode);
			} else {
				System.out.println("发生了hash冲突，在第:" + i + "次，值是：" + bookHashCode);
			}
		}
		System.out.println(hashSet.size());
	}
}
```

case1 输出结果：
```
2080
2112

2112
2112

851553
851553
```

case2 输出结果：
```java
发生了hash冲突，在第:105673次，值是：2134400190
发生了hash冲突，在第:111120次，值是：651156501
发生了hash冲突，在第:121781次，值是：1867750575
发生了hash冲突，在第:145304次，值是：2038112324
发生了hash冲突，在第:146237次，值是：1164664992
149995
```


#### 为什么会产生 Hash 冲突
在 `HashSet<String>` 中，如果元素值为 `"Aa"` 和 `"BB"`，它们的哈希码确实相同。Java 的 `String` 类 `hashCode()` 方法的实现导致这两个字符串具有相同的哈希值。这是因为 `String` 的 `hashCode()` 方法的实现是基于字符串中字符的序列和位置计算的，具体公式如下：

```
hashCode = s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

对于字符串 `"Aa"` 和 `"BB"`，计算结果如下：

- `"Aa"` 的哈希码为：
  ```
  'A' * 31^1 + 'a' * 31^0
  = 65 * 31 + 97
  = 2015 + 97
  = 2112
  ```

- `"BB"` 的哈希码为：
  ```
  'B' * 31^1 + 'B' * 31^0
  = 66 * 31 + 66
  = 2046 + 66
  = 2112
  ```

可以看到，它们的哈希码都为 `2112`，因此它们在 `HashSet` 中会发生哈希冲突。

#### 哈希冲突处理过程
在 `HashSet<String>` 集合中存储的对象出现哈希冲突时，Java 会通过链地址法（即分离链接法）来处理这些冲突。下面详细解释在 `HashSet<String>` 中哈希冲突的处理过程：

1. **计算哈希码**：
   当你向 `HashSet` 中添加一个字符串（如 `Aa`）时，`HashSet` 会调用该字符串的 `hashCode()` 方法计算哈希码。

2. **定位桶（bucket）**：
   通过哈希码确定该整数应放置的桶的位置。如果两个整数的哈希码相同，它们将被放置到同一个桶中。

3. **处理冲突**：
    - **链地址法**：`HashSet` 使用链地址法处理冲突。如果两个或多个整数的哈希码相同，它们将被放在同一个桶中，但以链表或树的形式存储。
        - 当桶中的元素数量较少时，使用链表存储冲突的对象。
        - 当桶中的元素数量超过一定阈值时（默认为8），链表会转换为红黑树，以提高查找性能。

4. **检查相等性**：
   即使两个整数的哈希码相同，`HashSet` 仍会使用 `equals()` 方法来检查它们是否真正相等。如果 `equals()` 方法返回 `true`，`HashSet` 会认为这两个整数是重复的，不会再次添加。如果 `equals()` 方法返回 `false`，则认为它们是不同的对象，添加到同一个桶的链表或树中。

以下是示例代码，展示在 `HashSet<String>` 中处理这种哈希冲突的过程：
```java
import java.util.HashSet;
import java.util.Set;

public class HashSetExample {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        // 添加字符串 "Aa" 和 "BB"
        set.add("Aa");
        set.add("BB");

        // 打印集合，查看两个字符串是否都被添加
        System.out.println(set);

        // 添加重复的字符串 "Aa"
        set.add("Aa");

        // 打印集合，查看是否有重复元素
        System.out.println(set);
    }
}
```
输出结果：

```
[Aa, BB]
[Aa, BB]
```

### 整型包装类型 Integer
#### Integer的构造方法在java9后过时
```java
    // 构造方法在 java9 以后不推荐
    Integer i1 = new Integer(1200);
    Integer i2 = Integer.valueOf(1200);
```
![](Integer的构造方法在java9后过时.png)

#### 使用 Integer 对象比较的一些坑
```java
/**
 * @author andy_ruohan
 * @description Integer 对象比较的一些坑
 * @date 2024/8/8 23:31
 */
public class IntegerBugDemo {
	public static void main(String[] args) {
		// Integer 对象比较
		Integer a = Integer.valueOf(600);
		Integer b = Integer.valueOf(600);
		int c = 600;
		System.out.println(a == b);
		System.out.println(a.equals(b));
		System.out.println(a == c);

		System.out.println("===================");

		Integer x = Integer.valueOf(99);
		Integer y = Integer.valueOf(99);
		System.out.println(x == y);
		System.out.println(x.equals(y));
	}
}
```
以上代码输出结果为：
```java
false
true
true
===================
true
true
```
原因可参考源码，或下面的《阿里巴巴开发手册》摘录的内容：
```java
【强制】所有整型包装类对象之间值的比较，全部使用 equals 方法比较。
说明：对于 Integer var = ? 在-128 至 127 之间的赋值，Integer 对象是在 IntegerCache.cache 产生，
会复用已有对象，这个区间内的 Integer 值可以直接使用==进行判断，但是这个区间之外的所有数据，都
会在堆上产生，并不会复用已有对象，这是一个大坑，推荐使用 equals 方法进行判断。
```