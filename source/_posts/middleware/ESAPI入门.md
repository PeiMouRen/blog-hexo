---
title: ESAPI入门
date: 2022-10-17 09:53:26
categories:
  - ESAPI
tags:
  - ESAPI
  - security
---

# 简介

> ESAPI (The OWASP Enterprise Security API) is a free, open source, web application security control library that makes it easier for programmers to write lower-risk applications.
>
> ESAPI（OWASP Enterprise Security API）是一个免费的、开源的、Web 应用程序安全控制库，它使程序员更容易编写低风险的应用程序。

**官方地址：**`https://github.com/ESAPI/esapi-java-legacy`

**文档：**`https://owasp.org/www-project-enterprise-security-api/`

# 使用

**版本**

- JDK: 8
- ESAPI: 2.5.0.0

**maven坐标**

```xml
<dependency>
    <groupId>org.owasp.esapi</groupId>
    <artifactId>esapi</artifactId>
    <version>2.5.0.0</version>
</dependency>
```

**配置文件**

​		复制官网项目中`configuration/esapi`下的`ESAPI.properties`和`validation.properties`到操作系统用户目录下的`.esapi`目录（默认目录）；

​		若需要将配置文件放到其他目录，则需要在启动项目时指定`VM opton`：

`-Dorg.owasp.esapi.resources="/path/to/.esapi"`;

**使用slf4j**

`ESAPI.properties`中设置`ESAPI.Logger`属性值为`org.owasp.esapi.logging.slf4j.Slf4JLogFactory`;

## 使用api

```java
// 编码
String input = "xxx";
input = ESAPI.encoder().encodeForHTML(input);
input = ESAPI.encoder().encodeForHTMLAttribute(input);
input = ESAPI.encoder().encodeForJavaScript(input);
input = ESAPI.encoder().encodeForCSS(input);
input = ESAPI.encoder().encodeForURL(input);

// 验证
ESAPI.validator().isValidInput("",input,"Email",11,false);
```

