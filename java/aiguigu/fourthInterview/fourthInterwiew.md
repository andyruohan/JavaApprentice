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

## BigDecimal 的一些坑
### 1. 使用 BigDecimal(double) 存在精度损失风险
当使用float/double这些浮点数据时，会丢失精度String/int则不会，BigDecimal(double) 存在精度损失风险
```java
【强制】禁止使用构造方法 BigDecimal(double)的方式把 double 值转化为 BigDecimal 对象。
说明：BigDecimal(double)存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。
如：BigDecimal g = new BigDecimal(0.1f); 实际的存储值为：0.10000000149
正例：优先推荐入参为 String 的构造方法，或使用 BigDecimal 的 valueOf 方法，此方法内部其实执行了
Double 的 toString，而 Double 的 toString 按 double 的实际能表达的精度对尾数进行了截断。
 BigDecimal recommend1 = new BigDecimal("0.1");
 BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```

### 2. BigDemcial 的等值比较应使用 compareTo()
equals() 方法会比较值和精度，(1.0) 与 (1.00) 返回结果为 false，而 compareTo() 则会忽略精度。
```java
【强制】浮点数之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals 来判断。
说明：浮点数采用“尾数+阶码”的编码方式，类似于科学计数法的“有效数字+指数”的表示方式。二进制无法精确表示大部分的十进制小数，具体原理参考《码出高效》。
反例：
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
if (a == b) {
 // 预期进入此代码快，执行其它业务逻辑
 // 但事实上 a==b 的结果为 false
}
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)) {
 // 预期进入此代码快，执行其它业务逻辑
 // 但事实上 equals 的结果为 false
}
正例：
(1) 指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的。
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
float diff = 1e-6f;
if (Math.abs(a - b) < diff) {
 System.out.println("true");
}
(2) 使用 BigDecimal 来定义值，再进行浮点数的运算操作。
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
if (x.equals(y)) {
 System.out.println("true");
}
```

### 3. 除法商的结果需要指定精度
```java
    private static void m3() {
	    BigDecimal amount1 = new BigDecimal("2.0");
	    BigDecimal amount2 = new BigDecimal("3.0");
	    System.out.println(amount1.divide(amount2)); //Non-terminating decimal expansion; no exact representable decimal result.
    }
```
直接除，会报下面的错误：
```
Exception in thread "main" java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
	at java.math.BigDecimal.divide(BigDecimal.java:1707)
	at test.BigDecimalBugDemo.m3(BigDecimalBugDemo.java:67)
	at test.BigDecimalBugDemo.main(BigDecimalBugDemo.java:25)
```
应加上精度处理，如下模式也就是我们常说的我们的“四舍五入”：
```java
        System.out.println(amount1.divide(amount2, 2, RoundingMode.HALF_UP)); 
```


### 4. 科学计数法的使用问题
```java
	private static void m4() {
		BigDecimal amount1 = BigDecimal.valueOf(1234567890123456789.3141592631415926);
		//输出结果使用了科学计数法: 1.23456789012345677E+18
		System.out.println(amount1);
		//输出结果使用了科学计数法: 1.23456789012345677E+18
		System.out.println(amount1.toString());
		//输出结果未使用科学计数法
		System.out.println(amount1.toPlainString());
		
		System.out.println();
		BigDecimal amount2 = new BigDecimal("1234567890123456789.3141592631415926");
	    //输出结果未使用科学计数法
		System.out.println(amount2);
	    //输出结果未使用科学计数法
		System.out.println(amount2.toString());
	    //输出结果未使用科学计数法
		System.out.println(amount2.toPlainString());
	}
```
上例的输出结果为：
```
1.23456789012345677E+18
1.23456789012345677E+18
1234567890123456770

1234567890123456789.3141592631415926
1234567890123456789.3141592631415926
1234567890123456789.3141592631415926
```

## List 去重的几种方法
以下是你提供的代码中去除List重复元素的几种方法，并对其优化空间进行分析：

### 1. for循环遍历判断是否含有，没有就新增到`newList`里
```java
private static void m1() {
    // 原始列表
    List<Integer> initList = Arrays.asList(70,70,-1,5,3,3,4,4,4,4,99);
    List<Integer> srcList = new ArrayList<>(initList);
    List<Integer> newList = new ArrayList<>();

    for (int i = 0; i < srcList.size(); i++) {
        if(!newList.contains(srcList.get(i))) {
            newList.add(srcList.get(i));
        }
    }
    System.out.println(newList);
    System.out.println();
}
```
**优化空间：**
- **时间复杂度：** 该方法时间复杂度为O(n²)，因为每次调用`contains`方法都需要遍历`newList`。
- **优化建议：** 可以使用一个`HashSet`来保存已经遇到的元素，并检查`srcList`的元素是否在`HashSet`中。如果不在则添加到`newList`和`HashSet`中，这样可以将时间复杂度降低到O(n)。

### 2. 结合`HashSet`或`LinkedHashSet`去重
```java
private static void m2() {
    List<Integer> srcList = Arrays.asList(70,70,-1,5,3,3,4,4,4,4,99);
    List<Integer> newList = new ArrayList<>(new HashSet<>(srcList));
    newList.forEach((s) -> System.out.print(s+" "));
    System.out.println();
    System.out.println();

    newList = new ArrayList<>(new LinkedHashSet<>(srcList));
    newList.forEach((s) -> System.out.print(s+" "));
    System.out.println();
    System.out.println();
}
```
**优化空间：**
- **时间复杂度：** 使用`HashSet`和`LinkedHashSet`的时间复杂度是O(n)，已经是最优的解决方案之一。
- **优化建议：** `HashSet`会丢失原列表的顺序，而`LinkedHashSet`保留顺序。代码已经考虑到两者的差异，除非特别需要控制内存占用或者提高特定场景的性能，否则不需要进一步优化。

### 3. 结合 Stream 流式计算
```java
private static void m3() {
    List<Integer> initList = Arrays.asList(70,70,-1,5,3,3,4,4,4,4,99);
    List<Integer> srcList = new ArrayList<>(initList);
    List<Integer> newList = null;

    newList = srcList.stream().distinct().collect(Collectors.toList());

    newList.forEach((s) -> System.out.print(s+", "));
}
```
**优化空间：**
- **时间复杂度：** 该方法在内部使用了`LinkedHashSet`，时间复杂度为O(n)。
- **优化建议：** 这个方法非常简洁，且在多核处理器上可能具有更好的性能（由于Stream的并行特性）。如果需要优化性能，可以尝试使用`parallelStream()`，但在小规模数据集上并行处理可能得不偿失。

### 4. 循环坐标去除重复
```java
private static void m4() {
    List<Integer> initList = Arrays.asList(70,70,-1,5,3,3,4,4,4,4,99);
    List<Integer> srcList = new ArrayList<>(initList);
    List<Integer> newList = new ArrayList<>(initList);

    for (Integer element : srcList) {
        if(newList.indexOf(element) != newList.lastIndexOf(element)) {
            newList.remove(newList.lastIndexOf(element));
        }
    }
    newList.forEach((s) -> System.out.print(s+", "));
    System.out.println();
    System.out.println();
}
```
**优化空间：**
- **时间复杂度：** 该方法的时间复杂度接近O(n²)，因为`indexOf`和`lastIndexOf`都是O(n)操作。
- **优化建议：** 可以优化为通过`HashSet`记录已经遇到的元素，并在遍历过程中检查重复项，而不是使用`indexOf`和`lastIndexOf`。

### 5. 双 for 循环对比
```java
private static void m5() {
    List<Integer> initList = Arrays.asList(70,70,-1,5,3,3,4,4,4,4,99);
    List<Integer> srcList = new ArrayList<>(initList);
    List<Integer> newList = new ArrayList<>(initList);

    for (int i = 0; i < newList.size()-1; i++) {
        for (int j = newList.size()-1; j > i ; j--) {
            if(newList.get(j).equals(newList.get(i))){
                newList.remove(j);
            }
        }
    }
    newList.forEach((s) -> System.out.print(s+" "));
    System.out.println();
    System.out.println();
}
```
**优化空间：**
- **时间复杂度：** 该方法的时间复杂度为O(n²)。
- **优化建议：** 可以使用类似于`m1`方法中的`HashSet`来记录已经遇到的元素，避免使用双重循环，减少时间复杂度。

### 总结
总体来说，`m2`（使用`HashSet`或`LinkedHashSet`）和`m3`（Stream流式计算）是最优的解决方案。对于其他方法，可以通过使用`HashSet`来降低时间复杂度。

## == 和 equals
### 代码示例
```java
/**
 * @author andy_ruohan
 * @description
 * @date 2024/8/18 13:57
 *   == 和 equals
 *  1 比较范围
 *    1.1 == 既可以比较基本类型也可以比较引用类型
 *    1.2 equals只能比较引用类型，equals(Object obj)
 *  2 比较规则
 *    equals比较规则，看是否被重写过
 *    2.1 没有被重写，出厂默认就是==
 *    2.2 如果被重写，具体看实现方法
 */
public class EqualsDemo {
	public static void main(String[] args) {
		String s1 = new String("abc");
		String s2 = new String("abc");
		System.out.println(s1 == s2);
		System.out.println(s1.equals(s2));
		Set<String> set01 = new HashSet<>();
		set01.add(s1);
		set01.add(s2);
		System.out.println(set01.size());
		System.out.println(s1.hashCode() + "\t" + s2.hashCode());
		System.out.println("================================");

		Person p1 = new Person("abc");
		Person p2 = new Person("abc");
		System.out.println(p1 == p2);
		System.out.println(p1.equals(p2));
		Set<Person> set02 = new HashSet<>();
		set02.add(p1);
		set02.add(p2);
		System.out.println(set02.size());
		System.out.println(p1.hashCode() + "\t" + p2.hashCode());
		System.out.println("================================");
		System.out.println();
	}

	@AllArgsConstructor
	static class Person {
		private String name;
	}
}
```
上述代码运行结果为：
```
false
true
1
96354	96354
================================
false
false
2
758529971	2104457164
================================
```

### 原因分析
1. 当你不重写equals和hashCode方法时，Java会使用Object类的默认实现，导致比较的是对象的引用，而不是内容。
2. 对于String类，它重写了equals和hashCode方法，使得它可以正确地比较字符串的内容，并且在集合中不会出现重复元素（如果内容相同）。
3. 自定义类（如Person）如果想要在集合中正确判断相等性，通常需要重写equals和hashCode方法。
