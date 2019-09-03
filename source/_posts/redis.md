---
title: redis
tags: redis
date: 2018-12-06 18:10:43
---

# redis是什么
redis是一个开源的，使用C语言编写的，支持网络交互的、可基于内存也可持久化的Key-Value数据库

# 安装过程
待完善...

# linux操作redis

## 连接redis
`./redis-cli`
`127.0.0.1:6379>`

## 配置redis
redis的配置文件位于Redis安装目录下，文件名为 redis.conf，可以通过CONFIG命令查看或者设置配置项。
```
127.0.0.1:6379> config get loglevel
1) "loglevel"
2) "notice"
```
使用`*`获取所有配置项
`config get *`
很多配置，这里就不贴了。[参考这里](https://www.cnblogs.com/ruanraun/p/redis.html)
可以通过修改redis.conf文件或者使用config set 命令来修改配置
```
127.0.0.1:6379> config set loglevel 'notice'
OK
```

## 如何停止/启动/重启redis服务
如果是用apt-get或者yum install安装的redis，可以直接通过下面的命令停止/启动/重启redis
```
/etc/init.d/redis-server stop
/etc/init.d/redis-server start
/etc/init.d/redis-server restart
```
如果是通过源码安装的redis，可以通过redis的客户端程序redis-cli的shutdown命令来停止redis

`redis-cli -h 127.0.0.1 -p 6379 shutdown`
或
`./redis-cli shutdown`
没错，虽然启动的是server，但是这里停止的是`redis-cli`
如果上述方式都没有成功停止redis，则可以使用终极武器 kill -9

启动redis：
`cd src`
`./redis-server`
上面这种启动redis使用的是默认配置，也可以通过启动参数告诉redis使用指定配置
`./redis-server ../redis.conf`

让redis后台运行
`./redis-server &`
然后输入`bg`

# redis命令
[参考redis中文官网](https://www.redis.net.cn/)

## key
### del
del用于删除已存在的键，不存在的 key 会被忽略
`del key_name`
返回被删除key的数量

### dump
DUMP 命令用于序列化给定 key ，并返回被序列化的值
`DUMP KEY_NAME`
如果 key 不存在，那么返回 nil 。 否则，返回序列化之后的值。

### exist
EXISTS 命令用于检查给定 key 是否存在。
`EXISTS KEY_NAME`
若 key 存在返回 1 ，否则返回 0 。

### expire
Expire 命令用于设置 key 的过期时间。key 过期后将不再可用。
`Expire KEY_NAME TIME_IN_SECONDS`
设置成功返回1， 当key不存在或者不能为key设置过期时间时返回0。

### expireat
Expireat 命令用于以 UNIX 时间戳(unix timestamp)格式设置 key 的过期时间。key 过期后将不再可用。
`Expireat KEY_NAME TIME_IN_UNIX_TIMESTAMP`
设置成功返回1，不成功或不存在返回0。
示例：
`EXPIREAT w3ckey 1293840000`

### keys
Keys 命令用于查找所有符合给定模式 pattern 的 key
`KEYS PATTERN`
返回符合给定模式的 key 列表 (Array)。

查找以w3c开头的key：
`keys w3c*`
查找所有的key
`keys *`

### move
move命令用于将当前数据库的key移动到给定的数据库db当中。
`MOVE KEY_NAME DESTINATION_DATABASE`
移动成功返回 1 ，失败则返回 0 。

示例：
```
SELECT 0     # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
set w3c redis  #设置一个key
move w3c 1   #转移到1库
exist w3c  #发现w3c已经没有了
select 1 #选择1库
exist w3c #发现w3c已经转移到1库中
```
1. 不存在的key无法转移
2. 当源数据库和目标数据库有相同的 key 时，key无法转移。

### persist
persist命令用于移除给定 key 的过期时间，使得 key 永不过期。
`PERSIST KEY_NAME`
当过期时间移除成功时，返回 1 。 如果 key 不存在或 key 没有设置过期时间，返回 0 。

### pttl
Pttl 命令以毫秒为单位返回 key 的剩余过期时间。
`PTTL KEY_NAME`
当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。

### TTL
TTL 命令以秒为单位返回 key 的剩余过期时间。
`TTL KEY_NAME`
当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。

### randomkey
RANDOMKEY 命令从当前数据库中随机返回一个 key 。
`randomkey`
当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil 。

### rename
Rename 命令用于修改 key 的名称 。
`RENAME OLD_KEY_NAME NEW_KEY_NAME`
改名成功时提示 OK ，失败时候返回一个错误。
当 OLD_KEY_NAME 和 NEW_KEY_NAME 相同，或者 OLD_KEY_NAME 不存在时，返回一个错误。 
当 NEW_KEY_NAME 已经存在时， RENAME 命令将覆盖旧值。

### renamenx
Renamenx 命令用于在新的 key 不存在时修改 key 的名称 。
`RENAMENX OLD_KEY_NAME NEW_KEY_NAME`
修改成功时，返回 1 。 如果 NEW_KEY_NAME 已经存在，返回 0 。

### type
Type 命令用于返回 key 所储存的值的类型。
`TYPE KEY_NAME `
返回 key 的数据类型，数据类型有：
```
none (key不存在)
string (字符串)
list (列表)
set (集合)
zset (有序集)
hash (哈希表)
```

## string
### set
SET 命令用于设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型。
`SET KEY_NAME VALUE`
从 Redis 2.6.12 版本开始， SET 在设置操作成功完成时，才返回 OK 。

### get
Get 命令用于获取指定 key 的值。
`GET KEY_NAME`
如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误。

### getrange
getrange命令用于获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)。
`getrange KEY_NAME start end`

示例：首先，设置 mykey 的值并截取字符串。
```
redis 127.0.0.1:6379> SET mykey "This is my test key"
OK
redis 127.0.0.1:6379> GETRANGE mykey 0 3
"This"
redis 127.0.0.1:6379> GETRANGE mykey 0 -1
"This is my test key"
```

### getset
getset 命令用于设置指定 key 的值，并返回 key 旧的值。
`getset key_name value`
返回给定 key 的旧值。 当 key 没有旧值时，即 key 不存在时，返回 nil 。
当 key 存在但不是字符串类型时，返回一个错误。
示例：首先，设置 mykey 的值并截取字符串。
```
redis 127.0.0.1:6379> GETSET mynewkey "This is my test key"
(nil)
redis 127.0.0.1:6379> GETSET mynewkey "This is my new value to test getset"
"This is my test key"
```

### getbit
getbit 命令用于对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
`getbit key_name offset`
返回字符串值指定偏移量上的位(bit)。
当偏移量 OFFSET 比字符串值的长度大，或者 key 不存在时，返回 0 。

### setbit
setbit命令用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
`setbit key_name offset`
返回指定偏移量原来储存的位。

### mget
mget命令返回所有(一个或多个)给定 key 的值。 如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。
`MGET KEY1 KEY2 .. KEYN`

### setex
setex 命令为指定的 key 设置值及其过期时间。如果 key 已经存在， SETEX 命令将会替换旧的值。
`setex key_name timeout value`
设置成功时返回 OK 。
说白了就是在设置一个值的时候直接就指定了过期时间。

### setnx
setnx(SET if Not eXists)命令在指定的 key 不存在时，为 key 设置指定的值。
`setnx key_name value`
设置成功，返回 1 。 设置失败，返回 0 。
```
redis> EXISTS job                # job 不存在
(integer) 0
redis> SETNX job "programmer"    # job 设置成功
(integer) 1
redis> SETNX job "code-farmer"   # 尝试覆盖 job ，失败
(integer) 0
redis> GET job                   # 没有被覆盖
"programmer"
```

### setrange
setrange 命令用指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 offset 开始。
`SETRANGE KEY_NAME OFFSET VALUE`
返回被修改后的字符串长度。
就是从指定位置修改字符串的内容。

### strlen
strlen 命令用于获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误。
`strlen key_name`
返回字符串值的长度。 当 key 不存在时，返回 0。

### mset
mset 命令用于同时设置一个或多个 key-value 对。
`MSET key1 value1 key2 value2 .. keyN valueN `
总是返回OK

### msetnx
msetnx 命令用于所有给定 key 都不存在时，同时设置一个或多个 key-value 对。
`MSETNX key1 value1 key2 value2 .. keyN valueN `
当所有 key 都成功设置，返回 1 。 如果所有给定 key 都设置失败(至少有一个 key 已经存在)，那么返回 0 。

msetnx是原子性操作，只要有一个值失败，所有的值都不会被设置。

### psetex
psetex 命令以毫秒为单位设置 key 的生存时间。
`PSETEX key1 EXPIRY_IN_MILLISECONDS value1 `
设置成功时返回 OK 。

### incr
incr 命令将 key 中储存的数字值增一。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`incr key_name`
返回执行incr命令之后 key 的值。

### incrby
incrby命令将 key 中储存的数字加上指定的增量值。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCRBY 命令。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`incr key_name incr_amount`
返回加上指定的增量值之后， key 的值。

### incrbyfloat
incrbyfloat命令为 key 中所储存的值加上指定的浮点数增量值。
`incrbyfloat key_name incr_amount`
返回执行命令之后 key 的值。

用 SET 设置的值可以是指数符号`314e-2`，但执行 INCRBYFLOAT 之后格式会被改成非指数符号`3.14`
SET 设置的值小数部分可以是 0`3.0`，但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数`4`

### decr
decr 命令将 key 中储存的数字值减一。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECR 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`decr key_name`
返回执行命令之后 key 的值。

### decrby
decrby 命令将 key 所储存的值减去指定的减量值。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECRBY 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`decrby key_name decrement_amount`
返回减去指定减量值之后， key 的值。

### append
append 命令用于为指定的 key 追加值。
如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。
如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。
`append key_name new_value`
返回追加指定值之后， key 中字符串的长度。

## hash
### hdel
hdel 命令用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略。
`HDEL KEY_NAME FIELD1.. FIELDN `
被成功删除字段的数量，不包括被忽略的字段。

### hexists
hexists 命令用于查看哈希表的指定字段是否存在。
`HEXISTS KEY_NAME FIELD_NAME `
如果哈希表含有给定字段，返回 1 。 如果哈希表不含有给定字段，或 key 不存在，返回 0 。

### hget
hget 命令用于返回哈希表中指定字段的值。
`HGET KEY_NAME FIELD_NAME `
返回给定字段的值。如果给定的字段或 key 不存在时，返回 nil 。

### hgetall
hgetall 命令用于返回哈希表中，所有的字段和值。
在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍。
`HGETALL KEY_NAME `
以列表形式返回哈希表的字段及字段值。 若 key 不存在，返回空列表。

### hincrby
hincrby 命令用于为哈希表中的字段值加上指定增量值。
增量也可以为负数，相当于对指定字段进行减法操作。
如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。
如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。
对一个储存字符串值的字段执行 HINCRBY 命令将造成一个错误。
本操作的值被限制在 64 位(bit)有符号数字表示之内。
`HINCRBY KEY_NAME FIELD_NAME INCR_BY_NUMBER `

### hincrbyfloat
hincrbyfloat 命令用于为哈希表中的字段值加上指定浮点数增量值。
如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。
`HINCRBYFLOAT KEY_NAME FIELD_NAME INCR_BY_NUMBER `
返回执行 Hincrbyfloat 命令之后，哈希表中字段的值。

### hkeys
hkeys 命令用于获取哈希表中的所有字段名。
`HKEYS KEY_NAME FIELD_NAME INCR_BY_NUMBER `
包含哈希表中所有字段的列表。 当 key 不存在时，返回一个空列表。

### hlen
hlen 命令用于获取哈希表中字段的数量。
`HLEN KEY_NAME `
返回哈希表中字段的数量。 当 key 不存在时，返回 0 。

### hmget
hmget 命令用于返回哈希表中，一个或多个给定字段的值。
如果指定的字段不存在于哈希表，那么返回一个 nil 值。
`HMGET KEY_NAME FIELD1...FIELDN `
返回一个包含多个给定字段关联值的表，表值的排列顺序和指定字段的请求顺序一样。

### hmset
hmset 命令用于同时将多个 field-value (字段-值)对设置到哈希表中。
此命令会覆盖哈希表中已存在的字段。
如果哈希表不存在，会创建一个空哈希表，并执行 HMSET 操作。
`HMSET KEY_NAME FIELD1 VALUE1 ...FIELDN VALUEN`

### hset
hset 命令用于为哈希表中的字段赋值 。
如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。
如果字段已经存在于哈希表中，旧值将被覆盖。
`HSET KEY_NAME FIELD VALUE`
返回如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1 。 如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0 。


### hsetnx
hsetnx 命令用于为哈希表中不存在的的字段赋值 。
如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。
如果字段已经存在于哈希表中，操作无效。
如果 key 不存在，一个新哈希表被创建并执行 HSETNX 命令。
`HSETNX KEY_NAME FIELD VALUE`
设置成功，返回 1 。 如果给定字段已经存在且没有操作被执行，返回 0 。

### hvals
hvals 命令返回哈希表所有字段的值。
`HVALS KEY_NAME FIELD VALUE`
返回一个包含哈希表中所有值的表。 当 key 不存在时，返回一个空表。

# JAVA操作redis基本使用
## 初始化
``` java
public class RedisClient {

private Jedis jedis;// 非切片客户端连接
private JedisPool jedisPool;// 非切片连接池
private ShardedJedis shardedJedis;// 切片客户端连接
private ShardedJedisPool shardedJedisPool;// 切片连接池

public RedisClient() {
	initialPool();
	initialShardedPool();
	shardedJedis = shardedJedisPool.getResource();
	jedis = jedisPool.getResource();
}

//初始化非切片连接池
private void initialPool() {
	// 池基本配置
	JedisPoolConfig config = new JedisPoolConfig();
	//可用连接实例的最大数目，默认值为8
	config.setMaxActive(20);
	//控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8
	config.setMaxIdle(5);
	//等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException
	config.setMaxWait(1000l);
	config.setTestOnBorrow(false);

	jedisPool = new JedisPool(config, "127.0.0.1", 6379);
}

//初始化切片池
private void initialShardedPool() {
	// 池基本配置
	JedisPoolConfig config = new JedisPoolConfig();
	config.setMaxActive(20);
	config.setMaxIdle(5);
	config.setMaxWait(1000l);
	config.setTestOnBorrow(false);
	// slave链接
	List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>();
	shards.add(new JedisShardInfo("127.0.0.1", 6379, "master"));

	// 构造池
	shardedJedisPool = new ShardedJedisPool(config, shards);
}

public void show() {
	KeyOperate();
	StringOperate();
	ListOperate();
	SetOperate();
	SortedSetOperate();
	HashOperate();
	jedisPool.returnResource(jedis);
	shardedJedisPool.returnResource(shardedJedis);
}
```

## key
```java
/**
 * key 功能 
 * 1.清空所有数据 jedis.flushDB()
 * 2.判断键是否存在 shardedJedis.exists("key1")
 * 3.新增键值对 shardedJedis.set("key001", "value001")
 * 4.删除某个key,若key不存在，则忽略该命令 jedis.del("key002")
 * 5.设置 key001的过期时间 jedis.expire("key001", 5)
 * 6.查看某个key的剩余生存时间,单位【秒】.永久生存或者不存在的都返回-1  jedis.ttl("key001")
 * 7.移除某个key的生存时间  jedis.persist("key001")
 * 8.查看key所储存的值的类型 jedis.type("key001")
 * 
 */
private void KeyOperate() {
	System.out.println("======================key==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());
	// 判断key否存在
	System.out.println("判断key999键是否存在：" + shardedJedis.exists("key999"));
	System.out.println("新增key001,value001键值对：" + shardedJedis.set("key001", "value001"));
	System.out.println("判断key001是否存在：" + shardedJedis.exists("key001"));
	// 输出系统中所有的key
	System.out.println("新增key002,value002键值对：" + shardedJedis.set("key002", "value002"));
	System.out.println("系统中所有键如下：");
	Set<String> keys = jedis.keys("*");
	Iterator<String> it = keys.iterator();
	while (it.hasNext()) {
		String key = it.next();
		System.out.println(key);
	}
	// 删除某个key,若key不存在，则忽略该命令。
	System.out.println("系统中删除key002: " + jedis.del("key002"));
	System.out.println("判断key002是否存在：" + shardedJedis.exists("key002"));
	// 设置 key001的过期时间
	System.out.println("设置 key001的过期时间为5秒:" + jedis.expire("key001", 5));
	try {
		Thread.sleep(2000);
	} catch (InterruptedException e) {
	}
	// 查看某个key的剩余生存时间,单位【秒】.永久生存或者不存在的都返回-1
	System.out.println("查看key001的剩余生存时间：" + jedis.ttl("key001"));
	// 移除某个key的生存时间
	System.out.println("移除key001的生存时间：" + jedis.persist("key001"));
	System.out.println("查看key001的剩余生存时间：" + jedis.ttl("key001"));
	// 查看key所储存的值的类型
	System.out.println("查看key所储存的值的类型：" + jedis.type("key001"));
	/*
	 * 一些其他方法：1、修改键名：jedis.rename("key6", "key0");
	 * 2、将当前db的key移动到给定的db当中：jedis.move("foo", 1)
	 */
}
```

## string
```java
/**
 * String功能
 * 1.一次性新增key201,key202  jedis.mset("key201", "value201", "key202", "value202")
 * 2.一次性删除key201,key202  jedis.del(new String[] { "key201", "key202" }
 * 3.一次性获取key201,key202  jedis.mget("key201", "key202")
 * 4.新增键值对时防止覆盖原先值  shardedJedis.setnx("key302", "value302_new")
 * 5.获取原值，更新为新值一步完成 shardedJedis.getSet("key302", "value302-after-getset")
 * 6.获取key302对应值中的子串   shardedJedis.getrange("key302", 5, 7)
 * 
 */
private void StringOperate() {
	System.out.println("======================String_1==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	jedis.set("key001", "value001");
	jedis.set("key002", "value002");
	jedis.set("key003", "value003");
	System.out.println("已新增的3个键值对如下：");
	System.out.println(jedis.get("key001"));
	System.out.println(jedis.get("key002"));
	System.out.println(jedis.get("key003"));

	System.out.println("=============删=============");
	System.out.println("删除key003键值对：" + jedis.del("key003"));
	System.out.println("获取key003键对应的值：" + jedis.get("key003"));

	System.out.println("=============改=============");
	// 1、直接覆盖原来的数据
	System.out.println("直接覆盖key001原来的数据：" + jedis.set("key001", "value001-update"));
	System.out.println("获取key001对应的新值：" + jedis.get("key001"));
	// 2、直接覆盖原来的数据
	System.out.println("在key002原来值后面追加：" + jedis.append("key002", "+appendString"));
	System.out.println("获取key002对应的新值" + jedis.get("key002"));

	System.out.println("=============增，删，查（多个）=============");
	/**
	 * mset,mget同时新增，修改，查询多个键值对 等价于： jedis.set("name","ssss");
	 * jedis.set("jarorwar","xxxx");
	 */
	System.out.println("一次性新增key201,key202,key203,key204及其对应值："
			+ jedis.mset("key201", "value201", "key202", "value202", "key203", "value203", "key204", "value204"));
	System.out.println(
			"一次性获取key201,key202,key203,key204各自对应的值：" + jedis.mget("key201", "key202", "key203", "key204"));
	System.out.println("一次性删除key201,key202：" + jedis.del(new String[] { "key201", "key202" }));
	System.out.println(
			"一次性获取key201,key202,key203,key204各自对应的值：" + jedis.mget("key201", "key202", "key203", "key204"));
	System.out.println();

	// jedis具备的功能shardedJedis中也可直接使用，下面测试一些前面没用过的方法
	System.out.println("======================String_2==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============新增键值对时防止覆盖原先值=============");
	System.out.println("原先key301不存在时，新增key301：" + shardedJedis.setnx("key301", "value301"));
	System.out.println("原先key302不存在时，新增key302：" + shardedJedis.setnx("key302", "value302"));
	System.out.println("当key302存在时，尝试新增key302：" + shardedJedis.setnx("key302", "value302_new"));
	System.out.println("获取key301对应的值：" + shardedJedis.get("key301"));
	System.out.println("获取key302对应的值：" + shardedJedis.get("key302"));

	System.out.println("=============超过有效期键值对被删除=============");
	// 设置key的有效期，并存储数据
	System.out.println("新增key303，并指定过期时间为2秒" + shardedJedis.setex("key303", 2, "key303-2second"));
	System.out.println("获取key303对应的值：" + shardedJedis.get("key303"));
	try {
		Thread.sleep(3000);
	} catch (InterruptedException e) {
	}
	System.out.println("3秒之后，获取key303对应的值：" + shardedJedis.get("key303"));

	System.out.println("=============获取原值，更新为新值一步完成=============");
	System.out.println("key302原值：" + shardedJedis.getSet("key302", "value302-after-getset"));
	System.out.println("key302新值：" + shardedJedis.get("key302"));

	System.out.println("=============获取子串=============");
	System.out.println("获取key302对应值中的子串：" + shardedJedis.getrange("key302", 5, 7));
}
```

## list
```java
/**
 * List功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void ListOperate() {
	System.out.println("======================list==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "ArrayList");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "LinkedList");
	shardedJedis.lpush("stringlists", "MapList");
	shardedJedis.lpush("stringlists", "SerialList");
	shardedJedis.lpush("stringlists", "HashList");
	shardedJedis.lpush("numberlists", "3");
	shardedJedis.lpush("numberlists", "1");
	shardedJedis.lpush("numberlists", "5");
	shardedJedis.lpush("numberlists", "2");
	System.out.println("所有元素-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	System.out.println("所有元素-numberlists：" + shardedJedis.lrange("numberlists", 0, -1));

	System.out.println("=============删=============");
	// 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
	System.out.println("成功删除指定元素个数-stringlists：" + shardedJedis.lrem("stringlists", 2, "vector"));
	System.out.println("删除指定元素之后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	// 删除区间以外的数据
	System.out.println("删除下标0-3区间之外的元素：" + shardedJedis.ltrim("stringlists", 0, 3));
	System.out.println("删除指定区间之外元素后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	// 列表元素出栈
	System.out.println("出栈元素：" + shardedJedis.lpop("stringlists"));
	System.out.println("元素出栈后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));

	System.out.println("=============改=============");
	// 修改列表中指定下标的值
	shardedJedis.lset("stringlists", 0, "hello list!");
	System.out.println("下标为0的值修改后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	System.out.println("=============查=============");
	// 数组长度
	System.out.println("长度-stringlists：" + shardedJedis.llen("stringlists"));
	System.out.println("长度-numberlists：" + shardedJedis.llen("numberlists"));
	// 排序
	/*
	 * list中存字符串时必须指定参数为alpha，如果不使用SortingParams，而是直接使用sort("list")，
	 * 会出现"ERR One or more scores can't be converted into double"
	 */
	SortingParams sortingParameters = new SortingParams();
	sortingParameters.alpha();
	sortingParameters.limit(0, 3);
	System.out.println("返回排序后的结果-stringlists：" + shardedJedis.sort("stringlists", sortingParameters));
	System.out.println("返回排序后的结果-numberlists：" + shardedJedis.sort("numberlists"));
	// 子串： start为元素下标，end也为元素下标；-1代表倒数一个元素，-2代表倒数第二个元素
	System.out.println("子串-第二个开始到结束：" + shardedJedis.lrange("stringlists", 1, -1));
	// 获取列表指定下标的值
	System.out.println("获取下标为2的元素：" + shardedJedis.lindex("stringlists", 2) + "\n");
}
```
## set
```java
/**
 * Set功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void SetOperate() {

	System.out.println("======================set==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("向sets集合中加入元素element001：" + jedis.sadd("sets", "element001"));
	System.out.println("向sets集合中加入元素element002：" + jedis.sadd("sets", "element002"));
	System.out.println("向sets集合中加入元素element003：" + jedis.sadd("sets", "element003"));
	System.out.println("向sets集合中加入元素element004：" + jedis.sadd("sets", "element004"));
	System.out.println("查看sets集合中的所有元素:" + jedis.smembers("sets"));
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("集合sets中删除元素element003：" + jedis.srem("sets", "element003"));
	System.out.println("查看sets集合中的所有元素:" + jedis.smembers("sets"));
	/*
	 * System.out.println("sets集合中任意位置的元素出栈："+jedis.spop("sets"));//注：
	 * 出栈元素位置居然不定？--无实际意义
	 * System.out.println("查看sets集合中的所有元素:"+jedis.smembers("sets"));
	 */
	System.out.println();

	System.out.println("=============改=============");
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("判断element001是否在集合sets中：" + jedis.sismember("sets", "element001"));
	System.out.println("循环查询获取sets中的每个元素：");
	Set<String> set = jedis.smembers("sets");
	Iterator<String> it = set.iterator();
	while (it.hasNext()) {
		Object obj = it.next();
		System.out.println(obj);
	}
	System.out.println();

	System.out.println("=============集合运算=============");
	System.out.println("sets1中添加元素element001：" + jedis.sadd("sets1", "element001"));
	System.out.println("sets1中添加元素element002：" + jedis.sadd("sets1", "element002"));
	System.out.println("sets1中添加元素element003：" + jedis.sadd("sets1", "element003"));
	System.out.println("sets1中添加元素element002：" + jedis.sadd("sets2", "element002"));
	System.out.println("sets1中添加元素element003：" + jedis.sadd("sets2", "element003"));
	System.out.println("sets1中添加元素element004：" + jedis.sadd("sets2", "element004"));
	System.out.println("查看sets1集合中的所有元素:" + jedis.smembers("sets1"));
	System.out.println("查看sets2集合中的所有元素:" + jedis.smembers("sets2"));
	System.out.println("sets1和sets2交集：" + jedis.sinter("sets1", "sets2"));
	System.out.println("sets1和sets2并集：" + jedis.sunion("sets1", "sets2"));
	System.out.println("sets1和sets2差集：" + jedis.sdiff("sets1", "sets2"));// 差集：set1中有，set2中没有的元素

}
```

## SortedSet
```java
/**
 * SortedSet功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void SortedSetOperate() {
	System.out.println("======================zset==========================");
	// 清空数据
	System.out.println(jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("zset中添加元素element001：" + shardedJedis.zadd("zset", 7.0, "element001"));
	System.out.println("zset中添加元素element002：" + shardedJedis.zadd("zset", 8.0, "element002"));
	System.out.println("zset中添加元素element003：" + shardedJedis.zadd("zset", 2.0, "element003"));
	System.out.println("zset中添加元素element004：" + shardedJedis.zadd("zset", 3.0, "element004"));
	System.out.println("zset集合中的所有元素：" + shardedJedis.zrange("zset", 0, -1));// 按照权重值排序
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("zset中删除元素element002：" + shardedJedis.zrem("zset", "element002"));
	System.out.println("zset集合中的所有元素：" + shardedJedis.zrange("zset", 0, -1));
	System.out.println();

	System.out.println("=============改=============");
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("统计zset集合中的元素中个数：" + shardedJedis.zcard("zset"));
	System.out.println("统计zset集合中权重某个范围内（1.0——5.0），元素的个数：" + shardedJedis.zcount("zset", 1.0, 5.0));
	System.out.println("查看zset集合中element004的权重：" + shardedJedis.zscore("zset", "element004"));
	System.out.println("查看下标1到2范围内的元素值：" + shardedJedis.zrange("zset", 1, 2));

}
```
	
## hash
```java
/**
 * Hash功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void HashOperate() {
	System.out.println("======================hash==========================");
	// 清空数据
	System.out.println(jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("hashs中添加key001和value001键值对：" + shardedJedis.hset("hashs", "key001", "value001"));
	System.out.println("hashs中添加key002和value002键值对：" + shardedJedis.hset("hashs", "key002", "value002"));
	System.out.println("hashs中添加key003和value003键值对：" + shardedJedis.hset("hashs", "key003", "value003"));
	System.out.println("新增key004和4的整型键值对：" + shardedJedis.hincrBy("hashs", "key004", 4l));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("hashs中删除key002键值对：" + shardedJedis.hdel("hashs", "key002"));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============改=============");
	System.out.println("key004整型键值的值增加100：" + shardedJedis.hincrBy("hashs", "key004", 100l));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("判断key003是否存在：" + shardedJedis.hexists("hashs", "key003"));
	System.out.println("获取key004对应的值：" + shardedJedis.hget("hashs", "key004"));
	System.out.println("批量获取key001和key003对应的值：" + shardedJedis.hmget("hashs", "key001", "key003"));
	System.out.println("获取hashs中所有的key：" + shardedJedis.hkeys("hashs"));
	System.out.println("获取hashs中所有的value：" + shardedJedis.hvals("hashs"));
	System.out.println();

}
```
	
# 操作命令
## 连接操作命令
quit：关闭连接（connection）
auth：简单密码认证
help cmd： 查看cmd帮助，例如：help quit

## 持久化
save：将数据同步保存到磁盘
bgsave：将数据异步保存到磁盘
lastsave：返回上次成功将数据保存到磁盘的Unix时戳
shundown：将数据同步保存到磁盘，然后关闭服务

## 远程服务控制
info：提供服务器的信息和统计
monitor：实时转储收到的请求
slaveof：改变复制策略设置
config：在运行时配置Redis服务器

## 对value操作的命令
exists(key)：确认一个key是否存在
del(key)：删除一个key
type(key)：返回值的类型
keys(pattern)：返回满足给定pattern的所有key
randomkey：随机返回key空间的一个
keyrename(oldname, newname)：重命名key
dbsize：返回当前数据库中key的数目
expire：设定一个key的活动时间（s）
ttl：获得一个key的活动时间
select(index)：按索引查询
move(key, dbindex)：移动当前数据库中的key到dbindex数据库
flushdb：删除当前选择数据库中的所有key
flushall：删除所有数据库中的所有key

## String
set(key, value)：给数据库中名称为key的string赋予值value
get(key)：返回数据库中名称为key的string的value
getset(key, value)：给名称为key的string赋予上一次的value
mget(key1, key2,…, key N)：返回库中多个string的value
setnx(key, value)：添加string，名称为key，值为value
setex(key, time, value)：向库中添加string，设定过期时间time
mset(key N, value N)：批量设置多个string的值
msetnx(key N, value N)：如果所有名称为key i的string都不存在
incr(key)：名称为key的string增1操作
incrby(key, integer)：名称为key的string增加integer
decr(key)：名称为key的string减1操作
decrby(key, integer)：名称为key的string减少integer
append(key, value)：名称为key的string的值附加value
substr(key, start, end)：返回名称为key的string的value的子串

## List 
rpush(key, value)：在名称为key的list尾添加一个值为value的元素
lpush(key, value)：在名称为key的list头添加一个值为value的 元素
llen(key)：返回名称为key的list的长度
lrange(key, start, end)：返回名称为key的list中start至end之间的元素
ltrim(key, start, end)：截取名称为key的list
lindex(key, index)：返回名称为key的list中index位置的元素
lset(key, index, value)：给名称为key的list中index位置的元素赋值
lrem(key, count, value)：删除count个key的list中值为value的元素
lpop(key)：返回并删除名称为key的list中的首元素
rpop(key)：返回并删除名称为key的list中的尾元素
blpop(key1, key2,… key N, timeout)：lpop命令的block版本。
brpop(key1, key2,… key N, timeout)：rpop的block版本。
rpoplpush(srckey, dstkey)：返回并删除名称为srckey的list的尾元素，并将该元素添加到名称为dstkey的list的头部

## Set
sadd(key, member)：向名称为key的set中添加元素member
srem(key, member) ：删除名称为key的set中的元素member
spop(key) ：随机返回并删除名称为key的set中一个元素
smove(srckey, dstkey, member) ：移到集合元素
scard(key) ：返回名称为key的set的基数
sismember(key, member) ：member是否是名称为key的set的元素
sinter(key1, key2,…key N) ：求交集
sinterstore(dstkey, (keys)) ：求交集并将交集保存到dstkey的集合
sunion(key1, (keys)) ：求并集
sunionstore(dstkey, (keys)) ：求并集并将并集保存到dstkey的集合
sdiff(key1, (keys)) ：求差集
sdiffstore(dstkey, (keys)) ：求差集并将差集保存到dstkey的集合
smembers(key) ：返回名称为key的set的所有元素
srandmember(key) ：随机返回名称为key的set的一个元素

## Hash
hset(key, field, value)：向名称为key的hash中添加元素field
hget(key, field)：返回名称为key的hash中field对应的value
hmget(key, (fields))：返回名称为key的hash中field i对应的value
hmset(key, (fields))：向名称为key的hash中添加元素field 
hincrby(key, field, integer)：将名称为key的hash中field的value增加integer
hexists(key, field)：名称为key的hash中是否存在键为field的域
hdel(key, field)：删除名称为key的hash中键为field的域
hlen(key)：返回名称为key的hash中元素个数
hkeys(key)：返回名称为key的hash中所有键
hvals(key)：返回名称为key的hash中所有键对应的value
hgetall(key)：返回名称为key的hash中所有的键（field）及其对应的value

# Spring Boot 使用RedisTemplate 存储键值出现乱码
最近使用spring-data-redis RedisTemplate 操作redis时发现存储在redis中的key不是设置的string值，前面还多出了许多类似\xac\xed\x00\x05t\x00这种字符串，如下
```
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04pass"
2) "\xac\xed\x00\x05t\x00\x04name"
3) "name"
```

解决：使用StringRedisTemplate，而不是RedisTemplate。

# 通过notify-keyspace-events实现订单自动关闭功能
修改Redis配置文件，开启过期通知功能。
在配置文件中搜索`notify`，找到`# notify-keyspace-events Ex`这一行，将注释去掉，同时注释掉下面一行`notify-keyspace-events ""`，
为了节约cup资源，事件通知默认是没有开启的，即`notify-keyspace-events ""`

修改之后在需要重启redis，然后通过cli查看配置是否生效，`config get notify-keyspace-events`，如果显示Ex，就生效了。