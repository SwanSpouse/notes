# 比特币网络节点的构成

比特币网络是一种**点对点的数字现金系统（P2P）**，网络节点中每台机器都彼此对等。P2P网络不存在任何服务端、层级关系或者中心化服务。

## 节点类型与分工

## ![](../.gitbook/assets/比特币网络节点分工.png)

一个全功能节点包含上述4个模块【钱包**W**allet】【矿工**M**iner】【完整区块链full **B**lock-chain database】【网络路由节点**N**etwork routing】

* 【网络路由节点】使得节点具有**参与验证并传播交易与区块信息，发现监听并维持点对点的链接**的能力
* 【完整区块链】具有此模块的节点被称为：全节点。它能够**独自自主的校验所有交易**，不需要任何其他信息。
* 【钱包】比特币的所有权是通过数字密钥、比特币地址和数字签名来确定的，数字密钥实际上并不是存储在网络中，而是由用户生成并存储在一个文件或简单的数据库中，称为钱包。有些节点仅仅保留**区块链的一部分**，通过一种”简易支付验证“（SPV Simplified Payment Verification）的方法来完成交易。
* 【矿工】挖矿节点以相互竞争的方式创造新的区块。有一些挖矿节点也是全节点，可以独立挖矿；还有一些参与矿池挖矿的节点是轻量级节点，**必须依赖矿池服务器维护全节点进行工作。**

拥有全部四个模块被称之为**核心客户端（Bitcoin Core）**，除了这些主要节点类型外，还有一些服务器及节点运行其他协议，如特殊矿池挖矿协议、轻量级客户端访问协议。

下表为扩展比特币网络的不同节点类型  
![](../.gitbook/assets/比特币网络的不同节点.png)

## 扩展比特币网络

要在全世界的网络中完成整个的交易，下图描述了一个扩展比特币网络，它包含了多重类型的节点、网关服务器、边缘路由器、钱包客户端以及它们互相连接所需要的各类协议，比特币互相连接的接口一般使用`8333端口`

## ![](../.gitbook/assets/扩展比特币网络.png)

## 如何控制区块产生速度恒定

### 难度系数

**工作量证明：**是寻找一个特殊数字使得SHA256函数的输出字符串的前`n`位是零

对于每一种不同的加密货币来说，都有**一个值**需要在建立货币的时候时候被定义，即**每一个新区块在当前全网算力的条件被发现的【平均时间】**，这也是难度系数的由来。

> 比特币10分钟；以太坊15秒；瑞波币3.5秒；莱特币2.5分钟

抛开代码算法层面来说，实现方法就是通过找前`n`位是0的方法。从概率角度来说，n值越大，意味找到这个这个数的解范围越小。

随着需求`0`的数目一个一个增加，**需要的计算时间将会程指数增长**。

### 难度调整方式

难度的调整实在每个完整节点中自动发生的。如果网络发现区块产生速率比10分钟要快时会增加难度。如果发现比10分钟慢时则降低难度。

例如比特币中的是这样定义的：每2016个区块后计算生成它们花费的时长，比上20160（14天）调整一次。有人可能会问，如果在这十四天内计算能力暴涨怎么办，其实这个10分钟的区块新建间隔的规定也只是一个估计要求，真实情况下，**这个时间会偏离10分钟这个设定值很多，但是这种偏差并不会对整个区块链的运行产生影响**

但是有人会问，这个过程是靠脚本（代码）来实现的，还是自己手动调整的呢？答案是，本地挖矿节点根据自己看到的链上信息自己调整。

问题又来了，为何我不自己降低难度，让自己更加容易新建区块呢？其实，因为链上所有节点确认新的区块（只有确认了你才能得到回报）是按照最长链并且计算难度最大来判断的，你如果用很小的难度新加的区块，是肯定跑不赢全网的其他矿工的。

## 比特币的交易处理能力

### 区块容量

比特币区块链从被**第一次部署**时，或者说源代码中已经规定，**区块容量是1M**。最初设计成1M的原因一方面，**防止DOS攻击**。另一方面，当年中本聪在创建区块链的时候的容量是32M，但是他通过一个说明为”Clear up“这样毫不起眼的Commit把区块容量改成了1M，为**防止区块链体积增长过快**，为区块容量这个问题添加了些神秘色彩。好吧，我承认，中本聪就已经非常具有神秘色彩了，是在神秘色彩上添上了些故事。

截止2018年4月1号，完整区块链大小已经有约**151GB**的大小了

上表中，可以观察到，**1M的容量**意味着比特币最大的处理交易数量在**约2400**（486882区块1034.39的大小很接近了），从代码及技术文档来看，一个区块的最大处理交易数量在2700笔，意味着在一定程度上区块利用率可以超过100%。

## 比特币的一笔交易过程中到底发生了什么

我们可以确认的是，每一笔都将记录在大账本中，那么我们需要研究的内容，就是区块中交易内容内的具体数据结构

### 交易结构

每一个交易块包含的内容如下表所示

| 大小 | 字段 | 描述 |
| :--- | :--- | :--- |
| 4 bytes | 版本 | 明确这笔交易参照的规则 |
| 1 - 9 bytes | 输入数量 | 输入值的数量 |
| 不定 | 输入 | **一个或多个交易输入** |
| 1 - 9 bytes | 输出数量 | 输出值的数量 |
| 不定 | 输出 | **一个或多个交易输出** |
| 4 bytes | 时钟时间 | UNIX时间戳或区块号 |

最后这个值是解锁时间，定义了能被加到区块链里的最早交易时间。大多是时候设为0，表示**立马执行**。

一笔比特币交易是一个含有**输入值**和**输出值**的**数据结构**。该数据结构**包含了将一笔资金从初始点（输入值）转移至目标地址（输出值）的代码信息**。比特币交易的输入值和输出值与账号或者身份信息无关。可以把它理解为一种**被特定秘密信息锁定的一定数量的比特币**。只有拥**有者或者知道这个秘密信息的人**可以解锁。

### 交易的输入和输出

比特币交易的基本单位是**未经使用的一个交易输出**，简称UTXO（unspent transaction outputs）  
可以把UTXO类比为我们使用的**人民币1，5，10，20，50，100的面值**，对于UTXO来说，它的面值可以是一”聪“的任意倍数（1BTC等于一亿聪）**但是**这个有着任意面值的”人民币“不能随意打开，还被加上一道类似**红包支付口令的密码**，只有拥有这个密码的人才可以使用这个UTXO，UTXO包含，币值+一段代码（锁，只有有钥匙的人才能打开）。

被交易消耗的UTXO被称为交易输入，由交易创建的UTXO被称为交易输出。

### 交易输出

**不同面值的UTXO是由交易输出来提供的**。你可以想象你需要购买一个3.1BTC的物品，你并不能从你的钱包中找到几个UTXO来得到3.1BTC，但是你刚好拥有一个4BTC的UTXO，你使用这个UTXO作为付款，那么你需要自己**手动构建一个0.9的UTXO返还给你自己**。

一个交易输出包含两个部分

* 一定量的比特币。被命名为“聪”（satoshis）
* 一个**锁定脚本**。给这个UTXO上锁，保证只有**收款人地址**的私钥才可以打开

### 交易输入

每个交易输入是在构造的一笔交易（使用UTXO），比如你需要支付0.015BTC，钱包会寻找一个0.01BTC和0.005BTC的UTXO来组成这一笔交易。交易输入中还会**包含一个解锁脚本**，这是一个签名，可以类比成支付宝红包密码的口令。

### 交易费

交易费 = 求和（所有输入） - 求和（所有输出）

这里有一个比较有意思的地方，就是因为找零的输出UTXO是交易的发起这自己构建的，如果很不幸，你忘记了自己构建找零的UTXO，那么这些多余的BTC就会变成矿工的劳务费。

例如，我需要和小明进行交易，需要购买一个商品，花费`0.8BTC`，为了确保这笔交易能被更快的处理（添加到大账本上），我要在其中添加一笔交易费，假设`0.01BTC`（忽略人傻钱多），那就意味着这笔交易会需要我从钱包中找到几个UTXO能组成`0.81BTC`。但如果我的钱包内找不出这样的UTXO，只有一个`1BTC`的UTXO可用，那么我就需要构建一个`0.19BTC`的UTXO作为找零回到自己的钱包

**交易费只和交易字段使用的字节大小有关，与参与交易的比特币币值无关**。UTXO是有尺寸的，比如某人想支付一笔很大的BTC交易，但是他的钱包中有很多小尺寸的UTXO，如果加入了很多个UTXO，就意味着他的交易会变复杂，存储空间需求多。当然，很多钱包会提供这方面的优化功能，保证你的交易使用的字段大小最优化。

### 解锁和锁定脚本

在实际实现的时候，这个“支付宝红包口令”被称为脚本，是一种基于逆波兰表示法的基于堆栈的执行语言。具体细节感兴趣的读者可以去[比特币的Github](https://github.com/bitcoin/bitcoin)研究代码。关于脚本有很多细节上的定义和实现方法，这里限于篇幅不展开描述了

### 矿工费和优先级

我们知道，**每一笔交易都是广播到区块链上，由矿工决定是不是加入到新区块上的**。那么这里就会涉及到一个问题，谁的交易的优先级更高，是先来后到？还是谁给前多谁就能加入到新区块中？

在[区块容量](https://charlesliuyx.github.io/2017/09/24/一文弄懂区块链-以比特币为例/#区块容量)一节中，有一张图表直观的展示了现在网络中一笔交易的等待时间，其中最长的，也就是30分钟，如果你不是一个超级急性子，很多时候还是可以接受的（毕竟跨国转账1-2个工作日）

优先级 = 输入值块龄 \* 输出值块龄 / 交易总长度

一个交易想成为**“较高优先级”**，需满足条件：**优先值大于57600000**，等价于1个BTC，年龄为1天，交易的大小为250字节

区块中前50KB的字节是保留给“较高优先级”的，其实这一机制也保证了一笔交易不会等待时间无现长。但是我们要知道，内存池（存放待处理交易的位置）中的交易，如果在没有处理后消失，所以钱包必须拥有不断重新广播未被处理交易的功能。

### 创币交易 - Coinbase

每一个**新建立的区块**，都会有新的比特币作为奖励被产生，这个交易是一个特殊交易，被称为**创币交易（Coinbase奖励）**

创币交易中不存在解锁脚本（也叫ScriptSig字段），被Coinbase的数据取代，长度最小2字节，最大100字节，除了开始的几个自己以外，矿工可以任意使用Coinbase的其他部分。比如创世区块中，Coinbase的输入中的字段是：The Times 03/Jan/2009 Chancellor on brink of second bailout for banks，是泰晤士日报当天的头版文章标题：财政大臣将再次对银行施以援手。

### Merkle树

**每个区块中的所有交易，都是用Merkle树来表示的**。换句话说，交易的存储数据结构是，Merkle树

Merkle树是一种**哈希二叉树**，它可以用来进行**快速查找和检验大规模数据完整性**。对于比特币网络来说，使用Merkle树来存储交易信息的目的是为了**高效的查找和校验某笔交易的信息是否存在。**

当N个数据元素经过加密（使用两次SHA256算法，也称double-SHA256），**至多**计算 2\*logN 次就能**检查出任意某元素是否在树中。**

### 构造Merkle树

假设我们有A B C D四笔交易字段，首先需要把这四个数据Hash化。然后把这些哈细化的**数据通过串联相邻叶子节点**的哈希值然后哈希化。基本过程如下图所示  
![](../.gitbook/assets/Merkle树.png)

叶子节点必须是偶数（平衡树），如果遇到奇数的情况，把最后一个节点自身复制一个，凑偶。

### Merkle树的效率

下表显示了**证明区块中存在某笔交易**所需转化为Merkle路径的数据量

| 交易数量 | 区块的近似大小 | 路径大小（哈希数量） | 路径大小（字节） |
| :--- | :--- | :--- | :--- |
| 16 | 4KB | 4个哈希 | 128 bytes |
| 512 | 128KB | 9个哈希 | 288 bytes |
| 2048 | 512KB | 11个哈希 | 352 bytes |
| 65535 | 16MB | 16个哈希 | 512 bytes |

可以发现，即使区块容量达到16MB规模，为证明交易存在的Merkle路径长度增长也极其缓慢（幂指数增长取对数变为线性增长）

## 参考

* [https://charlesliuyx.github.io/2017/09/24/一文弄懂区块链-以比特币为例/](https://charlesliuyx.github.io/2017/09/24/一文弄懂区块链-以比特币为例/)

