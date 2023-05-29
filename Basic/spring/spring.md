### 循环依赖
> <div style="text-align: center; font-weight: bold">Circular dependencies</div>
> If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.   
>    
> For example: Class A requires an instance of class B through constructor injection, and class B requires an instance of class A through constructor injection. If you configure beans for classes A and B to be injected into each other, the Spring IoC container detects this circular reference at runtime, and throws a BeanCurrentlyInCreationException.
>  
> <font color = 'blue'>One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. </font>Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.
>  
> Unlike the typical case (with no circular dependencies), a circular dependency between bean A and bean B forces one of the beans to be injected into the other prior to being fully initialized itself (a classic chicken-and-egg scenario).  
  
参考Spring官网：https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html

##### 构造方法不能解决循环依赖
```java
ObjectA objectA = new ObjectA(new ObjectB(new ObjectA(...)));
```

##### 使用setter可解决循环依赖
```java
ObjectA objectA = new ObjectA();
ObjectB objectB = new ObjectB();
objectA.setB(objectB);
objectB.setA(objectA);
```

##### Spring三级缓存
![](三级缓存ConcurrentHashMap.png)
第一层，singletonObjects 存放的是已经初始化好了的 Bean；  
第二层，earlySingletonObjects 存放的是实例化了，但是未初始化的 Bean；  
第三层，singletonFactories 存放的是 FactoryBean 。假如 A 类实现了 FactoryBean ，那么依赖注入的时候不是 A 类，而是 A 类产生的 Bean。

![](三级缓存主要方法.png)
1. A 创建过程中需要 B，于是 A 将自己放到三级缓里面，去实例化 B 。
2. B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有再查二级缓存，还是没有再查三级缓存，找到了 A 然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A 。
3. B 顺利初始化完毕，将自己放到一级缓存里面（此时 B 里面的 A 依然是创建中状态）。然后回来接着创建 A ，此时 B 已经创建结束，直接从一级缓存里面拿到 B，然后完成创建，并将 A 自己放到一级缓存里面。