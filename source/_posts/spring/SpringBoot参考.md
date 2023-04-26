---
title: SpringBoot参考
P: frame/SpringBoot参考
date: 2022-12-12 14:13:12
categories:
  - SpringBoot
tags:
  - SpringBoot 
---

# 说明

记录一些`SpringBoot`框架中常用的技术点。

# 功能

## 发送http请求

使用`RestTemplate`发送http请求，自定义`Bean`。更多配置详见`ClientHttpRequestFactory`接口的几个实现类。

```java
@Bean
public RestTemplate restTemplate() {
    SimpleClientHttpRequestFactory simpleClientHttpRequestFactory = new SimpleClientHttpRequestFactory();
    simpleClientHttpRequestFactory.setConnectTimeout(3000);
    simpleClientHttpRequestFactory.setReadTimeout(3000);
    RestTemplate restTemplate = new RestTemplate(simpleClientHttpRequestFactory);
    // 自定义异常处理
    restTemplate.setErrorHandler(new CustomResponseErrorHandler());
    return restTemplate;
}
```

---

`RestTemplate`在响应码不为`2XX`时会直接抛异常，若不想抛异常，可通过实现`ResponseErrorHandler`来自定义异常处理程序。

```java
public class CustomResponseErrorHandler implements ResponseErrorHandler {
    @Override
    public boolean hasError(ClientHttpResponse response) throws IOException {
        return false;
    }

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
		// hasError返回true时执行
    }
}
```

## redis配置

`application.yml`配置：

```yml
spring:
  redis:
    password: 123456
    database: 0
    cluster:
      nodes:
        - 172.158.0.1:6379
        - 172.158.0.2:6379
        - 172.158.0.3:6379
        - 172.158.0.4:6379
        - 172.158.0.5:6379
        - 172.158.0.6:6379
    lettuce:
      pool: # 需依赖commons-pool2
        enabled: true
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: -1ms
      cluster:
        refresh:
          adaptive: true
          period: 15s
```

注入`RedisTemplate`：

```java
@Bean
public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
    RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
    redisTemplate.setKeySerializer(RedisSerializer.string());
    redisTemplate.setValueSerializer(RedisSerializer.string());
    redisTemplate.setHashKeySerializer(RedisSerializer.string());
    redisTemplate.setHashValueSerializer(RedisSerializer.string());
    redisTemplate.setConnectionFactory(factory);
    return redisTemplate;
}
```

## cache

启动类上添加注解`@EnableCaching`。

使用redis作为缓存:

```java
@Configuration
public class RedisCacheConfig {

    private static final String CACHE_KEY_PREFIX = "CACHE_KEY_PREFIX_";

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {

        RedisSerializer<String> redisSerializer = new StringRedisSerializer();

        JsonMapper jsonMapper = JsonMapper.builder()
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                .build();
        jsonMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        jsonMapper.activateDefaultTyping(jsonMapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL);

        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        jackson2JsonRedisSerializer.setObjectMapper(jsonMapper);

        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .prefixCacheNameWith(CACHE_KEY_PREFIX)
                .entryTtl(Duration.ofMinutes(5))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();

        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();

    }
}
```

缓存方法上添加注解：

```java
@Cacheable(value = "CACHE_NAME", key = "#appId", unless = "#result == null")
public String getNameByAppId(String appId) {
    return appDao.getNameByAppId(appId);
}
```

## 上传下载文件

### 上传

```java
public String uploadFile(String fileName) {

        String filePath = "D:\\test\\";
        FileSystemResource fileResource = new FileSystemResource(filePath + fileName);

        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.MULTIPART_FORM_DATA);

        MultiValueMap<String, Object> formData = new LinkedMultiValueMap<>();
        formData.add("file", fileResource);

        HttpEntity<MultiValueMap> httpEntity = new HttpEntity<>(formData, httpHeaders);
      
        String url = "upload url";
        String s = restTemplate.postForObject(url, httpEntity, String.class);

        return s;
    }
```

### 下载

```java
  @GetMapping("/download")
public void downloadMedia(HttpServletResponse response) {
    log.debug("下载文件");
    ClassPathResource classPathResource = new ClassPathResource("static/1.png");

    response.reset();
    response.setContentType("application/octet-stream");
    response.setCharacterEncoding("utf-8");
    response.setHeader("Content-Disposition", "attachment;filename=1.png");

    try(BufferedInputStream bis = new BufferedInputStream(classPathResource.getInputStream())) {
        byte[] buff = new byte[1024];
        OutputStream os  = response.getOutputStream();
        int i = 0;
        while ((i = bis.read(buff)) != -1) {
            os.write(buff, 0, i);
            os.flush();
        }
    } catch (IOException e) {
        log.error("下载文件异常", e);
    }

}
```

```java
// 也可用byte[].class来替换ByteArrayResource.class
ResponseEntity<ByteArrayResource> responseEntity = restTemplate.exchange(downloadUrl, HttpMethod.GET, httpEntity, ByteArrayResource.class);
```

---

或者采用`execute`方法：

```java
public ResponseEntity downloadMedia(String fileUrl, String chatbotId) {

        HttpHeaders httpHeaders = new HttpHeaders();

        String url = "xxx";

        return restTemplate.execute(url, HttpMethod.GET,
                request -> request.getHeaders().addAll(httpHeaders),
                response -> {
                    HttpHeaders responseHeaders = response.getHeaders();

                    log.debug("response headers: {}", responseHeaders);
                    if (MediaType.APPLICATION_JSON.equalsTypeAndSubtype(responseHeaders.getContentType())) {
                        String errorBody = IOUtils.toString(response.getBody(), StandardCharsets.UTF_8);
                        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorBody);
                    }

                    ByteArrayOutputStream baos = new ByteArrayOutputStream();
                    IOUtils.copy(response.getBody(), baos);

                    log.debug("response headers: {}", responseHeaders);
                    return new ResponseEntity<>(new ByteArrayResource(baos.toByteArray()), responseHeaders, HttpStatus.OK);
                });

```

## 发送邮件

**maven依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

**yml配置**

```yaml
spring:
  mail:
    host: mail.xxx.com
    port: 25
    username: xxx@xxx.com
    password: xxx
    properties:
      mail:
        smtp:
          connectiontimeout: 10000
          timeout: 10000
          writetimeout: 10000
#          socks:
#            host: 100.10.10.10
#            port: 8989
```

**邮件设置**

```java
@Resource
private JavaMailSender javaMailSender;

public void send() {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage);
    try {
        InternetAddress from = new InternetAddress(paramMap.get(Constant.xxx), "xxx系统", StandardCharsets.UTF_8.name());
        String[] to = StringUtils.split(paramMap.get(Constant.xxx), ",");
        mimeMessageHelper.setFrom(from);
        mimeMessageHelper.setTo(to);
        if (StringUtils.isNotEmpty(paramMap.get(Constant.xxx))) {
            String[] cc = StringUtils.split(paramMap.get(Constant.xxx), ",");
            mimeMessageHelper.setCc(cc);
        }
        mimeMessageHelper.setSubject("xxx提醒");
        String content = paramMap.get(Constant.xxx);
        content = StringUtils.replaceEach(content,
                                          new String[]{"${xxx}", "${xxx}"},
                                          new String[]{String.valueOf(contentParam.get("xxx")), String.valueOf(contentParam.get("xxx"))});
        mimeMessageHelper.setText(content);
        javaMailSender.send(mimeMessage);

        log.debug("send email end");

    } catch (Exception e) {
        log.debug("send email error!", e);
    }
}
```

## db

**maven**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>xxx</version>
</dependency>
<!-- oracle dependency -->
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>21.9.0.0</version>
</dependency>
<dependency>
    <groupId>com.oracle.database.nls</groupId>
    <artifactId>orai18n</artifactId>
    <version>21.9.0.0</version>
</dependency>
<!-- mysql dependency -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.30</version>
</dependency>
```

**yml**

```yaml
spring:
  datasource:
  	#url: jdbc:mysql://127.0.0.1:3306/db1?useSSL=false&useUnicode=true&characterEncoding=utf-8
    #driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:oracle:thin:@127.0.0.1:1521:orcl
    driver-class-name: oracle.jdbc.OracleDriver
    username: db1
    password: 123456
    
mybatis:
  mapper-locations: classpath:mappers/*
  type-aliases-package: com.xxx.xxx.bean
  configuration:
    map-underscore-to-camel-case: true # 转换下划线为驼峰
    jdbc-type-for-null: 'null' # oracle数据库必须配置
```

**xxxMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xxx.xxx.dao.xxxDao">

</mapper>
```

