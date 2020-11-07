## WebFlux学习

### 入门

webflux处理请求(对应SpringMVC中的DispatcherServlet)：

```java
public interface WebHandler{
	Mono<Void> handle(ServerWebExchange exchange);
}
```

具体实现类DispatcherHandler

```java
@Override
public Mono<void> handle(ServerWebExchange exchange){
	if(this.handlerMappings == null){
        return createNotFoundError();
    }
    return Flux.fromIterable(this.handlerMapings)
        	.concatMap(mapping ->mapping.getHandler(exchange))
        	.next()
        	.switchIfEmpty(createNotFoundError())
        	.faltMap(handler -> invokeHandler(exchange,handler))
        	.flatMap(result -> handleResult(exchange,result));
}
```

1、ServerWebExchange对象中放置每一次HTTP请求响应信息，包括参数；

2、判断整个接口映射mapping集合是否为空，

3、根据具体的请求地址获取对应的handlerMapping

4、调用具体业务方法

5、返回结果