## 概述


## String
```html

    /**
     * 查询缓存
     *
     * @param key 缓存键 不可为空
     **/
    public Object get(String cacheName, String key);

    /**
     * 查询缓存 - 直接从远程缓存获取，如果没有远程则取本地缓存
     *
     * @param key 缓存键 不可为空
     **/
    public Object getRemote(String cacheName, String key);

    /**
     * 获取缓存剩余过期时间 - 直接从远程缓存获取，如果没有远程则取本地缓存（caffeine暂时没有找到获取ttl的功能，暂时返回cacheName对应的ttl）
     * 如果该key已经过期,将返回 -2
     *
     * @param key 缓存键 不可为空
     **/
    public Long getTtlRemote(String cacheName, String key);

    /**
     * 设置缓存过期时间 - 指定缓存失效时间 如果没有远程，只有caffeine， 则返回 false
     *
     * @param cacheName
     * @param key
     * @param time
     * @param timeUnit
     * @return
     */
    public boolean setTtlRemote(String cacheName, String key, long time, TimeUnit timeUnit);

    /**
     * 有效期依赖于 cacheName
     *
     * @param cacheName
     * @param key
     * @param value
     * @return
     */
    public boolean put(String cacheName, String key, String value);

    public boolean evict(String cacheName, String key);

    public String cacheNameKey(String cacheName, String key);

    public boolean hasKey(String cacheName, String key);

    /**
     * 根据 cacheName 查询下面包含的 key
     *
     * @param cacheName
     * @param limit
     * @return
     */
    public List<String> keysByCacheName(String cacheName, Integer limit);

    /**
     * 递增 +1 操作， 并返回 +1 操作之后的值
     *
     * @param cacheName
     * @param key
     * @return
     */
    public Long increment(String cacheName, String key);

    /**
     * 递减 -1 操作， 并返回 -1 操作之后的值
     *
     * @param cacheName
     * @param key
     * @return
     */
    public Long decrement(String cacheName, String key);

    public Map<String, Map<String, Object>> metricsInfo();

    // redis gene begin

    /**
     * Redis type 查看数据类型
     *
     * @param cacheName
     * @param key
     * @return
     */
    default String redisType(String cacheName, String key) {
        return null;
    }


    default String redisEncoding(String cacheName, String key) {
        return null;
    }

    // redis gene end
```
## Hash
```html

    /**
     * 实现命令 : HSET key field value
     * 添加 hash 类型的键值对，如果字段已经存在，则将其覆盖。
     *
     * @param key
     * @param field
     * @param value
     */
    default void redisHSet(String cacheName, String key, String field, Object value) {

    }

    default void redisHSet(String cacheName, String key, String field, Object value, Long timeout, TimeUnit timeUnit) {

    }

    /**
     * 实现命令 : HSET key field1 value1 [field2 value2 ...]
     * 添加 hash 类型的键值对，如果字段已经存在，则将其覆盖。
     *
     * @param key
     * @param map
     */
    default void redisHSet(String cacheName, String key, Map<String, Object> map) {

    }

    default void redisHSet(String cacheName, String key, Map<String, Object> map, Long timeout, TimeUnit timeUnit) {

    }

    /**
     * 实现命令 : HSETNX key field value
     * 添加 hash 类型的键值对，如果字段不存在，才添加
     *
     * @param key
     * @param field
     * @param value
     */
    default boolean redisHSetNx(String cacheName, String key, String field, Object value) {
        return false;
    }

    default boolean redisHSetNx(String cacheName, String key, String field, Object value, Long timeout, TimeUnit timeUnit) {
        return false;
    }

    /**
     * 实现命令 : HGET key field
     * 返回 field 对应的值
     *
     * @param key
     * @param field
     * @return
     */
    default Object redisHGet(String cacheName, String key, String field) {
        return null;
    }

    /**
     * 实现命令 : HMGET key field1 [field2 ...]
     * 返回 多个 field 对应的值
     *
     * @param key
     * @param fields
     * @return
     */
    default List<Object> redisHGet(String cacheName, String key, String... fields) {
        return null;
    }

    /**
     * 实现命令 : HGETALL key
     * 返回所以的键值对
     *
     * @param key
     * @return
     */
    default Map<Object, Object> redisHGetAll(String cacheName, String key) {
        return null;
    }

    /**
     * Redis 命令 `hgetall` 的时间复杂度为 O(n) Redis 命令 `scan` 的时间复杂度为 O(1)，可以无阻塞的匹配出列表，
     * 缺点是可能出现重复数据，这里用 Map 接收刚好可以解决这个问题，因为要求匹配的数据必须带顺序，所以在本方法中直接用 LinkedHashMap 来实现。
     *
     * @param cacheName
     * @param key
     * @param count
     * @return
     */
    default Map<Object, Object> redisHGetAlUsingScan(String cacheName, String key, Integer count) {
        return null;
    }


    /**
     * 实现命令 : HKEYS key
     * 获取所有的 field
     *
     * @param key
     * @return
     */
    default Set<Object> redisHKeys(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : HVALS key
     * 获取所有的 value
     *
     * @param key
     * @return
     */
    default List<Object> redisHValue(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : HDEL key field [field ...]
     * 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。
     *
     * @param key
     * @param fields
     */
    default Long redisHDel(String cacheName, String key, Object... fields) {
        return null;
    }


    /**
     * 实现命令 : HEXISTS key field
     * 判断 key 下指定的 field 是否存在
     *
     * @param key
     * @param field
     */
    default boolean redisHExists(String cacheName, String key, String field) {
        return false;
    }


    /**
     * 实现命令 : HLEN key
     * 获取 hash 中字段:值对的数量
     *
     * @param key
     */
    default Long redisHLen(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : HSTRLEN key field
     * 获取字段对应值的长度
     *
     * @param key
     * @param field
     */
    default Long redisHStrLen(String cacheName, String key, String field) {
        return null;
    }


    /**
     * 实现命令 : HINCRBY key field 整数
     * 给字段的值加上一个整数
     *
     * @param key
     * @param field
     * @return 运算后的值
     */
    default Long redisHInCrBy(String cacheName, String key, String field, long number) {
        return null;
    }


    /**
     * 实现命令 : HINCRBYFLOAT key field 浮点数
     * 给字段的值加上一个浮点数
     *
     * @param key
     * @param field
     * @return 运算后的值
     */
    default Double redisHInCrByFloat(String cacheName, String key, String field, Double number) {
        return null;
    }

```
## List
```html

    /**
     * 实现命令 : LPUSH key 元素1 [元素2 ...]
     * 在最左端推入元素
     *
     * @param cacheName
     * @param key
     * @param values
     * @return 执行 LPUSH 命令后，列表的长度。
     */
    default Long redisLPush(String cacheName, String key, Object... values) {
        return null;
    }


    /**
     * 实现命令 : RPUSH key 元素1 [元素2 ...]
     * 在最右端推入元素
     *
     * @param cacheName
     * @param key
     * @param values
     * @return 执行 RPUSH 命令后，列表的长度。
     */
    default Long redisRPush(String cacheName, String key, Object... values) {
        return null;
    }


    /**
     * 实现命令 : LPOP key
     * 弹出最左端的元素
     *
     * @param cacheName
     * @param key
     * @return 弹出的元素
     */
    default Object redisLPop(String cacheName, String key) {
        return null;
    }

    /**
     * 实现命令 : RPOP key
     * 弹出最右端的元素
     *
     * @param cacheName
     * @param key
     * @return 弹出的元素，无数据返回 null
     */
    default Object redisRPop(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : BLPOP key
     * (阻塞式)弹出最左端的元素，如果 key 中没有元素，将一直等待直到有元素或超时为止
     *
     * @param cacheName
     * @param key       参数 0 表示阻塞等待时间无无限制
     * @param timeout   等待的时间，单位秒
     * @return 弹出的元素，无数据返回 null
     */
    default Object redisBLPop(String cacheName, String key, int timeout, TimeUnit timeUnit) {
        return null;
    }

    /**
     * 实现命令 : BRPOP key
     * (阻塞式)弹出最右端的元素，将一直等待直到有元素或超时为止
     *
     * @param cacheName
     * @param key
     * @return 弹出的元素
     */
    default Object redisBRPop(String cacheName, String key, int timeout, TimeUnit timeUnit) {
        return null;
    }


    /**
     * 实现命令 : LINDEX key index
     * 返回指定下标处的元素，下标从0开始
     *
     * @param cacheName
     * @param key
     * @param index
     * @return 指定下标处的元素
     */
    default Object redisLIndex(String cacheName, String key, int index) {
        return null;
    }


    /**
     * 实现命令 : LINSERT key BEFORE|AFTER 目标元素 value
     * 在目标元素前或后插入元素
     *
     * @param cacheName
     * @param key
     * @param position  BEFORE|AFTER
     * @param pivot
     * @param value
     * @return 执行 LInsert 命令后，列表的长度。
     */
    default Long redisLInsert(String cacheName, String key, String position, Object pivot, Object value) {
        return null;
    }


    /**
     * 实现命令 : LRANGE key 开始下标 结束下标
     * 获取指定范围的元素,下标从0开始，包括开始下标，也包括结束下标，比如传递[0, 1]，则级返回位于0的数据，也返回位于1的数据。
     * 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。
     *
     * @param cacheName
     * @param key
     * @param start
     * @param end
     * @return 返回指定范围的元素集合
     */
    default List<Object> redisLRange(String cacheName, String key, int start, int end) {
        return null;
    }


    /**
     * 实现命令 : LLEN key
     * 获取 list 的长度
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisLLen(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : LREM key count 元素
     * 删除 count 个指定元素, 将key域列表中,前count次,值为value的元素删除:
     *
     * @param cacheName
     * @param key
     * @param count     count > 0: 从头往尾移除指定元素。count < 0: 从尾往头移除指定元素。count = 0: 移除列表所有的指定元素。(未验证)
     * @param value
     * @return 被移除元素的数量。 列表不存在时返回 0 。
     */
    default Long redisLRem(String cacheName, String key, int count, Object value) {
        return null;
    }


    /**
     * 实现命令 : LSET key index 新值
     * 更新指定下标的值,下标从 0 开始，支持负下标，-1表示最右端的元素
     *
     * @param cacheName
     * @param key
     * @param index
     * @param value
     * @return
     */
    default void redisLSet(String cacheName, String key, int index, Object value) {
    }


    /**
     * Redis Ltrim 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
     * 下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。
     * 举个例子，执行命令 LTRIM list 0 2 ，表示只保留列表 list 的前三个元素，其余元素全部删除。
     * 裁剪 list [01234] 的 `LTRIM key 1 -2` 的结果为 [123]
     *
     * @param cacheName
     * @param key
     * @param start
     * @param end
     * @return
     */
    default void redisLTrim(String cacheName, String key, int start, int end) {
    }


    /**
     * 实现命令 : RPOPLPUSH 源list 目标list
     * 将 源list 的最右端元素弹出，推入到 目标list 的最左端，
     *
     * @param cacheName
     * @param sourceKey 源list
     * @param targetKey 目标list
     * @return 弹出的元素
     */
    default Object redisRPopLPush(String cacheName, String sourceKey, String targetKey) {
        return null;
    }


    /**
     * 实现命令 : BRPOPLPUSH 源list 目标list timeout
     * (阻塞式)将 源list 的最右端元素弹出，推入到 目标list 的最左端，如果 源list 没有元素，将一直等待直到有元素或超时为止
     *
     * @param cacheName
     * @param sourceKey 源list
     * @param targetKey 目标list
     * @param timeout   超时时间，单位秒， 0表示无限阻塞
     * @return 弹出的元素
     */
    default Object redisBRPopLPush(String cacheName, String sourceKey, String targetKey, int timeout, TimeUnit timeUnit) {
        return null;
    }

```
## Set
```html

    /**
     * 实现命令 : SADD key member1 [member2 ...]
     * 添加成员
     *
     * @param cacheName
     * @param key
     * @param values
     * @return 添加成功的个数
     */
    default Long redisSAdd(String cacheName, String key, Object... values) {
        return null;
    }


    /**
     * 实现命令 : SREM key member1 [member2 ...]
     * 删除指定的成员
     *
     * @param cacheName
     * @param key
     * @param values
     * @return 删除成功的个数，value全部删除的话key也丢失
     */
    default Long redisSRem(String cacheName, String key, Object... values) {
        return null;
    }


    /**
     * 实现命令 : SCARD key
     * 获取set中的成员数量
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisSCard(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : SISMEMBER key member
     * 查看成员是否存在
     *
     * @param cacheName
     * @param key
     * @param value
     * @return
     */
    default boolean redisSIsMember(String cacheName, String key, Object value) {
        return false;
    }


    /**
     * 实现命令 : SMEMBERS key
     * 获取所有的成员
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Set<Object> redisSMembers(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : SMOVE 源key 目标key member
     * 移动成员到另一个集合
     *
     * @param cacheName
     * @param sourceKey 源key
     * @param targetKey 目标key
     * @param value
     * @return
     */
    default boolean redisSMove(String cacheName, String sourceKey, String targetKey, Object value) {
        return false;
    }


    /**
     * 实现命令 : SDIFF key [otherKey ...]
     * 求 key 的差集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param key
     * @param otherKeys
     * @return
     */
    default Set<Object> redisSDiff(String cacheName, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SDIFFSTORE 目标key key [otherKey ...]
     * 存储 key 的差集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param targetKey 目标key
     * @param key
     * @param otherKeys
     * @return
     */
    default Long redisSDiffStore(String cacheName, String targetKey, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SINTER key [otherKey ...]
     * 求 key 的交集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param key
     * @param otherKeys
     * @return
     */
    default Set<Object> redisSInter(String cacheName, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SINTERSTORE 目标key key [otherKey ...]
     * 存储 key 的交集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param targetKey 目标key
     * @param key
     * @param otherKeys
     * @return
     */
    default Long redisSInterStore(String cacheName, String targetKey, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SUNION key [otherKey ...]
     * 求 key 的并集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param key
     * @param otherKeys
     * @return
     */
    default Set<Object> redisSUnion(String cacheName, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SUNIONSTORE 目标key key [otherKey ...]
     * 存储 key 的并集 注意操作的对象都是缓存中的key
     *
     * @param cacheName
     * @param targetKey 目标key
     * @param key
     * @param otherKeys
     * @return
     */
    default Long redisSUnionStore(String cacheName, String targetKey, String key, String... otherKeys) {
        return null;
    }


    /**
     * 实现命令 : SPOP key [count]
     * 随机删除(弹出)指定个数的成员
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Object redisSPop(String cacheName, String key) {
        return null;
    }

    /**
     * 实现命令 : SPOP key [count]
     * 随机删除(弹出)指定个数的成员
     *
     * @param cacheName
     * @param key
     * @param count     个数
     * @return
     */
    default List<Object> redisSPop(String cacheName, String key, int count) {
        return null;
    }


    /**
     * 实现命令 : SRANDMEMBER key [count]
     * 随机返回指定个数的成员 只是随机返回，并不从key中删除
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Object redisSRandMember(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : SRANDMEMBER key [count]
     * 随机返回指定个数的成员 如果 count 为正数，随机返回 count 个不同成员  有可能重复
     * 此方法不建议使用，有可能会卡死，假设count传得特别大，在实际使用的时候，会有长时间卡顿 O(N)复杂度的不建议使用
     *
     * @param cacheName
     * @param key
     * @param count     个数
     * @return
     */
    default List<Object> redisSRandMember(String cacheName, String key, int count) {
        return null;
    }

```
## ZSet
```html

    /**
     * 实现命令 : ZADD key score member
     * 添加一个 成员/分数 对
     * 分数相同按照成员的字典序(从A到Z)进行排序
     * 由于 score 是浮点数，所有尽量确保 score 的不同，形如 新分数 = 旧分数 + 小数位的时间戳
     *
     * @param cacheName
     * @param key
     * @param value     成员
     * @param score     分数
     * @return ZADD 的返回值只计算添加的新元素的数量。 如果使用 zadd 更新成员的分数值 的时候，返回值为 false（如果想要修改返回值，后续考虑增加 CH 语法）
     */
    default boolean redisZAdd(String cacheName, String key, double score, Object value) {
        return false;
    }

    /**
     * 同 redisZAdd 相比， 增加了 CH 参数
     *
     * @param cacheName
     * @param key
     * @param score
     * @param value
     * @return CH: 修改返回值为发生变化的成员总数
     */
    default boolean redisZAddCH(String cacheName, String key, double score, Object value) {
        return false;
    }

    /**
     * 实现命令 : ZREM key member [member ...]
     * 删除成员
     *
     * @param cacheName
     * @param key
     * @param values
     * @return 成功删除的个数
     */
    default Long redisZRem(String cacheName, String key, Object... values) {
        return null;
    }


    /**
     * 实现命令 : ZREMRANGEBYRANK key start stop
     * 删除下标范围内的所有成员  start下标 和 end下标间的所有成员
     * 下标从0开始，支持负下标，-1表示最右端成员，包括开始下标也包括结束下标（start == end == 0 的时候，删除第一个元素）
     *
     * @param cacheName
     * @param key
     * @param start     开始下标
     * @param end       结束下标
     * @return 成功删除的个数
     */
    default Long redisZRemRangeByRank(String cacheName, String key, int start, int end) {
        return null;
    }


    /**
     * 实现命令 : ZREMRANGEBYSCORE key start stop
     * 删除分数段内的所有成员
     * 包括 min 也包括 max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @return 成功删除的个数
     */
    default Long redisZRemRangeByScore(String cacheName, String key, double min, double max) {
        return null;
    }


    /**
     * 实现命令 : ZSCORE key member
     * 获取成员的分数
     *
     * @param cacheName
     * @param key
     * @param value
     * @return 未找到成员返回 null
     */
    default Double redisZScore(String cacheName, String key, Object value) {
        return null;
    }


    /**
     * 实现命令 : ZINCRBY key 带符号的双精度浮点数 member
     * 增减成员的分数
     *
     * @param cacheName
     * @param key
     * @param value
     * @param delta     带符号的双精度浮点数
     * @return 返回更新过后的score(member成员的新分数值)， 如果没有对应的成员，则添加成员并初始化分数，返回成员的分数
     */
    default Double redisZInCrBy(String cacheName, String key, Object value, double delta) {
        return null;
    }


    /**
     * 实现命令 : ZCARD key
     * 获取集合中成员的个数
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisZCard(String cacheName, String key) {
        return null;
    }


    /**
     * 实现命令 : ZCOUNT key min max
     * 获取某个分数范围内的成员个数，包括min也包括max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @return
     */
    default Long redisZCount(String cacheName, String key, double min, double max) {
        return null;
    }


    /**
     * 实现命令 : ZRANK key member
     * 按分数从小到大获取成员在有序集合中的排名，下标从0开始
     *
     * @param cacheName
     * @param key
     * @param value
     * @return 没有对应的成员则返回 null
     */
    default Long redisZRank(String cacheName, String key, Object value) {
        return null;
    }


    /**
     * 实现命令 : ZREVRANK key member
     * 按分数从大到小获取成员在有序集合中的排名
     *
     * @param cacheName
     * @param key
     * @param value
     * @return
     */
    default Long redisZRevRank(String cacheName, String key, Object value) {
        return null;
    }


    /**
     * 实现命令 : ZRANGE key start end
     * 获取 start下标到 end下标之间到成员，并按分数从小到大返回
     * 下标从0开始，支持负下标，-1表示最后一个成员，包括开始下标，也包括结束下标(start == end == 0, 则返回第一个)
     *
     * @param cacheName
     * @param key
     * @param start     开始下标
     * @param end       结束下标
     * @return
     */
    default Set<Object> redisZRange(String cacheName, String key, int start, int end) {
        return null;
    }


    /**
     * 实现命令 : ZREVRANGE key start end
     * 获取 start下标到 end下标之间到成员，并按分数从大到小返回
     * 下标从0开始，支持负下标，-1表示最后一个成员，包括开始下标，也包括结束下标
     *
     * @param cacheName
     * @param key
     * @param start     开始下标
     * @param end       结束下标
     * @return
     */
    default Set<Object> redisZRevRange(String cacheName, String key, int start, int end) {
        return null;
    }


    /**
     * 实现命令 : ZRANGEBYSCORE key min max
     * 获取分数范围内的成员并按从小到大返回
     * 包括min也包括max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @return
     */
    default Set<Object> redisZRangeByScore(String cacheName, String key, double min, double max) {
        return null;
    }


    /**
     * 实现命令 : ZRANGEBYSCORE key min max LIMIT offset count
     * 分页获取分数范围内的成员并按从小到大返回
     * 包括min也包括max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @param offset    开始下标，下标从0开始
     * @param count     取多少条
     * @return
     */
    default Set<Object> redisZRangeByScore(String cacheName, String key, double min, double max, int offset, int count) {
        return null;
    }


    /**
     * 实现命令 : ZREVRANGEBYSCORE key min max
     * 获取分数范围内的成员并按从大到小返回
     * 包括min也包括max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @return
     */
    default Set<Object> redisZRevRangeByScore(String cacheName, String key, double min, double max) {
        return null;
    }


    /**
     * 实现命令 : ZREVRANGEBYSCORE key min max LIMIT offset count
     * 分页获取分数范围内的成员并按从大到小返回
     * 包括min也包括max
     *
     * @param cacheName
     * @param key
     * @param min       小分数
     * @param max       大分数
     * @param offset    开始下标，下标从0开始
     * @param count     取多少条
     * @return
     */
    default Set<Object> redisZRevRangeByScore(String cacheName, String key, double min, double max, int offset, int count) {
        return null;
    }

    /**
     * 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
     * 把 sourceKeys 这些有序集的数据，求交集放到 targetKey 这个有序集中
     *
     * @param cacheName
     * @param targetKey
     * @param sourceKeys
     * @return 存储在 targetKey 中的结果有序集的元素数量
     */
    default Long redisZInterStore(String cacheName, String targetKey, String... sourceKeys) {
        return null;
    }

    /**
     * 实现命令 : ZRANGEBYLEX key min max
     * 有序集合中给定的字典区间的所有成员
     * 此指令适用于分数相同的有序集合中 LEX结尾的指令是要求分数必须相同
     *
     * @param cacheName
     * @param key
     * @param min       字典中排序位置较小的成员,必须以"["开头,或者以"("开头,可使用"-"代替
     * @param max       字典中排序位置较大的成员,必须以"["开头,或者以"("开头,可使用"+"代替
     * @return
     */
    default Set<Object> redisZRangeByLex(String cacheName, String key, String min, String max) {
        return null;
    }

    /**
     * 计算给定的一个或多个有序集的并集并将结果集存储在新的有序集合 key 中
     * 把 sourceKeys 这些有序集的数据，求并集放到 targetKey 这个有序集中
     *
     * @param cacheName
     * @param targetKey
     * @param sourceKeys
     * @return
     */
    default Long redisZUnionStore(String cacheName, String targetKey, String... sourceKeys) {
        return null;
    }

    // zremrangebylex zlexcount zpopmax zpopmin bzpopmax bzpopmin 命令需要提升 redis 相关的 JAR 的版本

```
## Bitmaps
```html

    /**
     * Redis BITMAP SETBIT
     *
     * @param cacheName
     * @param key
     * @param offset
     * @param b
     * @return
     */
    default boolean redisVSetBit(String cacheName, String key, long offset, boolean b) {
        return false;
    }

    /**
     * Redis BITMAP GETBIT
     *
     * @param cacheName
     * @param key
     * @param offset
     * @return
     */
    default boolean redisVGetBit(String cacheName, String key, long offset) {
        return false;
    }

    /**
     * Redis BITMAP BITCOUNT 计算给定字符串中，被设置为 1 的比特位的数量
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisVBitCount(String cacheName, String key) {
        return null;
    }

    /**
     * 返回位图中第一个值为 0或者1 的二进制位的位置
     *
     * @param cacheName
     * @param key
     * @param value
     * @param start
     * @param end
     * @return
     */
    default Long redisVBitPos(String cacheName, String key, boolean value, long start, long end) {
        return null;
    }

    @Deprecated
    default void redisVBitFieldTesting() {

    }

    /**
     * 类似 BloomFilter 的添加功能 , 采用 BitMap 存储
     * 自定义的 BloomFilter(CustomerBloomFilterConstant)
     *
     * @param cacheName
     * @param key
     * @param value
     */
    default void redisVCustomerBloomFilterAdd(String cacheName, String key, String value) {

    }

    /**
     * 类似 BloomFilter 的查看功能 , 从 BitMap 查看是否包含
     * 自定义的 BloomFilter(CustomerBloomFilterConstant)
     *
     * @param cacheName
     * @param key
     * @param value
     * @return
     */
    default boolean redisVCustomerBloomFilterContains(String cacheName, String key, String value) {
        return false;
    }

```
## HyperLogLog
```html

    /**
     * 添加
     *
     * @param cacheName
     * @param key
     * @param value
     */
    default Long redisHLLAdd(String cacheName, String key, String value) {
        return null;
    }

    /**
     * size
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisHLLSize(String cacheName, String key) {
        return null;
    }

    /**
     * 合并
     *
     * @param cacheName
     * @param targetKey
     * @param sourceKeys
     * @return
     */
    default Long redisHLLUnion(String cacheName, String targetKey, String... sourceKeys) {
        return null;
    }

    /**
     * 删除
     *
     * @param cacheName
     * @param key
     */
    default void redisHLLDelete(String cacheName, String key) {
    }

```
## GEO
```html

    /**
     * 添加
     *
     * @param cacheName
     * @param key
     * @param lon
     * @param lat
     * @param member
     * @return
     */
    default Long redisGAdd(String cacheName, String key, double lon, double lat, String member) {
        return null;
    }

    /**
     * 两点距离
     *
     * @param cacheName
     * @param key
     * @param member1
     * @param member2
     * @return
     */
    default Distance redisGDistance(String cacheName, String key, String member1, String member2) {
        return null;
    }

    /**
     * 添加
     *
     * @param cacheName
     * @param key
     * @param members
     * @return
     */
    default List<Point> redisGPos(String cacheName, String key, String... members) {
        return null;
    }

    /**
     * 获取某个成员附近（距离范围内）的成员
     *
     * @param key
     * @param member 成员
     * @param v      距离
     * @param metric 度规（枚举）（km、m）
     * @return
     */
    default List<Object> redisGRadiusByMember(String key, String member, double v, Metrics metric) {
        return null;
    }

```
## Stream(Redis5)
```html

    /**
     * 组信息
     * @param cacheName
     * @param key
     * @return
     */
    default StreamInfo.XInfoGroups redisSmGroups(String cacheName, String key) {
        return null;
    }

    /**
     * 创建消费组
     *
     * @param cacheName
     * @param key
     * @param group
     * @return
     */
    default String redisSmCreateGroup(String cacheName, String key, String group) {
        return null;
    }

    /**
     * stream 信息
     *
     * @param cacheName
     * @param key
     * @return
     */
    default StreamInfo.XInfoStream redisSmInfo(String cacheName, String key) {
        return null;
    }

    /**
     * 消费组信息
     *
     * @param cacheName
     * @param key
     * @param group
     * @return
     */
    default StreamInfo.XInfoConsumers redisSmConsumers(String cacheName, String key, String group) {
        return null;
    }

    /**
     * 确认已消费
     *
     * @param cacheName
     * @param key
     * @param group
     * @param recordIds
     * @return
     */
    default Long redisSmAck(String cacheName, String key, String group, String... recordIds) {
        return null;
    }

    /**
     * 查询Group内，被消费者取走但还没有Ack的消息
     *
     * @param cacheName
     * @param key
     * @param group
     * @return
     */
    default PendingMessagesSummary redisSmPending(String cacheName, String key, String group) {
        return null;
    }

    /**
     * 查看没有ack消息的列表
     *
     * @param cacheName
     * @param key
     * @param consumerGroup
     * @param consumerName
     * @return
     */
    default PendingMessages redisSmPending(String cacheName, String key, String consumerGroup, String consumerName) {
        return null;
    }

    /**
     * 添加消息
     *
     * @param cacheName
     * @param key
     * @param field
     * @param value
     * @return
     */
    default String redisSmAdd(String cacheName, String key, String field, Object value) {
        return null;
    }

    /**
     * 删除消息，这里的删除仅仅是设置了标志位，不影响消息总长度
     * 消息存储在stream的节点下，删除时仅对消息做删除标记，当一个节点下的所有条目都被标记为删除时，销毁节点
     *
     * @param cacheName
     * @param key
     * @param recordIds
     * @return
     */
    default Long redisSmDel(String cacheName, String key, String... recordIds) {
        return null;
    }

    /**
     * 长度
     *
     * @param cacheName
     * @param key
     * @return
     */
    default Long redisSmLen(String cacheName, String key) {
        return null;
    }

    /**
     * 从头开始读
     *
     * @param cacheName
     * @param key
     * @return
     */
    default List<MapRecord<String, Object, Object>> redisSmRead(String cacheName, String key) {
        return null;
    }

    /**
     * 从指定的 recordId 开始读
     *
     * @param cacheName
     * @param key
     * @param recordId
     * @return
     */
    default List<MapRecord<String, Object, Object>> redisSmRead(String cacheName, String key, String recordId) {
        return null;
    }
```
