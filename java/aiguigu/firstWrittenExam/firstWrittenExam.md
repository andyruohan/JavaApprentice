## mysql 最左前缀法则示例
假设index(a,b,c)  

| Where语句                                                | 索引是否被使用                                                                                     |
|--------------------------------------------------------|---------------------------------------------------------------------------------------------|
| where a = 3                                            | 是，使用到 a                                                                                     |
| where a =  3 and b = 5                                 | 是，使用到 a、b                                                                                   |
| where a =  3 and b = 5 and c = 4                       | 是，使用到 a、b、c                                                                                 |
| where b =  3 或者 where b = 3 and c =  4 或者 where c =  4 | 否                                                                                           |
| where a =  3 and c = 5                                 | 使用到 a，但是 c 不可以，b 中间断了                                                                       |
| where a =  3 and b > 4 and c = 5                       | 使用到 a 和 b，c 不能用在范围之后，b断了                                                                    |
| where a is  null and b is not null                     | is null 支持索引，但是 is not null 不支持，所以 a 可以使用索引，但是 b 不一定能用上索引（该描述基于mysql 8.0，也可能不成立取决于具体数据库的版本） |
| where a  <> 3                                          | 不能使用索引                                                                                      |
| where  abs(a) =3                                       | 不能使用索引                                                                                      |
| where a =  3 and b like 'kk%' and c = 4                | 是，使用到 a、b、c                                                                                 |
| where a =  3 and b like '%kk' and c = 4                | 是，只用到 a                                                                                     |
| where a =  3 and b like '%kk%' and c = 4               | 是，只用到 a                                                                                     |
| where a =  3 and b like 'k%kk%' and c =  4             | 是，使用到 a、b、c                                                                                 |

可以结合字典排序来理解:
1) 如`where a =  3 and c = 5`，就类似于查找单词找`a%c`，由于中间这一段不知道是啥，因此只能根据索引找到`a`，剩余的只能通过全表扫描。
2) 如`where a =  3 and b > 4 and c = 5`，就类似查找单词，先根据索引找到字母为`a`的，然后根据索引找到字母大于`b`，但在大于`b`后再找字母为`c`的就很难找了，需要对`ac、ae、...、az`打头的单词逐个核对。
3) 如`where a  <> 3`，就类似于查找单词，除`a`打头的单词全部查找一遍。

### 如何分析 explain 语句
如下面的 explain 语句：
```
mysql> EXPLAIN SELECT * FROM emp where age = 33 and deptId = '2027' and name = 'rPZBJR';
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys       | key                 | key_len | ref               | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------------------+------+----------+-------+
|  1 | SIMPLE      | emp   | NULL       | ref  | idx_age_deptid_name | idx_age_deptid_name | 93      | const,const,const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+-------------------+------+----------+-------+
1 row in set, 1 warning (0.01 sec)
```
每列代表的意义为：
- **id**: 查询中的每个操作步骤都有一个唯一的标识符，这里只有一个操作步骤，所以标识符为 1。
- **select_type**: SIMPLE 表示这是一个简单的 SELECT 查询，没有 UNION 或子查询等复杂的结构。
- **table**: 表示查询涉及的表名，这里是 emp 表。
- **partitions**: 如果查询涉及分区表，此列将显示相关分区信息。
- **type**: 这是非常重要的一列，表示了在表中找到行的方式。在这里，类型是 ref，表示查询使用了索引的“引用”（ref）访问方法。
- **possible_keys**: 这列显示了可能被查询使用的索引，这里可能使用的索引是 idx_age_deptid_name。
- **key**: 表示实际使用的索引，这里使用了索引 idx_age_deptid_name。
- **key_len**: 表示索引的长度，这里是 93 个字节。
- **ref**: 显示了在索引中使用的列和值，这里是 const,const,const，表示在索引中使用了常量值。
- **rows**: 表示 MySQL 估计查询需要检查的行数，这里是 1 行。
- **filtered**: 表示通过索引条件过滤的行的百分比，这里是 100.00%，表示索引条件过滤了 100% 的行。
- Extra: 提供了其他信息，这里为 NULL，表示没有额外的操作。
**综上所述**，查询使用了索引 idx_age_deptid_name，并且使用了索引的引用（ref）访问方法来检索数据，通过索引条件过滤了所有行，并且没有进行额外的排序操作。

其中，需要重点关注的列：
- **type**: 这列显示了在表中找到行的方式。常见的类型包括：
  - **ALL**: 表示全表扫描，即检索表中的所有行。
  - **index**: 表示全索引扫描，即扫描整个索引而不访问实际的数据行。
  - **range**: 表示使用索引进行范围扫描，即根据索引列上的范围条件来检索数据。
  - **ref**: 表示使用索引引用（ref）访问方法，通常与等值查询条件一起使用。
  - **const**: 表示使用索引常量（const）访问方法，通常与等值查询条件一起使用。
  - **eq_ref**: 表示连接查询中使用了索引唯一性的情况。
- **possible_keys**: 这列显示了可能被查询使用的索引。如果存在多个索引，这些索引中的一个或多个可能会被用于查询。
- **key**: 这列显示了实际被查询使用的索引。通常，你希望看到查询使用了一个合适的索引。
- **rows**: 这列表示 MySQL 估计查询需要检查的行数。较小的值表示较高的性能，因为查询需要检查的行越少，查询的执行速度越快。
- **filtered**: 这列表示通过索引条件过滤的行的百分比。通常情况下，你希望这个值越高越好，因为这表示索引条件过滤了更多的行，减少了查询的处理量。
- **Extra**: 这列提供了关于查询执行过程的额外信息。一些常见的值包括 "Using index"（表示查询使用了覆盖索引）、"Using where"（表示查询使用了 WHERE 子句中的条件）、"Using temporary"（表示查询需要使用临时表）、"Using filesort"（表示查询需要对结果进行排序）等。


### 案例实测
#### 无过滤不索引
1) 查看索引
```
mysql> SHOW INDEX FROM emp;
+-------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table | Non_unique | Key_name            | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| emp   |          0 | PRIMARY             |            1 | id          | A         |      499086 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| emp   |          1 | idx_age_deptid_name |            1 | age         | A         |          20 |     NULL |   NULL | YES  | BTREE      |         |               | YES     | NULL       |
| emp   |          1 | idx_age_deptid_name |            2 | deptId      | A         |      188125 |     NULL |   NULL | YES  | BTREE      |         |               | YES     | NULL       |
| emp   |          1 | idx_age_deptid_name |            3 | name        | A         |      499086 |     NULL |   NULL | YES  | BTREE      |         |               | YES     | NULL       |
+-------+------------+---------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
4 rows in set (0.07 sec)
```
可以看到名为 idx_age_deptid_name 的组合索引：age + deptId + name。

2) 无过滤条件`(即无where)`执行 sql
```
mysql> EXPLAIN SELECT * FROM emp ORDER BY age,deptid;
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+----------------+
|  1 | SIMPLE      | emp   | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 499086 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+--------+----------+----------------+
1 row in set, 1 warning (0.01 sec)
```
Extra 栏使用了 `filesort`。

3) 有过滤条件`(即有where)`执行 sql
```
mysql> EXPLAIN SELECT * FROM emp where age > 1000 ORDER BY age,deptid;
+----+-------------+-------+------------+-------+---------------------+---------------------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys       | key                 | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------------+---------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | emp   | NULL       | range | idx_age_deptid_name | idx_age_deptid_name | 5       | NULL |    1 |   100.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------------+---------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.01 sec)
```
Extra 栏使用了 `index condition`。


## JVM 内存模型中，为什么要区分新生代和年老代，对于新生代为什么又要区分 eden 区、survivor 区?
### 为什么要区分新生代和年老代核心要点
- 不同年龄代收集算法不同
- 对内存连续空间的处理不同（亦即垃圾碎片的处理）

### 新生代为什么又要区分 eden 区、survivor 区核心要点
- 为了更有效的区分哪些对象应该被复制到老年代

### 知识延伸
#### 新生代的流转过程
![image-20230312072639148](img/image-20230312072639148.png)
1) 新对象会被保存到 eden 区（开始是空的所以内存连续），eden 区满了会把有效对象复制到 s0（s0 也是空的所以也是连续空间） 
2) 清空 eden 区（再次写入时又是连续空间） 
3) s0 和 s1 在命名上互换，原来的 s1 等待写入（空的） 
4) eden区再次满了，重复上面步骤

#### 不同年代的代表算法
1) 新生代：复制算法，如半空间复制算法。
2) 老年代：标记-整理-清除算法，如 CMS（没有整理环节，有可能产生垃圾碎片）、G1 算法。

## 远程调用 RPC(Remote Procedure Call) 有哪几种
远程调用RPC有几种常见的实现方式，包括：
1) **基于HTTP协议的RESTful API**：使用HTTP请求和响应进行通信，常见于Web服务和RESTful API。
2) **基于TCP/IP的Socket编程**：直接通过套接字进行数据传输，可以实现自定义的远程调用协议。
3) **基于SOAP（Simple Object Access Protocol）的Web服务**：使用XML作为消息格式，在Web服务中较为常见，但已逐渐被RESTful API替代。
4) **基于消息队列（Message Queue）的RPC**：通过消息队列实现异步通信，例如使用AMQP（Advanced Message Queuing Protocol）或者其他消息队列系统。
5) **基于Dubbo**：Dubbo是阿里巴巴开源的基于Java的RPC框架，提供高性能、透明化的远程方法调用，支持多种协议、集群容错、负载均衡等特性，广泛用于企业级分布式应用开发。
6) **基于WebFlux和Spring Data Reactive**：利用WebFlux构建响应式的Web服务，同时结合Spring Data Reactive处理数据存取，实现异步响应式的RPC调用。

## 远程调用需要注意哪些问题？
需要注意以下问题：
1. **接口文档**: 编写清晰的接口文档以便于维护和团队协作。
2. **统一化的报文结构**: 确保输入和输出的数据格式统一，便于处理和解析。
3. **标准化的服务状态码**: 使用统一的状态码标识请求结果，方便客户端处理。
4. **统一化请求日志记录及异常记录**: 记录请求日志和异常信息以便追踪和排查问题。
5. **全局异常处理**: 捕获并处理服务异常，保证系统稳定运行。
6. **快速失败**: 当请求延迟过高时，及时返回失败结果，避免长时间等待。
7. **重试机制**: 在请求失败时进行重试，提高请求成功率。
8. **服务列表和重试次数控制**: 在请求失败时切换服务列表并控制重试次数，避免无限重试。
9. **事务处理**: 在需要保证数据一致性的场景下使用事务，确保操作要么全部成功，要么全部失败。
10. **分布式事务处理**: 在分布式系统中，确保跨服务的事务操作要么全部成功，要么全部失败。
    >如TCC(Try-Confirm-Cancel)
11. **分布式锁**: 处理并发修改数据时使用分布式锁保证数据一致性。
12. **数据一致性**: 在并发修改数据时，使用合适的策略保证数据的一致性，例如分布式锁或乐观锁。

## @Transactional
### @Transactional放在类上
当`methodB`中发生异常时，确保整个事务被回滚，包括`methodA`中的其他操作。下面是一个更新后的示例：

```java
@Service
@Transactional
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public void methodA() {
        try {
            userRepository.save(new User("Alice"));
            methodB();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void methodB() {
        userRepository.save(new User("John"));
    }
}
```

在这个示例中，如果`methodB`中的操作抛出异常，则会捕获异常并进行处理。整个事务将被回滚，包括`methodA`中的其他操作，比如userRepository.save(new User("Alice"));。

### @Transactional放在方法上
如果想在`methodA`和`methodB`中使用不同的事务，可以将`@Transactional`注解放在方法级别而不是类级别，并使用适当的事务传播行为。在Spring中，可以通过`propagation`属性来指定事务的传播行为。
```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void methodA() {
        try {
            userRepository.save(new User("Alice"));
            // 在新的事务中调用methodB
            methodB();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void methodB() {
        userRepository.save(new User("John"));
    }
}
```

在这个示例中，`methodA`和`methodB`都被注解为`@Transactional`，但是`methodB`的事务传播行为被设置为`REQUIRES_NEW`，这意味着它会在一个新的事务中执行，而不受调用者的事务影响。


### 异常传播
`methodB`默认传播机制是`Propagation.REQUIRED`，这意味着如果调用者有事务，`methodB`将加入调用者的事务；如果调用者没有事务，它将开启一个新的事务。
```java
import org.springframework.transaction.annotation.Transactional;

public class ExampleService {

    @Transactional
    public void methodA() {
        try {
            methodB();
        } catch (Exception e) {
            // 异常处理
        }
    }

    @Transactional
    public void methodB() {
        // 执行一些操作，可能会抛出异常
        throw new RuntimeException("Exception in methodB");
    }
}
```
上例中，当`methodB`中抛出异常时，它将导致`methodA`所在的事务回滚。

### 防止异常传播（在被调用的方法捕获异常）
如果在`methodB`中捕获了异常，而不是在`methodA`中捕获，那么事务的回滚行为将取决于异常是否被重新抛出。
```java
import org.springframework.transaction.annotation.Transactional;

public class ExampleService {

    @Transactional
    public void methodA() {
        try {
            methodB();
        } catch (Exception e) {
            // 异常处理
        }
    }

    @Transactional
    public void methodB() {
        try {
            // 执行一些操作，可能会抛出异常
            throw new RuntimeException("Exception in methodB");
        } catch (Exception e) {
            // 在methodB中捕获了异常但没有重新抛出
            // 可以做一些处理
        }
    }
}
```
上例中，`methodB`中捕获了异常，但没有重新抛出，`methodA`的事务仍然会继续进行，不会回滚。

### 防止异常传播（被调用的方法抛出异常，调用方手动完成事务）
如果`methodB`抛出了异常，而`methodA`捕获了这个异常，并且希望避免`methodA`的事务回滚，可以手动将事务标记为已完成，或者使用Spring的`TransactionAspectSupport`类来手动提交事务。
```java
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.interceptor.TransactionAspectSupport;

public class ExampleService {

    @Transactional
    public void methodA() {
        try {
            methodB();
        } catch (Exception e) {
            // 异常处理
            // 手动标记事务为已完成
            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        }
    }

    @Transactional
    public void methodB() {
        // 执行一些操作，可能会抛出异常
        throw new RuntimeException("Exception in methodB");
    }
}
```
在这个示例中，使用`TransactionAspectSupport`类手动将事务标记为已完成，这样就阻止了事务回滚。


## SpringBoot如何实现自动装配
在Spring Boot中，自动装配（Auto-Configuration）是一种强大的功能，它使得开发人员能够快速构建Spring应用而无需手动配置大量的Bean。以下是实现Spring Boot自动装配的几个关键步骤和概念：

1. 启用自动装配
   Spring Boot项目通常包含一个主类，该类使用@SpringBootApplication注解来启动应用程序。这个注解实际上是一个组合注解，包括@Configuration、@EnableAutoConfiguration和@ComponentScan。
    ```java
    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
    ```
   

2. 自动装配注解  
    **@EnableAutoConfiguration**  
    `@EnableAutoConfiguration`注解告诉Spring Boot根据应用程序的依赖自动配置Spring应用程序上下文。例如，如果类路径下有HSQLDB数据库的依赖，Spring Boot会自动配置一个内存数据库连接。  
    >Spring Boot在启动时会扫描类路径下的所有 META-INF/spring.factories 文件，并从中读取自动配置类的信息。每个 spring.factories 文件都包含一个 org.springframework.boot.autoconfigure.EnableAutoConfiguration 属性，该属性列出了所有应该由Spring Boot自动配置的类。spring.factories 文件的加载过程： 
    >- 启动阶段：当Spring Boot应用启动时，@EnableAutoConfiguration 注解会触发 SpringFactoriesLoader 来加载所有在 META-INF/spring.factories 文件中列出的配置类。
    >- 类路径扫描：SpringFactoriesLoader 会扫描类路径中的所有 META-INF/spring.factories 文件。
    >- 自动配置加载：根据 spring.factories 文件中的配置，Spring Boot会加载并应用相应的自动配置类。

    **@ComponentScan**  
    `@ComponentScan`注解扫描`@Component`、`@Service`、`@Repository`和`@Controller`等注解的类，并注册为Spring Bean。  
   

3. 条件装配（Conditional Configuration）  
   **@Conditional**
   `@Conditional`注解允许在特定条件下进行Bean的创建。Spring Boot使用了许多`@Conditional`注解来实现自动装配。例如： 
   - `@ConditionalOnClass`：当类路径下存在某个类时进行装配。
   - `@ConditionalOnMissingBean`：当Spring上下文中没有定义某个特定Bean时进行装配。
   - `@ConditionalOnProperty`：当某个配置属性存在且满足特定条件时进行装配。
 

4. 创建自动配置类  
   创建一个自动配置类需要遵循以下步骤：  
   (1) 创建一个配置类，并使用@Configuration注解。  
   (2) 使用@Conditional注解来指定条件。  
   (3) 在类路径下创建一个文件META-INF/spring.factories，并在其中配置自动配置类。  
   例如，创建一个自定义的自动配置类：
    ```java
    @Configuration
    @ConditionalOnClass(MyService.class)
    public class MyServiceAutoConfiguration {
        @Bean
        @ConditionalOnMissingBean
        public MyService myService() {
            return new MyService();
        }
    }
    ```

5. 使用注解装配Bean
   Spring Boot提供了多种注解来简化Bean的装配：
   - @Autowired：用于自动注入依赖。
   - @Value：用于注入外部配置属性。
   - @Component、@Service、@Repository、@Controller：用于声明Spring管理的Bean。

6. 外部配置和属性注入
   Spring Boot允许通过application.properties或application.yml文件来进行外部配置。可以使用@Value注解注入这些属性。   
   例如，在application.properties文件中：
    ```yaml
    myapp.message=Hello, World!
    ```
   在Spring Bean中注入这个属性：
    ```java
    @Component
    public class MyBean {
        @Value("${myapp.message}")
        private String message;
    
        // getters and setters
    }
    ```