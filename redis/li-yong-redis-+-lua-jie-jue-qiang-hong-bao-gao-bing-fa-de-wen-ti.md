### 抢红包的需求分析

抢红包的场景有点像秒杀，但是要比秒杀简单点。

因为秒杀通常要和库存相关。而抢红包则可以允许有些红包没有被抢到，因为发红包的人不会有损失，没抢完的钱再退回给发红包的人即可。

另外像小米这样的抢购也要比淘宝的要简单，也是因为像小米这样是一个公司的，如果有少量没有抢到，则下次再抢，人工修复下数据是很简单的事。而像淘宝这么多商品，要是每一个都存在着修复数据的风险，那如果出故障了则很麻烦。



淘宝的专家丁奇有个文章有写到淘宝是如何应对秒杀的：《秒杀场景下MySQL的低效–原因和改进》http://blog.nosqlfan.com/html/4209.html



基于redis的抢红包方案

下面介绍一种基于redis的抢红包方案。



把原始的红包称为大红包，拆分后的红包称为小红包。

1.小红包预先生成，插到数据库里，红包对应的用户ID是null。生成算法见另一篇blog：http://blog.csdn.net/hengyunabc/article/details/19177877

2.每个大红包对应两个redis队列，一个是未消费红包队列，另一个是已消费红包队列。开始时，把未抢的小红包全放到未消费红包队列里。

未消费红包队列里是json字符串，如{userId:'789', money:'300'}。

3.在redis中用一个map来过滤已抢到红包的用户。

4.抢红包时，先判断用户是否抢过红包，如果没有，则从未消费红包队列中取出一个小红包，再push到另一个已消费队列中，最后把用户ID放入去重的map中。

5.用一个单线程批量把已消费队列里的红包取出来，再批量update红包的用户ID到数据库里。

上面的流程是很清楚的，但是在第4步时，如果是用户快速点了两次，或者开了两个浏览器来抢红包，会不会有可能用户抢到了两个红包？

为了解决这个问题，采用了lua脚本方式，让第4步整个过程是原子性地执行。

下面是在redis上执行的Lua脚本：

```lua

-- 函数：尝试获得红包，如果成功，则返回json字符串，如果不成功，则返回空
-- 参数：红包队列名， 已消费的队列名，去重的Map名，用户ID
-- 返回值：nil 或者 json字符串，包含用户ID：userId，红包ID：id，红包金额：money

-- 如果用户已抢过红包，则返回nil
if redis.call('hexists', KEYS[3], KEYS[4]) ~= 0 then
  return nil
else
  -- 先取出一个小红包
  local hongBao = redis.call('rpop', KEYS[1]);
  if hongBao then
    local x = cjson.decode(hongBao);
    -- 加入用户ID信息
    x['userId'] = KEYS[4];
    local re = cjson.encode(x);
    -- 把用户ID放到去重的set里
    redis.call('hset', KEYS[3], KEYS[4], KEYS[4]);
    -- 把红包放到已消费队列里
    redis.call('lpush', KEYS[2], re);
    return re;
  end
end
return nil

``` 

测试结果20个线程，每秒可以抢2.5万个，足以应付绝大部分的抢红包场景。


如果是真的应付不了，拆分到几个redis集群里，或者改为批量抢红包，也足够应付。

#### 总结：
redis的抢红包方案，虽然在极端情况下（即redis挂掉）会丢失一秒的数据，但是却是一个扩展性很强，足以应付高并发的抢红包方案。


#### reference

* https://blog.csdn.net/hengyunabc/article/details/19433779 



