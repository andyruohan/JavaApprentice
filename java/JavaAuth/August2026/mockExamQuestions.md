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

## 21．【Spring Security：认证流程】

某系统使用标准的用户名、密码认证流程。数据库中保存的是 BCrypt 密码哈希，用户信息查询正常，账号也未过期或锁定。开发人员配置了下面的密码校验逻辑：

```java
public boolean matches(
        CharSequence rawPassword, String encodedPassword) {
    return encode(rawPassword).equals(encodedPassword);
}
```

其中 `encode()` 每次都使用随机盐生成 BCrypt 哈希。正确密码也经常校验失败，最合理的修复是？（单选）

A. 查询用户时，先把数据库中的哈希再次编码，再交给认证组件  
B. 使用 BCrypt 自带的匹配方法，将输入的明文密码与数据库哈希进行校验  
C. 注册和登录都使用同一个编码器实例，使随机盐保持一致  
D. 将比较方式改为忽略大小写，避免哈希字符串大小写不同导致失败

## 22．【MyBatis：缓存范围】

程序直接使用 MyBatis，分别通过同一个 `SqlSessionFactory` 打开 `sessionA` 和 `sessionB`。两个会话独立，一级缓存使用默认配置，二级缓存关闭。

假设数据库最初保存用户名字为 `"旧名字"`，依次执行：

```text
① sessionA 查询用户 1，得到“旧名字”
② sessionB 更新用户 1 为“新名字”，并提交
③ sessionA 再次执行与①完全相同的查询
```

期间 `sessionA` 没有提交、回滚、更新或清理缓存，也没有插件改变缓存行为。下列说法最准确的是？（单选）

A. 两个会话来自同一个 Factory，因此②会自动清理 A 的一级缓存  
B. B 提交后会更新 A 中已缓存的对象，因此③直接从缓存得到“新名字”  
C. 一级缓存只缓存 SQL，不缓存查询结果，因此③一定访问数据库  
D. ③可以命中 A 的一级缓存，仍返回“旧名字”；B 的更新不会自动清理 A 的一级缓存

**解析：正确答案 D。**

虽然两个会话来自同一个 `SqlSessionFactory`，但它们的一级缓存彼此独立。一级缓存保存的是查询结果，不只是 SQL。

```text
              SqlSessionFactory
                 /         \
                ↓           ↓
           sessionA      sessionB
                │           │
           一级缓存 A    一级缓存 B
                \           /
                 ↓         ↓
                   数据库
```

| 步骤 | 操作 | A 的一级缓存 | 数据库 |
|---|---|---|---|
| ① | A 查询用户 1 | 缓存“旧名字” | 旧名字 |
| ② | B 更新用户 1 并提交 | **仍是“旧名字”** | 新名字 |
| ③ | A 再次执行相同查询 | 命中缓存，返回“旧名字” | 新名字 |

**B 更新数据库，不会自动修改或清理 A 已缓存的查询结果。**

**一级缓存的清理情况**

| 操作 | 是否清理当前会话的一级缓存 | 出处原文或关键源码 |
|---|---|---|
| 当前会话执行 `insert`、`update`、`delete`，或调用 `commit()`、`rollback()`、`close()` | **会** | “The local cache is cleared upon `update`, `commit`, `rollback` and `close`.” [官方原文](https://mybatis.org/mybatis-3/java-api.html#Local_Cache)<br>`insert()`、`delete()` 也会转调 `update()`：[源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/session/defaults/DefaultSqlSession.html#L183) |
| 当前会话调用 `clearCache()` | **会** | `executor.clearLocalCache();` [源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/session/defaults/DefaultSqlSession.html) |
| 执行配置了 `flushCache="true"` 的普通查询 | **查询前清理** | `if (queryStack == 0 && ms.isFlushCacheRequired()) { clearLocalCache(); }` [源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/executor/BaseExecutor.html#L148) |
| 使用 `localCacheScope=STATEMENT` | **一次最外层查询结束后清理** | `if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) { clearLocalCache(); }` [源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/executor/BaseExecutor.html#L169) |
| **另一个会话**执行更新、提交 | **不会自动清理当前会话** | `this.localCache = new PerpetualCache("LocalCache");` [源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/executor/BaseExecutor.html#L68)<br>**源码推导：**各执行器拥有独立缓存，清理仅作用于自身。 |
| 查询参数改变 | **不会仅因此清理** | `cacheKey.update(value);` [源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/executor/BaseExecutor.html#L228)<br>**源码推导：**实际 SQL 参数值参与生成缓存键，变化通常导致缓存不命中，而不是清理旧缓存。 |

**与第 17 题对比**

| 场景 | 对 A 的一级缓存的影响 |
|---|---|
| A 自己执行更新，即使更新的是其他记录 | 清理 A 的整个一级缓存 |
| B 执行更新并提交，即使更新的是 A 缓存的记录 | 不自动清理 A 的一级缓存 |

如果需要让 A 不再使用旧缓存，可以执行：

```java
sessionA.clearCache();
User user = mapperA.findById(1L);
```

在本题二级缓存关闭的前提下，这次查询会重新访问数据库。但**清理 MyBatis 缓存不等于刷新数据库事务快照**，能否读取到新值，还取决于数据库隔离级别和事务状态。

**记忆：自己的更新清自己的缓存；别人的更新不会自动清我的缓存。**

## 23．【Spring Boot：线程池任务调度】

某 `ThreadPoolTaskExecutor` 的配置如下：

```java
executor.setCorePoolSize(2);
executor.setMaxPoolSize(4);
executor.setQueueCapacity(2);
executor.setRejectedExecutionHandler(
    new ThreadPoolExecutor.AbortPolicy()
);
```

线程池初始为空，连续提交 6 个任务。假设这些任务都执行很久，**提交第 6 个任务时，前面已经开始的任务均未结束**，线程创建正常。

提交完这 6 个任务后，最符合该配置的状态是？（单选）

A. 4 个任务已分配给工作线程，2 个任务在队列中，无任务被拒绝  
B. 2 个任务已分配给工作线程，2 个任务在队列中，另外 2 个任务被拒绝  
C. 4 个任务已分配给工作线程，另外 2 个任务由提交任务的线程执行  
D. 6 个任务都已分配给工作线程，因为队列容量会增加最大线程数

## 24．【Spring：异步调用与事务】

系统使用普通的 Spring 代理式事务管理和异步执行。两个方法位于不同的 Spring Bean 中，均通过代理调用：

```java
@Transactional
public void createOrder() {
    orderMapper.insert(order);
    notificationService.sendAsync(order.getId());
}
```

```java
@Async
public void sendAsync(Long orderId) {
    // 在另一个线程中查询订单并发送通知
}
```

没有额外配置事务上下文传播或提交后回调。下列说法正确的是？（单选）

A. 异步方法自动加入创建订单的事务，查询时一定能读取到尚未提交的订单  
B. `@Async` 会自动等到外层事务提交成功后，再开始执行任务  
C. 异步线程不会自动继承调用线程的事务，任务可能在订单提交前开始；若要求提交成功后才触发，应显式安排提交后的处理  
D. 异步方法中只要抛出异常，创建订单的事务就一定回滚

## 25．【架构设计：数据库与消息的一致性】

订单服务需要同时完成两件事：

- 将订单写入关系型数据库。
- 向消息队列发送“订单已创建”事件。

系统不使用跨数据库与消息队列的分布式事务。要求服务崩溃重启后仍能补发事件，允许消息重复，但消费者必须避免重复处理。

哪种设计最符合要求？（单选）

A. 先发送消息，再写订单；发送成功即可保证数据库最终一定存在该订单  
B. 先提交订单，再直接发送消息；发送失败只记录应用日志，不保存待发送任务  
C. 将数据库操作与消息发送放进同一个 `@Transactional` 方法，即可自动保证两者一起提交或回滚  
D. 在同一个数据库事务中保存订单和待发送事件，由后台任务重试投递，并由消费者按事件唯一标识实现幂等处理

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 21 | B | B | ✓ |
| 22 | B | D | ✗ |
| 23 | A | A | ✓ |
| 24 | C | C | ✓ |
| 25 | D | D | ✓ |

**合计：答对 4/5，正确率 80%。**

# MyBatis 缓存专项练习：26～30 题

以下均为单选题，假设使用标准 MyBatis 行为，没有插件修改缓存逻辑。

## 26．【一级缓存：清理范围】

同一个 `SqlSession` 中，一级缓存采用默认的 `SESSION` 范围，二级缓存关闭。依次执行：

```java
userMapper.findById(1L);        // 查询用户
productMapper.findById(10L);    // 查询商品

userMapper.updateName(2L, "新名字");

productMapper.findById(10L);    // 再次查询同一商品
```

所有 Mapper 均来自该会话，初始缓存为空，SQL 正常执行。最后一次商品查询会怎样？

A. 命中一级缓存，因为更新的是用户表，不是商品表  
B. 命中一级缓存，因为两个查询属于不同 Mapper 命名空间  
C. 重新访问数据库，因为更新操作清理了该会话的一级缓存  
D. 是否访问数据库，取决于更新操作是否提交

## 27．【二级缓存：查询结果何时共享】

`UserMapper` 已开启二级缓存，查询使用默认缓存配置。A、B 是两个独立的 `SqlSession`。

依次发生：

```text
① 初始二级缓存为空
② A 查询用户 1，查询成功，但 A 尚未提交或关闭
③ 新打开的 B 查询同一用户，查询语句与参数完全相同
```

没有其他操作或并发事务，也没有配置阻塞式缓存。下列说法正确的是？

A. B 可以命中二级缓存，因为 A 的查询已经返回结果  
B. B 需要查询数据库，因为 A 的查询结果尚未提交到共享二级缓存  
C. B 可以访问 A 的一级缓存，因为它们使用同一个 Mapper  
D. B 必须等待 A 关闭，因为所有二级缓存读取都会阻塞到其他会话结束

## 28．【一级与二级缓存：flushCache】

`UserMapper` 已启用二级缓存，更新语句被显式配置为：

```xml
<update id="updateName" flushCache="false">
    UPDATE users SET name = #{name} WHERE id = #{id}
</update>
```

更新前，共享二级缓存中已经有用户 1 的旧查询结果。某会话执行该更新，将用户 1 改名并成功提交。期间没有查询、缓存淘汰或其他清理操作。

下列说法正确的是？

A. 一级、二级缓存都不清理，因为 `flushCache="false"` 控制所有缓存  
B. 一级缓存不清理，但二级缓存会在提交时自动清空  
C. 一级、二级缓存都会清理，因为提交必然清空所有缓存  
D. 当前会话的一级缓存会清理，但该更新不会触发二级缓存清理，旧查询结果可能仍存在

## 29．【二级缓存：按表还是按命名空间清理】

`UserMapper` 和 `ReportMapper` 分别配置了各自独立的 `<cache/>`，没有使用 `cache-ref`。

- `UserMapper` 可以更新 `users` 表。
- `ReportMapper` 可以查询 `users` 表，并已将一个旧查询结果存入自己的二级缓存。

现在通过 `UserMapper` 更新 `users` 表并提交，使用默认刷新配置。之后，一个新会话通过 `ReportMapper` 执行与之前完全相同的查询。假设没有缓存过期、淘汰或其他操作。

下列说法正确的是？

A. `ReportMapper` 可能返回旧结果，因为此次更新没有自动清理它独立的二级缓存  
B. 一定查询数据库，因为 MyBatis 会根据被更新的表名清理所有相关缓存  
C. 一定返回新结果，因为 `UserMapper` 会把更新后的字段同步写入两个缓存  
D. 无法执行查询，因为 MyBatis 禁止不同 Mapper 缓存同一张表的数据

## 30．【缓存配置：如何确保本次访问数据库】

`UserMapper` 已开启二级缓存。当前会话使用默认一级缓存配置，刚执行过一次查询，一级缓存中已有该查询结果；共享二级缓存中也可能存在相应结果。

开发人员希望：**下一次查询不复用一、二级缓存，而是真正向数据库发起查询**。这里只讨论是否访问数据库，不要求一定读取到其他事务最新提交的数据。

下列哪种做法能确保达到目的？

A. 只调用 `sqlSession.clearCache()`，然后执行原查询  
B. 只将查询设置为 `useCache="false"`，然后在当前会话再次执行相同查询  
C. 调用 `sqlSession.clearCache()`，并通过设置了 `useCache="false"` 的查询语句执行查询  
D. 只将 `localCacheScope` 配置为 `STATEMENT`，保留查询的二级缓存配置

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 26 | C | C | ✓ |
| 27 | B | B | ✓ |
| 28 | D | D | ✓ |
| 29 | A | A | ✓ |
| 30 | C | C | ✓ |

**合计：答对 5/5，正确率 100%。**

## 补充问答：缓存清理主要看 Session、Mapper，还是表？

**我的疑问：**

> 所以一级缓存跟 Session，二级缓存跟 Mapper。虽然它们与同一张表的数据有关，但主要清理规则取决于 Session 和 Mapper，可以这样理解吗？

**解答：**

可以。**清理范围主要取决于 SqlSession 和 Mapper 关联的缓存，不是数据库表。**

| 缓存 | 所属范围 | 更新时的清理规则 |
|---|---|---|
| **一级缓存** | 当前 `SqlSession` | 通过该会话执行增删改，会清理该会话的整个一级缓存，不区分表或 Mapper |
| **二级缓存** | 通常是 Mapper 命名空间；也可通过 `cache-ref` 共享 | 更新语句在关联了二级缓存、且 `flushCache="true"` 时，标记该缓存待清理，提交时清理整个关联缓存；更新语句的 `flushCache` 默认是 `true` |

**“同一张表”是数据上的关联，不是 MyBatis 自动清理缓存的依据。**

例如：

- **同一个 Session 查询不同表：**查询了用户表和商品表后，更新用户表，也会清掉该 Session 中的商品查询缓存。
- **不同 Mapper 查询同一张表：**如果两个 Mapper 各有独立二级缓存，更新其中一个 Mapper，不会自动清理另一个 Mapper 的二级缓存。
- **不同 Mapper 共享缓存：**如果通过 `cache-ref` 关联到同一个二级缓存，清理这个共享缓存就会影响双方。

**记忆：一级看会话，二级看关联缓存，不按表自动联动。**

# 专项练习：31～35 题

以下均为单选题。

## 31．【Redis：数据类型与内部编码】

在一个空的 Redis 数据库中执行：

```redis
SET stock 100
```

随后检查发现：

```text
TYPE stock             → string
OBJECT ENCODING stock  → int
```

下列解释正确的是？

A. `stock` 实际是整数类型，`TYPE` 只是为了兼容客户端而返回 `string`  
B. 这两个结果互相矛盾，正常情况下字符串类型只能使用 `raw` 编码  
C. `stock` 对外是字符串类型，内部采用整数编码节省空间；逻辑类型与内部编码不是一回事  
D. `int` 表示该键已经变成整数集合，后续应使用集合命令操作

## 32．【HDFS：文件修改方式】

某系统每天向 HDFS 写入一份大型交易明细文件，文件写入完成并关闭后，会被多个分析任务反复读取。现在发现文件中间有一条交易金额错误，需要修正。

下列处理思路最符合 HDFS 的文件访问模型的是？

A. 对文件中间位置执行原地覆盖，只要修改前后的字节数一致即可  
B. 生成修正后的新文件并替换旧版本，或另存修正记录，在后续处理时合并  
C. 在文件末尾追加正确金额，HDFS 会自动定位并替换原交易记录  
D. 先截断文件末尾，HDFS 会自动恢复中间错误记录的正确值

**解析：正确答案 B。**

HDFS 不支持任意位置的原地覆盖。修正文件中间的记录，可以采用以下方式：

- **生成修正后的新文件**，再替换旧版本。
- **单独保存修正记录**，后续分析时由业务程序根据规则合并。

例如：

```text
原文件：订单 1001，金额 500 元
修正记录：订单 1001，正确金额 300 元
                  ↓
后续分析程序合并，按 300 元计算
```

**识别订单并应用修正的是业务程序，HDFS 不会理解交易记录并自动修复数据。**

| 选项 | 判断 | 原因 |
|---|---|---|
| A | 错误 | 即使修改前后的字节数相同，也不支持任意位置原地覆盖 |
| B | 正确 | 通过重写文件或另存修正记录，避免原地修改文件中间的数据 |
| C | 错误 | 追加只是在末尾增加内容，不会自动替换原记录 |
| D | 错误 | 截断是缩短文件，不会自动纠正中间记录 |

## 33．【Spring Cloud Gateway：路由匹配】

某 Spring Cloud Gateway WebFlux 应用配置了两条路由。假设路由定义正常加载，没有其他自定义逻辑改变排序与匹配行为：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: general
          order: 0
          uri: http://general-service
          predicates:
            - Path=/api/**

        - id: orders
          order: 10
          uri: http://order-service
          predicates:
            - Path=/api/orders/**
```

请求路径为：

```text
/api/orders/123
```

开发人员希望它优先进入 `orders` 路由。下列说法正确的是？

A. 当前配置已经满足要求，因为更具体的路径一定优先于宽泛路径  
B. 当前配置已经满足要求，因为 `order` 数值越大，优先级越高  
C. 只需要把 `orders` 写到 YAML 前面，保留两个 `order` 值，就一定优先匹配它  
D. 当前会优先匹配 `general`；可将 `orders` 的 `order` 改为 `-1`，让它先参与匹配

## 34．【Reactor：Mono 与 Flux】

某接口需要一次返回一个 JSON 数组，数组中包含 20 个用户。开发人员使用：

```java
Mono<List<User>> result;
```

另一位开发人员认为：“有 20 个用户，就不能用 Mono，必须改成 Flux。”

以下判断正确的是？

A. 该说法不正确：Mono 限制的是最多发出一个元素，这个元素本身可以是包含 20 个用户的 List  
B. 该说法正确：Mono 要求返回对象内部也只能包含一条业务数据  
C. 该说法正确：只要最终 JSON 是数组，就只能用 `Flux<User>` 表示  
D. 该说法不正确，但原因是 Mono 可以像 Flux 一样连续发出任意多个 User 元素

## 35．【Spring：事务自调用】

应用采用普通的 Spring 代理式事务管理。下面两个方法位于同一个 Bean 中，外部通过代理调用 `outer()`：

```java
@Transactional
public void outer() {
    orderMapper.insert(order);
    this.inner();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void inner() {
    auditMapper.insert(audit);
    throw new IllegalStateException("模拟失败");
}
```

假设两个 Mapper 使用同一个事务管理器管理的数据源，`outer()` 没有捕获异常，也没有其他特殊配置。最终结果是什么？

A. 订单提交，审计记录回滚，因为 `inner()` 使用了 `REQUIRES_NEW`  
B. 订单和审计记录都提交，因为自调用使整个调用链的事务都失效  
C. 订单和审计记录都回滚；`inner()` 没有通过代理开启新事务，其数据库操作参与了 `outer()` 已开启的事务  
D. 订单回滚，审计记录提交，因为审计操作在另一个方法中，默认独立提交

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 31 | C | C | ✓ |
| 32 | ？ | B | 未作答 |
| 33 | D | D | ✓ |
| 34 | A | A | ✓ |
| 35 | C | C | ✓ |

**合计：答对 4/5，未作答 1 题；按未作答不得分，得分 80 分。已作答的 4 题全部正确。**

## 补充问答：HDFS 是什么？

**我的疑问：**

> 我有点忘了 HDFS 是什么了，能再介绍一下吗？

**解答：**

**HDFS 全称是 Hadoop Distributed File System，中文是“Hadoop 分布式文件系统”。**它是 Hadoop 中负责存储文件的组件，适合将大量、大型文件分散保存在多台服务器上。

例如，一个大文件可以拆成多个块，再分布存储。下面以每块保存两个副本为例：

```text
一个大文件
    ↓ 拆成数据块
  块 A、块 B、块 C
    ↓ 分散存储，并保存副本
服务器 1：块 A、块 C
服务器 2：块 B、块 A
服务器 3：块 C、块 B
```

| 概念 | 作用 |
|---|---|
| **NameNode** | 管理文件目录、文件由哪些块组成、块的位置等元数据 |
| **DataNode** | 保存实际的数据块，处理数据读写 |
| **Block（数据块）** | 文件拆分后的存储单位 |
| **副本** | 同一数据块保存在不同节点上，提高容错能力 |

HDFS 常用于保存日志、交易明细、历史数据，供批量分析任务反复读取。

**主要访问特点：适合大文件、顺序读写、一次写入后多次读取；支持追加和截断，但不支持随意原地修改文件中间的数据。**

“一次写入”并不是说文件永远不能变化，而是在强调它的主要使用方式。

# 专项练习：36～40 题

以下均为单选题。

## 36．【Spring：事务异常被捕获】

应用采用普通的 Spring 代理式事务管理。两个方法位于同一个 Bean 中，外部通过代理调用 `outer()`：

```java
@Transactional
public void outer() {
    orderMapper.insert(order);

    try {
        this.inner();
    } catch (IllegalStateException e) {
        // 记录日志，不再抛出
    }
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void inner() {
    auditMapper.insert(audit);
    throw new IllegalStateException("模拟失败");
}
```

假设两次插入均成功，使用同一个事务管理器管理的数据源，没有数据库异常，也没有手动设置回滚标记。最终结果是什么？

A. 订单提交，审计记录回滚，因为 `inner()` 声明了 `REQUIRES_NEW`  
B. 订单和审计记录都提交，因为自调用没有开启新事务，且异常被捕获，外层正常结束  
C. 订单和审计记录都回滚，因为事务中只要出现过运行时异常就必须回滚  
D. 订单回滚，审计记录提交，因为 `inner()` 的事务注解未生效，其操作自动提交

**我的答案：C。正确答案：B。**

### 解析：异常被捕获，为什么可以提交？

本题有两个关键条件：

- `this.inner()` 是自调用，没有经过内层事务代理，因此 `REQUIRES_NEW` 没有生效。
- 异常被 `outer()` 捕获，没有传播出外层方法，且没有其他回滚标记。

```text
外部通过代理调用 outer()
          ↓
开启外层事务
          ↓
插入订单
          ↓
this.inner()：未经过内层事务代理
          ↓
插入审计记录，参与外层已有事务
          ↓
抛出运行时异常
          ↓
outer() 捕获异常，不再抛出
          ↓
outer() 正常返回，事务未被标记为 rollback-only
          ↓
订单、审计记录一起提交
```

**事务不是“出现过运行时异常就一定回滚”。**需要判断异常是否被事务拦截器捕获并触发回滚规则，或者事务是否因其他原因被标记为只能回滚。

### 补充疑问：REQUIRED 内层抛异常，外层捕获后不是仍然回滚吗？

**我的疑问：**

> 我是根据这条规则作答的：REQUIRED 内层方法抛异常，外层捕获后依然回滚，因为共享事务已经被标记为 rollback-only。难道这句话不对吗？

**解答：**

这条规则的核心正确，但需要补充前提：**内层方法的事务注解确实生效，符合回滚规则的异常经过了内层事务代理。**

另外，`REQUIRED` 更准确地说是“加入已有事务”，不要与 `NESTED`（基于保存点的嵌套事务）混淆。

**场景一：经过代理调用 REQUIRED 方法**

外层调用另一个 Bean 的事务方法：

```java
@Transactional
public void outer() {
    orderMapper.insert(order);

    try {
        innerService.inner(); // 经过内层事务代理
    } catch (IllegalStateException e) {
        // 捕获异常，不再抛出
    }
}
```

另一个 Bean 中：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void inner() {
    auditMapper.insert(audit);
    throw new IllegalStateException("模拟失败");
}
```

在默认事务配置下：

```text
inner() 抛出运行时异常
          ↓
异常经过内层事务代理
          ↓
代理按回滚规则，将共享事务标记为 rollback-only
          ↓
异常继续传播，被 outer() 捕获
          ↓
outer() 正常返回，尝试提交
          ↓
共享事务已经只能回滚
          ↓
回滚，并通常抛出 UnexpectedRollbackException
```

**外层捕获异常时，回滚标记已经设置好了，捕获操作不会撤销这个标记。**

出处：[Spring 官方 REQUIRED 传播机制说明](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)

**场景二：第 36 题的自调用**

```java
this.inner(); // 没有经过内层事务代理
```

执行过程：

```text
inner() 抛出运行时异常
          ↓
没有内层事务代理拦截、设置 rollback-only
          ↓
异常直接被 outer() 捕获
          ↓
outer() 正常返回
          ↓
外层事务代理看到正常结束，且无回滚标记
          ↓
提交
```

**即使把第 36 题中 `inner()` 的注解改为 `REQUIRED`，在相同自调用条件下，结果也一样。**

出处：[Spring 官方事务注解与自调用说明](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)

**两种情况对比**

| 对比点 | 经过代理调用 REQUIRED 方法 | 第 36 题：自调用 |
|---|---|---|
| 内层调用经过事务代理吗？ | 是 | 否 |
| 内层事务注解生效吗？ | 是 | 否 |
| 内层异常经过事务代理吗？ | 是 | 否 |
| 是否因该异常设置回滚标记？ | 默认回滚规则下会 | 本题不会 |
| 外层捕获异常并正常结束后 | 仍然回滚 | 提交 |

**完善后的记忆规则：**

> 在默认事务配置下，内层 REQUIRED 方法通过事务代理加入外层事务。若符合回滚规则的异常传播出内层方法，内层事务代理会将共享事务标记为 rollback-only。即使外层随后捕获异常，也无法正常提交。自调用绕过内层事务代理时，不能直接套用这条规则。

## 37．【SQL：COUNT 与外连接】

有以下数据：

```text
customer
id | name
1  | 张三
2  | 李四

orders
id  | customer_id
101 | 1
102 | 1
```

其中 `orders.id` 是非空主键。执行：

```sql
SELECT c.id,
       COUNT(*) AS row_count,
       COUNT(o.id) AS order_count
FROM customer c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id;
```

客户李四对应的结果是什么？

A. `id=2，row_count=0，order_count=0`  
B. `id=2，row_count=1，order_count=1`  
C. 不会返回李四，因为他没有订单  
D. `id=2，row_count=1，order_count=0`

## 38．【Spring Cloud：限流、熔断与重试】

某服务调用库存接口。库存接口近期持续慢响应。系统已经配置调用超时，但开发人员又增加了以下策略：

```text
每次调用超时后，立即重试两次。
所有重试都继续访问同一个库存服务。
```

在库存服务已经过载、大量请求同时超时的情况下，下列判断最合理的是？

A. 重试可能放大下游压力；应限制重试次数与节奏，并结合熔断降级，避免持续调用不健康的依赖  
B. 配置了调用超时，就不会再占用任何下游资源，因此立即重试不会加重压力  
C. 只要增加调用方线程池大小，就可以消除下游慢响应导致的故障扩散  
D. 熔断与重试不能同时存在，因此只能保留重试，不能再配置熔断

## 39．【消息可靠性：确认与重复消费】

某消费者使用手动确认模式。处理流程如下：

```text
① 收到消息
② 在数据库事务中完成业务更新，并提交成功
③ 向消息队列发送消费确认
```

消费者在②完成后、③执行前崩溃。消息队列随后将这条未确认消息重新投递。若业务要求避免重复产生效果，最合适的设计是？

A. 改为先确认消息再处理业务，即可同时避免重复处理和业务丢失  
B. 保持原流程即可，因为数据库提交后，消息队列会自动知道业务已经成功  
C. 使用稳定的消息唯一标识，将去重记录与业务更新放在同一数据库事务中；重新投递时识别已完成的消息  
D. 将最近处理过的消息标识保存在消费者进程内存中，重启后仍可可靠去重

## 40．【MyBatis：二级缓存与回滚】

某 Mapper 已开启二级缓存，使用默认更新刷新配置。共享二级缓存中已经保存用户 1 的名字 `"旧名字"`。

随后一个手动管理事务的 `SqlSession` 执行：

```java
mapper.updateName(1L, "新名字");
session.rollback();
session.close();
```

假设更新成功执行但最终回滚，期间没有查询、其他事务、缓存淘汰或自定义缓存行为。之后，一个新会话执行此前缓存过的相同用户查询。

下列说法正确的是？

A. 新会话会命中二级缓存并得到“新名字”，因为更新语句执行时会直接修改缓存对象  
B. 新会话可以命中原有二级缓存并得到“旧名字”，因为回滚放弃了本次待执行的二级缓存清理  
C. 新会话一定访问数据库，因为 UPDATE 一执行就立即清空了共享二级缓存，回滚不能撤销  
D. 新会话不能使用二级缓存，因为任意一个会话关闭都会清空该 Mapper 的共享缓存

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 36 | C | B | ✗ |
| 37 | D | D | ✓ |
| 38 | A | A | ✓ |
| 39 | C | C | ✓ |
| 40 | B | B | ✓ |

**合计：答对 4/5，正确率 80%。**

# 专项练习：41～50 题

以下均为单选题。

## 41．【Spring：REQUIRED 与回滚标记】

两个方法位于不同的 Spring Bean 中，调用均经过事务代理。使用同一个事务管理器和默认回滚规则：

```java
@Transactional
public void outer() {
    orderMapper.insert(order);
    try {
        auditService.saveAudit();
    } catch (IllegalStateException e) {
        // 记录日志，不再抛出
    }
}
```

```java
@Transactional(propagation = Propagation.REQUIRED)
public void saveAudit() {
    auditMapper.insert(audit);
    throw new IllegalStateException("模拟失败");
}
```

假设两次插入成功，没有其他特殊配置。外层结束时，最符合默认行为的是？

A. 订单提交，审计记录回滚，因为每个事务方法可以独立回滚  
B. 两项都提交，因为异常没有传播出 `outer()`  
C. 两项都回滚，外层尝试提交时通常抛出 `UnexpectedRollbackException`  
D. 两项都回滚，但外层调用者一定不会收到异常，因为业务异常已被捕获

**我的答案：D。正确答案：C。**

### 解析：捕获了业务异常，为什么仍然会报错？

业务异常被捕获，不代表事务的回滚标记被撤销。

```text
saveAudit() 抛出 IllegalStateException
          ↓
内层事务代理将共享事务标记为 rollback-only
          ↓
外层 catch 捕获 IllegalStateException
          ↓
outer() 正常返回
          ↓
外层事务代理尝试提交，发现事务只能回滚
          ↓
执行回滚，抛出 UnexpectedRollbackException
```

**这里有两个不同的异常：**

| 异常 | 发生位置 | 是否被题目中的 catch 捕获 |
|---|---|---|
| `IllegalStateException` | 内层业务方法 | 是 |
| `UnexpectedRollbackException` | 外层方法返回后的事务提交阶段 | 否 |

### 补充问答：既然已经捕获异常，调用方还会收到异常吗？

**我的疑问：**

> 既然已经捕获异常了，这段代码还会报错吗？我在实际项目中没有遇到过这种情况。

**解答：**

**会。`outer()` 方法体结束，不代表事务已经提交；提交发生在方法体外面的事务代理中。**

可以将代理的工作简化理解为：

```java
// 示意代码：Spring 事务代理
public void 调用outer() {
    开启事务();

    target.outer(); // 业务代码，里面的 catch 已经执行完

    提交事务();     // 发现 rollback-only，回滚并抛出新异常
}
```

`outer()` 内部的 `catch` 无法捕获该方法返回以后、代理提交事务时发生的异常。

例如：

```java
@Transactional
public void outer() {
    orderMapper.insert(order);

    try {
        auditService.saveAudit();
    } catch (IllegalStateException e) {
        System.out.println("① 已捕获业务异常");
    }

    System.out.println("② outer 方法体执行结束");
}
```

调用方：

```java
try {
    orderService.outer();
    System.out.println("③ 调用成功");
} catch (UnexpectedRollbackException e) {
    System.out.println("④ 提交失败，事务实际已回滚");
}
```

在题目条件下，输出是：

```text
① 已捕获业务异常
② outer 方法体执行结束
④ 提交失败，事务实际已回滚
```

**③ 不会输出。**要等事务代理也处理成功，这次调用才算正常返回。

实际项目中没有遇到，可能是以下条件不同：

| 实际情况 | 与第 41 题的区别 |
|---|---|
| 同类中使用 `this.inner()` | 没经过内层事务代理；对应第 36 题 |
| 异常在内层方法里面就被捕获 | 异常没有传播到内层事务代理，不会由该代理因这个异常触发回滚 |
| 内层使用生效的 `REQUIRES_NEW` | 内外是独立事务，内层回滚不直接将外层标记为只能回滚；外层若捕获异常，可以继续提交 |
| 更上层或全局异常处理器捕获了异常 | 异常实际发生了，但可能被统一转换成响应，因此调用者没有直接看到异常堆栈 |

前两种情况还需要排除数据库错误、手动设置 `rollback-only` 等其他回滚原因；**不能理解为“只要捕获异常，就一定能提交”。**

Spring 官方说明：内层将共享事务标记为只能回滚后，外层尝试提交时需要收到 `UnexpectedRollbackException`，避免调用者误以为数据已经提交。

出处：[Spring 官方事务传播说明](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)

**记忆：捕获业务异常，不会撤销已经设置的 rollback-only；提交阶段还可能抛出新异常。**

## 42．【Spring：REQUIRES_NEW 与独立提交】

两个方法位于不同的 Spring Bean 中，调用均经过事务代理。数据库连接充足，两项操作不存在锁冲突或外键依赖：

```java
@Transactional
public void createOrder() {
    orderMapper.insert(order);
    auditService.saveAudit();
    throw new IllegalStateException("订单处理失败");
}
```

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    auditMapper.insert(audit);
}
```

审计方法正常完成，异常未被外层捕获。最终结果是什么？

A. 订单回滚，审计记录提交  
B. 订单和审计记录都回滚，因为异常最终离开了最外层方法  
C. 订单提交，审计记录回滚，因为新事务优先承担回滚  
D. 两项都提交，因为内部新事务提交会同时提交外层事务

## 43．【SQL：NOT IN 与 NULL】

表中数据如下：

```text
customer
id
1
2
3

blocked_customer
customer_id
2
NULL
```

执行：

```sql
SELECT id
FROM customer
WHERE id NOT IN (
    SELECT customer_id
    FROM blocked_customer
);
```

在标准 SQL 的 NULL 比较语义下，结果是什么？

A. 返回 `1、3`，因为 NULL 会被 `NOT IN` 自动忽略  
B. 返回 `1、2、3`，因为子查询有 NULL，整个条件被忽略  
C. 语法报错，因为 `NOT IN` 子查询不能返回 NULL  
D. 不返回任何行，因为没有一行的条件计算结果为 TRUE

**我的答案：A。正确答案：D。**

### 解析：NOT IN 不会自动忽略 NULL

```sql
id NOT IN (2, NULL)
```

可以理解为：

```sql
id <> 2 AND id <> NULL
```

SQL 中，与 `NULL` 使用普通比较运算，结果是 **UNKNOWN（未知）**。

| id | `id <> 2` | `id <> NULL` | AND 的结果 | WHERE 是否保留 |
|---|---|---|---|---|
| 1 | TRUE | UNKNOWN | UNKNOWN | 否 |
| 2 | FALSE | UNKNOWN | FALSE | 否 |
| 3 | TRUE | UNKNOWN | UNKNOWN | 否 |

**WHERE 只保留条件为 TRUE 的行**，因此本题不返回任何行。

如果希望查询未被拉黑的客户，可以过滤子查询中的 NULL：

```sql
SELECT id
FROM customer
WHERE id NOT IN (
    SELECT customer_id
    FROM blocked_customer
    WHERE customer_id IS NOT NULL
);
```

也可以使用关联的 `NOT EXISTS`：

```sql
SELECT c.id
FROM customer c
WHERE NOT EXISTS (
    SELECT 1
    FROM blocked_customer b
    WHERE b.customer_id = c.id
);
```

在本题数据下，这两种改写都返回客户 `1、3`。

## 44．【SQL：同时统计全部订单与已支付订单】

需要返回**所有客户**，并统计每个客户的：

- 全部订单数；
- 已支付订单数。

没有订单的客户，两项都应为 0。`orders.id` 是非空主键。

哪条 SQL 满足要求？

**A.**
```sql
SELECT c.id,
       COUNT(o.id) AS total_count,
       COUNT(o.id) AS paid_count
FROM customer c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.status = 'PAID'
GROUP BY c.id;
```

**B.**
```sql
SELECT c.id,
       COUNT(o.id) AS total_count,
       SUM(CASE WHEN o.status = 'PAID' THEN 1 ELSE 0 END) AS paid_count
FROM customer c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id;
```

**C.**
```sql
SELECT c.id,
       COUNT(*) AS total_count,
       SUM(CASE WHEN o.status = 'PAID' THEN 1 ELSE 0 END) AS paid_count
FROM customer c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id;
```

**D.**
```sql
SELECT c.id,
       COUNT(o.id) AS total_count,
       SUM(CASE WHEN o.status = 'PAID' THEN 1 ELSE 0 END) AS paid_count
FROM customer c
JOIN orders o ON o.customer_id = c.id
GROUP BY c.id;
```

**我的答案：D。正确答案：B。**

### 解析：统计正确，还要保证客户没有被遗漏

D 的聚合表达式正确，但 `JOIN` 默认是 `INNER JOIN`，没有订单的客户会被排除，不满足“所有客户”的要求。

B 的三个关键点：

| 写法 | 作用 |
|---|---|
| `LEFT JOIN` | 保留所有客户 |
| `COUNT(o.id)` | 只计算非 NULL 的订单编号，没有订单时得到 0 |
| `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` | 已支付订单计 1，其他情况计 0 |

| 选项 | 问题 |
|---|---|
| A | WHERE 过滤后只剩已支付订单，既会遗漏客户，也无法统计全部订单 |
| B | 正确，保留所有客户并分别统计两类数量 |
| C | `COUNT(*)` 会把无订单客户的补 NULL 行也计入，得到 1 |
| D | INNER JOIN 会遗漏没有订单的客户 |

## 45．【MyBatis：二级缓存与其他会话的一级缓存】

`UserMapper` 已开启二级缓存，一级缓存范围为 `SESSION`，查询和更新均采用默认缓存配置。初始二级缓存为空，按以下顺序执行：

```text
① A 查询用户 1，得到“旧名字”，A 不提交、不关闭
② B 更新用户 1 为“新名字”，并提交
③ A 再次执行与①相同的查询
```

A、B 是独立会话。期间没有其他查询、操作或插件干预。

下列说法正确的是？

A. A 必然得到“新名字”，因为 B 提交时会同步更新所有缓存对象  
B. A 必然访问数据库，因为清理二级缓存会同时清理各会话的一级缓存  
C. A 可以先发现二级缓存未命中，再命中自己的一级缓存，返回“旧名字”  
D. A 的查询会失败，因为 B 更新后，A 的一级缓存被标记为不可访问

## 46．【线程池：CallerRunsPolicy】

某线程池配置为：

```java
corePoolSize = 1;
maximumPoolSize = 1;
workQueue = new ArrayBlockingQueue<>(1);
rejectedExecutionHandler = new ThreadPoolExecutor.CallerRunsPolicy();
```

线程池正常运行、未关闭。主线程依次调用 `execute()` 提交任务 A、B、C。提交 C 时：

- A 正在工作线程中执行，尚未结束；
- B 仍在队列中；
- C 是一个耗时任务。

下列说法正确的是？

A. C 由提交它的主线程执行，主线程要等 C 执行结束，才从本次 `execute()` 返回  
B. C 会创建第二个工作线程执行，因为拒绝策略可以突破最大线程数  
C. C 会替换队列中的 B，B 被丢弃  
D. C 被静默丢弃，主线程立即从 `execute()` 返回

## 47．【消息可靠性：生产者确认与消费者确认】

某 RabbitMQ 应用同时启用了生产者确认和消费者手动确认。下列理解正确的是？

A. 生产者收到发布确认，就说明消费者的数据库事务已经提交  
B. 消费者发送 ACK 后，生产者才会收到对应的发布确认  
C. 启用这两类确认后，消息不会重复投递，业务不再需要幂等  
D. 两类确认关注不同阶段；生产者收到发布确认，不代表消费者已经完成业务处理

## 48．【架构设计：读写分离与读己之写】

系统采用主从异步复制。用户提交资料修改后，主库事务已成功提交，页面随即发起查询，但从库尚未同步，页面显示旧资料。

如果要求**本次修改后的紧接查询能看到这次修改**，哪种方案最直接且可靠？

A. 修改成功后清理应用缓存，紧接查询仍随机路由到任意从库  
B. 将这次紧接查询路由到主库，并在能够看到已提交修改的新事务中执行  
C. 固定等待 50 毫秒，再随机查询从库，即可保证复制完成  
D. 为从库查询增加 Redis 分布式锁，即可保证主从数据一致

## 49．【MyBatis：自定义二级缓存】

开发人员实现了一个基于 Redis 的 MyBatis `Cache`，并通过 Mapper 的 `<cache type="..."/>` 接入。

假设每个 Mapper 使用独立的缓存标识。关于自定义缓存的 `clear()` 方法，下列设计最合理的是？

A. 直接执行 Redis `FLUSHALL`，因为 MyBatis 要求清理所有 Mapper 的缓存  
B. 只清理当前线程最近查询的那个键，因为 `clear()` 的范围就是一条记录  
C. 清理该缓存实例所属的全部条目，并通过缓存标识隔离，避免删除其他 Mapper 或业务的数据  
D. 不执行任何操作，因为数据库更新后 Redis 会自动识别并删除相关查询结果

## 50．【架构设计：设计模式与扩展】

系统根据客户类型计算折扣。当前代码包含大量分支：

```java
if (type == NORMAL) {
    // 普通客户折扣
} else if (type == VIP) {
    // VIP 折扣
} else if (type == PARTNER) {
    // 合作伙伴折扣
}
```

现在希望将不同折扣算法独立封装，按客户类型选择，并让新增算法尽量不修改已有算法实现。

哪种设计最符合这一目标？

A. 使用单例模式，保证所有折扣算法只存在一个实例  
B. 使用策略模式，为不同折扣算法提供统一接口，并通过注册表等方式选择实现  
C. 使用代理模式，为原来的整个分支方法添加统一日志，但保留所有算法分支  
D. 使用模板方法模式，仅把所有折扣分支原样搬进父类的一个固定方法中，不设置可替换步骤

## 答案与作答记录

| 题号 | 我的答案 | 正确答案 | 结果 |
|---|---|---|---|
| 41 | D | C | ✗ |
| 42 | A | A | ✓ |
| 43 | A | D | ✗ |
| 44 | D | B | ✗ |
| 45 | C | C | ✓ |
| 46 | A | A | ✓ |
| 47 | D | D | ✓ |
| 48 | B | B | ✓ |
| 49 | C | C | ✓ |
| 50 | B | B | ✓ |

**合计：答对 7/10，正确率 70%。**