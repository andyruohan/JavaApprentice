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