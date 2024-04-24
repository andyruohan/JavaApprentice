## mysql 最左前缀法则示例
假设index(a,b,c)  

| Where语句                                                | 索引是否被使用                                                             |
|--------------------------------------------------------|---------------------------------------------------------------------|
| where a = 3                                            | 是，使用到 a                                                             |
| where a =  3 and b = 5                                 | 是，使用到 a、b                                                           |
| where a =  3 and b = 5 and c = 4                       | 是，使用到 a、b、c                                                         |
| where b =  3 或者 where b = 3 and c =  4 或者 where c =  4 | 否                                                                   |
| where a =  3 and c = 5                                 | 使用到 a，但是 c 不可以，b 中间断了                                               |
| where a =  3 and b > 4 and c = 5                       | 使用到 a、b 和 c 不能用在范围之后，b断了                                            |
| where a is  null and b is not null                     | is null 支持索引，但是 is not null 不支持，所以 a 可以使用索引，但是 b 不一定能用上索引（mysql8.0） |
| where a  <> 3                                          | 不能使用索引                                                              |
| where  abs(a) =3                                       | 不能使用索引                                                              |
| where a =  3 and b like 'kk%' and c = 4                | 是，使用到 a、b、c                                                         |
| where a =  3 and b like '%kk' and c = 4                | 是，只用到 a                                                             |
| where a =  3 and b like '%kk%' and c = 4               | 是，只用到 a                                                             |
| where a =  3 and b like 'k%kk%' and c =  4             | 是，使用到 a、b、c                                                         |

可以结合字典排序来理解:
1) 如`where a =  3 and c = 5`，就类似于查找单词找`a%c`，由于中间这一段不知道是啥，因此只能根据索引找到`a`，剩余的只能通过全表扫描。
2) 如`where a =  3 and b > 4 and c = 5`，就类似查找单词，先根据索引找到字母为`a`的，然后根据索引找到字母大于`b`，但在大于`b`后再找字母为`c`的就很难找了，需要对`ac、ae、...、az`打头的单词逐个核对。
3) 如`where a  <> 3`，就类似于查找单词，除`a`打头的单词全部查找一遍。