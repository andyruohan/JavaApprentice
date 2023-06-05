### mac 电脑 redis 的安装
1. 安装 redis
```
brew install redis
```
一般安装成功后，程序文件存放目录: /usr/local/opt/redis，配置文件存放目录: /usr/local/etc/...。
2. 添加 path 
```
vim ~/.bash_profile
```
如果使用的是默认安装路径，一般安装在/usr/local/opt/redis。

```
export REDIS = /usr/local/opt/reids/bin;
export = $REDIS:$PATH
```
3. 启动 redis-server  
```
brew services start redis
```
未进入 redis 命令行前，可通过查看如下代码查看 redis 版本
``` 
redis-server --version
```
4. 进入 redis 命令行
```
redis-cli
```
进入 redis 命令行后，可通过查看如下代码查看 redis 版本
``` 
INFO server
```

![](redis安装成功查看版本号.png)


| 商品操作                          | redis命令                               |
|:------------------------------|:--------------------------------------|
| 商品编号1001的销量是9，商品编号1002的销量是15  | zadd goods:sellsort 9 1001 15 1002    |
| 有一个客户又买了2件商品1001，商品编号1001销量加2 | zincrby goods:sellsort 2 1001         |
| 求商品销量前10名                     | zrange goods:sellsort 0 10 withscores |

```
127.0.0.1:6379> zadd goods:sellsort 9 1001 15 1002
(integer) 2
127.0.0.1:6379> zrange goods:sellsort 0 10 withscores
1) "1001"
2) "9"
3) "1002"
4) "15"
127.0.0.1:6379> zincrby goods:sellsort 2 1001
"11"
127.0.0.1:6379> zrange goods:sellsort 0 10 withscores
1) "1001"
2) "11"
3) "1002"
4) "15"
```

### redis实践
```java {.line-numbers}
@RestController
public class GoodsController {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Value("${server.port}")
    private String severPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        String result = stringRedisTemplate.opsForValue().get("goods:001");
        int goodsNumber = result == null ? 0: Integer.parseInt(result);

        if (goodsNumber > 0) {
            int realNumber = goodsNumber - 1;
            stringRedisTemplate.opsForValue().set("goods:001", String.valueOf(realNumber));
            System.out.println("成功买到商品，库存还剩下" + realNumber + "件" + "\t服务提供端口 " + severPort);

            return "成功买到商品，库存还剩下" + realNumber + "件" + "\t服务提供端口 " + severPort;
        } else {
            System.out.println("商品已经售完/活动结束/调用超时，欢迎下次光临" + "\t服务提供端口 " + severPort);
        }
        return "商品已经售完/活动结束/调用超时，欢迎下次光临" + "\t服务提供端口 " + severPort;
    }
}
```
运行结果：
![](redis_demo运行结果.png)

以上代码在高并发环境会出现的问题：
1. 11~12行非原子性
2. 14行条件会被越过

##### 错误优化：加synchronized
```java {.line-numbers}
@RestController
public class GoodsController {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Value("${server.port}")
    private String severPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        synchronized (this) {
            String result = stringRedisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0: Integer.parseInt(result);

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                stringRedisTemplate.opsForValue().set("goods:001", String.valueOf(realNumber));
                System.out.println("成功买到商品，库存还剩下" + realNumber + "件" + "\t服务提供端口 " + severPort);

                return "成功买到商品，库存还剩下" + realNumber + "件" + "\t服务提供端口 " + severPort;
            } else {
                System.out.println("商品已经售完/活动结束/调用超时，欢迎下次光临" + "\t服务提供端口 " + severPort);
            }
            return "商品已经售完/活动结束/调用超时，欢迎下次光临" + "\t服务提供端口 " + severPort;
        }
    }
}
```
在11行处添加synchronized (this)，实际生产环境解决不了高并发问题：因为服务器端会通过nginx代理多台服务器，高并发的请求还是会进来，处理不了`超卖`、`多卖`的情况。
![](实际生产部署拓扑样例.png)

> 加锁场景拓展补充：  
> synchronized：不见不散，适用场景：等不到就一直等
reentrantLock：过时不候，适用场景：等了一段时间不想等了

