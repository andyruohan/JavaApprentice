## Java 基础
### Java 常见的代理
| 名称 | 全称或名称含义 | 核心方式           | 代表使用场景 |
|---|---|----------------|---|
| **ASM** | 官方直接使用 ASM 这个名称，没有给出正式英文展开 | 直接操作字节码，生成或修改类 | **代码覆盖率统计、字节码插桩**，例如 JaCoCo |
| **CGLIB** | **Code Generation Library**，官方项目描述为 Byte Code Generation Library，字节码生成库 | 生成目标类的子类(继承)   | **Spring AOP 的类代理**，例如为 Service 添加事务、日志 |
| **JDK 动态代理** | **Java Development Kit Dynamic Proxy**；JDK 指 Java 开发工具包 | 生成实现指定接口的代理类   | **MyBatis Mapper 接口代理**、Spring AOP 的接口代理 |
| **Javassist** | **Java Programming Assistant**，Java 编程助手 | 提供字节码操作和动态代理工具 | **运行时生成或修改类、创建代理对象**，例如通过 `ProxyFactory` 拦截方法 |

### Throwable 介绍
```text
Throwable（所有错误和异常的超类）
├── Error（系统级严重错误，程序无法处理，不应捕获）
│   ├── OutOfMemoryError（内存溢出）
│   ├── StackOverflowError（栈溢出）
│   ├── NoClassDefFoundError（类定义找不到）
│   └── ...（一般不捕获）
│
└── Exception（程序可处理的问题）
    ├── RuntimeException（非受检异常 / Unchecked，编译期不强制处理）
    │   ├── NullPointerException（空指针）
    │   ├── IllegalArgumentException（非法参数）
    │   ├── IndexOutOfBoundsException（下标越界）
    │   ├── ClassCastException（类型转换异常）
    │   └── ...（开发中常见，可避免）
    │
    └── 非RuntimeException（受检异常 / Checked，编译期必须 try-catch 或 throws）
        ├── IOException（I/O 异常）
        ├── SQLException（数据库异常）
        ├── FileNotFoundException（文件未找到）
        ├── ClassNotFoundException（类未找到）
        └── ...（必须显式处理）
```

### 过滤器与拦截器的执行过程
```text
正常流程：
Filter前置 → preHandle → Controller → postHandle → 视图渲染（如有）→ afterCompletion(ex=null) → Filter后置

异常被成功处理流程：
Filter前置 → preHandle → Controller报错 → @ControllerAdvice中匹配的异常处理方法 → afterCompletion(ex=null) → Filter后置

异常未被处理流程：
Filter前置 → preHandle → Controller报错 → afterCompletion(ex=异常) → 异常向Filter传播 → Filter的finally执行，普通后置代码跳过
```

## Mybatis
### Mybatis 一级、二级缓存的执行过程

**二级缓存已开启、且本次查询允许使用时：先查二级，再查一级，最后查数据库。**

```text
执行查询
   ↓
二级缓存
   ├─ 命中 → 直接返回
   └─ 未命中
        ↓
      一级缓存（当前 SqlSession）
        ├─ 命中 → 返回结果
        └─ 未命中
             ↓
           查询数据库
             ↓
           结果放入一级缓存
             ↓
           返回结果

从一级缓存或数据库取得的结果
   ↓
暂存为待写入二级缓存的结果
   ↓
缓存提交时，写入共享二级缓存
```

源码中，`CachingExecutor` 先尝试读取二级缓存，未命中才调用底层执行器；底层 `BaseExecutor` 再检查一级缓存。[MyBatis 查询源码](https://mybatis.org/mybatis-3/xref/org/apache/ibatis/executor/CachingExecutor.html#L94)

| 查询条件 | 查询顺序 |
|---|---|
| 二级缓存开启，查询允许使用 | **二级 → 一级 → 数据库** |
| 未开启二级缓存，或查询配置 `useCache="false"` | **一级 → 数据库** |


### Mybatis 一级、二级缓存的清理

| 对比项 | 一级缓存 | 二级缓存 |
|---|---|---|
| **清理范围** | 当前 `SqlSession` 的整个缓存 | 语句关联的共享缓存，通常对应 Mapper 命名空间 |
| **执行增删改** | 执行时清理 | 默认先标记，提交时实际清理 |
| **更新设 `flushCache="false"`** | 仍清理 | 不触发清理 |
| **提交** | 清理 | 应用待执行的清理和写入，不一定清空 |
| **回滚更新事务** | 清理 | 放弃本次待执行的缓存变化 |
| **`sqlSession.clearCache()`** | 清理 | 不影响 |
| **关闭会话** | 缓存不再可用 | 共享缓存不会因此被清空 |

最直观的例子：

| 步骤 | 共享二级缓存 | A 的一级缓存 |
|---|---|---|
| ① A 查询用户 1 | A 的查询结果尚未提交到共享缓存，仍为空 | 保存“旧名字” |
| ② B 更新用户 1 并提交 | B 提交时清理关联的二级缓存，仍为空 | **不受影响，仍保存“旧名字”** |
| ③ A 再次查询 | 先查二级，未命中 | 再查一级，命中“旧名字” |

```text
B 执行 UPDATE，尚未提交
    ├─ B 的一级缓存：已清理
    ├─ 共享二级缓存：标记为提交时清理
    └─ A 的一级缓存：不受影响

B 提交
    ├─ 共享二级缓存：实际清理
    └─ A 的一级缓存：仍不受影响
```

**所以可以记成：一级清“当前会话”，二级清“关联的共享缓存”；一级立即清，二级的更新失效通常等提交。**

## Kafka
### Kafka 的基本架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Kafka 集群：3个Broker、2个Topic                        │
│                                                                         │
│  ┌─────────────────────┬─────────────────────┬─────────────────────┐    │
│  │      Broker 1       │      Broker 2       │      Broker 3       │    │
│  │                     │                     │                     │    │
│  ╞═════════════════════╪═════════════════════╪═════════════════════╡    │
│  │                 Topic：order-events（订单消息）                   │    │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤    │
│  │ P0 [Leader]         │ P0 [Follower]       │ P1 [Follower]       │    │
│  │ P2 [Follower]       │ P1 [Leader]         │ P2 [Leader]         │    │
│  │                     │                     │                     │    │
│  ╞═════════════════════╪═════════════════════╪═════════════════════╡    │
│  │                Topic：payment-events（支付消息）                  │    │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤    │
│  │ P0 [Follower]       │ P1 [Leader]         │ P0 [Leader]         │    │
│  │                     │                     │ P1 [Follower]       │    │
│  │                     │                     │                     │    │
│  └─────────────────────┴─────────────────────┴─────────────────────┘    │
│                                                                         │
│  说明：                                                                  │
│  1. 竖向每一列是一个Broker，可承载多个Topic的分区副本。                        │
│  2. 横向每一带是一个Topic，展示其副本在各Broker上的分布。                      │
│  3. P表示Partition（分区）；订单有3个分区，支付有2个分区。                      │
│  4. 每个分区有2份副本：1个Leader、1个Follower，共10份副本。                   │
│  5. 不同Topic的P0是不同分区；同一分区的Follower复制Leader日志。               │
│  6. Follower可能短暂落后，ISR不表示每一瞬间数据都完全相同。                    │
│  7. 删除订单Topic，相当于清理订单这一横带的数据；                              │
│     三个Broker仍运行，支付Topic仍保留。                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Kafka 主要运行过程
```
Producer（生产者）       Broker（服务端）       Consumer（消费者）
       │                      │                      │
       ├──── 发送消息 ────────▶│                      │
       │                      │                      │
       │               保存Partition副本              │
       │                  的消息日志                  │
       │                      │                      │
       │                      │◀─────  拉取请求 ──────┤
       │                      ├────── 返回消息 ──────▶│
       │                      │                      │
       
       注：这里的消息日志就是消息数据本身，不是程序运行日志。
```

### 保证消息可靠性的三个措施
```
生产者 ───────▶ RabbitMQ ───────▶ 消费者
      Confirm   持久化     Ack
      发送确认   存储保障   消费确认
```

### Kafka 选举机制类比表

| 公司层级 | Kafka 对应组件 | 管理对象 | 选举/指派机制 | 详细过程 |
|:---|:---|:---|:---|:---|
| **公司总部** | **ZooKeeper 集群** | 管理临时节点和 Watcher 通知 | 提供选举协调服务 | 总部前台有一块"领导公示板"（`/controller` 临时节点）。每位员工入职时都在前台登记，总部通过定时签到（会话心跳）检测员工是否在职。员工离职或失联超过时限，总部自动从公示板上擦除其名字，并通知所有等候的员工"重新竞选"。 |
| **部门领导** | **Controller（控制器）** | 管理 **Broker**（集群节点上下线、Topic 创建删除、元数据同步） | **Controller 选举**（ZooKeeper 临时节点竞争） | 所有员工（Broker）在入职时都跑到总部前台，抢着在"领导公示板"上贴自己的名字，**第一个贴上去的人成为部门领导（Controller）**。其他人贴不上，就在前台等候通知（注册 Watcher）。现任领导因失联被总部从公示板上擦除名字后，前台立刻通知所有等候的员工，大家**再次一拥而上抢着贴名字**，先贴上去的人成为新领导。 |
| **项目小组长** | **Partition Leader** | 管理 **Partition**（分区内数据的读写请求、Follower 数据同步、ISR 维护） | **Partition Leader 选举**（由 Controller 从 ISR 中指派） | 新部门领导上任后，第一件事是检查所有项目组（Partition），看哪些组的组长（Leader）位置空缺。对于空缺的组，领导查阅该组的**核心骨干名单（ISR，即与前任组长数据完全同步的组员）**，**按名单顺序指定第一个人当新组长**。随后领导在总部备案任命结果（更新 ZK 元数据），并发邮件通知全部门所有员工和外部客户更新通讯录。 |



## Redis
### Redis 的基础数据类型
| 数据类型              | 常见内部编码                          |
|-------------------| ------------------------------- |
| String            | int、embstr、raw                  |
| List              | quicklist                       |
| Hash              | listpack、hashtable              |
| Set               | intset、hashtable、listpack（较新版本） |
| ZSet (Sorted Set) | listpack、skiplist               |

[Redis官方文档](https://redis.io/docs/latest/commands/object-encoding/)

Redis objects can be encoded in different ways:

- **Strings** can be encoded as:

   - **raw**, normal string encoding.
   - **int**, strings representing integers in a 64-bit signed interval, encoded in this way to save space.
   - **embstr**, an embedded string, which is an object where the internal simple dynamic string, sds, is an unmodifiable string allocated in the same chuck as the object itself. embstr can be strings with lengths up to the hardcoded limit of OBJ_ENCODING_EMBSTR_SIZE_LIMIT or 44 bytes.

---
- **Lists** can be encoded as:
   - **linkedlist**, simple list encoding. No longer used, an old list encoding.
   - **ziplist**, Redis <= 6.2, a space-efficient encoding used for small lists.
   - **listpack**, Redis >= 7.0, a space-efficient encoding used for small lists.
   - **quicklist**, encoded as linkedlist of ziplists or listpacks.


- **Sets** can be encoded as:

   - **hashtable**, normal set encoding.
   - **intset**, a special encoding used for small sets composed solely of integers.
   - **listpack**, Redis >= 7.2, a space-efficient encoding used for small sets.

- **Hashes** can be encoded as:

   - **zipmap**, no longer used, an old hash encoding.
   - **hashtable**, normal hash encoding.
   - **ziplist**, Redis <= 6.2, a space-efficient encoding used for small hashes.
   - **listpack**, Redis >= 7.0, a space-efficient encoding used for small hashes.

- **Sorted Sets** can be encoded as:
   - **skiplist**, normal sorted set encoding.
   - **ziplist**, Redis <= 6.2, a space-efficient encoding used for small sorted sets.
   - **listpack**, Redis >= 7.0, a space-efficient encoding used for small sorted sets.

- **Streams** can be encoded as:
  - **stream**, encoded as a radix tree of listpacks.  
  
All the specially encoded types are automatically converted to the general type once you perform an operation that makes it impossible for Redis to retain the space saving encoding.

### Redis 三种缓存异常现象
| 类型 | 核心特征 | 典型场景 |
|---|---|---|
| 缓存穿透 | 查询不存在的数据 | 反复查询不存在的用户 ID |
| 缓存击穿 | 热点 Key 失效 | 热门商品缓存突然过期 |
| 缓存雪崩 | 大量 Key 同时失效或缓存服务故障 | 大量缓存同时过期 |


## 数据库
### 数据库四类现象
| 现象     | 核心特征                       |
| ------ | -------------------------- |
| 脏读     | 读到了其他事务尚未提交的数据             |
| 不可重复读  | 同一事务两次读取同一行，字段值发生变化        |
| 幻读（虚读） | 同一事务重复查询某个条件，符合条件的记录集合发生变化 |
| 丢失更新   | 并发更新使其中一个更新结果被覆盖           |

### 索引分区
- 全局非分区索引：在分区表上创建的全局普通索引，索引没有被分区
- 局部分区索引：在分区表上创建的索引，在每个表分区上创建独立的索引，索引的分区范围与表一致（按照表分区对索引进行分区）
- 全局分区索引：在分区表或非分区表上创建的索引，索引单独指定分区的范围，与表的分区范围或是否分区无关

记忆：**局部索引跟着表分区；全局分区索引独立分区；全局非分区索引自身不分区**。

> [Oracle 官方文档](https://docs.oracle.com/database/121/TDPDW/tdpdw_physdes.htm#TDPDW365):
> Index Partitioning
>
>
>- **Local Indexes**
   >
   >  A local index is an index on a partitioned table that is coupled with the underlying partitioned table, **'inheriting' the partitioning strategy from the table.**
>
>- **Global Partitioned Indexes**
   >
   >  A global partitioned index is an index on a partitioned or nonpartitioned table that **is partitioned using a different partitioning-key or partitioning strategy than the table**. Global-partitioned indexes can be partitioned using range or hash partitioning and **are uncoupled from the underlying table.**
>
>- **Global Non-Partitioned Indexes**
   >
   >  **The index structure is not partitioned** and uncoupled from the underlying table.


## SQL
### SQL 的执行顺序
> 简化版：FROM → WHERE → GROUP BY → 聚合计算 → HAVING → ORDER BY

> 完整版：FROM → JOIN → WHERE → GROUP BY → 聚合 → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT

### SqlSession 的三个核心对象

| 对象 | 作用                                      | 是否可多线程共享 |
|---|-----------------------------------------|---|
| `SqlSessionFactory` | 创建SqlSession，**通常配置为单例**，随应用长期复用        | ✅ |
| `SqlSessionTemplate` | 统一调用入口，**通常配置为单例**，通过代理获取对应会话并委托执行 | ✅ |
| `DefaultSqlSession` | 实际会话，关联连接、事务、一级缓存等状态                    | ❌ |

### SqlSession 的常规调用流程
```
       userMapper.findById(1L)
                │
                ▼
         Mapper代理解析调用
                │
                ▼
    SqlSessionTemplate.selectOne(...)
                │
                ▼
    内部动态代理 sqlSessionProxy
                │
                ▼
    SqlSessionInterceptor.invoke()
                │
                ▼
    获取当前事务对应的SqlSession
       ├── 已有 → 复用
       └── 没有 → SqlSessionFactory创建
                   （通常单例，可共享）
                │
                ▼
    底层SqlSession.selectOne(...)
                │
                ▼
    通过执行器查询缓存或数据库
```

## SpringCloud
### openFein 的日志级别
| 级别 | 请求方法、URL、响应状态码、耗时 | 请求／响应头 | 请求／响应正文 |
|---|---|---|---|
| `NONE`（默认） | ❌ | ❌ | ❌ |
| `BASIC` | ✅ | ❌ | ❌ |
| `HEADERS` | ✅ | ✅ | ❌ |
| `FULL` | ✅ | ✅ | ✅ |

> OpenFeign日志说明：https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html#feign-logging  
> The Logger.Level object that you may configure per client, tells Feign how much to log. Choices are:
> - NONE, No logging (DEFAULT).
> - BASIC, Log only the request method and URL and the response status code and execution time.
> - HEADERS, Log the basic information along with request and response headers.
> - FULL, Log the headers, body, and metadata for both requests and responses.


### Eureka 的自我保护机制
Eureka 是 Netflix 开源的服务注册与发现系统，Spring Cloud Netflix 提供了相应集成。[Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix?utm_source=chatgpt.com)

Eureka Server 默认每隔15分钟尝试更新预期心跳续约阈值，该阈值通常为预期每分钟续约次数的85%。当最近1分钟实际收到的续约次数不超过该阈值时，若开启自我保护机制，Eureka Server 将暂停正常的实例过期剔除，防止因网络异常导致大量健康实例被错误移除。

                    Eureka Server
                         │
                         ▼
              每隔15分钟尝试更新阈值
                         │
                         ▼
            计算预期每分钟心跳续约次数
                         │
                         ▼
              预期次数 × 85%
                         │
                         ▼
                 心跳续约阈值
                         │
                         ▼
             获取最近1分钟实际续约次数
                         │
                         ▼
              是否超过心跳续约阈值？
                    /          \
                  是            否
                  │             │
                  ▼             ▼
           允许正常剔除     是否开启自我保护？
           过期实例           /       \
                             是       否
                             │        │
                             ▼        ▼
                         暂停自动    允许正常
                         过期剔除    过期剔除

### LoadBalancer 的运行过程
```
                  Spring 应用（服务消费者）
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   RestTemplate / RestClient                             │
│              │                                          │
│              ▼                                          │
│   BlockingLoadBalancerClient                            │
│   （默认单例 Bean，处理同步负载均衡请求）                     │
│              │                                          │
│              │ 根据 serviceId                            │
│              ▼                                          │
│   LoadBalancerClientFactory.getInstance(serviceId)      │
│   （默认单例 Bean，管理不同 serviceId 的独立子容器）          │
│              │                                          │
│       ┌──────┴──────────────────┐                       │
│       ▼                         ▼                       │
│ ┌──────────────────┐  ┌──────────────────┐              │
│ │ order-service    │  │ user-service     │              │
│ │ 独立子容器         │  │ 独立子容器        │              │
│ │                  │  │                  │              │
│ │ LoadBalancer     │  │ LoadBalancer     │              │
│ │ 轮询算法          │  │ 随机算法           │              │
│ │       │          │  │       │          │              │
│ │       ▼          │  │       ▼          │              │
│ │ ServiceInstance  │  │ ServiceInstance  │              │
│ │ ListSupplier     │  │ ListSupplier     │              │
│ │       │          │  │       │          │              │
│ │       ▼          │  │       ▼          │              │
│ │ 实例列表          │  │ 实例列表           │              │
│ │ ├─ order-1       │  │ ├─ user-1        │              │
│ │ ├─ order-2       │  │ ├─ user-2        │              │
│ │ └─ order-3       │  │ └─ user-3        │              │
│ │       │          │  │       │          │              │
│ │       ▼          │  │       ▼          │              │
│ │ 选中 order-2     │  │ 选中 user-1       │              │
│ └──────────────────┘  └──────────────────┘              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## 云服务集群
### Kubernetes Pod 的三种重启策略

| 策略        | 正常退出（0） | 异常退出（非 0） |
| --------- | ------- | --------- |
| Always    | 重启      | 重启        |
| OnFailure | 不重启     | 重启        |
| Never     | 不重启     | 不重启       |

>[Kubernetes重启策略官方说明](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy):
> The spec of a Pod has a `restartPolicy` field with possible values **Always, OnFailure, and Never**. The default value is **Always**.


## 架构设计
### CAP 基本概念
CAP 分别表示：

- C（Consistency）：一致性，通常指线性一致性。
- A（Availability）：可用性，每个到达正常节点的请求最终都能得到非错误响应。
- P（Partition Tolerance）：分区容错性，网络分区出现时仍能按设计运行。

当网络分区真正发生时，分布式系统无法同时保证 C 和 A。没有网络分区时，同时满足一致性和可用性是可以的。

> [CAP原始论文](https://www.cs.princeton.edu/courses/archive/spr22/cos418/papers/cap.pdf):
> Theorem 1 It is impossible in the asynchronous network model to implement a read/write data object that guarantees the following properties: • Availability • Atomic consistency in all fair executions (including those in which messages are lost).
