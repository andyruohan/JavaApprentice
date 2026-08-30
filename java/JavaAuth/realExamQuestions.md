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

11.假设我们要设计扣减库存的操作，下列设计方案不合理的是？<br>
A. 数据库中扣减，成功后更新 Redis 缓存<br>
B. 把库存扣减从异步写转为同步写<br>
C. 先扣减 Redis 缓存，同步扣减数据库，如果失败则回滚 Redis 缓存<br>
D. 先扣减 Redis 缓存，同时向队列中发送一条扣减数据库库存的消息，异步进行数据库扣减，实现最终一致性。

12.Java 中 synchronized 和 lock 的相同点是？<br>
A. 可以知道有没有成功获取锁<br>
B. 可以让等待锁的线程响应中断<br>
C. 可以保证原子性<br>
D. 发生异常时，会自动释放线程占有的锁


13.电话号码表 t_phonebook 中含有100万条数据，其中号码字段PhoneNo上创建了唯一索引，且电话号码全部由数字组成，要统计号码头为321的电话号码的数量，下面写法执行速度最慢的是？  
A. `select count(*) from t_phonebook where substr(phoneno, 1,3) = '321'`  
B. `select count(*) from t_phonebook where phoneno >= '321' and phoneno < '321A'`  
C. 各选项的执行方式差异不大，性能基本一样  
D. `select count(*) from t_phonebook where phoneno like '321%'`


14.部署支持故障转移（允许1 台服务器宕机而不影响服务）的Kafka 集群，至少需要几台服务器？<br>
A. 1<br>
B. 4<br>
C. 3<br>
D. 2<br>

15.通过request对象获取以下用户提交的信息
```
GET /day06/response1?1339484005562 HTTP/1.1
Accept: */* .
Referer: http://localhost/day06/response/demo7/regist.html
Accept-Language: zh
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/4.0|compatible; MSIE 6.0; Windows NT 5.1;
SV1; . NET CLR 2.0.50727)
Host: localhost
Connection: Keep-Alive
```
则以下语句返回什么？<br>
`System.out.println(request.getRequestURI());`<br>
A. `http://192.168.1.114/day06/response1/`<br>
B. `/day06/response1`<br>
C. `http://192.168.1.114/day06/response/demo7/regist.html`<br>
D. `/day06/response/demo7/regist.html`

16【数据库原理】.以下关于子查询，说法不正确的是？  
A. 从逻辑结果上看，所有使用JOIN关键字编写的连接查询，都可以通过使用子查询（如IN、EXISTS等）的方式重写以实现相同的查询目标  
B. FROM子句中使用派生表时需要指定一个表别名  
C. 从逻辑结果上看，所有形式的子查询，都可以通过使用JOIN关键字编写的连接查询来等价替换  
D. 当外部查询需要引用派生表中的计算列（例如函数或表达式结果）时，该计算列在子查询内部需要定义列别名

17【SQL】.有如下语句：  
`select a.id, b.id, a.name, b.name from a full outer join b where a.id is not null or b.id is null;`  
下列说法正确的是？  
A. 结果是两个表中不在交集的部分  
B. 其他说法都不对  
C. 语法会报错  
D. 结果是两个表的交集

18【SpringMVC】.请分析拦截器源码中执行拦截请求的核心代码，描述错误的一项是？
```
public class HandlerExecutionChain {
    private final Object handler;
    private HandlerInterceptor[] interceptors;
    public void addInterceptor(HandlerInterceptor interceptor) {
        initInterceptorList().add(interceptor);
    }
    boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HandlerInterceptor[] interceptors = getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for (int i = 0; i < interceptors.length; i++) {
                HandlerInterceptor interceptor = interceptors[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    triggerAfterCompletion(request, response, null);
                    return false;
                }
            }
        }
        return true;
    }
    void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
        HandlerInterceptor[] interceptors = getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for (int i = interceptors.length - 1; i >= 0; i--) {
                HandlerInterceptor interceptor = interceptors[i];
                interceptor.postHandle(request, response, this.handler, mv);
            }
        }
    }
}
```  
A. 如果 preHandle 方法执行失败，则会执行 triggerAfterCompletion 方法  
B. 在真正执行 controller 的业务代码的前后，会分别执行 applyPreHandle 方法和 applyPostHandle 方法  
C. applyPreHandle 方法执行成功后，就会调用 applyPostHandle 方法  
D. 拦截器是递归调用 applyPreHandle 方法来拦截客户端发送来的请求

19【SpringBoot】.使用下列哪段代码可以返回多个非阻塞响应？  
A. `Flux<String> people = request.bodyToFlux(String.class);`  
B. `String string = request.parseString(String.class);`  
C. `RestTemplate.readResponse((String) result);`  
D. `Mono<String> string = request.bodyToMono(String.class);`

20.假设结算页核心服务的下游是PC端结算页Web、手机App、微信入口等，上游是62个依赖服务接口。下列选项中针对上游的主要降级手段不合理的是？  
A. 按照用户质量，将高风险用户、爬虫优先降级  
B. 根据依赖的影响程度和范围进行降级  
C. 限流降级  
D. 按照上游系统等级，将低级别系统的资源给高级别系统使用