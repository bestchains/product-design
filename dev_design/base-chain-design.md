## 基础链设计

### 意图

1. 我们（运营方）部署并维护一条链。第 3 方可以部署其中一些节点来监听链上的事件。前期我们需要保证链上治理权。
2. 每个平台上的用户和链上一个账户一一对应。由用户掌握私钥。
3. 在我们平台上的操作尽量能反映到链上。（比如创建[提案 proposal](proposal-vote-design.md#proposal-crd)，参与[投票 vote](proposal-vote-design.md#vote-crd)）初期可以通过在 [fabric-operator](https://github.com/bestchains/fabric-operator) 中写代码，后期我们会开放链操作的 API，用户可以直接操作链。

### 优势

1. 防篡改，可信。用户的私链使用了 [hyperledger fabric](https://github.com/hyperledger/fabric) 来保证安全和隐私，用户在我们平台的操作也需要安全保证，我们可以使用区块链技术保证用户的操作真实反映了用户的意图，运营方和第三方无法篡改。
2. 透明。用户可以通过我们 Web 界面或者自己监控链上事件看到其他用户的反映。

### 劣势

1. 透明即是优势也是劣势，上链后的隐私性较差，随着链上事件的广播，任何用户的任何操作可以被任何可以查看链事件的用户看到。
2. 与操作 kubernetes 中的资源相比，链上操作更加繁琐，意味着对我们来说，代码量和调试时间都会增加。

### 方案选型

#### 1. 直接使用 [hyperledger fabric](https://github.com/hyperledger/fabric) 链

##### 优势

* 我们比较熟悉 [hyperledger fabric](https://github.com/hyperledger/fabric)

##### 劣势

* [hyperledger fabric](https://github.com/hyperledger/fabric) 使用 [chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/smartcontract/smartcontract.html)，没有选用更广泛的 [evm](https://ethereum.org/zh/developers/docs/evm/)，且将 [evm](https://ethereum.org/zh/developers/docs/evm/) 带入 [chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/smartcontract/smartcontract.html) 的 [fabric-chaincode-evm](https://github.com/hyperledger-archives/fabric-chaincode-evm) 项目因为社区成员不感兴趣而归档。

#### 2. 开发新链 （ ✅ 目前选择此方案)

如果要写新链，基于快速上线，减少重复造轮子，避免安全性等等原因，我们应该选择一个经过现实世界检验的区块链快速开发脚手架工具。
我们选择了 [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) 区块链框架和 [Ignite CLI](https://github.com/ignite/cli) 基于 [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) 的开发者友好工具。

##### 优势

* 从技术上，[Cosmos SDK](https://github.com/cosmos/cosmos-sdk) 安全可信无法篡改值得信赖。
* [Ignite CLI](https://github.com/ignite/cli) 提供了一个快速开发的脚手架，开发后端，前端相对直接使用 [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) 更容易一些。
* 可以使用库 [ethermint](https://github.com/evmos/ethermint) 集成 [evm](https://ethereum.org/zh/developers/docs/evm/)，使用 [evm](https://ethereum.org/zh/developers/docs/evm/) 生态现有的大量智能合约，且这个库是 [evmos](https://evmos.org/) 链使用的底层库，这个链已经公网上线。
* [Cosmos SDK](https://docs.cosmos.network/main) 原生的 [Group 管理](https://docs.cosmos.network/main/modules/group#concepts) 类似我们的提案投票策略，迁移更加容易。

##### 劣势

* 需要我们自己运营。
* 我们拥有绝对管控权，用户可能担心我们通过回滚交易来篡改数据。

#### 3. 使用公链，比如 ETH 或者 BTC

开发智能合约来实现界面上对应的管理功能。

##### 优势

* 从技术和实际运行上都是安全可信无法篡改值得信赖的。

##### 劣势

* 费用贵。贵。贵。
* 出块慢，意味或者让用户操作后等待链确认，要么将用户操作和上链异步。
