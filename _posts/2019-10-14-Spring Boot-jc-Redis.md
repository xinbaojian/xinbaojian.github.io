# Spring Boot2.X 集成Redis

引入jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
```

`application.xml` 添加`redis`配置信息

```yaml
spring:
  redis:
    port: 6379
    host: 192.168.0.149
```



编写`RedisConfig.java` 配置文件

```java
package com.hngd.config;

import com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer;
import lombok.extern.log4j.Log4j2;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

/**
 * RedisConfig
 *
 * @author xin.bj
 * @date 2019-10-10 16:29
 */
@Log4j2
@EnableCaching
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
public class RedisConfig extends CachingConfigurerSupport {

    private final RedisConnectionFactory redisConnectionFactory;

    public RedisConfig(RedisConnectionFactory redisConnectionFactory) {
        this.redisConnectionFactory = redisConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // set key serializer
        StringRedisSerializer serializer = TedisCacheManager.STRING_SERIALIZER;
        // 设置key序列化类，否则key前面会多了一些乱码
        template.setKeySerializer(serializer);
        template.setHashKeySerializer(serializer);

        // fastjson serializer
        GenericFastJsonRedisSerializer fastSerializer = TedisCacheManager.FASTJSON_SERIALIZER;
        template.setValueSerializer(fastSerializer);
        template.setHashValueSerializer(fastSerializer);
        // 如果 KeySerializer 或者 ValueSerializer 没有配置，则对应的 KeySerializer、ValueSerializer 才使用这个 Serializer
        template.setDefaultSerializer(fastSerializer);

        log.info("redis: {}", redisConnectionFactory);
        LettuceConnectionFactory factory = (LettuceConnectionFactory) redisConnectionFactory;
        log.info("spring.redis.database: {}", factory.getDatabase());
        log.info("spring.redis.host: {}", factory.getHostName());
        log.info("spring.redis.port: {}", factory.getPort());
        log.info("spring.redis.timeout: {}", factory.getTimeout());
        log.info("spring.redis.password: {}", factory.getPassword());

        // factory
        template.setConnectionFactory(redisConnectionFactory);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (o, method, objects) -> {
            StringBuilder sb = new StringBuilder(32);
            sb.append(o.getClass().getSimpleName());
            sb.append(".");
            sb.append(method.getName());
            if (objects.length > 0) {
                sb.append("#");
            }
            String sp = "";
            for (Object object : objects) {
                sb.append(sp);
                if (object == null) {
                    sb.append("NULL");
                } else {
                    sb.append(object.toString());
                }
                sp = ".";
            }
            return sb.toString();
        };
    }

    @Bean
    @Override
    public CacheManager cacheManager() {
        // 初始化一个RedisCacheWriter
        RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);

        // 设置默认过期时间：30 分钟
        RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))
                //是否允许缓存null值
                // .disableCachingNullValues()
                // 使用注解时的序列化、反序列化
                .serializeKeysWith(TedisCacheManager.STRING_PAIR)
                .serializeValuesWith(TedisCacheManager.FASTJSON_PAIR);
        return new TedisCacheManager(cacheWriter, defaultCacheConfig);
    }

    /**
     * 注册Redis容器监听器
     * @return
     */
    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(){
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory);
        return container;
    }
}

```

自定义 `CacheManager`

```java
package com.hngd.config;

import com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer;
import com.hngd.annotation.CacheExpire;
import lombok.extern.log4j.Log4j2;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.cache.Cache;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.data.redis.cache.RedisCache;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.util.ReflectionUtils;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.Callable;

/**
 * @program: security-parent
 * @description: 自定义Redis缓存管理器
 * @author: xin.bj
 * @date: 2019-01-21 13:34
 */
@Log4j2
public class TedisCacheManager extends RedisCacheManager implements ApplicationContextAware, InitializingBean {

    private ApplicationContext applicationContext;

    private Map<String, RedisCacheConfiguration> initialCacheConfiguration = new LinkedHashMap<>();

    /**
     * key serializer
     */
    public static final StringRedisSerializer STRING_SERIALIZER = new StringRedisSerializer();

    /**
     * value serializer
     * <pre>
     *     使用 FastJsonRedisSerializer 会报错：java.lang.ClassCastException
     *     FastJsonRedisSerializer<Object> fastSerializer = new FastJsonRedisSerializer<>(Object.class);
     * </pre>
     */

    public static final GenericFastJsonRedisSerializer FASTJSON_SERIALIZER = new GenericFastJsonRedisSerializer();

    /**
     * key serializer pair
     */
    public static final RedisSerializationContext.SerializationPair<String> STRING_PAIR = RedisSerializationContext
            .SerializationPair.fromSerializer(STRING_SERIALIZER);
    /**
     * value serializer pair
     */
    public static final RedisSerializationContext.SerializationPair<Object> FASTJSON_PAIR = RedisSerializationContext
            .SerializationPair.fromSerializer(FASTJSON_SERIALIZER);

    public TedisCacheManager(RedisCacheWriter cacheWriter, RedisCacheConfiguration defaultCacheConfiguration) {
        super(cacheWriter, defaultCacheConfiguration);
    }

    @Override
    public Cache getCache(String name) {
        Cache cache = super.getCache(name);
        return new RedisCacheWrapper(cache);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void afterPropertiesSet() {
        String[] beanNames = applicationContext.getBeanNamesForType(Object.class);
        for (String beanName : beanNames) {
            final Class clazz = applicationContext.getType(beanName);
            add(clazz);
        }
        super.afterPropertiesSet();
    }

    @Override
    protected Collection<RedisCache> loadCaches() {
        List<RedisCache> caches = new LinkedList<>();
        for (Map.Entry<String, RedisCacheConfiguration> entry : initialCacheConfiguration.entrySet()) {
            caches.add(super.createRedisCache(entry.getKey(), entry.getValue()));
        }
        return caches;
    }

    private void add(final Class clazz) {
        ReflectionUtils.doWithMethods(clazz, method -> {
            ReflectionUtils.makeAccessible(method);
            CacheExpire cacheExpire = AnnotationUtils.findAnnotation(method, CacheExpire.class);
            if (cacheExpire == null) {
                return;
            }
            Cacheable cacheable = AnnotationUtils.findAnnotation(method, Cacheable.class);
            if (cacheable != null) {
                add(cacheable.cacheNames(), cacheExpire);
                return;
            }
            Caching caching = AnnotationUtils.findAnnotation(method, Caching.class);
            if (caching != null) {
                Cacheable[] cs = caching.cacheable();
                if (cs.length > 0) {
                    for (Cacheable c : cs) {
                        if (cacheExpire != null && c != null) {
                            add(c.cacheNames(), cacheExpire);
                        }
                    }
                }
            } else {
                CacheConfig cacheConfig = AnnotationUtils.findAnnotation(clazz, CacheConfig.class);
                if (cacheConfig != null) {
                    add(cacheConfig.cacheNames(), cacheExpire);
                }
            }
        }, method -> null != AnnotationUtils.findAnnotation(method, CacheExpire.class));
    }

    private void add(String[] cacheNames, CacheExpire cacheExpire) {
        for (String cacheName : cacheNames) {
            if (cacheName == null || "".equals(cacheName.trim())) {
                continue;
            }
            long expire = cacheExpire.expire();
            log.info("cacheName: {}, expire: {}", cacheName, expire);
            if (expire >= 0) {
                // 缓存配置
                RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                        .entryTtl(Duration.ofSeconds(expire))
                        .disableCachingNullValues()
                        // .prefixKeysWith(cacheName)
                        .serializeKeysWith(STRING_PAIR)
                        .serializeValuesWith(FASTJSON_PAIR);
                initialCacheConfiguration.put(cacheName, config);
            } else {
                log.warn("{} use default expiration.", cacheName);
            }
        }
    }

    protected static class RedisCacheWrapper implements Cache {
        private final Cache cache;

        RedisCacheWrapper(Cache cache) {
            this.cache = cache;
        }

        @Override
        public String getName() {
            try {
                return cache.getName();
            } catch (Exception e) {
                log.error("getName ---> errmsg: {}", e.getMessage(), e);
                return null;
            }
        }

        @Override
        public Object getNativeCache() {
            try {
                return cache.getNativeCache();
            } catch (Exception e) {
                log.error("getNativeCache ---> errmsg: {}", e.getMessage(), e);
                return null;
            }
        }

        @Override
        public ValueWrapper get(Object o) {
            try {
                return cache.get(o);
            } catch (Exception e) {
                log.error("get ---> o: {}, errmsg: {}", o, e.getMessage(), e);
                return null;
            }
        }

        @Override
        public <T> T get(Object o, Class<T> aClass) {
            try {
                return cache.get(o, aClass);
            } catch (Exception e) {
                log.error("get ---> o: {}, clazz: {}, errmsg: {}", o, aClass, e.getMessage(), e);
                return null;
            }
        }

        @Override
        public <T> T get(Object o, Callable<T> callable) {
            try {
                return cache.get(o, callable);
            } catch (Exception e) {
                log.error("get ---> o: {}, errmsg: {}", o, e.getMessage(), e);
                return null;
            }
        }

        @Override
        public void put(Object o, Object o1) {
            try {
                cache.put(o, o1);
            } catch (Exception e) {
                log.error("put ---> o: {}, o1: {}, errmsg: {}", o, o1, e.getMessage(), e);
            }
        }

        @Override
        public ValueWrapper putIfAbsent(Object o, Object o1) {
            try {
                return cache.putIfAbsent(o, o1);
            } catch (Exception e) {
                log.error("putIfAbsent ---> o: {}, o1: {}, errmsg: {}", o, o1, e.getMessage(), e);
                return null;
            }
        }

        @Override
        public void evict(Object o) {
            try {
                cache.evict(o);
            } catch (Exception e) {
                log.error("evict ---> o: {}, errmsg: {}", o, e.getMessage(), e);
            }
        }

        @Override
        public void clear() {
            try {
                cache.clear();
            } catch (Exception e) {
                log.error("clear ---> errmsg: {}", e.getMessage(), e);
            }
        }
    }

}
```
自定义注解，可以在缓存方法上添加过期时间。

```java
package com.hngd.annotation;

import org.springframework.core.annotation.AliasFor;

import java.lang.annotation.*;

/**
 * 自定义注解，可以在缓存方法上添加过期时间。
 *
 * @author xin.bj
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheExpire {

    /**
     * expire time, default 60s
     */
    @AliasFor("expire")
    long value() default 60L;

    /**
     * expire time, default 60s
     */
    @AliasFor("value")
    long expire() default 60L;
}

```

在启动类上引入`RedisConfig.java`

```java
@Import(`RedisConfig.class)
```

> Tips:
>
> 当初查资料完成这项功能的博客等没有记录，也忘记了，这里就不在写了
>
> 希望原作者勿怪。

