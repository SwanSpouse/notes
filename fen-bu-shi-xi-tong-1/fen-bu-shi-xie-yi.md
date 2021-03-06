# 分布式协议

## Paxos

Paxos协议是一个解决分布式系统中，多个节点之间就某个值达成一致的通信协议。它能处理在少数节点离线的情况下，剩余的多数节点仍能够达成一致。

### Paxos协议简介

Paxos协议是一个两阶段协议，分为Prepare阶段和Accept阶段。

该协议涉及到2个参与者角色，Proposer和Accceptor，Proposer是提议提案的服务器，而Acceptor是批准提案的服务器。二者在物理上可以是同一台机器。

* Prepare阶段1： Proposer 发送 Prepare
  * Proposer生成全局唯一且递增的提案ID，向Paxos集群的所有机器发送请求，这里无须携带提案内容，只携带提案ID即可。ID称作Pn
* Prepare阶段2： Acceptor 应答 Propare
  * Acceptor收到提案请求后，做出以下约定
    * 不再应答 &lt;=Pn 的Prepare请求；
    * 对于 &lt;Pn的Accept请求亦不做处理。
  * Acceptor做的处理包括：
    * 应答前要在本地持久化当前提案的ID；
    * 如果现在请求的提案ID Pn大于此前存放的proposalID，做以下逻辑:
      * If Pn &gt; proposalID then proposalID = Pn
    * 如果该Acceptor Accept过的提案，则返回提案中proposalID最大的那个提案的内容，否则返回空值。
* Accpet 阶段: Proposer 发送Accept
  * Proposer收集到多数派的应答Prepare阶段的返回值后，从中选择ProposalID最大的提案内容，作为要发起Accept的提案，如果这个提案为空值，

    则可以自己随意决定提案的内容。然后携带上当前的proposalID，向Paxos集群所有机器发送Accept请求。
* Accept 阶段：Acceptor 应答Accept
  * Acceptor 收到Accept请求后，检查不违背自己之前做出约定的情况下，持久化前ProposalID和提案的内容。最后Proposal收集到多数派应答的Accept回复后，形成决议。

## 3PC

3PC分为3次交互，

* 第一阶段，投票，事务协调器询问参与者是否能提交，得到肯定答复后进行第二阶段。
* 第二阶段是预提交，都确认预提交成功后，进行第三阶段。
* 第三阶段是真是的提交，成功则完成事务。

3PC在2PC的基础之上增加了一次交互，preCommit，只要预提交成功，则一定要保证doCommit成功。这是协议的基本思想，一般通过重试补偿策略保证doCommit提交成功。

## Raft协议

在Raft中，任何时刻一个服务器可以扮演一下任一角色

* 领导者 处理所有客户端交互、日志复制等动作，一般一次只有一个领导者。
* 选民，完全被动的角色，这样的服务器等待被通知投票。
* 候选人，候选人就是在选举过程中提名自己的实体，一旦选举成功，则成为领导者。

Raft算法分为两个阶段，1.选举过程，2. 在选举出来的领导人带领进行正常的操作，比如日志复制等。

* 任何一个服务器都可以成为一个候选者，它向其他服务器发出要求选举自己的请求，
* 其他服务器都回复同意指令。只要达半数以上，候选人还是可以成为领导者的。
* 成为领导者后，它可以向选民们发出执行具体操作动作的指令。
* 一旦Leader宕机，那么Follower中会有一个成为候选者，发出投票选举。

