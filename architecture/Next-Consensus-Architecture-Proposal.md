#### Next-Consensus-Architecture-Proposal

*本文翻译于 https://github.com/hyperledger-archives/fabric/wiki/Next-Consensus-Architecture-Proposal*

讨论和留言在[这里](http://github.com/hyperledger/fabric/issues/1631)

作者：Elli Androulaki, Christian Cachin, Angelo De Caro, Konstantinos Christidis, Chet Murthy, Binh Nguyen, Alessandro Sorniotti, and Marko Vukolić

本篇记录了区块链基础设施的架构，每一个区块链中的节点角色被划分为 *peers* 角色(维护状态和总账)和 *consenters* 角色(对包含在区块链状态里的交易最终顺序达成共识)。在大部分区块链架构中(包括2016年7月份版本的 Hyperledger fabric)，这些角色是统一的(例如Hyperledger fabric 里的验证节点)。该架构同时引进了 *endorsing peers* ，一种特殊类型的 *peers*，来负责模拟执行和为交易背书(对应于 Hyperledger fabric 0.5 版本中执行和验证交易的行为)。

该架构与以前的设计( peers/consenters/endorsers 是一体的)相比有以下优势：
- **链上编码信任的灵活性(Chaincode trust flexibility)**。该架构使对链上编码的信任假设从对共识的信任假设中分离开来。换一种方式来说，共识服务可以由一系列的节点(consenters)来提供，允许部分节点失败或者有恶意行为；而对于每一个链上编码 endorsers 可以是不同的。

- **可扩展性(Scalability)**。 当负责特指的链上编码的 endorser 节点与 consenters 节点是统计上独立的时候，系统的扩展性相对于全部的功能被相同的节点实现这一方式要更好。尤其是，当不同的链上编码定义了不相交的 endorsers 集合的时候，这样就会产生链上编码之间的分离，可以允许并行的执行链上编码(endorsement)。还会将可能很耗时的链上编码的执行过程也与共识服务过程分离开来。

- **机密性(Confidentiality)**。该架构使得对它所属的交易的内容、状态更新有具体机密性要求的链上编码的部署更加便利。

- **共识模块化(Consensus modularity)**。该架构使模块化的，并且允许可插拨的共识实现。

#### 目录
1. 系统架构
2. 交易背书(endorsement)的基本工作流程
3. 背书原则
4. 区块链数据结构
5. 状态转移与检查点(checkpoint)
6. 机密性

---
#### 1. 系统架构

区块链是一个由许多相互交流的节点组成的分布式系统。区块链运行着称为链上编码的程序，持有状态和总账数据，执行交易。链上编码是关键性元素：交易是一系列在链上编码上唤醒的操作，并且只有链上编码能改变状态。交易必须被认可(背书)，并且只有背书过的交易才能被提交来影响区块链上的状态。可能会存在一个或多个特殊的链上编码来管理函数、参数，这些被称为*系统级链上编码*。

##### 1.1. 交易

交易有以下两种类型：

- *部署交易 (Deploy transactions)* 创建新的链上编码，使用一个程序作为参数。当一个部署交易成功执行后，链上编码就被安装在了区块链上。

- *唤醒交易 (Invoke transactions)* 在某个具体的链上编码中执行具体的操作。一个唤醒交易指向一个链上编码提供的函数。当该交易成功执行后，链上编码会执行相应的函数，该函数可能会修改相应的状态并且返回一个输出。

就像随后描述的那样，部署交易是唤醒交易中一种特殊的类型，因为部署交易会创建新的链上编码，对应于在某一个系统级链上编码的唤醒交易。

**Remark：** *该文献当前假设了一个交易要么创建新的链上编码，要么唤醒已经部署在区块链上的某个链上编码中的某个操作。该文献还没有描述：a)支持跨链上编码的交易，b)查询(read-only)交易的优化。*

##### 1.2. 状态

**区块链状态。** 区块链的状态 (**world state**) 有一个简单的结构，被建模为一个版本化的键值存储 (KVS)，其中，键是名称，值是任意的数据。这些键值对条目被运行在区块链上的链上编码 (应用程序) 通过 `put` 和 `get` KVS 指令操作。状态被持久化的存储，并且对状态的更新操作会被日志记录。注意，版本化的 KVS 是作为一种状态模型适配的，一个具体的实现可以使用实际的 KVSs，但是亦可以是 RDBMS (关系型数据库管理系统) 或者其它的解决方案。

更加正式地，区块链状态 `s` 被建模为映射关系 `K -> (V X N)` 的一个元素：
- `K` 是键的集合
- `V` 是值得集合
- `N` 是一个无限有序的版本号集合。一对一映射函数 `netx : N -> N`，输入 `N` 的一个元素，返回下一个版本号。

`V` 和 `N` 都包含一个特殊的元素 `\bot`，暗示 `N` 中最小的元素。初始的时候，所有的键都被映射到 `(\bot, \bot)`。对于 `s(k)=(v, ver)`，我们通过 `s(k).value` 来表示 `v`，`s(k).version` 来表示 `ver`。

KVS 操作建模为如下：
- `put(k, v)`，对于 `k \in K` 和 `v \in V`，取出区块链状态 `s` 并且将它变为 `s'`，有 `s' = (v, next(s(k).version))` 并且对于所有的 `k' != k`，`s'(k') = s(k')`。

- `get(k)` 返回 `s(k)`。

**状态分割 (State partitioning)。** 根据 KVS 中的键的名称，我们能识别出它们具体属于哪一个链上编码，在某种意义上来说，只有属于某个特定链上编码的交易才能改变属于它自己键的值。原则上，任何链上编码都可以读取属于其它链上编码的键的内容(有机密性要求的链上编码的状态不能被明文读取 - 第6章有提及)。对于*跨链上编码交易(可以修改属于两个或更多链上编码的状态)* 的支持将会在未来添加。

**总账 (Ledger)。** 区块链状态的演化被记录在 *总账* 中。总账是包含交易的区块形成的哈希链。总账中的交易是完全有序的。

区块链状态和总账会在第四章有更详细的描述。

##### 1.3. 节点

节点是区块链中交流的实体，某种意义上来说，一个节点就是一个逻辑上的函数功能，大量不同类型的节点能运行在同一台物理服务器上。关键的是节点如何以可信域 (trust domains) 被组织，并且与控制它们的逻辑实体相关联。

有3种不同类型的节点：

1. **客户端或者提交客户端 (Client or submitting-client)**：提交一个实际的唤醒交易请求。

2. **对等体 (Peer)**：提交认可交易并且维护状态和总账的副本。peers 有以下两种特殊角色：a. **submitting peer or submitter**，b. **endorsing peer or endorser**。

3. **共识服务节点 (Consensus-service-node or consenter)**：运行着实现了传输保证 (例如原子性的广播)的 交流通信服务的节点，传输保证由运行共识算法来实现。

注意客户端和共识服务节点不维护总账和区块链状态，只有 peers 维护。

###### 1.3.1. 客户端

客户端代表终端用户行为的实体。它必须与一个 peer 相连接才可以与区块链交流通信。客户端可以按照自己的选择连接 peer。客户端创建和唤醒交易。

###### 1.3.2. Peer

Peers 与共识服务节点交流通信并且维护区块链状态和总账。如此一来，Peers 从共识服务节点处接收有序的状态更新操作，并且将它们应用到本地持有的状态中。

Peers 有以下两种角色组成。

- **Submitting peer。** 一个 *submitting peer* 是 一个 peer 的一种特殊角色，它提供一个接口给客户端，如此，客户端便能连接上它来唤醒交易并且获取结果。Peer 与其它的区块链节点相联系来执行一个或多个客户端提交的交易请求。

- **Endorsing peer。** *endorsing peer* 的特殊函数功能相对于特定的链上编码发生，并且在一个交易提交之前为其背书。每一个链上编码都可能会具体一种 *背书策略 (endorsement policy)*，该策略指向一个 *endorsing peers* 的集合。该策略定义了一个有效交易背书 (典型的是 endorsers 集合的签名) 发生的必要且有效的条件。就像第二章和第三章描述的那样。部署交易这种特殊的交易类型，它的背书策略被具体为系统级链上编码的背书策略。

为了强调既不是 Submitting peer 也不是 endorsing peer 的 peer，这种 peer 有时候会被认为是 *committing peer*。

###### 1.3.3. 共识服务节点 (Consenters)

*consenters* 构成了*共识服务*，例如一种提供传输保证的交流通信 fabric。共识服务可以以不同的方式实现：从一个中心化的服务 (在开发环境和测试环境中使用 ) 到分布式的协议 (旨在解决异构网络和拜占庭错误节点的模型)。

Peers 是 consenters 的客户端，Consenters 提供一个共享的交流通信的管道给 peers，该管道提供包含交易的消息的广播服务。Peers 连接上这个管道，用来发送和接收消息。管道支持所有消息原子性的传输，也就是，消息的传输是一个完全有序的并且可靠的。换句话说，管道向所有连接它的 peers 输出相同的消息并且消息是相同逻辑顺序的。原子性的交流通信也被称为*完全有序的广播 (total-order broadcast)* 或者是分布式系统中的共识。通过管道传播的消息是要包括进区块链状态的候选交易。

**分割 (共识管道) (Patitioning (consensus channels))。** 与发布/订阅消息系统的*主题(Topics)* 相似，共识服务可以支持多管道。客户端连接到一个给定的管道上，然后发送消息和获取到达的消息。多管道可以被认为是隔离的 - 客户端不会意识到其它管道的存在，但是客户端可以连接多个管道。简单起见，在本篇文献的剩余部分，除非显示地提到，否则我们假设共识服务由单 channel/topic组成。

**共识服务程序接口 (Consensus service API)。** Peers 通过共识服务提供的程序接口连接到由共识服务提供的管道上。共识服务程序接口由两个基本的操作组成 (更为普遍是*异步事件* - *asynchronous events*)：

- `broadcast(blob)`：submitting peer 调用该接口广播任意的消息 `blob` 在管道上传播。在 BFT 上下文环境中，这也被称作为 `request(blob)`，发送请求给服务。

- `deliver(seqno, prevhash, blob)`：共识服务调用该接口，传送带有具体的非负整数序列号 (`seqno`) 和最近刚传输的消息的哈希值 (`prevhash`) 的消息 `blob` 给 peers。换句话说，它是来自共识服务的一个输出事件。`deliver()` 在一个发布/订阅的系统中被称为 `notify()` 或者在 BFT 系统中被称为 `commit()`。

  注意共识服务客户端 (peers) 只能通过 `broadcast()` 和 `deliver()` 事件来与共识服务产生交互。

**共识属性。** 共识服务 (或者是原子性广播的管道) 需提供以下的保证。它们回答以下问题：*广播过的消息发生了什么？传送过的消息之间存在什么联系？*

1. **安全性 (一致性保证) (Safety (consistency guarantees) )**：
    只要 peers 足够长时间的连接到管道 (它们可以断开连接、宕机，但是会重新启动、重新连接)，它们将会看到一个相同序列的 *delivered* `(seqno, prevhash, blob)` 消息。这意味着，输出 (`deliver() 事件`) 在所有的节点上以相同的顺序发生，并且序列号是相同的消息，其内容 `(prevhash, blob)` 也是相同的。注意这仅仅是*逻辑上的顺序*，在某一 peer 上发生的 `deliver(seqno, prevhash, blob)` 不需要跟在其它 peer 上发生的同一输出 `deliver(seqno, prevhash, blob)` 有任何真实时间上的联系。换句话说，给定一个 `seqno`，没有两个正确的 peer 会接收到不同 `prevhash` 和 `blob`。还有就是，没有 `blob` 会被传送，除非某个共识服务客户端 (peer) 实际上调用了 `broadcast(blob)`，并且，优选的是，每一个广播过的 `blob` 仅仅被传送*一次*。

    此外，`deliver()` 事件 包括前一个 `deliver()` 事件的加密哈希值 (prevhash)。当共识服务实现了原子性广播的保证后，`prevhash` 是带有序列号为 `seqno-1` 的 `deliver()`事件的加密哈希。这就建立了一条跨 `deliver()` 事件的哈希链，可以被用来帮助验证共识服务输出的完整性，会在第四章和第五章有详细讨论。在特殊的第一个 `deliver()` 事件里，`prevhash` 有一个默认值。

2. **活性 (传送保证) (Liveness (delivery guarantee) )**：共识服务的活性保证通过具体的共识服务实现来定义，依赖于网络和节点错误模型。

    原则上来讲，如果 submitting peer 不会失败，共识服务应该保证每一个连接到共识服务的节点最终会收到每一笔提交的交易。

总体上来说，共识服务保证了下列特性：

- *同意 (Agreement)。* 对于发生在正确 peer 上的两个 `deliver()` 事件：`deliver(seqno, prevhash0, blob0)` 与 `deliver(seqno, prevhash1, blob1)`，有 `prevhash0==prevhash1` 和 `blob0==blob1`；

- *哈希链完整性 (Hashchain integrity)。* 对于发生在正确 peer 上的两个 `deliver()` 事件：`deliver(seqno-1, prevhash0, blob0)` 与 `deliver(seqno, prevhash, blob)`，有 `prevhash = HASH(seqno-1, prevhash0, blob0)`。

- *不会跳过 (No skipping)。* 如果共识服务在一个 peer p 上输出了 `deliver(seqno, prevhash, blob)`，那么 p 已经接受过 `deliver(seqno-1, prevhash0, blob0)` 事件。

- *不会创建 (No creation)。* 发生在节点上的任何 `deliver(seqno, prevhash, blob)` 事件一定是由某个节点 (可能是不同) 上的 `broadcast(blob)` 事件产生的。

- *不会重复 (No duplication) (可选的，强烈需要的)。* 对于任何两个事件 `broadcast(blob)` 和 `broadcast(blob')`，当两个事件 `deliver(seqno0, prevhash0, blob)` 和 `deliver(seqno1, prevhash1, blob')` 发生在正确的 peer 的时候，并且 `blob==blob'`，那么有 `seqno0==seqno1` 和 `prevhash0==prevhash1`。
(译者：跨 `deliver()` 事件的哈希链中不会出现两个具有相同 `blob` 的事件)

- *活性 (Liveness)。* 如果一个正确的 peer 唤醒了 `broadcast(blob)` 事件，那么每一个正确的 peer 最终都会发行一个 `deliver(*, *, blob)` 事件，* 代表任意值。

#### 2. 交易背书的基本流程

接下来我们将给出交易请求工作流程的大概轮廓。

**Remark：** *注意下列的协议没有假设交易是决定性的，例如，它允许非确定性的交易。*

##### 2.1. 客户端创建一笔交易，然后按照它的选择将交易发送给一个 submitting peer

为了唤醒一笔交易，客户端将下列消息发送给一个 submitting peer `spID
`。

`<SUBMIT, tx, retryFlag>`：

- `tx=<clientID, chaincodeID, txPayload, clientSig>`:

  - `clientID` 是 submitting client 的  ID，
  - `chaincodeID` 指向交易属于的 chaincode，
  - `txPayload` 是包含提交的交易内容的有效载荷，
  - `clientSig` 是客户端对 `tx` 其它部分的签名。


- `retryFlag` 是布尔变量，告诉 submitting peer 当交易失败的时候是否再次进行提交。

`txPayload` 的细节在唤醒交易和部署交易之间会有所不同，对于**唤醒交易**来说，`txPayload` 由一个字段组成

- `invocation = <operation, metadata>`，

  - `operation` 意味着链上编码的函数操作和参数，
  - `metadata` 意味着与唤醒相关的属性。

对于**部署交易**，`txPayload` 由两个字段组成：

- `chainCode = <source, metadata>`，

  - `source` 代表着链上编码的源代码，
  - `metadata` 代表着与链上编码和应用程序相关的属性。


- `policies` 包含于链上编码相关的策略，可以被所有的 peers 所访问，例如，背书策略

**TODO：** 决定是否显示包含客户端本地/逻辑时钟 (时间戳)。

##### 2.2. submitting peer 准备一笔交易，然后发送给 endorsers 来获得一个交易的背书

从一个客户端接收到 `<SUBMIT, tx, retryFlag>` 消息之后，submitting peer 首先验证客户端的签名 `clientSig`，然后准备一笔交易。这包括了 submitting peer 通过唤醒交易中指向的链上编码的操作和本地持有的状态先暂时执行一笔交易 (`txPayload`)。

作为执行交易后的结果，submitting peer 计算出一个*状态更新 (`stateUpdate`)* 和 *版本依赖 (`verDep`)*，在数据库语言中也被称为*MVCC+postimage info*。

回想到状态有键值对组成。所有的键值对条目是版本化的，也就是，每一个条目都包含有序的版本信息，当
