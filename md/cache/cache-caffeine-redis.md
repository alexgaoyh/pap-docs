## 概述

    配置文件调整：
      spring.cache.type=multi
      spring.redis.host=127.0.0.1
      spring.redis.port=6379
      pap.cache.multiCaffeineCacheNamesTtl.baseFileService=10
      pap.cache.multiRedisCacheNamesTtl.baseFileService=20
      pap.cache.multiCaffeineCacheNamesTtl.test=10
      pap.cache.multiRedisCacheNamesTtl.test=20
    
        1、spring.cache.type 设置为 multi , 说明一级缓存使用 caffeine， 二级缓存使用 redis
        2、按照 spring redis 相关，进行 host port 相关的配置
        3、配置相关的 cacheName ， 这里建议 caffeine 的过期时间远小于 redis，确保当前二级缓存的作用仅仅是为了减少网络开销；
        4、其他额外的非DB对应的缓存信息，可以使用 CacheOperation 进行处理：
                cacheOperation.put("test", "test", new Date().getTime() + "");
                    cacheOperation.get("test", "test");
               这里建议 cacheName 的参数采用其他的命名规则；
        
        5、建议对现有方法进行改造，比如 update 方法的返回值建议直接返回对象，这样的话， @CachePut 注解就可以使用（不建议）。
          或者说不使用 @CachePut 注解， 更新操作的话，直接删除缓存。

    二级缓存
        对于二级缓存，当前的使用只是建议减少 redis 的网络开销，将部分数据放到 caffeine 里面，这里建议 caffeine 的过期时间远小于 redis。        


## 二级缓存（Caffeine + Redis）
```html
package com.pap.cache;

import com.github.benmanes.caffeine.cache.Caffeine;
import com.pap.cache.config.PapCacheProperties;
import com.pap.cache.l2.CaffeineRedisCacheManager;
import com.pap.cache.util.CacheOperation;
import com.pap.cache.util.l2.CaffeineRedisCacheOperation;
import io.lettuce.core.cluster.ClusterClientOptions;
import io.lettuce.core.cluster.ClusterTopologyRefreshOptions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.data.redis.LettuceClientConfigurationBuilderCustomizer;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeUnit;

@Configuration
@ConditionalOnProperty(name = "spring.cache.type", havingValue = "multi", matchIfMissing = false)
public class PapCaffeineRedisConfiguration implements LettuceClientConfigurationBuilderCustomizer {

    private Logger logger = LoggerFactory.getLogger(PapCaffeineRedisConfiguration.class);

    @Autowired
    private PapCacheProperties papCacheProperties;

    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "multi", matchIfMissing = false)
    public CacheOperation cacheOperation(LettuceConnectionFactory lettuceConnectionFactory) {
        return new CaffeineRedisCacheOperation(cacheManager(lettuceConnectionFactory), redisTemplate(lettuceConnectionFactory), papCacheProperties);
    }
    @Override
    public void customize(LettuceClientConfiguration.LettuceClientConfigurationBuilder clientConfigurationBuilder) {
        /*
        ClusterTopologyRefreshOptions配置用于开启自适应刷新和定时刷新。如自适应刷新不开启，Redis集群变更时将会导致连接异常！
         */
        ClusterTopologyRefreshOptions topologyRefreshOptions = ClusterTopologyRefreshOptions.builder()
                //开启自适应刷新
//                .enableAdaptiveRefreshTrigger(ClusterTopologyRefreshOptions.RefreshTrigger.MOVED_REDIRECT, ClusterTopologyRefreshOptions.RefreshTrigger.PERSISTENT_RECONNECTS)
                .enableAllAdaptiveRefreshTriggers()
                .adaptiveRefreshTriggersTimeout(Duration.ofSeconds(10))
                //开启定时刷新,时间间隔根据实际情况修改
                .enablePeriodicRefresh(Duration.ofSeconds(15))
                .build();

        clientConfigurationBuilder.clientOptions(
                ClusterClientOptions.builder().topologyRefreshOptions(topologyRefreshOptions).build());
    }

    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "multi", matchIfMissing = false)
    public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
        //解决redis每隔一段时间强制关闭远程连接的问题
        lettuceConnectionFactory.setValidateConnection(true);
        // value序列化
        // 配置redisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());// key序列化
        redisTemplate.setValueSerializer(RedisSerializer.json());// value序列化
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());// Hash key序列化
        redisTemplate.setHashValueSerializer(RedisSerializer.json());// Hash value序列化
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "multi", matchIfMissing = false)
    public CacheManager cacheManager(LettuceConnectionFactory lettuceConnectionFactory) {
        // 使用Caffeine做本地缓存
        SimpleCacheManager caffeineCacheManager = new SimpleCacheManager();
        List<String> caffeineCacheNameList = new ArrayList<String>();

        ArrayList<CaffeineCache> caches = new ArrayList<>();
        for (Map.Entry<String, Integer> entry : papCacheProperties.getMultiCaffeineCacheNamesTtl().entrySet()) {
            CaffeineCache caffeineCache = new CaffeineCache(entry.getKey(), Caffeine.newBuilder()
                    .expireAfterWrite(entry.getValue(), TimeUnit.SECONDS)
                    .maximumSize(100)
                    // 按需开启， 打开统计信息收集
                    .recordStats()
                    .build());
            caches.add(caffeineCache);
            caffeineCacheNameList.add(entry.getKey());
        }
        caffeineCacheManager.setCaches(caches);

        // 使用redis做远程缓存
        // 配置 json 序列化器 - Jackson2JsonRedisSerializer
        List<String> redisCacheNameList = new ArrayList<String>();

        RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(RedisSerializer.json()))
                .entryTtl(Duration.ofSeconds(papCacheProperties.getDefaultTtlSecond()));

        // 针对不同cacheName，设置不同的过期时间
        Map<String, RedisCacheConfiguration> initialCacheConfiguration = new HashMap<String, RedisCacheConfiguration>();

        if (papCacheProperties.getMultiRedisCacheNamesTtl() != null && !papCacheProperties.getMultiRedisCacheNamesTtl().isEmpty()) {
            for (Map.Entry<String, Integer> entry : papCacheProperties.getMultiRedisCacheNamesTtl().entrySet()) {
                initialCacheConfiguration.put(entry.getKey(),
                        RedisCacheConfiguration.defaultCacheConfig()
                                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(RedisSerializer.json()))
                                .entryTtl(Duration.ofSeconds(entry.getValue())));
                redisCacheNameList.add(entry.getKey());
            }
        }

        RedisCacheManager redisCacheManager = RedisCacheManager.builder(lettuceConnectionFactory)
                .cacheDefaults(defaultCacheConfig) // 默认配置（强烈建议配置上）。  比如动态创建出来的都会走此默认配置
                .withInitialCacheConfigurations(initialCacheConfiguration) // 不同cache的个性化配置
                .build();

        return new CaffeineRedisCacheManager(caffeineCacheManager, redisCacheManager,
                caffeineCacheNameList, redisCacheNameList,
                redisTemplate(lettuceConnectionFactory));
    }

}

```

```html
package com.pap.cache.l2;

import com.github.benmanes.caffeine.cache.stats.CacheStats;
import org.springframework.cache.Cache;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.AbstractCacheManager;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisTemplate;

import java.util.*;

public class CaffeineRedisCacheManager extends AbstractCacheManager {
    private final SimpleCacheManager localCacheManger;

    private final RedisCacheManager remoteCacheManager;

    private final List<String> caffeineCacheNameList;

    private final List<String> redisCacheNameList;

    private final RedisTemplate<String, Object> redisTemplate;

    public CaffeineRedisCacheManager(SimpleCacheManager localCacheManager, RedisCacheManager remoteManager,
                                     List<String> caffeineCacheNameList, List<String> redisCacheNameList,
                                     RedisTemplate<String, Object> redisTemplate) {
        this.localCacheManger = localCacheManager;
        this.remoteCacheManager = remoteManager;
        this.caffeineCacheNameList = caffeineCacheNameList;
        this.redisCacheNameList = redisCacheNameList;
        this.redisTemplate = redisTemplate;
    }

    /**
     * 外部获取， hasKey()
     * @return
     */
    public RedisCacheManager getRemoteCacheManager() {
        return remoteCacheManager;
    }

    @Override
    protected Collection<? extends Cache> loadCaches() {
        ((SimpleCacheManager) localCacheManger).initializeCaches();
        ((RedisCacheManager) remoteCacheManager).initializeCaches();

        Set<String> localCacheNames = new HashSet<>(caffeineCacheNameList);
        Set<String> remoteCacheNames = new HashSet<>(redisCacheNameList);
        Collection<Cache> caches = new ArrayList<>();
        localCacheNames.forEach(name -> {
            if (remoteCacheNames.contains(name)) {
                caches.add(new CaffeineRedisCache(name, localCacheManger.getCache(name), remoteCacheManager.getCache(name),
                        localCacheManger, remoteCacheManager));
            } else {
                caches.add(localCacheManger.getCache(name));
            }
        });

        remoteCacheNames.forEach(name -> {
            if (!localCacheNames.contains(name)) {
                caches.add(remoteCacheManager.getCache(name));
            }
        });
        return caches;
    }

    /**
     * 统计 l2 二级缓存部分 Caffeine 和 Redis 的信息，可用于性能监控。
     * @return
     */
    public Map<String, Map<String, Object>> metricsInfo() {
        Map<String, Map<String, Object>> returnMap = new HashMap<>();

        // redis info
        Map<String, Object> redisInfoMap = new HashMap<>();
        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        Properties redisProperties = connection.info();
        for (final String name: redisProperties.stringPropertyNames()) {
            redisInfoMap.put(name, redisProperties.getProperty(name));
        }

        // caffeine info
        Map<String, Object> caffeineInfoMap = new HashMap<>();
        if(caffeineCacheNameList != null && caffeineCacheNameList.size() > 0) {
            for(String caffeineCacheName : caffeineCacheNameList) {
                Cache caffeineCache = localCacheManger.getCache(caffeineCacheName);
                if(caffeineCache != null && caffeineCache instanceof CaffeineCache) {
                    CaffeineCache tmp = (CaffeineCache) caffeineCache;
                    CacheStats stats = CacheStats.empty();
                    if (caffeineCache != null) {
                        stats = tmp.getNativeCache().stats();
                        caffeineInfoMap.put(caffeineCacheName + ":request-count", stats.requestCount());
                        caffeineInfoMap.put(caffeineCacheName + ":hit-count", stats.hitCount());
                        caffeineInfoMap.put(caffeineCacheName + ":miss-count", stats.missCount());
                        caffeineInfoMap.put(caffeineCacheName + ":load-success-count", stats.loadSuccessCount());
                        caffeineInfoMap.put(caffeineCacheName + ":load-failure-count", stats.loadFailureCount());
                        caffeineInfoMap.put(caffeineCacheName + ":load-failure-rate", stats.loadFailureRate());
                        caffeineInfoMap.put(caffeineCacheName + ":total-load-time", stats.totalLoadTime());
                        caffeineInfoMap.put(caffeineCacheName + ":eviction-count", stats.evictionCount());
                        caffeineInfoMap.put(caffeineCacheName + ":eviction-weight", stats.evictionWeight());
                    }

                }
            }
        }

        returnMap.put("redis", redisInfoMap);
        returnMap.put("caffeine", caffeineInfoMap);

        return returnMap;
    }

}

```

```html
package com.pap.cache.l2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.Cache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.data.redis.cache.RedisCacheManager;

import java.util.concurrent.Callable;

public class CaffeineRedisCache implements Cache {

    private Logger logger = LoggerFactory.getLogger(CaffeineRedisCache.class);

    private String name;

    private Cache localCache;

    private Cache remoteCache;

    private SimpleCacheManager localCacheManger;

    private RedisCacheManager redisCacheManager;

    public CaffeineRedisCache(String name, Cache localCache, Cache remoteCache,
                              SimpleCacheManager localCacheManger, RedisCacheManager redisCacheManager) {
        this.name = name;
        this.localCache = localCache;
        this.remoteCache = remoteCache;
        this.localCacheManger = localCacheManger;
        this.redisCacheManager = redisCacheManager;
    }

    @Override
    public String getName() {
        return this.name;
    }

    @Override
    public Object getNativeCache() {
        return this;
    }

    @Override
    public ValueWrapper get(Object key) {
        ValueWrapper valueWrapper = localCache.get(key);
        if (valueWrapper == null) {
            // logger.warn("local cache null : [{}]", key);
            valueWrapper = remoteCache.get(key);
            if (valueWrapper != null) {
                localCache.put(key, valueWrapper.get());
                // 这里额外统计一下，当前从远程缓存中获取的key的数量，后期可以用作热点操作
//                PapLRUCacheConstant.getKeyWithInitOrAdd(key.toString());
            }
        }
        return valueWrapper;
    }

    public ValueWrapper getRemote(Object key) {
        ValueWrapper valueWrapper = remoteCache.get(key);
        return valueWrapper;
    }

    @Override
    public <T> T get(Object key, Class<T> type) {
        T value = localCache.get(key, type);
        if (value == null) {
            // logger.warn("local cache null : [{}]", key);
            value = remoteCache.get(key, type);
            if (value != null) {
                localCache.put(key, value);
            }
        }
        return value;
    }

    @Override
    public <T> T get(Object key, Callable<T> valueLoader) {
        ValueWrapper valueWrapper = localCache.get(key);
        if (valueWrapper == null) {
            // logger.warn("local cache null : [{}]", key);
            T value = remoteCache.get(key, valueLoader);
            if (value != null) {
                localCache.put(key, value);
            }
            return value;
        } else {
            return (T) valueWrapper.get();
        }
    }

    @Override
    public void put(Object key, Object value) {
        if(true) {
            localCache.put(key, value);
        } else {
            // TODO 这里暂时未找到可以按照 remoteCache 处理的逻辑
        }
        if(true) {
            remoteCache.put(key, value);
        } else {
            // 如下代码的作用在于： 如果在使用过程中，某个 cacheName 对应的 ttl 过期时间要进行调整， 那么可以使用如下代码段落
            // 举例： cacheName 对应的过期时间想要动态增加，将对应的过期时间调整后放置到一个变量里面，判断当前操作的 cacheName 是不是这个在这个范围内。从而走不通的条件；
//        RedisCacheConfiguration redisCacheConfiguration = ((RedisCache) remoteCache).getCacheConfiguration();
//        ((RedisCache)remoteCache).getNativeCache().put(((RedisCache) remoteCache).getName(),
//                ByteUtils.getBytes(redisCacheConfiguration.getKeySerializationPair().write(redisCacheConfiguration.getKeyPrefixFor(((RedisCache) remoteCache).getName()) + key.toString())),
//                ByteUtils.getBytes(redisCacheConfiguration.getValueSerializationPair().write(key)),
//                Duration.ofSeconds(30));
        }

    }

    @Override
    public ValueWrapper putIfAbsent(Object key, Object value) {
        localCache.putIfAbsent(key, value);
        return remoteCache.putIfAbsent(key, value);
    }

    @Override
    public void evict(Object key) {
        localCache.evict(key);
        remoteCache.evict(key);
        // TODO 极端情况下这里可以做一个双删， 添加一个队列或者消息，隔一段时间再删除一下缓存，缓存一致性。
    }

    @Override
    public void clear() {
        localCache.clear();
        remoteCache.clear();
    }
}

```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/
