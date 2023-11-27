---
title: Redis预减库存，优化系统方案
date: 2021-12-03 21:32:33
updated: 2021-12-03 21:32:33
tags:
  - Redis
categories:
  - 技术
comments: true
---
Redis预减库存：**主要思路减少对数据库的访问，之前的减库存，直接访问数据库，读取库存，当高并发请求到来的时候，大量的读取数据有可能会导致数据库的崩溃。**



**思路：**

1. 系统初始化的时候，将商品库存加载到Redis 缓存中保存
2. 收到请求的时候,现在Redis中拿到该商品的库存值，进行库存预减，如果减完之后库存不足，直接返回逻辑Exception就不需要访问数据库再去减库存了，如果库存值正确，进行下一步
3. 将请求入队，立即给前端返回一个值，表示正在排队中，然后进行秒杀逻辑，后端队列进行秒杀逻辑，前端轮询后端发来的请求，如果秒杀成功，返回秒杀，成功，不成功就返回失败。

（后端请求 单线程 出队，生成订单，减少库存，走逻辑）前端同时轮询

1. 前端显示

### 第一步：预减库存

```
/**
 * 秒杀接口优化之---   第一步:  系统初始化后就将所有商品库存放入 缓存
 */
@Override
public void afterPropertiesSet() throws Exception {
    List<GoodsVo> goods = goodsService.getGoodsList();
    if (goods == null) {
        return;
    }
    for (GoodsVo goodsVo : goods) {
        redisService.set(GoodsKey.getGoodsStock, "" + goodsVo.getId(), goodsVo.getStockCount());
        isOverMap.put(goodsVo.getId(), false);//先初始化 每个商品都是false 就是还有
    }
}
/**秒杀接口优化之 ----第二步： 预减库存 从缓存中减库存
 * 利用 redis 中的方法，减去库存，返回值为 减去1 之后的值
 * */
long stock = redisService.decr(GoodsKey.getGoodsStock, "" + goodsId);
/*这里判断不能小于等于，因为减去之后等于 说明还有是正常范围*/
if (stock < 0) {
    isOverMap.put(goodsId, true);//没有库存就设置 对应id 商品的map 为true
    return Result.error(CodeMsg.MIAO_SHA_NO_STOCK);
}
```

### 预减库存：

1.先将所有数据读出来，初始化到缓存中，并以 stock + goodid 的形成存入Redis,

2.在秒杀的时候，先进行预减库存检测，从redis中，利用decr 减去对应商品的库存，如果库存小于0，说明此时 库存不足，则不需要访问数据库。直接抛出异常即可

### 内存标记：

由于接口优化很多基于Redis的缓存操作，当并发很高的时候，也会给Redis服务器带来很大的负担，如果可以减少对Redis服务器的访问，也可以达到的优化的效果。

于是，可以加一个内存map,标记对应商品的库存量是否还有，在访问Redis之前，在map中拿到对应商品的库存量标记，就可以不需要访问Redis 就可以判断没有库存了。

1.生成一个map，并在初始化的时候，将所有商品的id为键，标记false 存入map中。

```java
private Map<Long, Boolean> isOverMap = new HashMap<Long, Boolean>();

/**
 * 秒杀接口优化之---   第一步:  系统初始化后就将所有商品库存放入 缓存
 */
@Override
public void afterPropertiesSet() throws Exception {
    List<GoodsVo> goods = goodsService.getGoodsList();
    if (goods == null) {
        return;
    }
    for (GoodsVo goodsVo : goods) {
        redisService.set(GoodsKey.getGoodsStock, "" + goodsVo.getId(), goodsVo.getStockCount());
        isOverMap.put(goodsVo.getId(), false);//先初始化 每个商品都是false 就是还有
    }
}
    /**再优化： 优化 库存之后的请求不访问redis 通过判断 对应 map 的值
     * */
    boolean isOver = isOverMap.get(goodsId);
    if (isOver) {
        return Result.error(CodeMsg.MIAO_SHA_NO_STOCK);
    }


    if (stock < 0) {
        isOverMap.put(goodsId, true);//没有库存就设置 对应id 商品的map 为true
```

2.在预减库存之前，从map中取标记，若标记为false,说明库存，还有，

3.预减库存，当遇到库存不足的时候，将该商品的标记置为true,表示该商品的库存不足。这样，下面的所有请求，将被拦截，无需访问redis进行预减库存。