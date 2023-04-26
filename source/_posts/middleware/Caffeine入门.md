---
title: Caffeine入门
date: 2022-08-18 14:09:19
categories: 本地缓存
tags:
  - 缓存
  - Caffeine
---

# 1. 简介

> Caffeine是一个基于Java8开发的提供了近乎**最佳命中率**的**高性能**的本地缓存库。

官网：`https://github.com/ben-manes/caffeine`

# 2. 使用

## 2.1. maven坐标

> jdk11或更高版本使用`3.x`版本，其他使用`2.x`版本

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.1</version>
</dependency>
```

## 2.2. 添加缓存

> Caffeine提供了四种缓存添加策略：**手动加载**、**自动加载**、**手动异步加载**和**自动异步加载**。

### 2.2.1. 手动加载

```java
String key = "key";
String value;
// 定义缓存
Cache<String, String> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .build();

// 查找一个缓存元素，不存在时返回null
value = cache.getIfPresent(key);

// 查找缓存，若缓存不存在则生成缓存元素并返回，若无法生成则返回null，该方法为原子操作
value = cache.get(key, k -> "v1");

// 添加或更新一个缓存
cache.put(key, "v2");

// 移除一个缓存
cache.invalidate(key);

// 使用map对缓存进行操作
ConcurrentMap<String, String> cacheMap = cache.asMap();
cacheMap.put(key, "v3");
```

### 2.2.2. 自动加载

> 相当于把手动加载时生成缓存的策略(`value = cache.get(key, k -> "v1");`)直接配置到缓存对象中` .build(k -> k + "-v")`。

```java
String key = "key";
String value;
// 定义自动加载的缓存，加载策略为key + "-v"
LoadingCache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(k -> k + "-v");

// 查找缓存，不存在时生成缓存并返回，生成失败时返回null
value = cache.get("key");

List<String> keys = new ArrayList<>();
keys.add("k1");
keys.add("k2");
// 批量查找元素，不存在时生成缓存并返回，生成失败时返回null
Map<String, String> values = cache.getAll(keys);
```

### 2.2.3. 手动异步加载

```java
String key = "key";
CompletableFuture<String> value;
// 定义缓存对象
AsyncCache<String, String> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .buildAsync();

// 异步获取缓存，不存在时返回null
value = cache.getIfPresent(key);
if (value != null) {
    System.out.println(value.get());
}

// 获取缓存，缓存不存在时异步生成
value = cache.get(key, k -> k + "-v");
System.out.println(value.get());

// 添加或更新一个元素
CompletableFuture<String> v1 = CompletableFuture.supplyAsync(() -> "1111");
cache.put(key, v1);

// 移除一个缓存元素
// synchronous()方法给Cache提供了阻塞直到异步缓存生成完毕的能力
cache.synchronous().invalidate(key);
```

### 2.2.4. 自动异步加载

```java
String key = "key";
CompletableFuture<String> value;

// 定义缓存对象
AsyncLoadingCache<String, String> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .buildAsync(k -> k + "-v");

// 查找缓存元素，若不存在，则异步生成
value = cache.get(key);

List<String> keys = new ArrayList<>();
keys.add("k1");
keys.add("k2");
// 批量查找缓存元素，若不存在，则异步生成
CompletableFuture<Map<String, String>> vs = cache.getAll(keys);
```

## 2.3. 驱逐缓存

> 驱逐：缓存元素因满足策略而被移除。
>
> Caffeine提供了三种驱逐策略，分别是**基于容量**、**基于时间**和**基于引用**三种类型。

### 2.3.1. 基于容量

> **基于元素个数的驱逐**和**基于元素权重的驱逐**不能同时使用。

```java
// 基于缓存内的元素个数进行驱逐
Caffeine.newBuilder()
    .maximumSize(10_000)
    .build();

// 基于缓存内元素的权重进行驱逐
Caffeine.newBuilder()
    .maximumWeight(10_000)
    .weigher((String k, String v) -> v.length())
    .build();
```

### 2.3.2. 基于时间

```java
// 指定缓存在创建或者更新后，经过固定时间删除
Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

// 指定缓存在创建、更新或者读取后，经过固定时间删除
Caffeine.newBuilder()
    .expireAfterAccess(10, TimeUnit.MINUTES)
    .build();

Expiry<String, String> expiry = new Expiry<>() {
    @Override
    public long expireAfterCreate(String key, String value, long currentTime) {
        // TODO:计算该缓存在创建后多久失效
        return 0;
    }

    @Override
    public long expireAfterUpdate(String key, String value, long currentTime, @NonNegative long currentDuration) {
        // TODO:计算该缓存在更新后多久失效
        return 0;
    }

    @Override
    public long expireAfterRead(String key, String value, long currentTime, @NonNegative long currentDuration) {
        // TODO:计算该缓存在读取后多久失效
        return 0;
    }
};
// 自定义缓存失效策略
Caffeine.newBuilder()
    .expireAfter(expiry)
    .build();
```

### 2.3.3. 基于引用

```java
// 当key和缓存元素都不存在强引用时驱逐
Caffeine.newBuilder()
    .weakKeys()
    .weakValues()
    .build();

// 当进行GC时进行驱逐
Caffeine.newBuilder()
    .softValues()
    .build();
```

## 2.4. 失效缓存

> 失效：手动移除缓存元素。

```java
Cache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

// 移除指定key
cache.invalidate("key");

List<String> keys = new ArrayList<>();
keys.add("k1");
keys.add("k2");
keys.add("k3");
// 批量移除key
cache.invalidateAll(keys);

// 移除所有key
cache.invalidateAll();
```

## 2.5. 刷新缓存

> ​		刷新和驱逐并不相同。可以通过`LoadingCache.refresh(K)`方法，异步为key对应的缓存元素刷新一个新的值。与驱逐不同的是，在刷新的时候如果查询缓存元素，其旧值将仍被返回，直到该元素的刷新完毕后结束后才会返回刷新后的新值。
>
> ​		与 `expireAfterWrite`相反，`refreshAfterWrite` 将会使在写操作之后的一段时间后允许key对应的缓存元素进行刷新，但是只有在这个key被真正查询到的时候才会正式进行刷新操作。所以打个比方，你可以在同一个缓存中同时用到 `refreshAfterWrite`和`expireAfterWrite` ，这样缓存元素的在被允许刷新的时候不会直接刷新使得过期时间被盲目重置。当一个元素在其被允许刷新但是没有被主动查询的时候，这个元素也会被视为过期。
>
> ​		一个`CacheLoader`可以通过覆盖重写 `CacheLoader.reload(K, V)` 方法使得在刷新中可以将旧值也参与到更新的过程中去，这也使得刷新操作显得更加智能。

```java
LoadingCache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .refreshAfterWrite(5, TimeUnit.MINUTES)
    .build(k -> k + "-v");
```

## 2.6. 性能统计

> 通过使用`Caffeine.recordStats()`方法可以打开数据收集功能。`Cache.stats()`方法将会返回一个`CacheStats`对象，其将会含有一些统计指标。

```java
Cache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .recordStats()
    .build();

CacheStats stats = cache.stats();
double hitRate = stats.hitRate(); // 缓存命中率
long evictionCount = stats.evictionCount(); // 被驱逐的缓存数量
double averageLoadPenalty = stats.averageLoadPenalty(); // 加载新值的平均毫秒数
```

## 2.7. 缓存规范

```java
// 定义缓存规范
CaffeineSpec spec = CaffeineSpec.parse("maximumWeight=1000, expireAfterWrite=10m, recordStats");
// 利用缓存规范来创建缓存
Cache<Object, Object> cache = Caffeine.from(spec)
    .refreshAfterWrite(5, TimeUnit.MINUTES)
    .build();
```

# 3. Spring Boot集成

> 主要通过`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`两个接口实现缓存抽象。

## 3.1. 配置

**maven坐标**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.1</version>
</dependency>
```

**配置文件**

```yaml
spring:
  cache:
    type: caffeine
    cache-names: hello,hi
    caffeine:
      spec: maximumSize=500,expireAfterAccess=10m
```

**启动类添加注解**

```
@EnableCaching
```

## 3.2. 使用注解

### 3.2.1. @Cacheable

作用：触发缓存填充，即缓存特定参数下方法的返回值。

示例：缓存名为hello，生成密钥的key为参数name，生成缓存的条件为name的长度小于10

```java
@Cacheable(cacheNames = "hello", key = "#name", condition = "#name.length() < 10")
@GetMapping("/hello/{name}/{gender}/{age}")
public String hello(@PathVariable("name") String name, @PathVariable("gender") String gender, @PathVariable("age") int age) {
    try {
        System.out.println(name + " - " + gender + " - " + age);
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
    return "hello " + name;
}
```

### 3.2.2. @CachePut

作用：在不干扰方法执行的情况下更新缓存。

示例：更新key为name时，hello的缓存值。

```java
@CachePut(cacheNames = "hello", key = "#name")
@GetMapping("/hi")
public String updateHello(String name) {
    return "hi " + name;
}
```

### 3.2.3. @CacheEvict

作用：驱逐缓存。

示例：驱逐hello中所有缓存。

```java
@CacheEvict(cacheNames = "hello", allEntries = true)
@GetMapping("/evict")
public void evict() {
    System.out.println("evict cache");
}
```

### 3.2.4. @Caching

作用：组合多个注解。

```java
@Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
public Book importBooks(String deposit, Date date)
```

### 3.2.5. @CacheConfig

作用：类级别的注释，指定当前类中缓存注解的公共属性。

```java
@CacheConfig(cacheNames = "hello")
```

