
# 1～20
2026.1024 22:08
1 D 两个修改操作，在修改之前都加了S锁，会导致死锁。
2 A $ 恰恰会造成 sql 注入。
3 B ACD均有不同程度的错误。
4 C 首先D错误，可以使用聚合函数，故A也错误，其次B表述可对可错吧，看这里的结果是不是代表聚合结果，C是准确的。
5 C @Reques tMapping ("/say/helloworld", method = RequestMethod.POST) 需要指定第一个参数是什么
6 D D最准确，实际C也是对的
7 B ERROR不是其打印日志级别
8 D D最准确，分布式流量全部到专门数据库来取，实际A多点重复也有可能，因其是双主模式
9 D Kafka的消息是broker负责，集群的同步非Zookeeper的职责，Kafka的网络连接更不是，D是，因其是Kafka的元数据信息。
10 D ABC均有不同程度的错误。
11 ？ 选不出答案，ABCD均有不同程度的错误
12 C AB项synchronized不可以，D项lock不可以
13 A A项完全不能命中索引，BD一定程度上会名字
14 C 保证多数表决
15 B uri是问号之前，但我不代表ABD什么情况下会返回
16 C where EXISTS 很难通过join表征出来吧
17 C 缺少on条件
18 D 代码明显不是递归，而是循环，C选项也有点错误，抛异常了的话不会执行
19 A 根据Spring官方的描述就是这样，相反D选项返回是单个
20 A A不能解决本质流量问题

你这次第 1～20 题的整体判断比较准确，而且已经开始从“选出答案”转向“判断每个选项在技术上是否严格成立”。

其中有几个问题需要重点纠正：

- 第 5 题：`@RequestMapping` 第一个参数可以不写 `value=`，你对 C 的判断需要调整。
- 第 6 题：不仅 D 正确，B 实际上也能去除最终结果中的重复记录；C 则不能保证去重。
- 第 8 题：D 是合理答案，但双主架构与单一 ID 分配点需要分开理解。
- 第 11 题：你认为四个选项都有问题，这个判断有道理，题目缺少一致性约束。
- 第 18 题：C、D 都有问题，只是 D 更明显。

## 一、第 1～20 题答案核对

| 题号 | 你的答案 | 建议答案          | 结论 |
| -- | ---- | ------------- | -- |
| 1  | D    | D             |    |
| 2  | A    | A             |    |
| 3  | B    | B（题目不严谨）      |    |
| 4  | C    | C             |    |
| 5  | C    | C（仅按拼写错误）     |    |
| 6  | D    | B、D           |    |
| 7  | B    | B             |    |
| 8  | D    | D             |    |
| 9  | D    | D（仅限旧版 Kafka） |    |
| 10 | D    | D             |    |
| 11 | 未确定  | 倾向 C，但存在争议    |    |
| 12 | C    | C             |    |
| 13 | A    | A             |    |
| 14 | C    | C（有前提）        |    |
| 15 | B    | B             |    |
| 16 | C    | C（有争议）        |    |
| 17 | C    | C             |    |
| 18 | D    | C、D 都存在问题     |    |
| 19 | A    | A             |    |
| 20 | A    | A（按出题意图）      |    |

下面逐题解释你有疑问的知识点。

## 二、逐题解析

### 第 1 题：共享锁升级导致死锁

你选 D，正确。

两个事务都先对同一行获取 S 锁：

事务 T4

持有 R4 的 S 锁

请求 R4 的 X 锁

等待 T2 释放 S 锁

事务 T2

持有 R4 的 S 锁

请求 R4 的 X 锁

等待 T4 释放 S 锁

两个事务相互等待，形成循环等待关系

S 锁之间兼容，但 S 锁和 X 锁不兼容。

当两个事务都持有 S 锁，又都尝试升级成 X 锁时，就可能发生死锁。

A 仅说明另一个事务也获取 S 锁，如果它不请求冲突锁，并不足以形成死锁。

这里题干称“未升级为 X 锁”，更准确地说应是“尝试升级为 X 锁，但无法成功获取”。

### 第 2 题：MyBatis 的 `#{}` 和 `${}`

你选 A，正确。

| 对比       | `#{}`         | `${}`       |
| -------- | ------------- | ----------- |
| 处理方式     | 参数绑定，通常使用 `?` | 字符串直接替换     |
| SQL 注入防护 | 有效防止值参数注入     | 存在 SQL 注入风险 |
| 常见场景     | WHERE 条件值     | 动态表名、列名等标识符 |

例如：

```
SELECT * FROM user WHERE name = #{name}
```

会通过预编译参数绑定处理。

需要注意 B 的描述也不严格：`#{}` 并不是简单给字符串添加双引号，而是使用 JDBC 参数绑定，具体数据库类型由 TypeHandler 处理。

### 第 3 题：Kafka 减少分区

你选 B，符合出题意图，但原题说法不准确。

Kafka 常规管理 API 支持增加分区数量，不支持直接减少已有 Topic 的分区数量。

若要减少分区，通常需要重建 Topic，并重新迁移消息。

这样可能引起消息顺序、偏移量、数据迁移等问题。

所以 B 说“删除分区会导致数据不一致、消息乱序”，属于一种风险描述，而不是 Kafka 提供了删除单个分区的标准操作。

A、C、D 同样存在不同程度的错误，这道题不适合严格单选。

### 第 4 题：HAVING

你选 C，正确。

`HAVING` 是对分组结果进行筛选的条件子句，可以使用聚合函数。

```
SELECT department_id, COUNT(*)
FROM employee
GROUP BY department_id
HAVING COUNT(*) > 10;
```

B 说“在一个结果返回之后起作用”不准确。逻辑上 HAVING 在分组和聚合之后、最终结果输出之前进行筛选。

### 第 5 题：Spring MVC 的 `@RequestMapping`

你选 C，但是你的理由不正确。

下面两种写法是等价的：

```
@RequestMapping(
    "/say/helloworld",
    method = RequestMethod.POST
)
```

```
@RequestMapping(
    value = "/say/helloworld",
    method = RequestMethod.POST
)
```

原因是 `@RequestMapping` 的 `value` 属性在 Java 注解中有特殊地位。

当指定 `value` 属性时，可以省略属性名称。

因此，C 只有在题目字面写成 `@Reques tMapping`（中间多了空格）的情况下，才因注解名称拼写错误而不合法。

如果是录入题目时产生的空格，A、B、C、D 都是合法 Mapping 配置。

### 第 6 题：UNION 去重

这是你需要纠正的重点。

你认为 D 正确、C 也正确。

实际上：

B 和 D 均可以得到去重结果，C 不一定可以。

假设：

```
people:   张三、张三
member:   李四
employee: 张三
```

来看四个选项。

| 选项 | 运算表达式                 | 最终结果是否去重 |
| -- | --------------------- | -------- |
| A  | UNION ALL + UNION ALL | 否        |
| B  | UNION ALL + UNION     | 是        |
| C  | UNION + UNION ALL     | 否        |
| D  | UNION + UNION         | 是        |

原因是集合运算按出现顺序进行组合，而 `UNION` 会对其参与运算的结果去重。

B 可以理解为：

```
(
    SELECT name FROM people
    UNION ALL
    SELECT mname FROM member
)
UNION
SELECT ename FROM employee;
```

最后一次 `UNION` 会对最终合并结果去重。

但是 C：

```
(
    SELECT name FROM people
    UNION
    SELECT mname FROM member
)
UNION ALL
SELECT ename FROM employee;
```

最后一次 `UNION ALL` 可能重新引入重复值。

例如 `employee` 中也存在张三，最终会出现两次张三。

因此：

- D 是最直观的标准答案。
- B 也能完成去重。
- C 不能保证结果没有重复值。

另外，原题建表名称写的是 `eployee`，SQL 中却使用 `employee`。如果不是笔误，四个选项还会存在表名错误。

### 第 7 题：Feign 日志级别

你选 B，正确。

Feign 默认提供四种日志级别：

| 日志级别    | 含义                    |
| ------- | --------------------- |
| NONE    | 不记录日志，默认级别            |
| BASIC   | 记录请求方法、URL、响应状态码及耗时   |
| HEADERS | 在 BASIC 基础上增加请求头和响应头  |
| FULL    | 记录请求头、请求体、响应头、响应体及元数据 |

没有 `ERROR` 这个标准日志级别，因此选择 B。

### 第 8 题：MySQL 双主 ID 生成方案

你选 D，符合题意。

题目描述存在一个专门用于记录 `MaxId` 的数据库，所有业务都需要访问它获取新的 ID。

这会形成集中式 ID 分配瓶颈：

```
业务系统 A ──┐
业务系统 B ──┼──→ ID 生成数据库
业务系统 C ──┘
```

如果只有一个可写入口，那么即使底层部署双主，也不意味着 ID 分配天然实现了负载分担。

你担心双主会产生重复 ID，也有道理。

例如两个 MySQL 主节点如果都独立从 100 开始分配 ID，且没有协调机制，就可能重复。

MySQL 双主自增 ID 的一种常见配置是：

| 节点      | auto_increment_increment | auto_increment_offset |
| ------- | ------------------------ | --------------------- |
| MySQL A | 2                        | 1                     |
| MySQL B | 2                        | 2                     |

这样 A 生成奇数 ID，B 生成偶数 ID，降低双主自增 ID 冲突风险。

但这套方案仍需要解决节点故障、复制一致性、分配能力和扩容问题。

因此 D 是题目的主要答案，A 则是某些双主配置下可能存在的额外风险。

### 第 9 题：Kafka 与 ZooKeeper

你选 D，适用于旧版 ZooKeeper 模式 Kafka。

但是这里有一个非常重要的版本变化：

Kafka 4.0 起已移除 ZooKeeper 模式，只支持 KRaft。&#x20;

[image](https://www.google.com/s2/favicons?domain=https://kafka.apache.org\&sz=32)

Apache Kafka

+1



旧版 ZooKeeper 模式下：

- ZooKeeper 负责存储和协调部分集群元数据。
- Topic、Partition 相关元数据和集群协调工作涉及 ZooKeeper。
- 消息实际数据存储在 Broker，而不是 ZooKeeper。
- 分区副本同步主要由 Broker 完成。

不过，更严谨地说，ZooKeeper 不是直接负责所有分区管理操作；Kafka Controller 承担分区 Leader 选举等协调工作。

现在使用 KRaft：

```
Kafka Broker
      ↓
KRaft Controller Quorum
      ↓
元数据管理与 Controller 选举
```

所以这道题可以选 D，但建议在题库旁注明“仅适用于旧版 Kafka ZooKeeper 架构”。

### 第 10 题：MyBatis 整合 Redis 缓存

你选 D，正确。

MyBatis 本身提供 `Cache` 接口，可以通过自定义缓存实现把二级缓存存储到 Redis。

核心接口类似：

```
public interface Cache {

    String getId();

    void putObject(Object key, Object value);

    Object getObject(Object key);

    Object removeObject(Object key);

    void clear();

    int getSize();
}
```

例如：

```
public class RedisCache implements Cache {
    // 实现 Cache 接口
    // 调用 Redis 进行数据存取
}
```

然后在 Mapper XML 中配置：

```
<cache type="com.example.cache.RedisCache"/>
```

有一个术语需要纠正：是实现 `Cache` 接口，而不是“重写 Cache 类”。

此外，这种接入方式并不自动解决多实例下所有缓存一致性问题，尤其是跨 Mapper 更新、事务提交时机和 Redis 缓存失效等场景。

## 三、第 11～20 题重点解析

### 第 11 题：库存扣减设计

你说 A、B、C、D 都有不同程度的问题，我同意你指出的设计风险，但需要区分“方案有风险”和“方案本身不合理”。

| 方案                         | 是否可行  | 主要风险            |
| -------------------------- | ----- | --------------- |
| A. 先数据库后 Redis             | 可行    | 数据库提交成功但缓存更新失败  |
| B. 异步写改同步写                 | 可行    | 延迟、吞吐量可能受到影响    |
| C. Redis 预扣 + 同步数据库 + 失败回滚 | 可行但复杂 | 并发补偿、缓存回滚与一致性   |
| D. Redis 预扣 + MQ 异步数据库扣减   | 可行但复杂 | 消息丢失、重复消费、最终一致性 |

这里最重要的是：D 并非错误架构。

实际上，高并发库存扣减经常采用 Redis 预扣减、消息队列异步落库的设计。

但必须解决可靠投递、幂等、异常补偿、库存对账等问题。

C 的风险在于失败补偿不一定能简单地通过 Redis 加库存完成。

所以如果考试强制单选，我更倾向于 C，但题目本身不足以推出唯一正确答案。

### 第 12 题：synchronized 与 Lock

你选 C，正确。

- A：`Lock.tryLock()` 可以尝试获取锁并返回结果，`synchronized` 没有对应机制。
- B：`lockInterruptibly()` 可以让等待锁的线程响应中断，普通 `synchronized` 锁获取等待不具备这种中断机制。
- C：两者都可以实现临界区的互斥访问，保障其中复合操作的原子性。
- D：`synchronized` 在离开同步块时自动释放锁；显式 `Lock` 通常需要在 `finally` 中调用 `unlock()`。

### 第 13 题：SQL 函数与索引

你选 A，正确，但“完全不能命中索引”不够准确。

```
WHERE substr(phoneno, 1, 3) = '321'
```

对普通 `phoneno` B-tree 索引而言，数据库通常无法直接利用该索引进行前缀范围定位。

但有例外：

- Oracle 可以创建函数索引。
- 某些数据库支持表达式索引。
- 优化器有时可能扫描覆盖索引，而不是扫描整张表。

因此，A 通常最慢，但并不意味着它绝对无法使用任何索引。

B：

```
WHERE phoneno >= '321'
  AND phoneno < '321A'
```

对于全部由数字组成、采用常见字符排序规则的号码，该条件可以作为前缀范围筛选。

D：

```
WHERE phoneno LIKE '321%'
```

通常也适合利用前缀索引范围扫描。

要注意具体执行计划还取决于数据库类型、排序规则、统计信息以及索引结构。

### 第 14 题：Kafka 最少服务器数量

你选 C，正确。

按照常见 Kafka 高可用集群的预期，3 台服务器能够支持 1 台故障时维持元数据多数派。

但“保证多数表决”主要是 KRaft Controller Quorum 的机制，而不是 Kafka 所有分区的每一次读写都进行多数投票。

例如 KRaft：

```
Controller 1
Controller 2
Controller 3
```

三节点多数派为 2，所以允许一台 Controller 故障。

[image](https://www.google.com/s2/favicons?domain=https://kafka.apache.org\&sz=32)

Apache Kafka



还需要区分：

- Controller Quorum：负责元数据一致性。
- Partition Replica：负责业务消息副本。
- ISR：参与副本同步及可靠性控制。

生产环境还需要正确配置复制因子、`min.insync.replicas` 和 Producer `acks`，不能只依赖服务器数量。

### 第 15 题：getRequestURI()

你选 B，正确。

请求：

```
GET /day06/response1?1339484005562 HTTP/1.1
```

调用：

```
request.getRequestURI()
```

返回：

```
/day06/response1
```

几个方法可以一起记忆：

| 方法                 | 返回内容                 |
| ------------------ | -------------------- |
| `getRequestURI()`  | 不含域名和查询参数的请求 URI     |
| `getRequestURL()`  | 包含协议、域名、端口及请求路径的 URL |
| `getQueryString()` | `?` 后面的原始查询字符串       |
| `getContextPath()` | Web 应用上下文路径          |
| `getServletPath()` | Servlet 映射对应的路径部分    |

假设请求为：

```
http://localhost:8080/day06/response1?id=4
```

那么 `getRequestURI()` 返回：

```
/day06/response1
```

`getRequestURL()` 返回类似：

```
http://localhost:8080/day06/response1
```

而 `getQueryString()` 返回：

```
id=4
```

原题 A、C 的完整地址还与实际请求的 Host 不一致；C、D 则对应 Referer 所指页面，而不是当前请求路径。

### 第 16 题：子查询和 JOIN

你选 C，符合题库意图。

你说 `WHERE EXISTS` 很难通过 JOIN 表达出来，这个理解需要稍微调整。

例如：

```
SELECT *
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM customer c
    WHERE c.id = o.customer_id
);
```

在 `customer.id` 唯一的情况下，可以改写为：

```
SELECT o.*
FROM orders o
JOIN customer c
    ON c.id = o.customer_id;
```

如果 JOIN 的右表存在多条匹配记录，就可能导致左表重复，因此需要考虑去重或者改用半连接语义。

EXISTS 并非不能通过 JOIN 改写，但需要保证结果语义相同。

同时，A 声称所有 JOIN 都能用 IN、EXISTS 等子查询实现相同查询目标，也不够严谨。

尤其在返回两表所有字段、保留重复行、外连接补 NULL 等场景中，需要更复杂的改写。

所以这道题依然属于有歧义的单选题。

### 第 17 题：FULL OUTER JOIN

你选 C，正确。

原 SQL：

```
SELECT a.id, b.id, a.name, b.name
FROM a FULL OUTER JOIN b
WHERE a.id IS NOT NULL OR b.id IS NULL;
```

在 Oracle、PostgreSQL 等常见数据库中，`FULL OUTER JOIN` 缺少 `ON` 或 `USING` 连接条件，原 SQL 会报错。

因此选 C。

### 第 18 题：Spring MVC 拦截器

你选 D，符合最明显的出题意图。

但你指出 C 有问题，也是正确的。

C：

> applyPreHandle 方法执行成功后，就会调用 applyPostHandle 方法。

这个表述不一定成立。

例如 Controller 执行时抛出异常，正常的 `postHandle` 流程可能不会执行。

而 D：

> 拦截器是递归调用 applyPreHandle 方法来拦截请求。

源码中使用 `for` 循环，显然不是递归调用。

因此：

- D：明确错误。
- C：也存在过度绝对化的问题。

如果题目只有一个答案，优先选 D，但不能说 C 在所有情况下都正确。

### 第 19 题：Flux 与 Mono

你选 A，符合题库答案。

```
Flux<String> people =
    request.bodyToFlux(String.class);
```

`Flux` 表示能够异步发布零个到多个元素的响应式流。

而：

```
Mono<String> string =
    request.bodyToMono(String.class);
```

`Mono` 表示零个或一个元素。

但严格来说：

`bodyToFlux()` 只是把请求体读取成响应式流，并不是直接返回 HTTP 响应。

如果要返回多个响应式元素，还需要把 `Flux` 交给响应写入流程。

### 第 20 题：结算页服务降级

你选 A，按题库意图正确，但你的理由需要修改。

你说：

> A 不能解决本质流量问题。

实际上，A 可以解决一部分流量压力问题。

例如：

- 识别爬虫后提前拒绝请求。
- 对高风险用户施加更严格的访问限制。
- 减少低优先级流量对核心资源的占用。

这类策略能够降低系统整体流量。

但题目强调的是针对 62 个依赖接口的降级方式。

因此 A 更偏向入口流量治理，而 B、C、D 更直接涉及依赖服务的重要性、限流和资源分配。

所以 A 是最符合题意的答案，但不是说 A 本身是一种无效的降级措施。

## 四、最终建议

这组题里最值得记住的，不是单纯的答案，而是以下几个容易产生错误理解的地方：

| 题号 | 你需要修正或加强的理解                          |
| -- | ------------------------------------ |
| 5  | Java 注解的 `value` 属性可以省略名称            |
| 6  | 最后一次 UNION 可以对之前 UNION ALL 的结果整体去重   |
| 8  | 双主高可用不等于 ID 生成不存在集中式瓶颈               |
| 9  | Kafka 4.0 已经移除 ZooKeeper 模式          |
| 11 | 架构方案存在一致性风险，不等于方案必然不合理               |
| 13 | 函数作用于索引列通常影响普通索引的范围定位，但不是绝对不能使用索引    |
| 16 | EXISTS 可以在满足语义条件时通过 JOIN 或半连接等方式改写   |
| 18 | `preHandle` 成功不意味着 `postHandle` 一定执行 |
| 20 | 入口流量降级和依赖服务降级的侧重点不同                  |

整体评价： 你大部分答案都符合常见题库预期。建议把第 3、5、6、11、16、18 题标为有争议或需要注意特殊条件的题目；尤其是第 6 题，你对 C 的推断需要明确改成“不能保证去重”。

# 21～40
2026.10.24 14:40
21 C 有二级缓存后，会有相应加速。对一级二级缓存的加载顺序，需再加深记忆。
22 C 放到where条件里，在join时不起作用。
23 D zip不是redis字符串的内部编码。redis的几种内部编码，需加深记忆。
24 C没有排序，A没有on条件，D缺少group by。
25 D 重启服务不属于组件的功能。
26 C HDFS适用于一次写入、多次读取。
27 D 每次读取的结果都不一样，不可重复读。脏读、虚读的概念理解需加深。
28 D CAP不可同时满足，最多满足两个
29 B 首先AD都需取决于具体的SQL，其次left join左边是驱动表
30 B 核心线程池是4，能否同时执行4个线程的并发，无需等待
31 D 选D但不代表A、C就是全部描述正确的，AC兼容的数据类型即可，其次C有意选择错误的顺序，但是数据类型满足，能执行union，只是说业务逻辑不对。
32 D 读写分离是为了降低读库的压力
33 C broker既可以为消费者服务，又可以为生产者服务，但它既不是消费者又不是生产者
34 C 首先AD是有效的策略，其次C使用持久化我理解是极端情况下帮助数据恢复，来提升可靠性。而C使用多个分区，我不太明白这对可靠性有什么提升，难道是其中一个分区坏了，其他分区好的，降低坏的概率，那从这个角度BC感觉都行吧
35 B 谁在前面就优先匹配谁。这里如果加上order，该如何选，order越大优先级越高，还是越小优先级越高，我倾向于越小优先级越高，还有就是order出现负数代表什么。
36 B SqlSessionTemplate是线程安全的，它负责找到对应的sqlSession。
37 A Sentinel可以与Dubbo整合
38 D DaemonSet不属于重启策略，但它是干什么的不记得了，其次重启策略记忆需加深。
39 A md5、md4都是哈希算法，ladp忘了是干啥的了
40 C 多租户，不同用户不同库

这次第 21～40 题，你的整体掌握情况不错，尤其是 Redis、Kafka、Spring Cloud 和线程池的基础概念。

不过，有几道题需要纠正：第 32 题应选 C，而不是 D；第 40 题不能简单认为多租户就是不同用户使用不同数据库；第 21 题本身存在多个不严谨的选项。

我先给出完整核对结果，再针对你提出疑问的知识点展开分析。

## 一、21～40 题答案核对

| 题号 | 你的答案     | 建议答案         | 判断 |
| -- | -------- | ------------ | -- |
| 21 | C        | C / D（题目有歧义） |    |
| 22 | C        | C            |    |
| 23 | D        | D            |    |
| 24 | B（按解析推断） | B            |    |
| 25 | D        | D            |    |
| 26 | C        | C            |    |
| 27 | D        | D            |    |
| 28 | D        | D            |    |
| 29 | B        | B（传统逻辑理解）    |    |
| 30 | B        | B            |    |
| 31 | D        | D            |    |
| 32 | D        | C            |    |
| 33 | C        | C            |    |
| 34 | C        | C            |    |
| 35 | B        | B            |    |
| 36 | B        | B            |    |
| 37 | A        | A            |    |
| 38 | D        | D            |    |
| 39 | A        | A            |    |
| 40 | C        | C（依赖场景）      |    |

第 24 题你没有直接写答案字母，但解析排除了 A、C、D，因此按 B 统计。

## 二、重点纠错与知识点深化

### 第 21 题：MyBatis 一级缓存与二级缓存

你选 C，符合一种常见的出题预期，但需要纠正两个知识点。

1\. 一级、二级缓存的查询顺序

执行 SQL 查询

一级缓存（SqlSession 级别）

命中则直接返回

二级缓存（Mapper 命名空间级别）

命中则返回并填充一级缓存

数据库查询

注意：

- 一级缓存默认启用，作用域通常是 `SESSION`。
- 二级缓存的全局开关 `cacheEnabled` 默认是 `true`，但具体 Mapper 的二级缓存默认并未配置。
- XML Mapper 通常需要添加 `<cache/>`，才会真正启用该命名空间的二级缓存。

  [image](https://www.google.com/s2/favicons?domain=https://mybatis.org\&sz=32)

  Configuration – mybatis

  +1



2\. 为什么原题不严谨？

B 说必须在 `settings` 中配置开启二级缓存，并不准确，因为全局开关默认已经为 `true`。

C 说开启二级缓存后，跨 Session 相同查询不会加快执行，也不准确，因为缓存命中时通常可以避免访问数据库。

D 说二级缓存默认不开启，可以理解为 Mapper 层面默认不配置二级缓存，因此通常认为正确。

所以 B、C 都存在问题，如果是单选考试，可以按题库预期记 C，但从技术准确性看，这道题不够严谨。

### 第 22 题：分布式 SQL 中 JOIN 和 WHERE

你选 C，结论符合题目预期，但你的解释“放到 WHERE 条件里，在 JOIN 时不起作用”不完全正确。

对于 `INNER JOIN`，下面两种写法通常在语义上等价：

```
SELECT *
FROM a
JOIN b ON a.id = b.id;
```

```
SELECT *
FROM a, b
WHERE a.id = b.id;
```

数据库优化器一般能够识别连接条件，并进行谓词下推和执行计划优化。

因此不是 WHERE 条件不起作用，而是 把连接条件从 ON 移到 WHERE，并不能作为可靠的 SQL 性能优化手段。

对于 `LEFT JOIN`，ON 和 WHERE 更不能随意替换，因为可能改变查询结果。

### 第 23 题：Redis 内部编码

你选 D，正确。

这部分可以按 Redis 版本和数据类型一起记忆。

| 数据类型   | 常见内部编码                          |
| ------ | ------------------------------- |
| String | int、embstr、raw                  |
| List   | quicklist                       |
| Hash   | listpack、hashtable              |
| Set    | intset、hashtable、listpack（较新版本） |
| ZSet   | listpack、skiplist               |

特别注意，早期版本还使用过 `ziplist`、`linkedlist`、`zipmap` 等编码，不能将不同 Redis 版本的实现混为一谈。

[image](https://www.google.com/s2/favicons?domain=https://redis.io\&sz=32)

Docs



对于 String：

- `int`：可用整数形式存储的字符串。
- `embstr`：较短的字符串，一次内存分配存储对象及内容。
- `raw`：一般字符串编码。

所以 `zip` 不是 String 的内部编码。

### 第 27 题：数据库事务四类现象

你选 D，正确。

| 现象     | 核心特征                       |
| ------ | -------------------------- |
| 脏读     | 读到了其他事务尚未提交的数据             |
| 不可重复读  | 同一事务两次读取同一行，字段值发生变化        |
| 幻读（虚读） | 同一事务重复查询某个条件，符合条件的记录集合发生变化 |
| 丢失更新   | 并发更新使其中一个更新结果被覆盖           |

例如：

事务 A 第一次查询某行，余额为 100；事务 B 更新为 200 并提交；事务 A 第二次查询同一行，余额变成 200。

这就是不可重复读。

如果事务 B 插入了一行符合查询条件的新记录，使事务 A 第二次查询多出一行，通常属于幻读。

记忆方法：不可重复读看已有行的值变化，幻读看符合条件的行集合变化。

### 第 28 题：CAP 定理

你选 D，正确，但需要补充一个容易考到的细节。

CAP 分别表示：

- C（Consistency）：一致性，通常指线性一致性。
- A（Availability）：可用性，每个到达正常节点的请求最终都能得到非错误响应。
- P（Partition Tolerance）：分区容错性，网络分区出现时仍能按设计运行。

CAP 不是简单地让系统从 C、A、P 中任选两个。

准确理解：当网络分区真正发生时，分布式系统无法同时保证 C 和 A。

没有网络分区时，同时满足一致性和可用性是可以的。因此 D 所说的“大部分系统最多同时满足两个”只是流行的简化说法。

### 第 29 题：驱动表与被驱动表

你选 B，按照传统 `LEFT JOIN` 的逻辑理解是正确的。

```
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id;
```

逻辑上：

- A 是保留全部记录的左表，通常称驱动表。
- B 是右表，通常称被驱动表。
- 对于普通嵌套循环外连接，通常先访问 A，再匹配 B。

不过，现代优化器可能在满足条件时把外连接转换为内连接，进而重新安排连接顺序。因此不能脱离执行计划，把 SQL 左右位置永远等同于物理执行顺序。

[image](https://www.google.com/s2/favicons?domain=https://dev.mysql.com\&sz=32)

MySQL Developer Zone

+1



### 第 30 题：线程池执行顺序

你选 B，正确。

这里需要区分 线程创建数量 和 实际并发执行数量。

```
corePoolSize = 4;
maximumPoolSize = 8;
queueCapacity = 10;
```

对于新初始化的线程池，连续提交 4 个任务时，通常会创建 4 个核心线程，即使前面的任务已经执行完成，也可能继续创建核心线程，直到核心线程数达到 4。

因此四个任务可能分别由：

```
thread-1
thread-2
thread-3
thread-4
```

执行。

但打印顺序不确定，例如：

```
thread-3
thread-1
thread-4
thread-2
```

也有可能。核心线程数为 4 不意味着任何时刻都有 4 个任务在同时运行。

### 第 31 题：UNION

你选 D，正确，你对 C 的理解也基本到位。

`UNION` 的核心要求是：

1. 每条 SELECT 的列数一致。
2. 对应位置的列具有兼容的数据类型。
3. 根据位置进行列匹配，而不是根据列名匹配。

例如：

```
SELECT id, name FROM user_a
UNION
SELECT name, id FROM user_b;
```

如果对应位置的数据类型能够转换，有些数据库可能允许执行，但业务语义通常不正确。

此外，`UNION` 默认去重，`UNION ALL` 不去重。在无须去重时，优先考虑 `UNION ALL`。

### 第 32 题：数据库读写分离

需要纠正

你选 D，正确答案应为 C。

你的解释是“读写分离是为了降低读库的压力”，这里恰好反了。

应用服务

写请求

主库

写入 + 复制源

读请求

从库 1 / 从库 2

分担读流量

主库通过复制机制向从库同步数据

读写分离的本质是：

把一部分读请求从主库转移到从库，降低主库读负载，而不是使用多个写库分摊写压力。

所以：

- A：正确，经典的一主多从架构。
- B：作为设计目标有一定合理性，但“线性提升”和“消除锁冲突”过于绝对。
- C：错误，多个写库分摊写压力属于多主或分片等其他架构。
- D：正确，解决读性能瓶颈是读写分离的主要目标之一。

### 第 34 题：RabbitMQ 可靠性

你选 C，正确，而且你提出了一个很好的疑问：持久化和分区是不是都能提升可靠性？

答案是：持久化是 RabbitMQ 消息可靠性的直接保障手段，分区则不是经典 RabbitMQ 队列可靠性的必要机制。

| 机制                | 主要解决的问题                   |
| ----------------- | ------------------------- |
| Publisher Confirm | 生产者确认消息是否被 Broker 接收并处理   |
| 持久化               | 降低 Broker 重启后消息丢失的风险      |
| Consumer Ack      | 消费者成功处理消息后再确认             |
| 多副本（Quorum Queue） | 降低单节点故障造成的数据丢失风险          |
| 多分区               | 主要用于数据分布和并行处理，不能直接替代可靠性机制 |

这里需要区分 Kafka 与 RabbitMQ。

Kafka 的 Partition 是核心存储及并行处理单元，可以通过 Replica 提高容错能力，但分区数增加本身不等于副本数增加。

RabbitMQ 经典队列不依赖 Kafka 式的分区架构来保证可靠性，其可靠性通常通过 Confirm、Ack、持久化和 Quorum Queue 等机制实现。

[image](https://www.google.com/s2/favicons?domain=https://www.rabbitmq.com\&sz=32)

RabbitMQ

+1



还有一个重要细节：

持久化不意味着消息永远不会丢失。

例如消息已设置持久化，但生产者尚未收到可靠确认，Broker 就出现故障，仍可能产生不确定状态。生产者需要结合 Confirm 和重试机制处理。

实际开发建议同时考虑生产者确认、队列及消息持久化、消费者手动确认、消费幂等和必要的副本机制。

### 第 35 题：Spring Cloud Gateway 的 order

你选 B，正确。你对 `order` 的判断也正确。

Spring Cloud Gateway 中，order 值越小，优先级越高。负数表示比 order=0 的路由优先级更高。

例如：

```
spring:
  cloud:
    gateway:
      routes:
        - id: server_v2
          uri: http://127.0.0.1:8002
          order: -10
          predicates:
            - Path=/api/v2,/api/v2/**

        - id: server_v1
          uri: http://127.0.0.1:8001
          order: 0
          predicates:
            - Path=/api/**
```

| order | 优先级 |
| ----- | --- |
| -100  | 最高  |
| -10   | 较高  |
| 0     | 默认  |
| 10    | 较低  |
| 100   | 最低  |

负数没有特殊业务含义，只是排序值小于 0。

需要特别注意两件事：

第一，Gateway 不是自动选择最具体的 Path。当多个路由都匹配时，通常按路由排序结果选择第一个匹配项，所以建议显式配置 `order`。

第二，原题请求路径是 `/api/v2`，与 `/api/v2/**` 的匹配行为可能受 Spring 路径匹配版本和配置影响。B 显式同时写了 `/api/v2` 与 `/api/v2/**`，所以更稳妥。

[image](https://www.google.com/s2/favicons?domain=https://blogs.vmware.com\&sz=32)

Tanzu

+2



另外，不要把 Gateway 的路由 `order` 与 Gateway Filter 的 `order` 混为一谈。两者都使用排序概念，但分别决定路由匹配优先级和过滤器执行顺序。

### 第 36 题：MyBatis 线程安全

你选 B，正确。

还有一个小纠正：题目第 3 条说 `SqlSessionTemplate` 继承 `SqlSession`，本身就是错误描述。

实际上：

```
public class SqlSessionTemplate
        implements SqlSession, DisposableBean {
}
```

它是实现 `SqlSession` 接口，而不是继承一个普通类。

`DefaultSqlSession` 非线程安全，而 `SqlSessionTemplate` 是线程安全的，可以作为单例由多个 Mapper/DAO 共用。

`SqlSessionTemplate` 会通过 Spring 事务上下文获取合适的实际 `SqlSession`，并管理其生命周期。

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



### 第 38 题：Kubernetes Pod 重启策略与 DaemonSet

你选 D，正确。

Pod 的三种重启策略：

| 策略        | 正常退出（0） | 异常退出（非 0） |
| --------- | ------- | --------- |
| Always    | 重启      | 重启        |
| OnFailure | 不重启     | 重启        |
| Never     | 不重启     | 不重启       |

Pod 的 `restartPolicy` 默认是 `Always`。

[image](https://www.google.com/s2/favicons?domain=https://kubernetes.io\&sz=32)

Kubernetes



你忘记的 `DaemonSet`，它不是重启策略，而是 Kubernetes 的一种工作负载控制器。

它的主要作用是：确保集群中符合条件的每个节点上都运行一个指定的 Pod。

例如，一个 Kubernetes 集群有三个工作节点：

Node 1

日志采集 Pod

Node 2

日志采集 Pod

Node 3

日志采集 Pod

DaemonSet 确保符合调度条件的每个节点运行日志采集实例

常见用途：

- 日志采集：Fluent Bit、Filebeat
- 节点监控：Node Exporter
- 网络组件：部分 CNI 网络代理
- 存储组件：节点级存储服务

同时记住 Kubernetes 三个常考控制器：

| 控制器         | 主要用途                   |
| ----------- | ---------------------- |
| Deployment  | 管理无状态服务，支持副本数和滚动更新     |
| StatefulSet | 管理有状态服务，提供稳定标识和持久化存储关联 |
| DaemonSet   | 在每个符合条件的节点运行指定 Pod     |

### 第 39 题：Spring Security 密码保护与 LDAP

你选 A，正确。

这里要纠正一个术语：bcrypt、MD5、MD4 都属于散列算法（Hash），不是可逆的加密算法。

- bcrypt：专门适合密码散列的自适应算法，具有盐和可配置的计算成本。
- MD5：快速散列算法，不适合现代密码存储。
- MD4：更老的快速散列算法，同样不适合密码存储。
- LDAP：Lightweight Directory Access Protocol，轻量级目录访问协议，不是散列或加密算法。

LDAP 主要用于企业组织目录管理、用户查询和身份认证。

例如企业员工系统可以通过 LDAP 对接统一目录服务，查询组织架构或验证用户凭据。

Spring Security 支持 LDAP 认证，也支持 BCryptPasswordEncoder 等密码编码器。

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home



### 第 40 题：多租户数据库架构

你选 C，属于可能的考试预期，但你的解释“多租户，不同用户不同库”需要纠正。

首先，租户不等于用户。

例如一个银行企业网银 SaaS 平台中：

- 企业 A 是租户 A，有 100 名员工用户。
- 企业 B 是租户 B，有 50 名员工用户。

如果采用独立数据库方案，是每个租户一个数据库，而不是每个用户一个数据库。

多租户数据库常见三种设计：

共享数据库

方案一：共享数据库、共享表

使用 `tenant_id` 区分租户。

成本较低，但隔离性相对较弱，高并发下可能产生共享资源竞争。

共享库独立表

方案二：共享数据库、独立 Schema 或表

不同租户逻辑隔离，适用于租户数量和管理成本可控的场景。

独立数据库

方案三：不同租户独立数据库

租户隔离能力较强，可以分散负载，但数据库实例和运维管理成本更高。

再看原题的四个选项：

| 方案      | 主要解决的问题           |
| ------- | ----------------- |
| A. 垂直分库 | 按业务模块拆分数据库        |
| B. 水平分表 | 将单张大表拆分为多张表       |
| C. 水平分库 | 按租户、用户等分片键分散数据与负载 |
| D. 垂直分表 | 按字段拆分宽表           |

如果题目强调租户隔离与高并发下的数据库负载分散，C 是合理答案。

但如果共享数据库的数据量很大，读热点集中，水平分表、只读副本、缓存等也可能更适合。

所以不能得出“多租户高并发读操作一定要水平分库”的结论。

## 三、你这组题最值得巩固的知识点

结合你的解答，我建议优先巩固以下几项：

| 优先级 | 知识点                  | 建议掌握程度                                |
| --- | -------------------- | ------------------------------------- |
| 高   | MyBatis 一级、二级缓存      | 生命周期、命中顺序、失效机制                        |
| 高   | 数据库读写分离与分库分表         | 解决什么问题、不能解决什么问题                       |
| 高   | Spring Cloud Gateway | Route order、Path、Filter order         |
| 中   | RabbitMQ 可靠性         | Confirm、Ack、持久化、副本                    |
| 中   | SQL JOIN 优化          | ON/WHERE、驱动表、执行计划                     |
| 中   | Kubernetes           | Deployment、StatefulSet、DaemonSet、重启策略 |
| 中   | 多租户架构                | 共享表、独立 Schema、独立库                     |

本组核心结论：第 32 题需要改成 C，第 21 题存在歧义，第 40 题虽然倾向 C，但要按分片需求判断，而不是按用户数量直接判断。

你其余大部分解释能够抓到核心原理，接下来最需要提升的是区分“考试中的简化规则”和“实际工程中的成立条件”。

# 41～50 
2026.10.24 21:18
41 C 分数个数不会自动增长。
42 A 当该id没有被占用，主键允许更新。
43 D linger.ms不能盲目减少，对于C选项我不太清楚，为什么增加副本fetchers可以增加吞吐量。但根据题意应该选D。
44 B 根据代码显然可以得出是B。
45 D  首先BC是正确的，A选项我不知道对称性是什么意思，D仅根据每次请求携带数据进行相应的业务逻辑处理，看表述更像是与业务状态有关。
46 D ABC表述均正确。
47 A 私有方法不会有事务。C选项我不确定什么情况下会失效，D选项我不太确定protected对切面是否生效，感觉是不生效，所以D似乎也有点不对。
48 C B感觉也是对的，因为提到了自动提交已开启，但C相对更完备。
49 B 我其实没了解LoadBalancer，但我觉得AC表述均正确，至于D这个负载均衡配置我确实不知道怎么加载的，但综合一下AC是对的，推测是B选项。
50 C Mybatis使用JDK代理，但其余几个代理是干啥的需加深记忆。

结合完整题干，你第 41～50 题的答案基本正确。但有三个地方值得特别注意：第 45 题无状态的含义、第 47 题事务注解的方法可见性、第 48 题 MyBatis 的事务提交与并发安全。

## 一、答案核对

| 题号 | 你的答案 | 建议答案               | 判断 |
| -- | ---- | ------------------ | -- |
| 41 | C    | C                  |    |
| 42 | A    | A                  |    |
| 43 | D    | D                  |    |
| 44 | B    | B                  |    |
| 45 | D    | C（题目有歧义）           |    |
| 46 | D    | D                  |    |
| 47 | A    | A（版本相关）            |    |
| 48 | C    | B、C 均可，C 更符合常规事务管理 |    |
| 49 | B    | B                  |    |
| 50 | C    | C                  |    |

下面重点解释你不确定的选项，并且区分考试答案和实际工程实现。

## 二、逐题解析

### 第 41 题：Kafka 分区数量

答案：C，正确。

Kafka 的分区数量不会根据生产者和消费者数量自动增长。

例如一个 Topic 有 6 个分区：

- 生产者从 2 个增长至 20 个，不会自动增加分区。
- 同一消费者组从 3 个消费者增长至 10 个消费者，也不会自动增加分区。
- 10 个消费者中，最多通常只有 6 个消费者能同时承担该 Topic 的分区消费任务，其余可能空闲。

对于 D 选项，每个分区在同一个消费者组内同一时刻通常只会由一个消费者负责，但不同消费者组可以分别消费该分区。

所以 D 的表达不够严格，不过 C 明显错误。

### 第 42 题：数据库主键是否允许修改

答案：A，正确。

你的理解完全正确。

```
UPDATE student
SET studentid = 100
WHERE studentid = 101;
```

假设 100 不存在，且没有其他约束阻止修改，数据库可以将主键从 101 更新为 100。

需要记住：

主键约束要求唯一性和非空，不意味着主键不可修改。

但是在实际系统设计中，通常不建议修改主键，原因包括：

- 可能影响引用该主键的外键数据。
- 可能造成关联关系维护困难。
- 可能引发索引维护和额外锁竞争。

尤其是采用 ID 作为实体身份标识时，最好保持其不可变。

### 第 43 题：Kafka 吞吐量优化

答案：D，正确。

你问为什么增加 `num.replica.fetchers` 能提升吞吐量。

Kafka 存在不同的吞吐量指标：

| 吞吐量     | 涉及组件              |
| ------- |-------------------|
| 生产吞吐量   | Producer → Broker |
| 消费吞吐量   | Broker → Consumer |
| 副本复制吞吐量 | Leader → Follower |

`num.replica.fetchers` 是 Broker 的配置参数，控制副本数据拉取线程数量。

例如一个 Follower Broker 需要同步多个分区的数据：

Leader Broker

分区 P0、P1、P2、P3

Fetcher 1

Fetcher 2

Fetcher 3

Follower Broker

并行拉取不同分区的副本数据

增加 Fetcher 线程，可以提高多个分区副本的并行同步能力，在复制是瓶颈时改善吞吐量。

但这不意味着线程越多越好，因为还受到网络、CPU、磁盘 IO 的限制。

对于 D：

`linger.ms` 控制 Producer 为了凑批次而等待的时间。

- 适当增大：可能形成更大的 Batch，提升吞吐量。
- 减小：通常降低等待延迟，但可能导致批次更小、请求次数更多。

因此减少 `linger.ms` 不是通用的吞吐量优化手段。

[image](https://www.google.com/s2/favicons?domain=https://kafka.apache.org\&sz=32)

Apache Kafka



### 第 44 题：synchronized 线程同步

答案：B，正确。

```
private int data = 2;
int result = 0;
```

三个线程共享同一个 `Test` 对象，而且都使用：

```
synchronized (mv) {
    mv.m();
}
```

所以同一时刻只有一个线程可以进入 `mv.m()`。

三次执行结果：

```
2 2
4 2
6 2
```

这里容易忽略的是：虽然线程执行顺序不确定，但输出值的顺序是确定的，因为 `result += data` 和打印操作都在同一个临界区内。

### 第 45 题：业务无状态性

重点纠正

你选 D，我更倾向于 C，但这道题的表述本身有问题。

先看选项：

A. 多个模块（子系统）之间与对称性无关。

这里的“对称性”不是描述无状态性的常见标准术语，缺少上下文，不适合直接判定正确。

B. 请求提交到任何服务器，处理结果都是完全一样。

这属于对无状态服务的常见简化描述。准确来说，相同请求在相同业务状态和环境下，应能够由任意合适的服务实例处理，而不是保证任何时间都得到完全一样的结果。

C. 系统不存储业务的上下文信息。

这个说法过于绝对。

D. 仅根据每次请求携带数据进行相应的业务逻辑处理。

这是无状态架构的核心思想的一种简化表述，但还需要考虑请求所引用的持久化业务数据。

举个例子。

```
POST /orders/10001/pay
Authorization: Bearer xxx
```

服务端可以根据请求中的订单 ID 查询数据库中的订单状态。

数据库中可以存储：

```
订单ID：10001
订单状态：待支付
订单金额：200
```

完成支付后变为：

```
订单状态：已支付
```

虽然业务状态发生了变化，但服务本身仍然可以设计成无状态的。

因为它不要求某个用户下一次请求必须落到先前处理请求的同一台服务器。

关键区别：

- 无状态服务：不依赖服务器本地保存的客户端会话上下文。
- 有状态业务：可以持久化订单、库存、支付等业务状态。

因此，按照严格技术含义，我倾向于选 C。A 的措辞也无法直接判断，所以这道题不够规范。

### 第 46 题：数据库索引分区

答案：D，正确。

这道题主要考察 Oracle 等数据库中的索引分区类型。

| 类型      | 索引分区特点          |
| ------- | --------------- |
| 全局非分区索引 | 一个整体索引，不跟随表分区拆分 |
| 局部分区索引  | 索引分区与表分区对应      |
| 全局分区索引  | 索引独立按照指定规则分区    |

A、B、C 三项描述基本正确。

特别注意，局部分区索引具有较好的分区维护便利性，适合按分区键查询和管理的场景。

全局索引在某些跨分区查询中可能更有优势，但对分区维护操作需要额外考虑。

### 第 47 题：Spring `@Transactional` 的使用

你的答案：A，符合考试预期，但 D 也存在版本差异。

四个选项逐一分析：

| 选项 | 判断    | 说明                                           |
| -- | ----- | -------------------------------------------- |
| A  | 不严谨   | 类级注解不代表所有方法调用都能实际开启事务                        |
| B  | 通常正确  | 默认代理模式下，同类内部直接调用绕过事务代理                       |
| C  | 正确    | 接口上的事务注解可能因代理或织入方式而无法生效                      |
| D  | 取决于版本 | Spring 6 类代理支持 protected 方法；Spring 5.x 默认不支持 |

你对 A 的理解是正确的：类上添加 `@Transactional`，并不意味着所有方法调用都会受到事务代理拦截。

但是还需区分两个概念：

- 注解作用范围：类级 `@Transactional` 可以为类中符合条件的方法提供默认事务属性。
- 事务实际生效：需要满足代理机制、方法可见性、调用路径等条件。

Spring 官方也明确说明了类级注解和代理拦截的区别。

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



#### protected 方法究竟能否生效？

| Spring 版本       | JDK 动态代理 | CGLIB 类代理 |
| --------------- | -------- | --------- |
| Spring 5.x 默认配置 | 不支持      | 默认不支持     |
| Spring 6.x 默认配置 | 不支持      | 支持        |

例如：

```
@Service
public class OrderService {

    @Transactional
    protected void updateOrder() {
        // 更新订单
    }
}
```

在 Spring 6.x 的类代理模式下，满足外部代理调用等条件时，可以生效。

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home



但是如果：

```
public void execute() {
    updateOrder();
}
```

由同类内部直接调用，默认代理模式下仍然不会触发 `updateOrder()` 的事务拦截。

结论：第 47 题按常见题库选择 A，但由于没有指定 Spring 版本和代理方式，不能认为 D 无条件正确。

对于实际项目，我仍然建议把 `@Transactional` 放在明确的 `public` 业务方法上，并避免不必要的类级事务声明。

### 第 48 题：MyBatis SqlSession 与库存扣减

题目存在明显歧义

你选 C。C 是标准的手动事务提交写法，但 B 在语法及基本事务行为上同样正确。

#### B 选项

```
try (SqlSession session =
         sqlSessionFactory.openSession(true)) {

    ProductMapper mapper =
        session.getMapper(ProductMapper.class);

    mapper.updateStock(productId, quantity);
}
```

关键在于：

```
openSession(true)
```

表示启用自动提交。

因此执行成功时，通常不需要再手动调用 `commit()`。

#### C 选项

```
try (SqlSession session =
         sqlSessionFactory.openSession()) {

    ProductMapper mapper =
        session.getMapper(ProductMapper.class);

    mapper.updateStock(productId, quantity);

    session.commit();
}
```

默认 `openSession()` 不启用自动提交，需要手动执行 `commit()`。

如果出现异常，关闭未提交的 Session 会按 MyBatis 的事务管理行为进行回滚。MyBatis 官方文档对此有明确说明。

[image](https://www.google.com/s2/favicons?domain=https://mybatis.org\&sz=32)

Java API – mybatis



两者的区别：

| 对比            | B   | C  |
| ------------- | --- | -- |
| 自动提交          | 是   | 否  |
| 显式 commit     | 不需要 | 需要 |
| 多条 SQL 组成一个事务 | 不方便 | 支持 |
| 显式控制事务边界      | 较弱  | 较强 |
| 是否天然保证库存不超卖   | 否   | 否  |

最后一行非常重要。

手动提交事务不等于高并发库存安全。

假设：

```
库存 = 10

线程 A 查询库存 = 10
线程 B 查询库存 = 10

线程 A 扣减 8
线程 B 扣减 8
```

如果逻辑设计不合理，即使使用 C，也可能出现并发更新问题。

更推荐采用数据库条件更新：

```
UPDATE product
SET stock = stock - #{quantity}
WHERE id = #{productId}
  AND stock >= #{quantity};
```

然后判断影响行数：

```
int updated = mapper.updateStock(productId, quantity);

if (updated != 1) {
    throw new IllegalStateException("库存不足");
}
```

这样可以利用数据库的原子更新和条件判断防止库存扣成负数。

如果涉及订单生成、库存扣减、流水记录等多个数据库操作，还需要统一管理事务。

结论：如果按照题库预期，可以选 C；但严格来说 B 也能正确执行并提交更新，而且题目没有任何选项单凭自身就能保证高并发库存安全。

### 第 49 题：Spring Cloud LoadBalancer 工作原理

你的答案：B，正确，值得进一步掌握源码层面的调用关系。

你之前没有接触过 LoadBalancer，重点理解下面三个类：

- `BlockingLoadBalancerClient`
- `LoadBalancerClientFactory`
- `ServiceInstanceListSupplier`

#### 1. 整体调用关系

RestTemplate 发起请求

[http://order-service/orders](http://order-service/orders?utm_source=chatgpt.com)

BlockingLoadBalancerClient

处理同步负载均衡请求

LoadBalancerClientFactory

按 Service ID 获取对应 LoadBalancer

RoundRobinLoadBalancer / RandomLoadBalancer

使用负载均衡算法选择实例

ServiceInstanceListSupplier

提供可用的服务实例列表

10.0.0.1:8080

10.0.0.2:8080

10.0.0.3:8080

图示的是主要组件关系，实际实现中 LoadBalancer 通过 Supplier 获取实例，再返回被选中的服务实例。

#### 2. 为什么 A 是正确的？

```
@Bean
@ConditionalOnMissingBean
public LoadBalancerClient blockingLoadBalancerClient(
        LoadBalancerClientFactory factory) {
    return new BlockingLoadBalancerClient(factory);
}
```

在标准自动配置下，`BlockingLoadBalancerClient` 通常是应用上下文中的一个单例 Bean。

它可以处理多个不同服务的负载均衡请求。

但这不代表整个微服务集群中只有一个对象，而是通常每个相关的应用上下文有自己的实例。

#### 3. 为什么 C 是正确的？

Spring Cloud LoadBalancer 为不同的 Service ID 管理独立的负载均衡配置上下文。

例如：

```
order-service
    └── RoundRobinLoadBalancer

payment-service
    └── RandomLoadBalancer

inventory-service
    └── RoundRobinLoadBalancer
```

不同服务可以配置不同的负载均衡策略。

#### 4. 为什么 D 是正确的？

`BlockingLoadBalancerClient` 依赖 `LoadBalancerClientFactory`，根据服务名称取得对应服务的负载均衡器，并执行实例选择。

这些架构与 Spring Cloud 官方文档和自动配置源码相符。

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home

+1



所以：

A、C、D 都正确，选 B。

### 第 50 题：MyBatis 动态代理

答案：C，正确。

MyBatis Mapper 接口默认使用 JDK 动态代理。

例如：

```
public interface UserMapper {

    User selectById(Long id);
}
```

开发者没有实现这个接口，但仍然可以通过：

```
UserMapper mapper =
    sqlSession.getMapper(UserMapper.class);

User user = mapper.selectById(1L);
```

来执行查询。

本质是 MyBatis 创建代理对象，拦截接口方法，映射到对应 SQL。

另外几个选项：

| 技术        | 作用            |
| --------- | ------------- |
| ASM       | 底层字节码读取、分析与生成 |
| CGLIB     | 基于生成子类的代理技术   |
| JDK 动态代理  | 基于接口生成代理对象    |
| Javassist | 字节码操作和动态类生成   |

一句话记忆：

MyBatis Mapper 用 JDK 动态代理；Spring AOP 可以使用 JDK 动态代理或类代理；ASM 和 Javassist 则偏向底层字节码操作。

## 三、最终总结

你的第 41～50 题中，41、42、43、44、46、49、50 都可以直接保留原答案。其余三道题需要特别标记：

- 45： 你的 D 不建议保留为确定答案，C 更值得怀疑，但 A 的措辞也有问题。
- 47： A 是常见考试答案，不过 D 是否成立取决于 Spring 版本和代理方式。
- 48： C 是较完整的显式事务写法，但 B 同样可以正确提交单条更新；并发库存安全还需要条件更新等机制。

这十道题里最有实际工程价值的是 43、47、48、49。它们分别对应 Kafka 吞吐量调优、Spring 事务生效条件、库存并发一致性以及微服务负载均衡原理。