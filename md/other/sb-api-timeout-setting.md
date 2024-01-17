## 背景

&ensp;&ensp;《优化-Spring Boot项目服务端接口超时设置》近期断断续续处理了一些优化类问题，其中有一类问题是超时设置，本文对此进行一些介绍，避免使用不到造成未达到理想期望。

## 异步

&ensp;&ensp;很多开发者在遇到超时问题之后，第一反应是将请求改为异步，但是会遇到客户端收到了请求超时的信息，但是服务端仍在执行代码，造成前端展示与后端业务执行的误差，绝大部分解决方案如下所示。

### Callable异步

&ensp;&ensp;实现逻辑如下所示，配置文件中增加异步请求的超时时间，Controller接口中返回 Callable 对象。

```html

spring.mvc.async.request-timeout=5000

@GetMapping("/test")
public Callable<String> test() throws InterruptedException {
    Callable<String> callable = () -> { orderService.insert(new Order()); return "success"; };
    return callable;
}
```

### @Transactional(timeout = 5) 注解

&ensp;&ensp;实现逻辑如下所示，在 Service 实现类中，增加事务注解，并且设置超时时间，这样如果业务代码执行时间过长，则直接回滚代码

```html
@Override
@Transactional(timeout = 5)
public Order insert(Order order)  {
    for(int i = 0; i < 100000000; i++) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        orderRepository.save(new Order());
    }
    return order;
}
```

### 其他

&ensp;&ensp;还有类似使用 CompletableFuture 进行实现的方式，通过结合线程池的方式处理，本文不过多介绍。

## 总结

&ensp;&ensp;成本最低的解决方案是使用 @Transactional(timeout = 5) 注解，此方案不需要做额外的改动，可以快速增加服务端的超时设置，并且保证前后端理解一致。

## 参考
1. http://pap-docs.pap.net.cn/
2. https://gitee.com/alexgaoyh
