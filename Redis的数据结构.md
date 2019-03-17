# Redis的数据结构

redis是一种高级的key-value的存储系统，其中value支持五种数据类型。

- 字符串（String）
- 哈希（hash）
- 字符串列表（list）
- 字符串集合（set）
- 有序字符串集合（sorted set）

## 1. 存储字符串String
字符串类型是Redis中最为基础的数据存储类型，它在Redis中是二进制安全的，这便意味着该类型可以接受任何格式的数据，如JPEG图像数据或json对象描述信息等。在Redis中字符串类型的value最多可以容纳的数据长度是512M。
![String](https://upload-images.jianshu.io/upload_images/15060863-56fd5c5fc48cd9ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下是常用命令：

- 赋值
  设定key持有指定的字符串value，如果该key存在则进行覆盖操作。总是返回OK。

```
[root@itzhouq32 bin]# ./redis-cli 
127.0.0.1:6379> set username tom
OK
```

- 取值
  获取key的value。如果与该key关联的value不是String类型，redis将返回错误信息，因为get命令只能用于获取String value；如果key不存在，返回nil。
```
127.0.0.1:6379> get username
"tom"
127.0.0.1:6379> get xxx
(nil)
```
- 先取值再赋值
  getset key value ：先获取该key的值，然后再设置该key的值。
```
127.0.0.1:6379> getset username jack
"tom"
127.0.0.1:6379> get username
"jack"
127.0.0.1:6379>
```
-  删除
  del key ：删除指定key
```
127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> del age
(integer) 1
127.0.0.1:6379> get age
(nil)
127.0.0.1:6379> 
```
- 数值增减
  incr key : 将指定的key的value原子性的增加1。如果该key不存在，其初始值为0。在incr之后其值为1.如果value的值不能转成整型，该操作将执行失败并返回相应的错误信息。
```
127.0.0.1:6379> incr num
(integer) 1
127.0.0.1:6379> get num 
"1"
127.0.0.1:6379> incr num
(integer) 2
127.0.0.1:6379> get num
"2"
127.0.0.1:6379> incr hello
(integer) 1
127.0.0.1:6379> get hello
"1"
127.0.0.1:6379> set world china
OK
127.0.0.1:6379> incr world
(error) ERR value is not an integer or out of range
```
decr key : 将指定的key的value原子性的递减1.如果该key不存在，其初始值为0，在incr之后其值为-1.如果value的值不能转成整型，该操作将执行失败并返回相应的错误信息。
```
127.0.0.1:6379> decr num2
(integer) -1
127.0.0.1:6379> get num2
"-1"
```
incrby increment : 将指定的key的value原子性的增加increment。如果key不存在，其初始值为0.如果该值不能转化为整型，报错。
```
127.0.0.1:6379> incrby date 10
(integer) 10
127.0.0.1:6379> get date
"10"
```
decrby decrement ：将指定的值原子性减少。
```
127.0.0.1:6379> decrby day 5
(integer) -5
127.0.0.1:6379> get day
"-5"
```
- 拼接字符串
  append key value : 拼接字符串。如果key存在，则在原有的基础上追加该值；如果该key不存在，则重新创建一个key/value。返回值为字符串的长度。
```
127.0.0.1:6379> append username 123
(integer) 7
127.0.0.1:6379> get username
"jack123"
127.0.0.1:6379> append yyy 123
(integer) 3
127.0.0.1:6379> get yyy
"123"
```


## 2. 存储Hash
Redis中的Hash类型可以看成具有String key 和String value的map容器。所以该类型非常适合于存储对象的信息。如Username、password和age。如果Hash中包含少量的字段，那么该类型的数据也将仅占用很少的磁盘空间。
![Hash](https://upload-images.jianshu.io/upload_images/15060863-f91f51e2f18df001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


常用命令：

- 赋值
  hset key field value : 为指定的key设定field/value对（键值对）。
  hmset key field value [field2 value2 ......] : 设置key中多个field/value。
```
127.0.0.1:6379> hset myhash username jack
(integer) 1
127.0.0.1:6379> hset myhash age 18
(integer) 1
127.0.0.1:6379> hmset myhash2 username rose age 21
OK
```
- 取值
  hget key field : 返回指定key中的field的值
  hmget key field : 获取key中的多个field的值
  hgetall key : 获取key中所有的field-value
```
127.0.0.1:6379> hget myhash username
"jack"
127.0.0.1:6379> hmget myhash username age
1) "jack"
2) "18"
127.0.0.1:6379> hgetall myhash
1) "username"
2) "jack"
3) "age"
4) "18"
```
- 删除
  hdel key field [field ...] ： 可以删除一个或者多个字段，返回值是被删除的字段的个数。
  del key : 删除整个hash
```
127.0.0.1:6379> hgetall myhash2
1) "username"
2) "rose"
3) "age"
4) "21"
127.0.0.1:6379> hdel myhash2 username age
(integer) 2
127.0.0.1:6379> hgetall myhash2
(empty list or set)
```
```
127.0.0.1:6379> hmset myhash3 username marry age 20 addr shanghai
OK
127.0.0.1:6379> hgetall myhash3
1) "username"
2) "marry"
3) "age"
4) "20"
5) "addr"
6) "shanghai"
127.0.0.1:6379> del myhash3
(integer) 1
127.0.0.1:6379> hgetall myhash3
(empty list or set)
```
- 增加数字
  hincrby key field increment : 设置key中field的值增加increment。
```
127.0.0.1:6379> hget myhash age
"18"
127.0.0.1:6379> hincrby myhash age 5
(integer) 23
127.0.0.1:6379> hget myhash age
"23"
127.0.0.1:6379> hincrby myhash age -10
(integer) 13
127.0.0.1:6379> hget myhash age
"13"
```
- 判断
  hexists key field ： 判断指定的key中的field是否存在
```
127.0.0.1:6379> hexists myhash password
(integer) 0
127.0.0.1:6379> hexists myhash username
(integer) 1
```
- 获取key所包含的field的数量
```
127.0.0.1:6379> hgetall myhash
1) "username"
2) "jack"
3) "age"
4) "13"
127.0.0.1:6379> hlen myhash
(integer) 2
```
- 获取所有的ekey
  hkey key
```
127.0.0.1:6379> hkeys myhash
1) "username"
2) "age"
```
- 获取所有的value
```
127.0.0.1:6379> hvals myhash
1) "jack"
2) "13"
```

## 3. 存储List类型
在Redis中，List类型是按照插入顺序排序的字符串链表。和数据结构中的普通链表	一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。

![Lists](https://upload-images.jianshu.io/upload_images/15060863-2a9ddd06528e3c2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 从元素插入和删除的效率视角来看，如果我们是在链表的两头插入或删除元素，这将会是非常高效的操作，即使链表中已经存储了百万条记录，该操作也可以在常量时间内完成。然而需要说明的是，如果元素插入或删除操作是作用于链表中间，那将会是非常低效的。相信对于有良好数据结构基础的开发者而言，这一点并不难理解。

- ArrayList使用数组的方式存储数据，所以根据索引查询数据速度快，而新增或者删除元素时，涉及到位移操作，所以比较慢。

- LinkedList使用双向链表，每一个元素都记录前后元素的指针，所以插入、删除数据时只是更改前后元素的指针指向即可，速度非常快。然后通过记录下标查询元素时需要从头开始索引，所以比较慢。

以下为常用命令：

- 查看列表
  lrange key start end : 获取链表中从start到end的元素的值，start、end从0开始计数；也可以为负数，若为-1则表示链表尾部的元素。
```
127.0.0.1:6379> lrange mylist 0 -1
1) "c"
2) "b"
3) "a"
```
![从左向右添加时](https://upload-images.jianshu.io/upload_images/15060863-7f71e8c4035bfb5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- 两端添加
  lpush key value [value1  value2]... : 在指定的key所关联的list的头部插入所有的values，如果该key不存在，该命令在插入前创建一个与该key关联的空链表，之后再向链表的头部插入数据。插入成功，返回元素的个数。

  rpush key value [value1  value2]... : 在该list的尾部添加元素。
```
127.0.0.1:6379> lpush mylist2 a b c d 
(integer) 4
127.0.0.1:6379> rpush mylist2 1 2 3
(integer) 7
127.0.0.1:6379> lrange mylist2 0 -1
1) "d"
2) "c"
3) "b"
4) "a"
5) "1"
6) "2"
7) "3"
```
![image.png](https://upload-images.jianshu.io/upload_images/15060863-cfde415442450501.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 两端弹出
  lpop key ： 返回并弹出指定key关联得链表中得第一个元素，即头部元素。如果该key不存在，返回nil。
  rpop key : 从尾部弹出元素。
```
127.0.0.1:6379> lpop mylist2
"d"
127.0.0.1:6379> lrange mylist2 0 -1
1) "c"
2) "b"
3) "a"
4) "1"
5) "2"
6) "3"
127.0.0.1:6379> rpop mylist2
"3"
127.0.0.1:6379> lrange mylist2 0 -1
1) "c"
2) "b"
3) "a"
4) "1"
5) "2"
```
## 4. 存储set类型
在redis中，可以将Set类型看作是没有排序的字符集合，和List类型一样，我们也可以在该类型的数值上执行添加、删除和判断某一元素是否存在等操作。这些操作的时间复杂度为O(1),即常量时间内完成依次操作。

和List类型不同的是，Set集合中不允许出现重复的元素。

常用命令：

- 增加/删除元素
  sadd key values[ value1 value2...] : 向set中添加数据，如果key的值已经存在则不会重复添加。
  srem key members[member1 member2...] ： 删除set中指定的成员
```
127.0.0.1:6379> sadd myset a b c
(integer) 3
127.0.0.1:6379> srem myset 1 2 
(integer) 0
127.0.0.1:6379> srem myset a b 
(integer) 2
```
- 获得集合的元素
  smembers key : 获取set中所有的成员
  sismember key member：判断参数指定的成员是否在该set中，1表示存在，0表示不存在，或者该key本身不存在。
```
127.0.0.1:6379> smembers myset
1) "c"
127.0.0.1:6379> sismember myset a
(integer) 0
127.0.0.1:6379> sismember myset c
(integer) 1
```
- 集合的差集运算
  sdiff key1 key2...:返回key1与key2中相差的成员，而且与key的顺序有关，即返回差集。
```
127.0.0.1:6379> sadd mya1 a b c
(integer) 3
127.0.0.1:6379> sadd myb1 a c 1 2
(integer) 4
127.0.0.1:6379> sdiff mya1 myb1
1) "b"
```
- 集合的交集运算
  sinter key1 key2 key3 : 返回交集
```
127.0.0.1:6379> smembers mya1
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> smembers myb1
1) "a"
2) "1"
3) "2"
4) "c"
127.0.0.1:6379> sinter mya1 myb1
1) "a"
2) "c"
```
- 集合的并集运算
  sunion key1 key2 key3 ....:返回并集
```
127.0.0.1:6379> sunion mya1 myb1
1) "1"
2) "b"
3) "c"
4) "a"
5) "2"
```
- set的使用场景
  可以使用Redis的set数据类型跟踪一些唯一性数据，比如访问某一个博客的唯一IP地址信息。对于此场景，我们仅需在每次访问该博客时将访问者的IP存入redis中，set集合类型会自动保证IP地址的唯一性。

充分利用Set类型的服务端聚合操作方便、高效的特性，可以用于维护数据对象之间的关联关系。比如所有购买某一电子设备的客户ID被存储在一个特定的set中，而购买另一种电子产品的客户ID被存储在另外一个set中，如果此时我们想获取有哪些客户同时购买了这两种商品时，set的一些命令可以充分发挥优势了。

## 5. 存储sortedset类型
sortedset和set极为相似，他们都是字符串的集合，都不允许重复的成员出现在一个set中。他们之间的主要差别是sortedset中每一个成员都会有一个分数与之关联。redis正是通过分数来为集合的成员进行从小到大的排序。sortedset中分数是可以重复的。

常见命令：
 - 添加元素
    zadd key score member score2 member2...:将成员以及该成员的分数存放到sortedset中。如果该元素已经存在则会用新的分数替换原有的分数。返回值是新加入到集合的元素的个数，不包含之前已经存在的元素。
```
127.0.0.1:6379> zadd mysort 70 zhangsan 80 lisi 60 wangwu
(integer) 3
127.0.0.1:6379> zadd mysort 100 zhangsan
(integer) 0
```
- 获取元素
  zscore key member :返回指定成员的分数
```
127.0.0.1:6379> zscore mysort zhangsan
"100"
```
zcard key : 获取集合中成员数量
```
127.0.0.1:6379> zcard mysort
(integer) 3
```
- 删除元素
  zrem key member [member...] : 移除集合中指定的成员，可以指定多个成员
```
127.0.0.1:6379> zrem mysort wangwu lisi
(integer) 2
```
- 范围查询
  zrange key start end [withscores] : 获取集合中脚注为start-end的成员，[withscores]参数表明返回的成员包含其分数。
```
27.0.0.1:6379> zadd mysort 20 qianqi 89 zhaoba
(integer) 2
127.0.0.1:6379> zadd mysort 50 jack 98 rose
(integer) 2
127.0.0.1:6379> zrange mysort 0 -1
1) "qianqi"
2) "jack"
3) "zhaoba"
4) "rose"
5) "zhangsan"
127.0.0.1:6379> zrange mysort 0 -1 withscores
 1) "qianqi"
 2) "20"
 3) "jack"
 4) "50"
 5) "zhaoba"
 6) "89"
 7) "rose"
 8) "98"
 9) "zhangsan"
10) "100"
```
zrevrange key start stop [withscores] : 按照分数从大到小的顺序返回索引从start到stop之间的所有元素（包含两端的元素）。
```
127.0.0.1:6379> zrevrange mysort 0 -1 withscores
 1) "zhangsan"
 2) "100"
 3) "rose"
 4) "98"
 5) "zhaoba"
 6) "89"
 7) "jack"
 8) "50"
 9) "qianqi"
10) "20"
```
zremrangebyrank key start stop : 按照排名范围删除元素
```
127.0.0.1:6379> zremrangebyrank mysort 0 4
(integer) 5
```
- 使用场景
  可以用于一个大型在线游戏的积分排行榜。每当玩家的分数发生变化时，可以执行zadd命令更改新玩家的分数，此后再通过zrange命令获取积分top10的用户信息。

sortedset类型还可以用于构建索引数据。

