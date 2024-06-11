# Database
## 数据库索引的原理?创建索引的缺点是什么，什么情况索引失效?优化数据库的方法有哪些?

数据库索引的原理是通过创建一个额外的数据结构，该结构可以加速数据的检索。这个结构通常是一个树形结构，如B树或者B+树，它会按照索引列的值对数据进行排序，从而快速定位到符合查询条件的数据行。

创建索引的缺点包括：
1. **占用空间：** 索引需要额外的存储空间来维护索引结构。
2. **更新成本增加：** 索引的存在会导致插入、更新和删除操作的性能下降，因为除了更新数据表，还需要更新索引。
3. **可能引发锁竞争：** 在高并发的情况下，对数据库表进行更新操作可能会引发锁竞争，降低系统的并发性能。
    > 延伸：数据库管理系统通常使用MVCC，解决数据库中的并发控制问题。MVCC允许多个事务同时进行而不会互相阻塞，从而提高数据库的并发性能。

如何避免索引失效(可结合下文mysql最左前缀法则学习)
1. **避免在索引列上使用计算和函数**：直接使用列本身进行查询。
2. **避免在 LIKE 查询中使用前置通配符**：如可能，使用后置通配符。
3. **尽量避免不等于操作符**：如可能，使用其他逻辑重构查询条件。
    > (1) 不等于运算符（!= 或 <>）通常会导致索引失效，因为这种查询需要检查所有记录来确定哪些记录不符合条件。  
     (2) 使用 IS NOT NULL 可能导致索引失效，具体情况取决于数据和数据库版本。不过，IS NULL 通常是有效的。
4. **使用正确的数据类型**：确保查询中使用的值与列的数据类型一致。
5. **使用优化器提示**：如需确保使用索引，可以使用优化器提示（Hints）来强制优化器使用索引。
    >在某些情况下，数据库优化器会决定不使用索引，而是选择全表扫描或其他方法，因为优化器认为这样做的性能更好。


优化数据库的方法包括：
1. **硬件升级：** 通过升级硬件配置，如增加内存、优化磁盘性能等，提高数据库系统的整体性能。
2. **表结构设计优化**：表层面：减少联表查询，如通过添加冗余字段；字段层面：能使用varchar(20)就不要使用varchar(200)，避免大字段。
3. **合适的索引设计：** 根据实际查询需求设计合适的索引，避免创建过多或者不必要的索引。
4. **垂直与水平分库分表**：根据业务需求，将数据进行垂直切分（按业务模块划分数据库）或者水平切分（按数据行进行切分），提高数据库的并发处理能力和扩展性。
5. **优化sql语句：** 使用合适的sql语句，避免全表扫描或索引失效，提高运行效率。
    >(1) where优先等值匹配，范围匹配放在后面  
   > (2) where符合最左匹配原则，避免引起索引失效    
   > (3) 减少函数使用，避免引起索引失效  
   > (4) 减少多表关联查询、减少子查询、减少临时中间表
6. **事务优化**：注意数据库隔离级别（隔离级别越高，对读写性能影响越大），以及减少事务的使用，避免引起的额外开销。
   > 在事务优化时一定要注意锁的问题，如select for update等语句尽量少用
7. **缓存优化：** 使用缓存技术减少数据库访问次数，提高系统的响应速度。
8. **定期维护索引：** 定期对索引进行重新组织或者重建，以保证索引的有效性和性能。
9. **数据归档和清理（数据冷热存储）：** 定期清理和归档不再需要的数据，减少数据库表的数据量，提高查询效率。

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

## 数据库范式
使用地址和订单的例子来说明第一、第二和第三范式，以及去范式化的情况。

### 第一范式（1NF）

**定义**：所有字段都是原子性的，不可分割的。

#### 示例（未满足第一范式）

假设我们有一个订单表，其中地址信息是一个字段：

| 订单ID | 客户名 | 地址                            | 商品   | 数量 |
|--------|--------|---------------------------------|--------|------|
| 1      | 张三   | 北京市海淀区中关村南大街1号      | 手机   | 1    |
| 2      | 李四   | 上海市浦东新区世纪大道100号     | 电脑   | 2    |

这里的“地址”字段包含了多个信息（省、市、区、街道、门牌号），不满足第一范式。

#### 转换（满足第一范式）

将地址字段拆分为多个字段，每个字段包含一个原子值：

| 订单ID | 客户名 | 省份   | 城市   | 区      | 街道        | 门牌号 | 商品   | 数量 |
|--------|--------|--------|--------|---------|-------------|--------|--------|------|
| 1      | 张三   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    | 手机   | 1    |
| 2      | 李四   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  | 电脑   | 2    |

### 第二范式（2NF）

**定义**：在满足第一范式的基础上，表中的非主键字段必须完全依赖于主键，而不能只依赖于主键的一部分。

#### 示例（未满足第二范式）

假设我们有一个订单和客户信息的表：

| 订单ID | 客户ID | 客户名 | 省份   | 城市   | 区      | 街道        | 门牌号 | 商品   | 数量 |
|--------|--------|--------|--------|---------|-------------|--------|--------|------|---|
| 1      | 1001   | 张三   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    | 手机   | 1    |
| 2      | 1002   | 李四   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  | 电脑   | 2    |

**问题**：客户名和地址信息只依赖于客户ID，而不是完全依赖于订单ID，存在部分依赖。

#### 转换（满足第二范式）

将客户信息和订单信息拆分到不同的表中：

- 客户表：

  | 客户ID | 客户名 | 省份   | 城市   | 区      | 街道        | 门牌号 |
    |--------|--------|--------|--------|---------|-------------|--------|
  | 1001   | 张三   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    |
  | 1002   | 李四   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  |

- 订单表：

  | 订单ID | 客户ID | 商品   | 数量 |
    |--------|--------|--------|------|
  | 1      | 1001   | 手机   | 1    |
  | 2      | 1002   | 电脑   | 2    |

### 第三范式（3NF）

**定义**：在满足第二范式的基础上，表中的非主键字段必须直接依赖于主键，而不能依赖于其他非主键字段（消除传递依赖）。

#### 示例（未满足第三范式）

假设我们有一个包含客户、地址和订单信息的表：

| 订单ID | 客户ID | 客户名 | 省份   | 城市   | 区      | 街道        | 门牌号 | 商品   | 数量 |
|--------|--------|--------|--------|--------|---------|-------------|--------|--------|-----|
| 1      | 1001   | 张三   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    | 手机   | 1   |
| 2      | 1002   | 李四   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  | 电脑   | 2   |

**问题**：地址信息（省份、城市、区、街道、门牌号）依赖于客户ID，而不是直接依赖于订单ID，存在传递依赖。

#### 转换（满足第三范式）

将地址信息拆分到单独的表中：

- 客户表：

| 客户ID | 客户名 |
|--------|--------|
| 1001   | 张三   |
| 1002   | 李四   |

- 地址表：

  | 客户ID | 省份   | 城市   | 区      | 街道        | 门牌号 |
    |--------|--------|--------|---------|-------------|--------|
  | 1001   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    |
  | 1002   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  |

- 订单表：

  | 订单ID | 客户ID | 商品   | 数量 |
    |--------|--------|--------|------|
  | 1      | 1001   | 手机   | 1    |
  | 2      | 1002   | 电脑   | 2    |

### 去范式化

**定义**：为了提高查询性能或简化查询，故意引入冗余字段，即使这会违反规范化原则。

#### 示例（去范式化）

为了提高查询性能，我们可以将客户名和地址信息冗余到订单表中：

- 订单表（去范式化，增加冗余字段）：

  | 订单ID | 客户ID | 客户名 | 省份   | 城市   | 区      | 街道        | 门牌号 | 商品   | 数量 |
  |--------|--------|--------|--------|---------|-------------|--------|--------|------|---|
  | 1      | 1001   | 张三   | 北京市 | 北京市 | 海淀区  | 中关村南大街 | 1号    | 手机   | 1    |
  | 2      | 1002   | 李四   | 上海市 | 上海市 | 浦东新区 | 世纪大道    | 100号  | 电脑   | 2    |

### 去范式化的优劣势

**优势**：

1. **提高查询性能**：可以避免多表连接操作，直接在订单表中获取所需信息。
2. **简化查询**：查询变得更简单，因为所需的信息都可以直接从订单表中获取。

**劣势**：

1. **数据冗余**：由于订单表中包含冗余信息（客户名、地址信息），数据冗余增加，导致存储空间增加。
2. **数据一致性问题**：一旦客户或者地址信息发生变化，必须同步更新订单表中的冗余字段，否则可能导致数据不一致。

### 示例查询

**规范化查询**：

```sql
SELECT o.订单ID, c.客户名, a.省份, a.城市, a.区, a.街道, a.门牌号, o.商品, o.数量
FROM 订单表 o
JOIN 客户表 c ON o.客户ID = c.客户ID
JOIN 地址表 a ON c.客户ID = a.客户ID
WHERE o.订单ID = 1;
```

**去范式化查询**：

```sql
SELECT 订单ID, 客户名, 省份, 城市, 区, 街道, 门牌号, 商品, 数量
FROM 订单表
WHERE 订单ID = 1;
```

通过这种方式，我们展示了在第一范式、第二范式和第三范式的基础上进行数据库设计的过程，以及为了提高性能和简化查询而进行的去范式化设计。

# JVM
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



# Spring
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

## SpringBoot管理版本依赖
以SpringBoot基于Maven管理版本依赖为例，下面是基本工作原理：

1. **父级`pom.xml`文件**：Spring Boot项目通常会继承自Spring Boot提供的父级`pom.xml`文件。这个父级`pom.xml`文件定义了大多数常用库的版本号，比如Spring Framework、Spring Boot Starter等。通过继承这个父级`pom.xml`，你的项目可以继承父级中定义的依赖版本。

2. **Starter依赖项**：Spring Boot提供了一系列的"Starter"依赖项，这些依赖项通常以`spring-boot-starter-`为前缀，比如`spring-boot-starter-web`、`spring-boot-starter-data-jpa`等。当你在项目的`pom.xml`文件中添加这些Starter依赖项时，实际上是引入了一组预定义的依赖关系，这些依赖关系已经定义了特定功能的依赖库以及它们的版本号。

3. **依赖传递**：当添加了一个Starter依赖项时，它会自动引入一系列的传递性依赖，这些依赖通常是你项目所需的库和框架。这些传递性依赖也都是由Spring Boot的父级`pom.xml`文件中定义的版本管理的。

4. **版本冲突解决**：如果在项目中添加了一个依赖项，并且这个依赖项与Spring Boot的父级`pom.xml`中已定义的依赖版本发生冲突，Maven会自动解决这些冲突。通常情况下，它会选择与父级`pom.xml`中定义的版本兼容的最新版本，以确保整个项目的稳定性和兼容性。

总之，Spring Boot通过Maven的依赖管理机制，通过父级`pom.xml`文件中定义的版本号以及Starter依赖项，简化了依赖管理的过程，同时确保了项目中的库和框架的版本兼容性。

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

# Other
## Integer和int 的区别
### 关键点
1. **堆栈存储基础数据类型与对象**：
    - `int` 是基本数据类型，直接存储数值在栈上。
    - `Integer` 是 `int` 的包装类，存储在堆中，通过引用指向实际对象。

2. **值比对的时候注意java的自动拆箱**：
    - 自动拆箱是指Java自动将 `Integer` 对象转换为 `int` 基本类型。
    - 在进行值比较时，如果涉及 `Integer` 和 `int` 的混合比较，Java会自动进行拆箱操作，从而可能会影响结果。
    ```java
    Integer a = new Integer(5);
    int b = 5;
    if (a == b) {
        // 这里的 a 会被自动拆箱为 int 类型，然后进行值比较
        System.out.println("a 和 b 相等");
    }
    ```

3. **Integer 值大小在 -128 到 127 之内使用 IntegerCache**：
    - 为了优化性能，Java引入了 `IntegerCache`。对于范围在 -128 到 127 之间的整数，`Integer` 使用缓存来重用对象。
    - 当 `Integer` 值在这个范围内时，不会创建新的对象，而是从缓存中返回现有的对象。
    ```java
    Integer a = 127;
    Integer b = 127;
    if (a == b) {
        // a 和 b 指向同一个缓存对象
        System.out.println("a 和 b 引用相等");
    }
    
    Integer c = 128;
    Integer d = 128;
    if (c != d) {
        // c 和 d 指向不同的对象，因为 128 不在缓存范围内
        System.out.println("c 和 d 引用不相等");
    }
    ```

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

