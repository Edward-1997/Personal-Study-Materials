

## Recator 编程模式

### 时代局限性

-阻塞编程

-异步编程：callback，future

### 核心接口（优先使用Lambda表达式）

-Mono: 异步 0-1元素

-Flux：异步0-N元素，示例：

``` java
        Random random = new Random();
        //订阅并且处理（控制台输出）
        Flux.just(1,2,3)
                .map(value->{
                    if(random.nextInt(6) == 3){//当随机数为3时抛出异常
                     throw new RuntimeException("出现异常");
                    }
                    return value;
                })
                .subscribe(
                        System.out::println,//onNext 数据处理
                        System.out::println,//onError 数据出现异常
                        ()->{System.out.println("订阅成功");}//数据成功订阅后打印
                        );
```

多线程编程：

```java
        Flux.range(0,10)
                .publishOn(Schedulers.single())//异步单线程执行
                .subscribe(System.out::println);
        
        Flux.range(0,10)
                .publishOn(Schedulers.parallel())//异步并行执行
                .subscribe(System.out::println);
```

publishOn(Schedulers.xxxx())选择执行方式

-接口编程：

`Supplier`: 只出（返回）不进（参数）

```java
/**
 * Represents a supplier of results.
 */
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

`Consumer`：只进（参数）不出（返回）

```java
/**
*Represents an operation that accepts a single input argument *and returns no result
*/
@FunctionalInterface
public interface Consumer<T> {
	void accept(T t);
}
```

`Function`：又进（参数）又出（返回）

```java
//Represents a function that accepts one argument and produces a result
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

`BiFuncation` : 二元操作



