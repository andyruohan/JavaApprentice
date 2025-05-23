### 什么是云开发范式
![](images/云开发范式定义.png)

### 为何要统一术语
![](images/统一术语.png)

### 为什么要抽取实体
![](images/为什么要抽取实体.png)

### 领域服务和应用服务
![](images/领域层和服务层的Service.png)
如果不能区分领域服务，还是应用服务，先放到应用服务。

### 三层架构存在的问题
![](images/三层架构存在的问题.png)

### 四层架构业务逻辑、持久化、第三方系统交互如何处理
![](images/四层架构框架.png)

### 什么是依赖倒置
![](images/依赖倒置.png)

### 云开发范式如何分析和设计
![](images/云开发范式分析和设计流程1.png)
![](images/云开发范式分析和设计流程2.png)

### 存量应用如何抽取实体
![](images/存量应用抽取实体.png)

### 四层可以简化单元测试
![](images/四层单元测试.png)

软件工程领域，银弹是什么意思？
源自软件工程领域的一个著名论断，是弗雷德里克·布鲁克斯在他的论文《没有“银弹”：软件工程中的本质性和附属性的争论》中提出的观点。他认为，软件开发过程中面临的复杂性、一致性、可变性和不可见性等问题是固有的，无法通过单一的技术或方法来解决。‌

代码的趣味性

其他
- 第三方防腐接口定义放在领域层
- 领域层服务侧重于实体间的协同，不要调用第三方服务

如何确保组员四层代码理解一致
- 定期代码检视
- 代码架构守护
