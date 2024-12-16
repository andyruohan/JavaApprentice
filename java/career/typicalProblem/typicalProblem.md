### 如何解决大文件上传问题？（分片上传）
使用分片上传主要有下面 2 点好处：
- 断点续传 ：上传文件中途暂停或失败（比如遇到网络问题）之后，不需要重新上传，只需要上传那些未成功上传的文件分片即可。所以，分片上传是断点续传的基础。
- 多线程上传 ：我们可以通过多线程同时对一个文件的多个文件分片进行上传，这样的话就大大加快的文件上传的速度。
![](分片上传样例图.png)

#### 前端怎么生成文件分片呢？
前端可以通过 Blob.slice() 方法来对文件进行切割（File 对象是继承 Blob 对象的，因此 File 对象也有 slice() 方法）。
生成文件切片的示例代码如下：
![](前端代码.png)

#### 后端如何合并文件分片呢？
RandomAccessFile 类可以帮助我们合并文件分片，示例代码如下：
![](后端代码.png)

#### 何为秒传？  
秒传说的就是我们在上传某个文件的时候，首先根据文件的唯一标识判断一下服务端是否已经上传过该文件，如果上传过的话，直接就返回给用户文件上传成功即可。  
一般情况下，这个唯一标识都是通过对文件的名称、最后修改时间等信息取 MD5 值得到的，这个可以通过使用 spark-md5 这个库来生成。   
另外，还存在一种情况是我们要上传的文件已经上传了部分文件切片到服务端。这个时候，我们直接返回已上传的切片列表给前端即可。然后，前端再将剩余未上传的分片上传到服务端。
>需要注意的是：你不能根据文件名就决定文件是否已经上传到服务端，因为很可能存在文件名相同，但是，内容不同的情况。另外，体验更好的是文件内容不变，唯一标识就不应该改变。因此，我们可以根据文件的内容来计算 MD5 值。  


![](流程图.png)
>缺少每片数据的一致性校验和最后合包的一致性校验

参考链接：https://www.yuque.com/snailclimb/mf2z3k/akmquq

### 项目积累
#### 格式转换
参考本人GitHub链接：https://github.com/andyruohan/RTF2PDF.git

#### 科学计数问题
使用BigDecimal进行金额转String时，注意toString会产生科学计数，此时应使用toPlainString
![](产生科学记数法的问题点.PNG)

#### 或有金额计算问题
前台使用浮点数计算

#### MyBatis-plus 不会查出数据库为 [Null] 的数据？
[Null] 代表不确定的值（='a' 和 !='a' 都会不成立），数据库函数对 [Null] 和对 "" 的处理不一样，很多对 [Null] 不起作用。

#### 多线程使用样例
1. 格式转换多个word的启动
2. 一事通任务的派发
3. 异步事件表的运行

#### Spring AOP 切面问题
在Service类中，我有methodA和methodB两个方法，methodA中嵌套着methodB，我把切换数据源的切面@DataSoure加在methodA上可以生效，但加在methodB上不生效，如何解决这个问题呢？

### thymeleaf 版本问题，启动不报错，实际调用报错
Caused by: java.lang.NoClassDefFoundError: Could not initialise class ognl.OgnlRuntime

### RestTemplate 与 jackson-dataformat-xml
项目中使用的RestTemplate进行调用，当引入jackson-dataformat-xml后，RestTemplate的messageConverter里面会增加MappingJackson2XmlHttpMessageConverter，并且优先级高于MappingJackson2HttpMessageConverter，由于没有指定accept，导致调用的时候accept传递为xml

### 界面和后台二次校验
如扣费账号此前无筛选条件，但后期迭代有了过滤条件，需考虑暂存回显的情况

### Java程序DateTimeFormatter使用"YYYY"
DateTimeFormatter使用"YYYY"，代表基于周的年份，如果日期跨年，则年份不正确。如2024年12月31日，使用YYYY-MM-dd格式化后的日期变为2025-12-31。
