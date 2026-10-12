# 弱项专项练习：10 道单选题

## 1．【MyBatis：线程安全与会话管理】

在 Spring 与 MyBatis 正常整合、使用同一个 `SqlSessionFactory` 的情况下，两个线程同时调用同一个 Mapper Bean。下列说法正确的是？

A. Mapper Bean 是单例，因此两个线程必然共用同一个底层 `DefaultSqlSession`  
B. `SqlSessionTemplate` 通过为所有查询加全局锁，保证线程安全  
C. `SqlSessionTemplate` 获取适合当前调用的会话；在正常的 Spring 事务中，可复用当前线程绑定的事务会话  
D. 每次调用 Mapper 方法都会创建新的 `SqlSessionFactory`

## 2．【Spring Cloud：OpenFeign】

开发人员希望查看 Feign 请求与响应的头信息，但**不记录请求体和响应体**。在日志输出相关配置已生效的前提下，应该选择哪个 Feign 日志级别？

A. `BASIC`  
B. `HEADERS`  
C. `FULL`  
D. `ERROR`

**解析：正确答案 B。**

| 级别 | 请求方法、URL、响应状态码、耗时 | 请求／响应头 | 请求／响应正文 |
|---|---|---|---|
| `NONE`（默认） | ❌ | ❌ | ❌ |
| `BASIC` | ✅ | ❌ | ❌ |
| `HEADERS` | ✅ | ✅ | ❌ |
| `FULL` | ✅ | ✅ | ✅ |

以下用同一次请求展示各级别的简化日志。

**NONE**

```text
无 Feign 请求、响应日志
```

**BASIC**

```text
---> POST https://example.com/users HTTP/1.1
<--- HTTP/1.1 201 Created (35ms)
```

**HEADERS**

```text
---> POST https://example.com/users HTTP/1.1
Content-Type: application/json
Accept: application/json
---> END HTTP (16-byte body)

<--- HTTP/1.1 201 Created (35ms)
Content-Type: application/json
Location: /users/1
<--- END HTTP (23-byte body)
```

**FULL**

```text
---> POST https://example.com/users HTTP/1.1
Content-Type: application/json
Accept: application/json

{"name":"Alice"}
---> END HTTP (16-byte body)

<--- HTTP/1.1 201 Created (35ms)
Content-Type: application/json
Location: /users/1

{"id":1,"name":"Alice"}
<--- END HTTP (23-byte body)
```

官方描述中的 **metadata（元数据）**，指请求和响应的描述信息，例如请求方法、URL、响应状态码、耗时、正文大小等，**不是一个叫 `metadata` 的固定字段**。

元数据也不是 `FULL` 独有的信息；`BASIC` 已经记录了其中一部分。`HEADERS` 可以显示正文大小，但不输出正文内容。`ERROR` 不属于 Feign 的这组日志级别。

## 3．【消息中间件：分区与副本】

某 Kafka Topic 有 6 个分区，副本因子为 1。现在希望在一台 Broker 永久损坏、磁盘数据无法恢复时，仍有机会从其他 Broker 恢复受影响分区的数据。以下哪项调整最直接针对这个目标？

A. 将分区数从 6 增加到 12，副本因子仍为 1  
B. 增加消费者数量，使每个分区都有消费者处理  
C. 减小生产者的 `linger.ms`，让消息更快发出  
D. 增加分区的副本因子，将同一分区的副本分布在不同 Broker，并配合适当的确认与同步配置

## 4．【SQL：GROUP BY 与 HAVING】

订单表 `orders` 包含：

```text
id           订单编号
customer_id  客户编号
amount       订单金额
status       订单状态
```

现在要查询：**已支付订单总金额超过 1000 元的客户，以及他们的已支付订单总金额**。下面哪条 SQL 正确？

**A.**
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING SUM(amount) > 1000;
```

**B.**
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
WHERE status = 'PAID' AND SUM(amount) > 1000
GROUP BY customer_id;
```

**C.**
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
WHERE status = 'PAID' AND amount > 1000
GROUP BY customer_id;
```

**D.**
```sql
SELECT customer_id, SUM(amount) AS total
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 1000;
```

## 5．【MyBatis：动态代理】

项目中只有下面的 Mapper 接口，没有手写实现类：

```java
public interface UserMapper {
    User findById(Long id);
}
```

该接口已正确注册到 MyBatis，SQL 映射也已配置。为什么仍然可以调用 `userMapper.findById(1L)`？

A. Spring 自动生成一个 `UserMapper` 子类，使用 CGLIB 实现 SQL 查询  
B. Java 编译器会根据方法名自动生成 JDBC 查询代码  
C. MyBatis 使用 JDK 动态代理创建实现该接口的对象，由调用处理器将方法调用转换为对应的数据库操作  
D. MyBatis 要求接口方法必须是 `default` 方法，才能执行 SQL

## 6．【MyBatis：一级缓存】

在同一个 Spring 事务中，使用同一个 Mapper 连续执行以下操作。假设使用默认一级缓存配置、关闭二级缓存，且没有其他清理缓存的操作：

```java
User first = userMapper.findById(1L);
User second = userMapper.findById(1L);
```

下列说法正确的是？

A. 两次调用一定执行两次数据库查询  
B. 第二次查询可以复用当前 `SqlSession` 的一级缓存  
C. 第二次查询使用的是 `SqlSessionFactory` 的一级缓存  
D. 只有配置 Redis，第二次查询才能命中缓存

## 7．【Spring Cloud：Sentinel】

某服务依赖一个远程接口。该接口持续响应缓慢，开发人员希望达到设定条件后，暂时停止调用它，快速返回降级结果。最直接对应 Sentinel 的哪项能力？

A. 服务注册与发现  
B. 请求路由转发  
C. 数据库事务回滚  
D. 熔断降级

## 8．【数据库：S 锁与 X 锁】

假设两个事务对同一条记录执行以下操作，只考虑该记录的锁冲突：

```text
① T1 获得该记录的 S 锁
② T2 也获得该记录的 S 锁
③ T1 请求升级为 X 锁，但仍持有自己的 S 锁
④ T2 请求升级为 X 锁，但仍持有自己的 S 锁
```

在数据库采取死锁处理措施之前，下列分析正确的是？

A. 两个事务可能形成循环等待，导致死锁  
B. T1 先获得 S 锁，因此升级 X 锁时一定优先成功  
C. S 锁与 X 锁兼容，所以两个事务都能升级成功  
D. 第②步不可能成功，因为同一条记录只能有一个事务持有 S 锁

## 9．【Spring Security：BCrypt】

两个用户设置了相同的密码，系统分别调用 `BCryptPasswordEncoder.encode()` 生成密码哈希，使用相同的强度参数。下列说法正确的是？

A. 密码相同，生成的哈希一定完全相同  
B. 哈希不同意味着至少有一个用户的密码保存错误  
C. 因为使用随机盐，两次生成的哈希通常不同，但都可以通过 `matches()` 验证原密码  
D. 登录时必须先解密哈希，再与用户输入的密码比较

## 10．【SQL：关联子查询与 EXISTS】

有两张表：

```text
customer：id（客户主键）、name
orders：id（订单主键）、customer_id、amount
```

现在要查询：**至少有一笔订单金额超过 1000 元的客户，每个客户只返回一次。**

下面哪条 SQL 满足要求？

**A.**
```sql
SELECT c.id, c.name
FROM customer c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.amount > 1000
);
```

**B.**
```sql
SELECT c.id, c.name
FROM customer c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.amount > 1000
);
```

**C.**
```sql
SELECT c.id, c.name
FROM customer c
JOIN orders o ON o.customer_id = c.id
WHERE o.amount > 1000;
```

**D.**
```sql
SELECT c.id, c.name
FROM customer c
WHERE c.id IN (
    SELECT o.customer_id
    FROM orders o
    GROUP BY o.customer_id
    HAVING SUM(o.amount) > 1000
);
```

**解析：正确答案 B。**

| 选项 | 判断 | 原因 |
|---|---|---|
| A | 错误 | 没有关联当前客户。只要订单表中存在一笔超过 1000 元的订单，就会返回所有客户 |
| B | 正确 | 关联当前客户，判断是否存在达标订单；多笔匹配订单不会导致客户重复 |
| C | 错误 | 一个客户有多笔达标订单时，会返回多行该客户的信息 |
| D | 错误 | 判断的是订单累计金额超过 1000 元，不是单笔金额超过 1000 元 |

**举例**

客户表：

| id | name |
|---|---|
| 1 | 张三 |
| 2 | 李四 |
| 3 | 王五 |

订单表：

| id | customer_id | amount |
|---|---|---:|
| 101 | 1 | 1200 |
| 102 | 1 | 1500 |
| 103 | 2 | 600 |
| 104 | 2 | 700 |

| 选项 | 返回的客户 | 说明 |
|---|---|---|
| A | 张三、李四、王五 | 存在达标订单，导致所有客户都被选中 |
| B | 张三 | 张三有达标订单，只返回一次 |
| C | 张三、张三 | 两笔达标订单分别产生一行 |
| D | 张三、李四 | 李四累计 1300 元，但没有单笔超过 1000 元的订单 |

**知识点一：只写 JOIN 是什么连接？**

`JOIN` 默认表示 **`INNER JOIN`（内连接）**，只保留满足连接条件的匹配行。

| 写法 | 含义 |
|---|---|
| `JOIN` / `INNER JOIN` | 内连接，只保留匹配行 |
| `LEFT JOIN` | 左外连接，保留左表全部行 |
| `RIGHT JOIN` | 右外连接，保留右表全部行 |
| `FULL OUTER JOIN` | 全外连接，保留两表全部行 |

给 C 选项加上 `DISTINCT`，本题中也能满足要求：

```sql
SELECT DISTINCT c.id, c.name
FROM customer c
JOIN orders o ON o.customer_id = c.id
WHERE o.amount > 1000;
```

**知识点二：SELECT 1 是什么意思？**

`SELECT 1` 表示查询常量 `1`，**不是查询第一列，也不是只查询一行**。

```sql
SELECT 1 FROM orders;
```

如果订单表有四条记录，就返回四行，每行的值都是 `1`。

在本题的 `EXISTS` 中：

```sql
EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.amount > 1000
)
```

`EXISTS` 只关心子查询**是否返回至少一行**，不关心返回值是什么：

- 有匹配行：结果为真，保留当前客户。
- 没有匹配行：结果为假，排除当前客户。
- 有多条匹配行：仍只是判断为真，不会复制外层客户行。

**记忆：JOIN 组合匹配行；EXISTS 判断有没有匹配行；SELECT 1 返回常量。**

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 1 | C | C | ✓ |
| 2 | B | B | ✓ |
| 3 | D | D | ✓ |
| 4 | A | A | ✓ |
| 5 | C | C | ✓ |
| 6 | B | B | ✓ |
| 7 | D | D | ✓ |
| 8 | A | A | ✓ |
| 9 | C | C | ✓ |
| 10 | B | B | ✓ |

注：第 1～5 题出题时，末尾格式示例意外包含了正确答案，因此该组成绩不宜单独作为掌握程度的依据。

## 11．【Spring MVC：异常处理与回调】

在普通同步请求中，某个 Interceptor 的 `preHandle()` 已返回 `true`。随后 Controller 抛出异常，被 `@ControllerAdvice` 中匹配的异常处理方法成功处理并返回响应，后续没有发生新异常。

下列说法正确的是？（单选）

A. `postHandle()` 仍会执行，因为异常最终被成功处理  
B. `afterCompletion()` 会执行，其 `ex` 参数为 `null`  
C. `afterCompletion()` 会执行，其 `ex` 参数一定是 Controller 最初抛出的异常  
D. Controller 一旦抛异常，`postHandle()` 和 `afterCompletion()` 都不会执行

## 12．【架构设计：CAP】

某分布式读写系统发生网络分区，两组节点暂时无法相互通信。系统要求：写入成功后，后续读取必须符合线性一致性；同时，每个收到请求的非故障节点都必须最终完成请求，不能通过拒绝服务代替正常读写。

按照 CAP 定理，下列说法正确的是？（单选）

A. 只要增加副本，就能在任何网络分区下同时满足上述要求  
B. 使用异步复制，就能保证所有节点始终读取到最新写入  
C. CAP 中的 P 指性能，因此增加硬件即可解决冲突  
D. 如果网络分区持续存在，系统无法同时保证上述一致性和可用性要求

## 13．【Kafka：元数据协调与消息复制】

某 Kafka 集群明确采用 **ZooKeeper 模式，而非 KRaft 模式**。关于组件职责，下列说法正确的是？（单选）

A. ZooKeeper 用于元数据协调及控制器选举相关协调，分区消息副本的复制由 Broker 完成  
B. ZooKeeper 保存 Topic 的全部消息正文，Broker 只保存索引  
C. 消费者向 ZooKeeper 发送拉取请求，由 ZooKeeper 返回消息  
D. 两个 Kafka 集群之间的消息复制必须由 ZooKeeper 直接执行

## 14．【数据库：Oracle 索引分区】

一张订单表按照 `order_month` 进行范围分区，共有 12 个分区。现在要在这张表上创建一个索引，按照 `customer_id` 进行哈希分区，共有 8 个索引分区。

这属于哪种索引？（单选）

A. 局部分区索引，因为索引建立在分区表上  
B. 全局非分区索引，因为索引分区数与表不同  
C. 全局分区索引，因为索引采用独立于表的分区方式  
D. 无法创建，索引必须与表采用相同的分区键和分区数

## 15．【Spring Security：数据库用户认证】

某系统采用 Spring Security 标准的用户名、密码认证流程。开发人员需要实现其中的“用户数据查询”环节。数据库中保存了用户名、BCrypt 密码哈希、账号状态和角色信息，框架已配置相应的密码校验组件。

关于这个自定义环节与后续认证流程的职责划分，下列哪种设计正确？（单选）

A. 自定义环节根据用户名查询用户，将本次输入的密码重新进行 BCrypt 编码，与数据库哈希直接比较，再将比较结果交给框架  
B. 自定义环节返回数据库中的密码哈希、账号状态和权限信息，由后续认证组件完成密码匹配及账号检查；查询到用户不代表认证成功  
C. 自定义环节只返回账号状态和权限信息，密码哈希应保留在数据库中，由密码校验组件自行根据用户名查询  
D. 自定义环节返回用户信息后，框架即认为身份认证成功；后续只需根据角色判断是否允许访问目标资源

**解析：正确答案 B。**

“返回密码哈希”指的是：**在服务端内部，将数据库查询结果交给认证组件**，不是把密码哈希返回给浏览器或用户。

密码校验需要两份信息：

| 信息 | 来源 |
|---|---|
| 用户本次输入的明文密码 | 登录请求 |
| 注册时保存的密码哈希 | 数据库查询结果 |

```text
用户提交用户名和密码
          ↓
DaoAuthenticationProvider 协调认证
          ↓
UserDetailsService 根据用户名查询数据库
          ↓
返回 UserDetails
包含：密码哈希、账号状态、权限等
          ↓
使用 PasswordEncoder.matches(输入的明文密码, 保存的哈希)
          ↓
结合账号状态等检查，判断认证是否成功
```

| 选项 | 判断 | 原因 |
|---|---|---|
| A | 错误 | BCrypt 使用随机盐，重新编码后直接比较哈希通常无法验证密码；用户查询环节也不负责这种校验 |
| B | 正确 | 查询环节提供用户信息，后续认证组件负责密码校验和账号检查 |
| C | 错误 | `PasswordEncoder` 不负责查询数据库，也不知道用户名和数据库表结构 |
| D | 错误 | 查询到用户不代表身份认证成功，还需要校验密码、检查账号状态 |

**组件职责：**

| 组件 | 职责 |
|---|---|
| `UserDetailsService` | 加载用户信息，包括数据库中保存的密码哈希 |
| `UserDetails` | 承载用户信息 |
| `PasswordEncoder` | 密码编码与匹配 |
| `DaoAuthenticationProvider` | 协调用户加载、密码校验和账号检查 |

**记忆：查到用户不等于认证成功；内部传递密码哈希不等于向客户端暴露密码哈希。**

## 16．【Spring Security：密码校验】

系统直接使用同一个 `BCryptPasswordEncoder` 实例进行注册和登录。注册时执行：

```java
String storedHash = encoder.encode(rawPassword);
// 将 storedHash 保存到数据库
```

登录时，从数据库读取 `storedHash`，执行：

```java
String inputHash = encoder.encode(inputPassword);
boolean passed = inputHash.equals(storedHash);
```

用户输入了正确密码，但校验通常失败。最合适的修改是？（单选）

A. 注册和登录时使用相同的强度参数，即可保证两次哈希相同  
B. 将 `encoder` 配置为单例，即可保证两次编码使用相同的盐  
C. 使用 `encoder.matches(inputPassword, storedHash)` 校验  
D. 使用 `encoder.matches(storedHash, inputHash)` 校验

**解析：正确答案 C。**

BCrypt 使用随机盐，相同密码多次调用 `encode()`，通常会得到不同的哈希。因此，不能直接比较两次编码结果。

正确写法：

```java
boolean passed = encoder.matches(inputPassword, storedHash);
```

| 参数位置 | 应传内容 |
|---|---|
| 第一个参数 | 用户输入的明文密码 |
| 第二个参数 | 数据库存储的密码哈希 |

D 将数据库中的哈希当成了待验证的明文密码，验证对象不对。

BCrypt 保存的哈希包含验证需要的盐和成本参数，`matches()` 可以利用这些信息验证输入的密码，**不是解密哈希**。

**记忆：`matches(输入的明文, 保存的哈希)`。**

## 17．【MyBatis：一级缓存与更新】

在一个正常生效的 Spring 事务中，以下 Mapper 调用使用同一个 `SqlSession`。一级缓存采用默认的 `SESSION` 范围，二级缓存已关闭，更新语句使用默认的刷新缓存配置。

```java
User first = userMapper.findById(1L);

userMapper.updateName(2L, "新名字");

User second = userMapper.findById(1L);
```

假设第一条查询之前缓存为空，SQL 均正常执行，且没有插件改变缓存行为。下列说法正确的是？（单选）

A. 两次 `findById(1L)` 都会访问数据库，因为中间的更新会清理当前会话的一级缓存  
B. 第二次查询命中一级缓存，因为更新的是用户 2，查询的是用户 1  
C. 第二次查询命中一级缓存，因为事务尚未提交，缓存只在提交时清理  
D. 第二次查询是否访问数据库，取决于 `updateName()` 是否实际修改了至少一行

**解析：正确答案 A。**

默认情况下，更新操作会清理当前 `SqlSession` 的一级缓存，**不是只删除被更新用户的缓存**。

```text
第一次查询用户 1
          ↓
访问数据库，结果进入一级缓存
          ↓
更新用户 2
          ↓
清理当前 SqlSession 的一级缓存
          ↓
第二次查询用户 1
          ↓
缓存已清理，再次访问数据库
```

| 选项 | 判断 | 原因 |
|---|---|---|
| A | 正确 | 中间更新清理了会话缓存，因此第二次查询再次访问数据库 |
| B | 错误 | 缓存清理不是按本次更新的用户编号精确执行 |
| C | 错误 | 不需要等到事务提交才清理缓存 |
| D | 错误 | 默认更新流程会清理缓存，不以实际修改行数大于零为前提 |

**记忆：一级缓存按会话管理，默认更新操作会清理当前会话的一级缓存。**

## 18．【Spring MVC：Filter 的后置逻辑】

某个 Filter 的核心代码如下：

```java
System.out.println("F1");

try {
    chain.doFilter(request, response);
    System.out.println("F2");
} finally {
    System.out.println("F3");
}
```

本次是普通同步请求，Interceptor 的 `preHandle()` 返回 `true`。Controller 抛出的异常被 `@ControllerAdvice` 成功处理，响应也正常完成，后续没有新异常。只考虑本次 Filter 调用，下列说法正确的是？（单选）

A. 打印 `F1、F3`；Controller 抛过异常，因此不会执行 `F2`  
B. 打印 `F1、F2、F3`；Interceptor 的 `postHandle()` 也会执行  
C. 打印 `F1、F2、F3`；Interceptor 的 `afterCompletion()` 会收到 Controller 最初抛出的异常  
D. 打印 `F1、F2、F3`；Interceptor 的 `postHandle()` 不执行，`afterCompletion()` 的 `ex` 为 `null`

## 19．【SQL：LEFT JOIN 与过滤条件】

有以下数据：

```text
customer
id | name
1  | 张三
2  | 李四
3  | 王五

orders
id  | customer_id | status
101 | 1           | PAID
102 | 2           | UNPAID
```

执行：

```sql
SELECT c.id, o.id AS order_id
FROM customer c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'PAID';
```

结果是什么？以下每一对表示 `(客户编号, 订单编号)`。（单选）

A. `(1, 101)、(2, NULL)、(3, NULL)`  
B. `(1, 101)`  
C. `(1, 101)、(2, 102)、(3, NULL)`  
D. `(1, 101)、(3, NULL)`

## 20．【架构设计：重试与幂等】

支付服务收到请求后已经成功扣款，但返回响应时网络中断。调用方不知道是否成功，于是使用**相同的支付请求号**重试。

假设扣款和支付记录都保存在同一个关系型数据库中，目标是：同一支付请求即使被并发重试，也最多扣款一次。以下哪种设计最可靠？（单选）

A. 先查询支付请求号是否存在，不存在就扣款；查询、扣款和写入记录分别提交，不设置唯一约束  
B. 使用一个数据库事务完成扣款和记录插入，但支付请求号不设置唯一约束，也不做其他并发控制  
C. 对支付请求号设置唯一约束，在同一事务内插入支付记录并扣款；重复请求不再扣款，并读取已提交的处理结果  
D. 将支付请求号记录在当前服务进程的内存集合中，集合存在该请求号时直接返回成功，否则执行扣款

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 11 | B | B | ✓ |
| 12 | D | D | ✓ |
| 13 | A | A | ✓ |
| 14 | C | C | ✓ |
| 15（新版） | C | B | ✗ |
| 16 | D | C | ✗ |
| 17 | B | A | ✗ |
| 18 | D | D | ✓ |
| 19 | B | B | ✓ |
| 20 | C | C | ✓ |

**合计：答对 7/10，正确率 70%。**