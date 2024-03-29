## 系统设计怎么做？
### Step1:问清楚系统具体要求
### Step2:对系统进行抽象设计
### Step3:考虑系统目前需要优化的点
### Step4:优化你的系统抽象设计

大型网站架构设计必备的三板斧：
高性能架构设计：熟悉系统常见性能优化手段比如引入 读写分离、缓存、负载均衡、异步 等等。  
高可用架构设计：CAP理论和BASE理论、通过集群来提高系统整体稳定性、超时和重试机制、应对接口级故障：降级、熔断、限流、排队。  
高扩展架构设计：说白了就是懂得如何拆分系统。你按照不同的思路来拆分软件系统，就会得到不同的架构。  


### 性能相关的指标
#### 响应时间
响应时间RT(Response-time)就是用户发出请求到用户收到系统处理结果所需要的时间。
RT是一个非常重要且直观的指标，RT数值大小直接反应了系统处理用户请求速度的快慢。
#### 并发数
并发数可以简单理解为系统能够同时供多少人访问使用也就是说系统同时能处理的请求数量。
并发数反应了系统的负载能力。
#### QPS 和 TPS
- QPS（Query Per Second） ：服务器每秒可以执行的查询次数；
- TPS（Transaction Per Second） ：服务器每秒处理的事务数（这里的一个事务可以理解为客户发出请求到收到服务器的过程）；
书中是这样描述 QPS 和 TPS 的区别的。  

>QPS vs TPS：QPS 基本类似于 TPS，但是不同的是，对于一个页面的一次访问，形成一个TPS；但一次页面请求，可能产生多次对服务器的请求，服务器对这些请求，就可计入“QPS”之中。如，访问一个页面会请求服务器2次，一次访问，产生一个“T”，产生2个“Q”。  
 
#### 吞吐量
**吞吐量指的是系统单位时间内系统处理的请求数量。**  

一个系统的吞吐量与请求对系统的资源消耗等紧密关联。请求对系统资源消耗越多，系统吞吐能力越低，反之则越高。
TPS、QPS都是吞吐量的常用量化指标。  

- QPS（TPS） = 并发数/平均响应时间(RT)
- 并发数 = QPS * 平均响应时间(RT)  

### 系统活跃度
介绍几个描述系统活跃度的常见名词，建议牢牢记住。你不光会在回答系统设计面试题的时候碰到，日常工作中你也会经常碰到这些名词。
#### PV(Page View)
访问量, 即页面浏览量或点击量，衡量网站用户访问的网页数量；在一定统计周期内用户每打开或刷新一个页面就记录1次，多次打开或刷新同一页面则浏览量累计。UV 从网页打开的数量/刷新的次数的角度来统计的。
#### UV(Unique Visitor)
独立访客，统计1天内访问某站点的用户数。1天内相同访客多次访问网站，只计算为1个独立访客。UV 是从用户个体的角度来统计的。
#### DAU(Daily Active User)
日活跃用户数量。
#### MAU(monthly active users)
月活跃用户人数。
举例：某网站 DAU为 1200w， 用户日均使用时长 1 小时，RT为0.5s，求并发量和QPS。  
平均并发量 = DAU（1200w）* 日均使用时长（1 小时，3600秒） /一天的秒数（86400）=1200w/24 = 50w  
真实并发量（考虑到某些时间段使用人数比较少） = DAU（1200w）* 日均使用时长（1 小时，3600秒） /一天的秒数-访问量比较小的时间段假设为8小时（57600）=1200w/16 = 75w  
峰值并发量 = 平均并发量 * 6 = 300w  
QPS = 真实并发量/RT = 75W/0.5=150w/s
