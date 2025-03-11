# Java异步并发提效招式一之CompletableFuture

在日常开发中，经常需要循环去做一些任务，对于这些任务，最直观的实现方式就是采用同步循环的串行方式。这样做的劣势就是有些任务需要花掉很长的时间，譬如说几分钟。我们可以利用JDK8
中的CompletableFuture进行异步并发的改造，通常几分钟完成的任务，几秒钟就能完成，性能提高了几十倍，还是很明显的。

## 场景一：没有返回值的任务

``` java
//异步暂停扣款
CompletableFuture<Void> allFutures = CompletableFuture.allOf(noticeDetails.stream()
.map(detail -> CompletableFuture.supplyAsync(() -> {
pause(detail);
return null;
}))
).toArray(CompletableFuture[]::new);

//所有暂停扣款成功后，更新代还通知单
allFutures.whenComplete((v, e) -> {
    if (e == null) {
        //执行后面的操作
        //...
    }
});
```

## 场景二：有返回值的任务

```java
//Guava把id list进行分片，每片长度500
Iterable<List<Long>> partitioned = Iterables.partition(ids, 500);
List<CompletableFuture<List<Result>>> futures = StreamSupport.stream(partitioned.spliterator(), false)
.map(partition -> CompletableFuture.supplyAsync(() - > resultDao.findAllWithIds(partition)))
.collect(Collectors.toList());

//等待所有future完成，然后获取结果
futures.stream().map(CompletableFuture:join).flatMap(List::Stream).collect(Collectors.toList());
```