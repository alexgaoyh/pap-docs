## 概述

    本地缓存

    健康检查（误引入 redis 相关 jar， 但是未使用 redis）：

        1、可以在配置文件中增加配置：
            management.health.redis.enabled=false  根据具体情况，判断是否需要动态关闭 redis 的健康检查。

        2、
            在使用过程中，如果误引入 spring-boot-starter-actuator 和 spring-boot-starter-data-redis
                会触发进行 Redis Health Check 健康检测，除了如上的方法，还可以采用如下方法，覆盖 RedisHealthIndicator.
                避免 Redis 健康检测影响项目.

            import org.springframework.boot.autoconfigure.condition.ConditionalOnExpression;
            import org.springframework.stereotype.Component;
    
            @Component("redisHealthIndicator")
            @ConditionalOnExpression("'${spring.cache.type}'.equals('none') || '${spring.cache.type}'.equals('caffeine')")
            public class RedisHealthIndicator implements org.springframework.boot.actuate.health.HealthIndicator {
    
                @Override
                public org.springframework.boot.actuate.health.Health health() {
                    return org.springframework.boot.actuate.health.Health.up().build();
                }
            }

        3、
            直接覆盖 Bean 方法。 
                @Bean
                @ConditionalOnProperty(name = "spring.cache.type", havingValue = "caffeine", matchIfMissing = false)
                public String redisHealthIndicator() {
                    return "spring.cache.type=caffeine";
                }


## 本地缓存Caffeine

```html
package com.pap.cache;

import com.github.benmanes.caffeine.cache.Caffeine;
import com.pap.cache.config.PapCacheProperties;
import com.pap.cache.util.CacheOperation;
import com.pap.cache.util.caffeine.CaffeineCacheOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.ArrayList;
import java.util.Map;
import java.util.concurrent.TimeUnit;

@Configuration
@ConditionalOnProperty(name = "spring.cache.type", havingValue = "caffeine", matchIfMissing = false)
public class PapCaffeineConfiguration {

    private Logger logger = LoggerFactory.getLogger(PapCaffeineConfiguration.class);
    @Autowired
    private PapCacheProperties papCacheProperties;

    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "caffeine", matchIfMissing = false)
    public CacheOperation cacheOperation() {
        return new CaffeineCacheOperation(cacheManager(), papCacheProperties);
    }

    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "caffeine", matchIfMissing = false)
    public CacheManager cacheManager() {
        SimpleCacheManager manager = new SimpleCacheManager();

        ArrayList<CaffeineCache> caches = new ArrayList<>();
        for (Map.Entry<String, Integer> c : papCacheProperties.getCacheNamesTtl().entrySet()) {
            CaffeineCache caffeineCache = new CaffeineCache(c.getKey(), Caffeine.newBuilder()
                    .expireAfterWrite(c.getValue(), TimeUnit.SECONDS)
                    .maximumSize(100)
                    // 按需开启， 打开统计信息收集
                    .recordStats()
                    .build());
            caches.add(caffeineCache);
        }

        manager.setCaches(caches);

        return manager;
    }

    /**
     * 在引入 actuator 的情况下，如果 误引入 redis, 会触发健康检测，这里对其进行覆盖，避免启动过程中Redis Health 报错
     * spring.cache.type = caffeine
     *
     * @return
     */
    @Bean
    @ConditionalOnProperty(name = "spring.cache.type", havingValue = "caffeine", matchIfMissing = false)
    public String redisHealthIndicator() {
        return "spring.cache.type=caffeine";
    }

}

```

```html
package com.pap.cache.util.caffeine;

import com.github.benmanes.caffeine.cache.stats.CacheStats;
import com.pap.cache.config.PapCacheProperties;
import com.pap.cache.util.CacheOperation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.Cache;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;

import java.util.*;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;

public class CaffeineCacheOperation implements CacheOperation {

    private Logger logger = LoggerFactory.getLogger(CaffeineCacheOperation.class);

    private final CacheManager cacheManager;

    private final PapCacheProperties papCacheProperties;

    public CaffeineCacheOperation(CacheManager cacheManager, PapCacheProperties papCacheProperties) {
        this.cacheManager = cacheManager;
        this.papCacheProperties = papCacheProperties;
    }

    @Override
    public Object get(String cacheName, String key) {
        try {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                Cache.ValueWrapper valueWrapper = cache.get(key);
                if (valueWrapper != null) {
                    Object o = valueWrapper.get();
                    return o;
                }
            }
        } catch (Exception e) {
            if (e instanceof Exception) {
                logger.error("pap CacheOperation has get Exception: [{}][{}][{}]", e, cacheName, key);
            }
        }
        return null;
    }

    @Override
    public Object getRemote(String cacheName, String key) {
        logger.warn("pap CacheOperation has getRemote Exception(get from local): [{}][{}][{}]", "CaffeineCacheOperation", cacheName, key);
        return get(cacheName, key);
    }

    @Override
    public Long getTtlRemote(String cacheName, String key) {
        Integer ttl = papCacheProperties.getCacheNamesTtl().get(cacheName);
        return Long.valueOf(ttl);
    }

    @Override
    public boolean setTtlRemote(String cacheName, String key, long time, TimeUnit timeUnit) {
        return false;
    }

    @Override
    public boolean put(String cacheName, String key, String value) {
        try {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                cache.put(key, value);
                return true;
            }
        } catch (Exception e) {
            if (e instanceof Exception) {
                logger.error("pap CacheOperation has get Exception: [{}][{}][{}]", e, cacheName, key);
            }
        }
        return false;
    }

    @Override
    public boolean evict(String cacheName, String key) {
        try {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                cache.evict(key);
                return true;
            }
        } catch (Exception e) {
            if (e instanceof Exception) {
                logger.error("pap CacheOperation has evict Exception: [{}][{}][{}]", e, cacheName, key);
            }
        }
        return false;
    }

    @Override
    public String cacheNameKey(String cacheName, String key) {
        return cacheName + "::" + key;
    }

    @Override
    public boolean hasKey(String cacheName, String key) {
        CaffeineCache caffeineCache = (CaffeineCache) cacheManager.getCache(cacheName);
        Cache.ValueWrapper valueWrapper = caffeineCache.get(key);
        if(valueWrapper == null) {
            return false;
        } else {
            return true;
        }
    }

    @Override
    public List<String> keysByCacheName(String cacheName, Integer limit) {
        List<String> keysList = new ArrayList<>();

        CaffeineCache caffeineCache = (CaffeineCache) cacheManager.getCache(cacheName);
        if(caffeineCache != null) {
            com.github.benmanes.caffeine.cache.Cache<Object, Object> nativeCache = caffeineCache.getNativeCache();
            Set<Object> keySet = nativeCache.asMap().keySet();

            if(keySet != null) {
                for (Object key : keySet) {
                    keysList.add(key.toString());
                    if(keysList.size() >= limit) {
                        break;
                    }
                }
            }
        }

        return keysList;
    }

    @Override
    public Long increment(String cacheName, String key) {
        CaffeineCache caffeineCache = (CaffeineCache) cacheManager.getCache(cacheName);
        Cache.ValueWrapper valueWrapper = caffeineCache.get(key);
        if(valueWrapper == null) {
            AtomicLong count = new AtomicLong(1);
            caffeineCache.put(key, count);
            return count.get();
        } else {
            ((AtomicLong)valueWrapper.get()).incrementAndGet();
            // 再写一次，根据 caffeine.expireAfterWrite 类似一个续期，重置一下过期时间
            caffeineCache.put(key, valueWrapper.get());
            return ((AtomicLong) valueWrapper.get()).get();
        }
    }

    @Override
    public Long decrement(String cacheName, String key) {
        CaffeineCache caffeineCache = (CaffeineCache) cacheManager.getCache(cacheName);
        Cache.ValueWrapper valueWrapper = caffeineCache.get(key);
        if(valueWrapper == null) {
            AtomicLong count = new AtomicLong(-1);
            caffeineCache.put(key, count);
            return count.get();
        } else {
            ((AtomicLong)valueWrapper.get()).decrementAndGet();
            // 再写一次，根据 caffeine.expireAfterWrite 类似一个续期，重置一下过期时间
            caffeineCache.put(key, valueWrapper.get());
            return ((AtomicLong) valueWrapper.get()).get();
        }
    }

    @Override
    public Map<String, Map<String, Object>> metricsInfo() {
        if (cacheManager instanceof SimpleCacheManager) {
            Map<String, Object> caffeineInfoMap = new HashMap<>();

            SimpleCacheManager simpleCacheManager = (SimpleCacheManager) cacheManager;
            Collection<String> cacheNames = simpleCacheManager.getCacheNames();
            if (cacheNames != null && cacheNames.size() > 0) {
                for (String caffeineCacheName : cacheNames) {
                    Cache simpleCache = simpleCacheManager.getCache(caffeineCacheName);
                    if (simpleCache != null && simpleCache instanceof CaffeineCache) {
                        CaffeineCache caffeineCache = (CaffeineCache) simpleCache;
                        CacheStats stats = CacheStats.empty();
                        if (caffeineCache != null) {
                            stats = caffeineCache.getNativeCache().stats();
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
            Map<String, Map<String, Object>> metricsInfoMap = new HashMap<>();
            metricsInfoMap.put("caffeine", caffeineInfoMap);
            return metricsInfoMap;
        }
        return new HashMap<String, Map<String, Object>>();
    }
}

```

## 参考

1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh/

