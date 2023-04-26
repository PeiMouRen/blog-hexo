---
title: SpringMVC替换Struts1
date: 2022-08-30 11:33:18
categories: 
  - Spring
tags:
  - SpringMVC
  - Struts1.x
  - Framework
---

# 1. 概述

web项目由`Spring + Struts1`架构变更为`Spring + SpringMVC`架构。

Spring版本：5.3.22

Struts版本：1.3.10

# 2. 步骤

## 2.1. 调整maven依赖

删除`Struts1`的依赖，添加`SpringMVC`的依赖，

**示例Spring + SpringMVC依赖：**

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>${spring.version}</version>
</dependency>
```

## 2.2. 调整配置文件

### 2.2.1. web.xml

去掉`ActionServlet`的配置，添加`SpringMVC`的配置。

<p style="color:red">注意：SpringMVC的url-pattern属性要设置为/,表示不拦截jsp请求，/*则会拦截所有请求，包括jsp。</p>

**示例Spring + SpringMVC配置：**

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<servlet>
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>

<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-*.xml</param-value>
</context-param>
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

### 2.2.2. spring-beans.xml

`Spring`的配置文件，配置不扫描`Controller`类，示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.xx">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    
    <!-- 省略其他配置 -->
</beans>
```

### 2.2.3. spring-mvc.xml

添加`SpringMVC`的配置文件，示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-2.0.xsd
            http://www.springframework.org/schema/context
            https://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/mvc
            https://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/aop
            https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 启用MVC配置 -->
    <mvc:annotation-driven/>

    <!-- 扫描controller -->
    <context:component-scan base-package="com.xx" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 配置视图解析 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    
    <!-- 配置文件上传（使用commons-fileupload） -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10240" />
        <property name="defaultEncoding" value="UTF-8" />
    </bean>
    
    <!-- 启用@AspectJ支持 -->
    <aop:aspectj-autoproxy />

    <!-- 配置静态资源映射 -->
    <mvc:resources mapping="/css/**" location="/css/"/>
    <mvc:resources mapping="/images/**" location="/images/"/>
    <mvc:resources mapping="/js/**" location="/js/"/>
</beans>
```

## 2.3. 调整form bean

将`Struts`的配置文件`struts-config.xml`中`<form-beans>`内配置的`form bean`去掉对`ActionForm`的继承。

`FormFile`可用`MultipartFile`替换。

## 2.4. 调整Action

​		根据`Struts`的配置文件`struts-config.xml`中`<action-mappings>`内配置的`path`和`Action`的映射，修改对应的`Action`。

- `struts-config.xml`中`Action`的`path`为页面请求路径去除掉`.do`后的部分，如页面请求路径为`/index.do?method=add`，则`path`配置为`/index`;

- `Action`不再继承`DispatchAction`类；
- 根据`struts-config.xml`中的映射，给`Action`添加`@Controller`和`@RequestMapping(path)`注解；
- `Action`类内部的方法调整：
  - 添加`@RequestMapping`注解，`value`设置为方法名；
  - 不再需要`ActionMapping`参数，`SpirngMVC`直接返回`jsp`路径，配合**视图解析器**来返回页面；
  - `ActionForm`参数替换成`2.3.步`调整后的`JavaBean`，`Spring`会自动将参数封装为对应的`JavaBean`；
  - 返回值不再是`ActionForward`，按情况调整为`String`或`void`，用于返回`jsp`路径或进行其他操作；

	## 2.5. 调整jsp

​		由于`struts`的`ActionServlet`拦截的是`*.do`的请求，而`SpringMVC`拦截的是`/`请求，所以`jsp`中涉及到请求路径的地方都需要调整：

- 静态资源使用绝对路径，如`${pageContext.request.contextPath}/css/xx.css`；
- 请求`url`由`xx.do?method=xx`修改为`SpringMVC`格式，如`index.do?method=top&info=1`替换为`${pageContext.request.contextPath}/index/top?info=1`；

# 3. 其他

## 3.1. 使用idea批量查找/替换

idea批量查找快捷键：`shift + ctrl + f`

idea批量替换快捷键：`shift + ctrl + r`

可以批量查找/替换静态资源、请求路径url等；

---

**批量替换静态资源url示例：**

将`src="js`批量替换为`src="${pageContext.request.contextPath}/js`;

将`href="./css`批量替换为`href="${pageContext.request.contextPath}/css`;

---

**批量替换请求路径示例：**

将`.do?method=`批量替换为`/`;

<p style="color:red;">注意：上面替换请求路径的方式存在问题：若请求路径为`index.do?method=top&info=1`用上面的方法替换过后，请求路径变为`index/top&info=1`,显然在此基础上还需要将`&`替换为`?`；所以，若使用了上面的替换方法，后续需要大量检查测试。</p>

## 3.2. 请求路径为空的问题

当请求路径为空时，表示提交到当前页面；

例如：当前页面请求路径为`/index/top`，页面中<form>表单中`action`属性的值为`?info=1`，提交表单后，请求提交路径为`/index/top?info=1`；

因此，在修改`jsp`中的请求路径时，若请求路径为空（可以带参数），则无需调整该路径；

## 3.3. 规范代码

若时间充裕，可将`Action`类的统一调整为`xxController`格式；
