---
title: cas入门
date: 2022-09-09 09:37:53
categories:
  -	单点登录
tags:
  -	sso
  -	cas
---

[toc]

# 简介

`CAS`，即`Central Authentication Service`，是一个开源的企业多语言单点登录的解决方案。

github:`https://github.com/apereo/cas`

home:`https://www.apereo.org/projects/cas`

# CAS协议

- `TGT(Ticket Granting Ticket)`: 代表用户的一个`sso session`会话。
- TGC: 保存`TGT`的`cookie`。
- `ST(Service Ticket)`: 作为参数放在`url`中传递，代表用户访问应用的凭证。

![cas流程图](cas流程.png)

# 使用

> cas6版本后基于gradle构建代码，并推荐使用`waroverlay`的方式来进行使用和开发。
>
> war overlay简介：`https://maven.apache.org/plugins/maven-war-plugin/overlays.html`
>
> 即相同目录、相同文件名的内容会被覆盖掉，可做针对型开发。

## 版本

`cas:6.4`、`jdk:11`

## 获取代码

项目`README.md`中介绍了可以使用的`task`，但有一些不能使用。

```bash
git clone -b 6.4 https://github.com/apereo/cas-overlay-template.git
cd cas-overlay-template
```

## 编译

```bash
./gradlew[.bat] clean build
```

编译完成后会生成`build`目录，包含war包、配置文件等信息。

## 运行

### 使用内部容器运行

```bash
./gradlew[.bat] run 
```

启动后访问`http://localhost:8443`，默认用户为`casuser/Mellon`。

### 使用外部容器运行

```bash
./gradlew[.bat] clean build
```

编译完成后，在`build/libs`目录下会生成`cas.war`，将war包放到外部tomcat的`webapps`目录下并启动tomcat，访问`http://localhost:8080`，默认用户为`casuser/Mellon`。

# 配置

## 获取内置资源

```bash
./gradlew[.bat] listTemplateViews
```

执行后，会在`build/cas-resources`下生成cas内置的资源，包括配置文件、页面等。

其中`build/cas-resources/application.properties`为默认配置文件，如需修改，只需在`src/main/resources/application.yml`中覆盖对应配置即可。

## 使用http

**`build.gradle`配置文件中`dependencies`添加配置**

```
// 读取json配置
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"
```

**修改`src/main/resources/application.yml`，添加如下配置**

```yaml
server:
  ssl:
    enabled: false
cas:  
  tgc:
    secure: false # 允许使用http协议发送cookie，默认为true
  service-registry:
    core:
      init-from-json: true # 允许从json中注册服务
    json:
      location: classpath:/services # json文件地址
```

**添加json配置文件**

```bash
./gradlew[.bat] listTemplateViews
```

将`build/cas-resources/services`目录复制到`src/main/resources`下，并修改json文件中`serviceId`属性，添加`http`。

## 使用ssl

**使用keytool生成证书**

```bash
# 生成证书，别名：cas 算法：RSA key密码：123456 存储库密码：123456 存储库位置：D/cas/keystore
keytool -genkeypair -alias cas -keyalg RSA -keypass 123456 -storepass 123456 -keystore D:/cas/keystore

# 导出证书 别名：cas 存储库位置：D/cas/keystore 存储库密码：123456 导出的证书位置：D:/cas/cas.cer
keytool -exportcert -alias cas -keystore D:/cas/keystore -storepass 123456 -file D:/cas/cas.cer

# 将生成的证书导入jdk的证书库，jdk证书库的默认密码为changeit
keytool -importcert -keystore D:/util/jdk/openJDK11/lib/security/cacerts -file D:/cas/cas.cer -alias cas
```

**修改cas配置文件**

修改`src/main/resources/application.yml`，添加如下配置，用于覆盖掉默认的`ssl`配置。

```yaml
server:
  ssl:
    enabled: true
    key-store: file:D:/cas/keystore
    key-store-password: 123456
    key-password: 123456
```

**设置域名映射**

修改`C:\Windows\System32\drivers\etc\hosts`，添加域名映射`127.0.0.1  sso.rhythm.com`；

刷新dns：`ipconfig /flushdns`

**重新编译运行**

```bash
./gradlew[.bat] clean build run
```

启动成功后，访问`https://sso.rhythm.com:8443`。

## 使用jdbc验证账号密码

**`build.gradle`配置文件中`dependencies`添加配置**

```
// jdbc验证账号密码
implementation "org.apereo.cas:cas-server-support-jdbc:${project.'cas.version'}"
implementation "org.apereo.cas:cas-server-support-jdbc-drivers:${project.'cas.version'}"
```

**修改`src/main/resources/application.yml`，添加如下配置，用于覆盖默认的安全策略**

```yaml
cas:
  authn:
    jdbc: # 使用jdbc验证账号密码
      query:
        - url: jdbc:oracle:thin:@172.17.0.3:1521:ORCL
          driver-class: oracle.jdbc.driver.OracleDriver
          dialect: org.hibernate.dialect.Oracle10gDialect
          user: zs
          password: 123456
          password-encoder:
            type: DEFAULT
            encoding-algorithm: MD5 # 密码加密算法
          field-password: PASSWORD_
          sql: select t.PASSWORD_ from T_USER_ t where lower(t.ACCOUNT_) = lower(?) and t.STATUS_ = 'Y'
    accept:
      enabled: false # 禁用默认的账户名密码
```

**重新编译运行**

```bash
./gradlew[.bat] clean build run
```

启动成功后，访问`https://localhost:8443`，并使用数据库中的账号密码登录。

## 自定义登录页面

在`src/main/resources/`下新建`templates/login`目录，在新建的`login`目录下新建登录页面`casLoginView.html`，页面使用的静态资源方位`/src/main/resources/static/`目录下，再参考默认的登录页面（`build/cas-resources/templates/login/casLoginView.html`）开发自己的登录页面。

示例：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>title</title>
    <style type="text/css">
        *{ padding:0; margin:0; border:0;}
        body { background:#153767; width:100%; }
        .box{ background:url(images/bg_1.jpg) no-repeat top center; width:100%; min-height:610px;_height:610px;}
        .login{width:422px;padding:0 0 0 48px; height:200px; position:relative; margin:0 auto;top:250px;}
        .wen{ color:#c9d9e4; font-size:12px; font-weight:bold; height:72px; overflow:hidden;}
        .denglu{ position:absolute; cursor: pointer; bottom:40px; right:40px; background:url(images/login_1.jpg) no-repeat; width:78px; height:60px; text-indent:-999px; overflow:hidden; margin:0; padding:0;}
        .login ul{ list-style:none; padding-top:5px;}
        .login ul li{ float:left; height:30px; width:100%; overflow:hidden;}
        .login ul li span{ float:left; width:60px; line-height:22px; color:#fff; font-size:12px;}
        .login ul li input{ border:1px solid #163865; background:#88adc0; width:115px; height:20px; line-height:17px;}
        .login ul li img{position:absolute; bottom:37px; right:145px; width:110px; height:30px; line-height:22px;}
        .STYLE1 {font-size: 16px}
        .img_captcha{
            cursor: pointer;
        }
    </style>
    <script type="text/javascript" src="js/jquery.js"></script>
    <script type="text/javascript">
        $(document).ready(function(){
            $("#captchawordimage").click(function(){
                reloadCaptchaImage();
            });
            reloadCaptchaImage();
        });

        function reloadCaptchaImage(){
            var path = "captcha?ct="+ new Date().getTime();
            $("#captchawordimage").attr("src", path);
        }
    </script>
</head>
<body>

<div class="box">
    <div class="login">
        <div class="wen">
            <span class="STYLE1">· </span>系统1<br> <span class="STYLE1">·</span>
            系统2 <br> <span class="STYLE1">·</span> 系统3
        </div>
        <form id="fm1" action="login" th:action="@{login(service=${param.service})}" method="post">
            <ul>
                <li><span>帐 户：</span> <input style="width: 215px;" id="username" name="username" type="text"></li>
                <li><span>密 码：</span> <input style="width: 215px;" type="password" id="password" name="password"></li>
                <li><span>验证码：</span> <input style="width: 80px;" type="text" id="captchaword" name="captchaword"> <img id="captchawordimage" alt="图片验证码" class="img_captcha"></li>
            </ul>
            <div>
                <input type="hidden" name="lt" th:value="${loginTicket}">
                <input type="hidden" name="execution" th:value="${flowExecutionKey}">
                <input type="hidden" name="_eventId" value="submit">
                <input type="submit" class="denglu" name="submit">
            </div>
        </form>
        <div id="status" style="position: absolute; left: 110px; top: 161px; font-family: Verdana, 宋体, Arial, Sans; font-size: 10pt; color: red">
            <div id="errorPanel" th:if="${flowRequestContext.messageContext.hasErrorMessages()}">
                <p th:each="message : ${flowRequestContext.messageContext.allMessages}"
                   th:utext="#{${message.text}}">
                    Error Message Text
                </p>
            </div>
            <div th:if="${param.ve + ''} eq '1'">
                <p>验证码错误！</p>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```

## 注入bean到spring容器

在`src/main/java`目录下新建bean后，不会注入到spring容器中，需要在`src/main/resources/META-INF/spring.factories`文件中进行配置。

示例：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.apereo.cas.config.CasOverlayOverrideConfiguration,\
  com.rhythm.controller.CaptchaController,\
  com.rhythm.filters.CaptchaValidateFilter,\
  com.rhythm.config.FilterConfig
```

# cas客户端

**java-client：**`https://github.com/apereo/java-cas-client`

## 使用`web.xml`进行配置

### 引入maven依赖

```xml
<dependency>
    <groupId>org.jasig.cas.client</groupId>
    <artifactId>cas-client-core</artifactId>
    <version>${java.cas.client.version}</version>
</dependency>
```

### web.xml中添加配置

**配置身份验证过滤器`AuthenticationFilter`**

- casServerUrlPrefix：指定cas server的地址
- service：指定验证成功后跳转的地址

```xml
<filter>
  <filter-name>CAS Authentication Filter</filter-name>
  <filter-class>org.apereo.cas.client.authentication.AuthenticationFilter</filter-class>
  <init-param>
    <param-name>casServerUrlPrefix</param-name>
    <param-value>https://battags.ad.ess.rutgers.edu:8443/cas</param-value>
  </init-param>
  <init-param>
    <param-name>service</param-name>
    <param-value>http://www.acme-client.com</param-value>
  </init-param>
</filter>
<filter-mapping>
    <filter-name>CAS Authentication Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**配置票据验证过滤器`TicketValidationFilter`**

- casServerUrlPrefix：指定cas server的地址
- service：指定验证成功后跳转的地址

```xml
<filter>
  <filter-name>CAS Validation Filter</filter-name>
  <filter-class>org.apereo.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter</filter-class>
  <init-param>
    <param-name>casServerUrlPrefix</param-name>
    <param-value>https://battags.ad.ess.rutgers.edu:8443/cas</param-value>
  </init-param>
  <init-param>
    <param-name>service</param-name>
    <param-value>http://www.acme-client.com</param-value>
  </init-param>
</filter>
<filter-mapping>
    <filter-name>CAS Validation Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**配置请求包装过滤器`HttpServletRequestWrapperFilter`**

```xml
<filter>
  <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
  <filter-class>org.apereo.cas.client.HttpServletRequestWrapperFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

**配置断言`AssertionThreadLocalFilter`**

```xml
<filter>
  <filter-name>CAS Assertion Thread Local Filter</filter-name>
  <filter-class>org.apereo.cas.client.AssertionThreadLocalFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>CAS Assertion Thread Local Filter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

**配置单点登出过滤器`SingleSignOutFilter`和监听器`SingleSignOutHttpSessionListener`**

```xml
<filter>
   <filter-name>CAS Single Sign Out Filter</filter-name>
   <filter-class>org.apereo.cas.client.session.SingleSignOutFilter</filter-class>
</filter>
<filter-mapping>
   <filter-name>CAS Single Sign Out Filter</filter-name>
   <url-pattern>/*</url-pattern>
</filter-mapping>
<listener>
    <listener-class>org.apereo.cas.client.session.SingleSignOutHttpSessionListener</listener-class>
</listener>
```

## 使用spring进行配置

在`web.xml`配置的基础上，可以使用`spring ioc`配置来代替`AuthenticationFilter`和`TicketValidationFilter`。

**注册`AuthenticationFilter`和`TicketValidationFilter`Bean**

```java
@Configuration
public class CasConfig {
    @Bean("authenticationFilter")
    public AuthenticationFilter authenticationFilter(ParamService paramService) {
        AuthenticationFilter authenticationFilter = new AuthenticationFilter();
        authenticationFilter.setCasServerUrlPrefix("https://battags.ad.ess.rutgers.edu:8443/cas");
        authenticationFilter.setService("http://www.acme-client.com");

        return authenticationFilter;
    }

    @Bean("ticketValidationFilter")
    public Cas20ProxyReceivingTicketValidationFilter validationFilter(ParamService paramService) {
paramService.getParamValueAsString("SSO_SERVER_PREFIX_URL");
        Cas20ProxyReceivingTicketValidationFilter validationFilter = new Cas20ProxyReceivingTicketValidationFilter();
        validationFilter.setService("http://www.acme-client.com");
        Cas20ServiceTicketValidator validator = new Cas20ServiceTicketValidator("https://battags.ad.ess.rutgers.edu:8443/cas");
        validationFilter.setTicketValidator(validator);
        return validationFilter;
    }

}
```

**修改web.xml配置**

`web.xml`中删除`AuthenticationFilter`和`TicketValidationFilter`相关配置，并添加如下配置：

```xml
<filter>
    <filter-name>CAS Authentication Filter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>authenticationFilter</param-value>
    </init-param>
  </filter>
<filter-mapping>
    <filter-name>CAS Authentication Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<filter>
    <filter-name>CAS Validation Filter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>ticketValidationFilter</param-value>
    </init-param>
  </filter>
<filter-mapping>
    <filter-name>CAS Validation Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 使用springboot进行配置

**maven依赖**

```xml
<dependency>
    <groupId>org.jasig.cas.client</groupId>
    <artifactId>cas-client-support-springboot</artifactId>
    <version>${java.cas.client.version}</version>
</dependency>
```

**application.yml配置**

```yaml
cas:
  server-url-prefix: http://localhost:8443/cas
  server-login-url: http://localhost:8443/cas/login
  client-host-url: http://localhost:8080
  validation-type: CAS
  single-logout:
    enabled: true
```

**其他配置**

启动类上添加`@EnableCasClient`注解
