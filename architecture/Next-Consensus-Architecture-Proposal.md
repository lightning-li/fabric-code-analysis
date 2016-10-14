#### Next-Consensus-Architecture-Proposal

*本文翻译于 https://github.com/hyperledger-archives/fabric/wiki/Next-Consensus-Architecture-Proposal*

讨论和留言在[这里](http://github.com/hyperledger/fabric/issues/1631)

作者：Elli Androulaki, Christian Cachin, Angelo De Caro, Konstantinos Christidis, Chet Murthy, Binh Nguyen, Alessandro Sorniotti, and Marko Vukolić

本篇记录了区块链基础设施的架构，每一个区块链中的节点角色被划分为 *peers* 角色(维护状态和总账)和 *consenters* 角色(对包含在区块链状态里的交易最终顺序达成共识)。在大部分区块链架构中(包括2016年7月份版本的 Hyperledger fabric)，这些角色是统一的(例如Hyperledger fabric 里的验证节点)。该架构同时引进了 *endorsing peers* ，一种特殊类型的 *peers*，来负责模拟执行和为交易背书(对应于 Hyperledger fabric 0.5 版本中执行和验证交易的行为)。
