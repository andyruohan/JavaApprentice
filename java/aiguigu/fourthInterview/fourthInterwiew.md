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