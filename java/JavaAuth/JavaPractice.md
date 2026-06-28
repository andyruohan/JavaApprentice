## Kafka
### Zookeeper 管理的元数据
#### 题目
以下哪项不属于Zookeeper管理的Kafka元数据？
A. Controller选举信息，包括当前Controller Broker ID
B. Broker注册信息，包括Broker ID和网络地址
C. 消费者组的offset位移信息
D. Topic配置信息，包括分区数量和副本分配

#### 答案
**C**

#### 精简解析
1. **A 属于ZK管理**
   Controller主节点选举信息持久化在ZK，集群依靠ZK完成主节点切换。

2. **B 属于ZK管理**
   Broker启动后在ZK注册节点，记录ID、IP端口，下线节点自动清除。

3. **C 不属于ZK管理**
   Kafka 0.9及以上版本，消费offset统一存内置主题`__consumer_offsets`，不再保存在ZK。

4. **D 属于ZK管理**
   Topic分区、副本数量与分配方案全部记录在ZK中。

#### 考点小结
传统ZK架构下，集群、Broker、Topic、Controller元数据由ZK维护；消费者偏移量由Kafka内部主题保存。

### Zookeeper 重新选举
#### 题目
Kafka在高可用环境中使用Zookeeper来管理集群状态。当Kafka集群中的控制器节点故障时，以下哪个过程会在Zookeeper的帮助下触发，以确保集群恢复正常？
A. Kafka所有Broker自动重启，重新加入集群
B. 由Zookeeper重新选举一个新的控制器节点来承担控制器角色
C. Zookeeper将所有现有分区的领导者切換到不同的Broker，以确保消息可用
D. Kafka生产者和消费者立即停止所有消息操作，等待手动恢复控制器

#### 正确答案
**B**

#### 简洁解析
1. **A错误**
   控制器故障不会让全部Broker重启，无该机制。
2. **B正确**
   控制器在ZK抢占临时节点，原控制器宕机后临时节点消失，其他Broker竞争创建节点，完成新控制器选举。
3. **C错误**
   分区leader切换是新控制器上线后执行的操作，不是ZK直接完成。
4. **D错误**
   集群会自动选新控制器，无需手动恢复，生产消费不会全部暂停。

#### 考点总结
ZK通过临时节点实现Kafka控制器自动重新选举，保障集群管理功能正常。