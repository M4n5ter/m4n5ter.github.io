# Blockchain

这块内容是区块链，本页面会介绍区块链基础知识。内容大多翻译自 substrate fundamentals 。

## 区块链基础知识

区块链是一种去中心化的账本，它以一系列块( blocks )的形式记录信息。块中包含的信息是一组有序的指令，可能会导致状态发生变化。

在一个区块链网络中，计算机个体 —— 被称为节点 —— 通过去中心化点对点网络（`P2P`）在彼此间交流。 没有可以掌控这个网络的中央机构，通常，参与区块生产的每个节点都存储构成规范链的区块的副本。

在大多数情况下，用户通过提交可能导致状态变化的请求与区块链进行交互，例如，更改文件所有者或将资金从一个帐户转移到另一个帐户的请求。这些交易请求被传播到网络上的其他节点，并由区块作者（一般称为矿工）组装成一个区块。为了确保链上数据的安全性和链的持续进展，节点使用某种形式的共识机制来商定每个区块中数据的状态以及执行交易的顺序。

## 何为区块链节点

在 high level 上，所有区块链节点都需要以下核心的组成部分：

- 数据存储 —— 用于记录作为交易结果的状态变化。

- 用于节点间去中心化沟通的点对点网络（P2P，一般使用 `libp2p`）

- 共识机制，用来防止恶意活动破坏链，并确保链的持续进展。

- 排序和处理 传入交易的逻辑。

- 用于为区块生成哈希摘要（hash digests）以及签名和验证与交易关联的签名的密码学。

由于构建区块链所需的核心组件所涉及的复杂性，大多数区块链项目从现有区块链代码库 fork 一个副本，以便开发人员可以修改现有代码以添加新功能，而不是从头开始编写所有内容。例如，Bitcoin 仓库被 fork 以创建 Litecoin、ZCash、Namecoin 和 Bitcoin Cash。类似地，Ethereum 被 fork 以创建 Quorum、POA Network、KodakCoin 和 Musicoin。

然而，大多数区块链平台的设计并不允许修改或定制。因此，通过 fork 来构建新的区块链有严重的限制，包括原来的区块链代码中固有的限制，比如可扩展性。

我们将会先了解大多数区块链共享的一些共同属性。

## 状态的转换和冲突

区块链本质上是一个状态机。在任意时间点，区块链都有一个当前的内部状态。当入站交易被执行时，它们会导致状态的变化，因此区块链必须从其当前状态转换到新的状态。然而，可能有多个有效的要转换的状态，它们会导致不同的未来状态，区块链必须选择一个可以商定的状态来转换。要在转换后就状态而言达成一致，区块链中的所有操作都必须是确定性的。为了使链能够成功进展下去，大多数节点必须对所有状态转换达成一致，包括：

- 链的初始状态，称为创世状态或创世块。

- 记录在每个块中的已执行的交易导致的一系列状态转换。

- 区块要包含在链中的最终状态。

在中心化的网络中，中央机构可以在互斥的状态转换之间进行选择。例如，被配置成主要机构的服务器可能会按照它看到的顺序来记录状态转换的变化，或者在发生冲突时使用加权过程在竞争的替代方案之间进行选择。在去中心化的网络中，节点们可能以不同的顺序看到交易，因此它们必须使用更复杂的方法来选择交易并在存在冲突的状态转换之间进行选择。

区块链用来将交易打包成区块并选择哪个节点（矿工挖到矿）可以向链提交区块的方法，被称为区块链的共识模型或共识算法。最常用的共识模型称为工作量证明（`POW`，proof-of-work）共识模型。有了工作量证明共识模型，首先完成计算问题的节点（挖到矿的矿工）有权向链提交区块。

为了使区块链具有容错能力并提供一致的状态视图，即使一些节点受到恶意行为者或网络中断的破坏，但是一些共识模型需要至少三分之二的节点在任何时候都是同意状态。这种方式确保网络是可以容错的，并且可以承受一些网络参与者的不良行为，无论行为是故意的还是意外的。

## 区块链经济学

所有区块链都需要资源 —— 处理器、内存、存储和网络带宽 —— 来执行操作。参与网络的计算机 —— 产生区块的节点（矿工） —— 向区块链用户提供这些资源。这些节点创建了一个分布式、去中心化的网络，满足参与者社区的需求。
为了支持一个社区并使区块链可持续发展，大多数区块链要求用户以交易费的形式来为他们使用的网络资源付费。支付交易费需要用户身份与持有某种类型资产的账户相关联。区块链通常使用代币来表示账户中资产的价值，网络参与者通过交易所，在链外购买代币。然后，网络参与者可以存入代币，使他们能够支付交易费用。

## 区块链的治理

一些区块链允许网络参与者提交能够影响网络运营或区块链社区的提案，并对此进行投票。通过提交提案和投票 —— 公民投票 —— 区块链社区可以决定区块链在民主的情况下如何发展。然而，链上治理相对较少，要参与其中，区块链可能需要用户在账户中持有大量代币，或者被选为其他用户的代表。

## 在区块链上运行的应用程序

在区块链上运行的应用程序 —— 通常被称为去中心化应用程序或 `dApp` —— 通常是使用前端框架编写的网络应用程序，但后端为智能合约，用于改变区块链状态。

智能合约是在区块链上运行并在特定条件下代表用户执行交易的程序。开发人员可以编写智能合约，以确保以开发者编写的逻辑执行的交易的结果被记录下来，并且不能被篡改。然而，仅凭智能合约，开发人员无法访问区块链的一些底层功能 —— 如共识、存储或交易层 —— 相反，他们必须遵守链的既定规则和限制。智能合约开发人员通常接受这些限制作为权衡，从而加快开发时间，减少核心设计决策。