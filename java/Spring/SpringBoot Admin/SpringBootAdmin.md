[TOC]



## SpringBoot Admin 踩坑记录

### 1、SpringBoot Admin 的配置
#### server的配置：
SpringBootAdmin服务端的版本必须是2.2.0以上否则会出现报错

pom引入依赖：
```
<dependency>
  <groupId>de.codecentric</groupId>
  <articfactId>spring-boot-admin-starter-server</articfactId>
  <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
application.yml配置：

```yml
spring:
  application:
    name: admin-server
server:
  port: 8100
```

启动类代码：
```java
@SpringBootApplication
@EnableAdminServer
public class AdminServerApplication {
    public static void main(String[] args) {
        SpringApplication.run( AdminServerApplication.class, args );
    }
}

```

然后点击启动即可，访问http://localhost:8100/ 可以看到页面：

![SpringBootAdmin Server端页面](../images/SpringBootAdmin控制台.png)

#### client的配置:

pom文件引入依赖

```maven
<dependency>
  <groupId>de.codecentric</groupId>
  <articfactId>spring-boot-admin-starter-client</articfactId>
  <version>2.2.0</version>
</dependency>
```

.yaml文件的配置

```yaml
spring:
  application:
    name: client
  boot:
    admin:
      client:
      #server的地址
        url: http://localhost:8100
        instance:
          server-base-url: http://localhost:${server.port}
server:
  port: 8110
#actuator配置：暴露敏感路径，默认情况下，敏感路径并不暴露
management:
  endpoints:
    web:
      exposure:
      #暴露多个用','分隔，暴露所有用端点用‘*’
        include: '*'
  endpoint:
    health:
    #是否展示健康详情
      show-details: ALWAYS
  #info信息会显示到SpringBootAdmin的srver端，这里取得是pom文件中的数据
  info:
  	version: @project.version@
  	groupId: @project.groupId@
  	artifactId: @project.artifactId@
```

然后点击启动即可，可以看到控制台中显示的磁盘内存信息：

![](H:\personal_file\personal_learning_storage\关于java\Spring\images\SpringAdminClient信息.PNG)

### 2、SpringBoot Admin 整合进nacos中

#### server端配置：

pom文件中引入：

```
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <articfactId>spring-cloud-starter-alibaba-nacos-discovery</articfactId>
</dependency>
```

yaml文件修改：

```yaml
spring:
  application:
    name: server
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.0.130:8848 #nacos服务中心的地址
server:
  port: 8100
#Actuator配置：暴露所有路径
managerment:
  endpoints: 
    web:
      exposure:
        include: "*"
  endpoints:
  	health:
  	  show-details: always
```

启动类代码：

```java
@SpringBootApplication
@EnableAdminServer
@EnableDiscoveryClient
public class AdminServerApplication {
    public static void main(String[] args) {
        SpringApplication.run( AdminServerApplication.class, args );
    }
}
```

client同server端配置即可，控制台上出现了admin的服务，如图：

![SpringBootAdmin整合nacos](H:\personal_file\personal_learning_storage\关于java\Spring\images\SpringBootAdmin整合nacosPNG.PNG)

因为SpringBootAdmin会自动从注册中心获取所有服务列表，因此客户端只需要在pom文件中：

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <articfactId>spring-boot-starter-actuator</articfactId>
</dependency>
```

而不需要引入SpringBootAdmin client的pom配置