# 公钥私钥

## 对称加密

## 1976年以前，所有的加密方法都是同一种模式：

> （1）甲方选择某一种加密规则，对信息进行加密；
>
> （2）乙方使用同一种规则，对信息进行解密。

由于加密和解密使用同样规则（简称"密钥"），这被称为["对称加密算法"](http://zh.wikipedia.org/zh-cn/对等加密)（Symmetric-key algorithm）。

这种加密模式有一个最大弱点：甲方必须把加密规则告诉乙方，否则无法解密。保存和传递密钥，就成了最头疼的问题。

## 非对称加密

1976年，两位美国计算机学家Whitfield Diffie 和 Martin Hellman，提出了一种崭新构思，可以在不直接传递密钥的情况下，完成解密。这被称为["Diffie-Hellman密钥交换算法"](http://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)。这个算法启发了其他科学家。人们认识到，加密和解密可以使用不同的规则，只要这两种规则之间存在某种对应关系即可，这样就避免了直接传递密钥。

这种新的加密模式被称为"非对称加密算法"。

> （1）乙方生成两把密钥（公钥和私钥）。公钥是公开的，任何人都可以获得，私钥则是保密的。
>
> （2）甲方获取乙方的公钥，然后用它对信息加密。
>
> （3）乙方得到加密后的信息，用私钥解密。

如果公钥加密的信息只有私钥解得开，那么只要私钥不泄漏，通信就是安全的。

### 公钥私钥

公钥（Public Key）与[私钥](https://baike.baidu.com/item/私钥)（Private Key）是通过一种算法得到的一个[密钥](https://baike.baidu.com/item/密钥)对（即一个公钥和一个私钥），公钥是密钥对中公开的部分，私钥则是非公开的部分。公钥通常用于加密会话[密钥](https://baike.baidu.com/item/密钥)、验证[数字签名](https://baike.baidu.com/item/数字签名)，或加密可以用相应的[私钥](https://baike.baidu.com/item/私钥)解密的数据。通过这种算法得到的[密钥](https://baike.baidu.com/item/密钥)对能保证在世界范围内是唯一的。使用这个[密钥](https://baike.baidu.com/item/密钥)对的时候，如果用其中一个[密钥加密](https://baike.baidu.com/item/密钥加密)一段数据，必须用另一个密钥解密。比如用[公钥加密](https://baike.baidu.com/item/公钥加密)数据就必须用[私钥](https://baike.baidu.com/item/私钥)解密，如果用私钥加密也必须用公钥解密，否则解密将不会成功。

## 参考

* [https://baike.baidu.com/item/公钥/6447788?fr=aladdin](https://baike.baidu.com/item/公钥/6447788?fr=aladdin)
* [http://www.ruanyifeng.com/blog/2013/06/rsa\_algorithm\_part\_one.html](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)

