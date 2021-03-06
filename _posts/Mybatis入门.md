---
title: Mybatis入门
categories:
  - 框架
tags:
  - 数据库
  - mybatis
date: 2018-09-30 13:42:05
---
 这是摘要
 <!-- more -->


### 实现mybatis二级缓存
1. 实现`org.apache.ibatis.cache.Cache`接口

```java
public class RedisCache implements Cache {
   private static final Logger logger = LoggerFactory.getLogger(RedisCache.class);

   private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
   private final String id; // cache instance id
   private RedisTemplate redisTemplate;

   private static final long EXPIRE_TIME_IN_MINUTES = 30; // redis过期时间

   public RedisCache(String id) {
       if (id == null) {
           throw new IllegalArgumentException("Cache instances require an ID");
       }
       this.id = id;
   }

   @Override
   public String getId() {
       return id;
   }

   /**
    * Put query result to redis
    *
    * @param key
    * @param value
    */
   @Override
   @SuppressWarnings("unchecked")
   public void putObject(Object key, Object value) {
       RedisTemplate redisTemplate = getRedisTemplate();
       ValueOperations opsForValue = redisTemplate.opsForValue();
       opsForValue.set(key, value, EXPIRE_TIME_IN_MINUTES, TimeUnit.MINUTES);
       logger.debug("Put query result to redis");
   }

   /**
    * Get cached query result from redis
    *
    * @param key
    * @return
    */
   @Override
   public Object getObject(Object key) {
       RedisTemplate redisTemplate = getRedisTemplate();
       ValueOperations opsForValue = redisTemplate.opsForValue();
       logger.debug("Get cached query result from redis");
       return opsForValue.get(key);
   }

   /**
    * Remove cached query result from redis
    *
    * @param key
    * @return
    */
   @Override
   @SuppressWarnings("unchecked")
   public Object removeObject(Object key) {
       RedisTemplate redisTemplate = getRedisTemplate();
       redisTemplate.delete(key);
       logger.debug("Remove cached query result from redis");
       return null;
   }

   /**
    * Clears this cache instance
    */
   @Override
   public void clear() {
       RedisTemplate redisTemplate = getRedisTemplate();
       redisTemplate.execute((RedisCallback) connection -> {
           connection.flushDb();
           return null;
       });
       logger.debug("Clear all the cached query result from redis");
   }

   @Override
   public int getSize() {
       return 0;
   }

   @Override
   public ReadWriteLock getReadWriteLock() {
       return readWriteLock;
   }

   private RedisTemplate getRedisTemplate() {
       if (redisTemplate == null) {
           redisTemplate = ApplicationContextHolder.getBean("redisTemplate");
       }
       return redisTemplate;
   }
}
```