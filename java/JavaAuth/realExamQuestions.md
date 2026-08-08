1.以下是一个事务T4在更新记录R4时加S锁，并在事务结束前未升级为X锁的SQL代码示例：  
```sql
BEGIN TRANSACTION;  
SELECT * FROM table1 WHERE id = 4 FOR SHARE;  
UPDATE tablel SET some_column = some_value WHERE id = 4;  
COMMIT;  
```
以下哪种情况可能会在以上代码中导致死锁？  
A. 有一个会话T2在查询表table1中id为4的记录时也执行了SELECT ... FOR SHARE  
B. 有一个会话T2在修改表table2中id为4的记录时阻塞了该事务  
C. 有一个会话T2在修改表table1中id为5的记录时阻塞了该事务  
D. 有一个会话T2在修改表table1中id为4的记录前也执行了SELECT ... FOR SHARE

2.下面有关 ibatis 中的＃与＄的区别，描述错误的是？   
A. ＄ 方式能够很大程度防止sql注入  
B. ＃ 将传入的数据都当成一个字符串，会对自动传入的数据加一个双引号  
C. ＄ 将传入的数据直接显示生成在sql中  
D. ＄方式一般用于传入数据库对象，例如传入表名  

3.下列关于 Kafka 的减少分区说法正确的是？  
A. 删除主题并不会对分区造成任何影响  
B. 删除分区会导致数据不一致，消息乱序  
C. 減少分区数量，只需要删除某个分区即可，不会对系统操作任何影响  
D. 减少分区等同于删除主题，两个功能实现的是同一种效果  

4.关于having子句说法正确的是？  
A. 其他答案均正确  
B. having是在一个结果返回之后起作用的  
C. having是一个约束说明  
D. having不能够使用聚合函数  

5.对于URL POST请求 http://domain/say/helloworld 的Mapping配置错误的是？  
A. @PostMapping("/say/helloworld")  
B. @PostMapping(value="/say/helloworld")  
C. @Reques tMapping ("/say/helloworld", method = RequestMethod.POST)  
D. @RequestMapping (value="/say/helloworld", method = RequestMethod.POST)  

6.已知表结构
   people (pid, name, email, telephone, birthday) , member(mid, mname, email, mlevel)，eployee(eid, ename)，下列可查询出不重复人名称的语句是？  
A. 
```sql
select name from people
UNION ALL
select mname from member
UNION ALL
select ename from employee
```

B. 
```sql
select name from people
UNION ALL
select mname from member
UNION
select ename from employee
```

C. 
```sql
select name from people
UNION
select mname from member
UNION ALL
select ename from employee
```

D. 
```sql
select name from people
UNION
select mname from member
UNION
select ename from employee
```

7.Feign默认提供的日志级别有哪些：
1. NONE：默认的，不显示任何日志；
2. BASIC：仅记录请求方法、URL、响应状态码及执行时间；
3. HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；
4. ERROR：会记录所以接口返回的错误信息  
A. 1.2.3.4  
B. 1.2.3  
C. 1,2,4  
D. 1,3.4  

8.假设我们想利用mysqI双主模式通过内置的自增索引为基础来实现一个全局唯一id生成服务，因此在一个分布式系统中设置一个专门数据库，记录当前的Maxld值，插入记录时来取这个MaxId，然后自增1后插入。这种方案可能会导致？  
A. 存在多点重复  
B. 存在多点瓶颈  
C. 存在单点重复  
D. 存在单点瓶颈  

9.以下关于Kafka中Zookeeper的功能和特性描述，哪个是正确的？  
A. Zookeeper用于存储Kafka的消息数据  
B. Zookeeper用于执行Kafka集群间的数据同步  
C. Zookeeper负责处理Kafka消费者的网络连接  
D. Zookeeper负责管理Kafka中的主题分区  

10.如何在项目中使用redis整合Mybatis缓存？  
A.
```sql
Mapper中配置添加二级缓存配置，对于增删改等更新数据库操作
<update id="activateCompany" parameterType="int" >
update sql ......
</update>
即可实现缓存
```
B. 在Mapper中添加二级缓存配置   
C. 开启缓存  
D. 通过重写Cache类中的方法，将mybatis中默认的缓存空间映射到redis空间中  