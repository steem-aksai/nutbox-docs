# Crosschain Bridge

Nutbox 需要处理多个区块链网络中资产的相互转移，由于不同的区块链网络区块共识算法以及区块验证逻辑都不同，Nutbox 需要分别去验证每一个区块链网络中的交易的合法性以实现跨链资产抵押。Nutbox 的跨链桥服务专门用于不同区块链网络的数据验证以及作为跨链资产转移的基础设施。

## What Is Crosschain Bridge

跨链桥是一种沟通两个不同区块链网络的一种技术手段。通过跨链桥，用户可以在不同的区块链网络中实现资产转移。例如，用户将 1BTC 兑换为以太坊上的 ERC20 资产 xBTC（假如 1BTC = 1xBTC），用户的比特币账户将减少 1BTC，与此同时以太坊网络中该用户账户下的 ERC20 资产将增加 1xBTC。用户可以在未来某个时候将 ERC20 资产兑换成 BTC。跨链桥技术根据应用场景的不同，有多种实现方式，每种方式的技术实现细节和去中心化程度也不一样。Nutbox 采用基于 LTBSV（如下文）实现的跨链桥技术实现跨链资产的转移和区块的数据的验证。

## Light Trustless Blokchain State Verification(LTBSV)

 ![Image text](http://wherein.mobi/wp-content/uploads/2021/03/math4.2.png)

LTBSV 不同于区块链网络里的全节点，不需要存储整个区块数据，也不需要对区块里的每一笔交易进行验证。它和轻节点唯一不同的是不需要具备钱包功能即不需要存储任何用户的转账信息，也不需要维护像比特币这种继续 UTXO 账本模型的账户余额。LTBSV 包含以下组成部分：

### Block Fetcher

从任何第三方获取最新的区块数据，无需信任第三方数据源。通常只需要提供给 Block Fetcher 对应区块链网络的全节点链接，Block Fetcher 将按照设置好的频率轮询最新的区块。如果由于某种原因 Block Fetcher 暂停后重启，依旧会从上次验证的最新区块开始根究区块高度获取截止到网络最新区块的所有区块数据并挨个验证每一个区块的合法性。

### Block Validator

Block Validator 负责区块数据的验证。由于 LTBSV 不关心区块里每一笔交易，某种程度上它只关心代理合约相关的交易，因此 LTBSV 结合交易 Merkle root hash，只对区块头进行验证。在轻节点中，需要存储所有区块头，即当节点启动后需要同步之前的所有区块头，即使只是同步区块头，对大多数现有的区块链网络来说，也需要耗费几个小时的时间。LTBSV 采用 Check Point 机制，它最早在比特币系统中引入，比特币钱包 electrum 即采用了 Check Point 机制[11]，即开发者事先硬编码已经被验证的区块头，在比特币系统中，由于难度值每 2016 个区块调整一次，因此，只需要编码区块高度是2016 * n{ n = 1,2,3,,,n } 的区块 hash 和对应的难度值 [12]。采用 Check Point 机制后，LTBSV 启动后几分钟时间便可完成最新区块的验证。

### Deposit Proof

 ![Image text](http://wherein.mobi/wp-content/uploads/2021/03/math4.2.3.png)

用户随后提交 Deposit proof 给TEG模块验证，TEG模块将会解析出用户抵押交易的区块高度，并结合当前高度进行比对，如果当前高度没有在设定的确认区块高度之后，则交易失败。验证通过后将会分发等额 tToken 到用户的 Nutbox 账户中。