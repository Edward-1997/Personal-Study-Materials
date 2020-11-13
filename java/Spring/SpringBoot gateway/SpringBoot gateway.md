## SpringBoot gateway中遇到的一些问题

这些坑究其原因还是对<font color=red>reactor编程</font>概念不理解

gateway底层是基于webflux的，这与SpringBoot中的 web包有冲突

解决方法：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- Maven整个生命周期内排除内置容器，排除内置容器导出成war包可以让外部容器运行spring-boot项目-->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

spring cloud gateway POST请求体的时候，按照命令行编程会有很多坑，获取不到数据：

```java
    //网上大多数的解决方案，在高版本中不起作用    
	AtomicReference<String> imageCode = new AtomicReference<>();
        exchange.getFormData().subscribe(map -> {
            imageCode.set(map.getFirst("imageCode"));
        });

        String codeInMemory = code.getCode();
```

