redis

1.特点
	单线程： 命令的单线程。 网络数据的处理是多线程


	内存型数据库：可以持久化

	IO多路复用

	key-value value指出多种数据类型
	
	db 数据是独立的
	没有做业务区分
	
2. 常见的数据类型
	list
		1.基本操作：有序的  lpush lrange 0 -1 (拿出所有) rpush rrange  lpop rpop blpop(阻塞的，有数据再弹出来)
		2.应用场景：
			消息队列（阻塞队列）： 不推荐 没法追踪到消费者
			有序列表 朋友圈
			
	string
		1.基本操作：setnx  set get  mget--批量获取
		2.应用场景： 
			缓存 get
			incr 主键/数据统计/自增流水号
			setnex  分布式锁  lua脚本--保证多个版本的原子性
	set
		1.基本操作 无序不重复，sadd smembers scard  spop srem sismember sdiff（获取前面集合有，后面集合没有） sinter sunion
		2.应用场景：共同好友，可能认识的人，抽奖，随机抽奖 spop
	hash
		1.基本操作：hset hget hmset hmget hkeys  hgetall hincrby
		2.应用场景：
			购物车 用户id   商品id  商品数量
			string 能做的都可以
	zset
		1.基本操作：有序不重复， 每个元素都会有一个score,如果score 相同根据key的ascII码排序。zadd  zrange 0 -1 zrevrange  zrangebyscore zincrby zrank zscore zrem
		2.应用场景：排行榜，成绩，网游战力榜