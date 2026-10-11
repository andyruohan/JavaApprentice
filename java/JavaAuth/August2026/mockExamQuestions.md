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