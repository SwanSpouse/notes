### 拜占庭将军问题\(byzantine-generals-problems\)

拜占庭将军问题是Leslie Lamport（2013年的图灵讲得住）用来为描述分布式系统一致性问题（Distributed Consensus）在论文中抽象出来一个著名的例子。

拜占庭帝国想要进攻一个强大的敌人，为此派出了10支军队去包围这个敌人。这个敌人虽不比拜占庭帝国，但也足以抵御5支常规拜占庭军队的同时袭击。这10支军队在分开的包围状态下同时攻击。他们任一支军队单独进攻都毫无胜算，除非有至少6支军队（一半以上）同时袭击才能攻下敌国。他们分散在敌国的四周，依靠通信兵骑马相互通信来协商进攻意向及进攻时间。困扰这些将军的问题是，他们不确定他们中是否有叛徒，叛徒可能擅自变更进攻意向或者进攻时间。在这种状态下，拜占庭将军们才能保证有多于6支军队在同一时间一起发起进攻，从而赢取战斗？

拜占庭将军问题中并不去考虑通信兵是否会被截获或无法传达信息等问题，即消息传递的信道绝无问题。**Lamport已经证明了在消息可能丢失的不可靠信道上试图通过消息传递的方式达到一致性是不可能的。**所以，在研究拜占庭将军问题的时候，已经假定了信道是没有问题的（消息可以被篡改、前不一致但是能够保证不丢失）。

#### **问题分析**

1. 先看在没有叛徒情况下，假如一个将军A提一个进攻提议（如：明日下午1点进攻，你愿意加入吗？）由通信兵通信分别告诉其他的将军，如果幸运中的幸运，他收到了其他6位将军以上的同意，发起进攻。如果不幸，其他的将军也在此时发出不同的进攻提议（如：明日下午2点、3点进攻，你愿意加入吗？），由于时间上的差异，不同的将军收到（并认可）的进攻提议可能是不一样的，这是可能出现A提议有3个支持者，B提议有4个支持者，C提议有2个支持者等等。
2. 再加一点复杂性，在有叛徒情况下，一个叛徒会向不同的将军发出不同的进攻提议（通知A明日下午1点进攻， 通知B明日下午2点进攻等等），一个叛徒也会可能同意多个进攻提议（即同意下午1点进攻又同意下午2点进攻）。

叛徒发送前后不一致的进攻提议，被称为“**拜占庭错误**”，而能够处理拜占庭错误的这种容错性称为「**Byzantine fault tolerance**」，简称为BFT。

消息可能丢失，可靠信道，可以利用Paxos算法来达成最终一致性。

消息不可能的丢失，不可靠信道（消息可以被篡改、前后不一致），可以利用“工作量证明” 来解决这个问题。

### reference

* [https://baijiahao.baidu.com/s?id=1591728006111720793픴=spider&for=pc](https://baijiahao.baidu.com/s?id=1591728006111720793&wfr=spider&for=pc)


