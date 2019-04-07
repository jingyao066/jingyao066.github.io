---
title: redis基本使用
tags: redis
date: 2018-12-06 18:10:43
---

# redis是什么
redis是一个开源的，使用C语言编写的，支持网络交互的、可基于内存也可持久化的Key-Value数据库

# 安装过程

# 基本使用
## 初始化
```
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
```
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
```
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
```
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
```
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
```
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
```
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
