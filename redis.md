Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

string：
SET name "bar"
GET name
  // bar


hash:
HMSET myhash field1 "hello" field2 "world"
HGET myhash field1
HGET myhash field2
HGETALL myhash   获得所有
  //  1)field1
  //  2)hello
  //  3)field2
  //  4)world

HDEL field1 [field2] 删除指定field
HEXISTS key field1   查询是否存在
HLEN myhash          所需field数量
HVAL


list:
lpush listname first
lpush listname second
lpush listname third
lrange listname 0 10

********************************
#操作key
DEL key
DUMP key
EXISTS key
EXPIRE key seconds   过期时间s
EXPIRE key timestamp 时间戳过期时间
PTTL key             过期剩余时间s
