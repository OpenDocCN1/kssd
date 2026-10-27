
```
contract Ballot {
//一个选民的构造体
struct Voter {
uint weight; // 权重（即他可以投几票）
bool voted; //是否已经投过票
address delegate; // 代表地址（他可以代表某个人进行投票）
uint vote; // index of the voted proposal
}
// 投票的提案的构造体
struct Proposal
{
bytes32 name; // 提案名称
uint voteCount; //获得的票数
}
address public chairperson;//会议主席
//地址 -选民 的map
mapping(address => Voter) public voters;
// 投票种类的动态数组
Proposal[] public proposals;
///构造函数
function Ballot(bytes32[] proposalNames) {
chairperson = msg.sender;//初始化会议主席
voters[chairperson].weight = 1;
//初始化所有的提案
for (uint i = 0; i < proposalNames.length; i++) {
proposals.push(Proposal({
name: proposalNames[i],
voteCount: 0
}));
}
}
// 给予投票权
function giveRightToVote(address voter) returns (bool b) {
if (msg.sender != chairperson || voters[voter].voted) {
//对于会议主席和已经投过票的选民这里不处理
return false;;
}
voters[voter].weight = 1;
return true;
}
/// 投票权转移函数
function delegate(address to) {
// 投票权转移的发起人
Voter sender = voters[msg.sender];
if (sender.voted)
throw;
//递归找到没有转移投票权的 选民
while (
voters[to].delegate != address(0) &&
voters[to].delegate != msg.sender
) {
to = voters[to].delegate;
}
if (to == msg.sender) {
throw;
}
//将发起人设置为已经投过票的状态
sender.voted = true;
//将代表设置为刚才递归获取的选民
sender.delegate = to;
Voter delegate = voters[to];
if (delegate.voted) {
//如果代表已经投过票就在他投票的提案的票数增加
proposals[delegate.vote].voteCount += sender.weight;
}
else {
//将代表的的票数增加
delegate.weight += sender.weight;
}
}
/// 投票函数
function vote(uint proposal) {
Voter sender = voters[msg.sender];
if (sender.voted)
throw;
sender.voted = true;
sender.vote = proposal;
//将投的提案票数增加
proposals[proposal].voteCount += sender.weight;
}
///获得票数最多的提案
function winningProposal() constant
returns (uint winningProposal)
{
uint winningVoteCount = 0;
for (uint p = 0; p < proposals.length; p++) {
if (proposals[p].voteCount > winningVoteCount) {
winningVoteCount = proposals[p].voteCount;
winningProposal = p;
}
}
}
```

解读

* `address`. 地址类型。`chairperson`是会议主席的钱包地址。这个地址会在合约的构造函数`function Ballot()`中被赋值。很多时候也称呼这种地址为'owner'（所有人）。

* `public`. 这个关键字表明变量可以被合约之外的对象使用。`private`修饰符则表示变量只能被本合约(或者衍生合约)内的对象使用。如果你想要在测试中通过`web3.js`使用合约中的某个变量，记得把它声明为`public`。

* Mapping或数组。`mapping(address => Voter)`为选民钱包地址和选民构造体的键值对。`Proposal[] public proposals`是一个提案构造体的数组。

* 有特殊的变量和函数总是在全局命名空间存在，主要用于提供有关blockchain信息，例如`msg`，`block`，`tx`，其中`msg.sender`为发起人的地址。

solidity语言更深入的理解可以阅读<a href="elink@bb0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3I4A6k6r3W2@1P5g2)9J5k6i4u0W2j5h3c8@1K9r3g2V1L8$3y4K6i4K6u0W2K9h3!0Q4x3V1k6W2L8W2)9J5c8X3I4S2N6r3g2K6N6q4)9J5c8Y4y4G2L8r3W2V1K9i4c8&6i4K6u0V1K9h3&6Q4x3X3c8V1k6i4m8@1K9q4)9J5k6h3S2@1L8h3H3`.">官方文档</a>。

### c. 使用`geth`部署合约

1. 启动一个测试节点

> `geth --testnet --fast --cache=512 --genesis CustomGenesis.json console`

这里的`CustomGenesis.json`是为了给测试的账户分配以太币

```
{
"coinbase": "0x0000000000000000000000000000000000000000",
"difficulty": "0x20000",
"extraData": "",
"gasLimit": "0x2fefd8",
"nonce": "0x0000000000000042",
"mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
"timestamp": "0x00",
"alloc": {
"0xe49c283bc6bf92c5833cc981b97679238dd3b5da": {
"balance": "111111111000000000000000000000000000"
},
"0xd8927c296b3ebe454a6409770a0c323ec4ed23ba": {
"balance": "222222222000000000000000000000000000"
}
}
}
```

`solc`下的内容要替换成你的测试账户地址。具体`geth`的用法请查看<a href="elink@26eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2@1K9r3c8G2j5%4y4Q4x3X3g2G2M7X3N6Q4x3V1k6W2L8W2)9J5c8X3I4S2N6r3g2K6N6q4)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6@1k6i4y4@1i4K6u0V1L8X3g2@1N6$3!0J5K9%4y4Q4x3X3g2Z5N6r3#2D9">官方文档</a>和<a href="elink@8f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6r3S2W2M7X3g2#2L8g2)9J5c8X3N6G2i4K6u0V1k6i4c8Z5k6i4u0W2N6h3@1`.">源码介绍</a>。

2. 使用`solc`编译智能合约，获得二进制代码

例如以下代码

```
contract test {
function multiply(uint a) returns(uint d) {
return a \* 7;
}
}
```

在`geth`中输入

> `source = "contract test { function multiply(uint a) returns(uint d) { return a \* 7; } }"`

> `clientContract = eth.compile.solidity(source).test`

编译返回的结果的JSON格式如下

![](upload/attach/201902/93\_HHBMQERNAQCCJQZ.png)

其中，

* `code`：编译后的EVM字节码
* `info`：编译器返回的metadata
* `abiDefination`: Application Binary Interface定义。具体接口规则参见这里
* `compilerVersion`：编译此代码的solidity编译器版本
* `developerDoc`：针对开发者的Natural Specification Format，类似于Doxygen
* `language`：合约语言
* `languageVersion`：合约语言版本
* `source`：源代码
* `userDoc`：针对用户的Ethereum的Natural Specification Format。

编译器返回的JSON结构反映了合约部署的两种不同的路径。`info`信息真实的存在于区中心化的云中，作为metadata信息来公开验证Blockchain中合约代码的实现。而`code`信息通过创建交易的方式部署到区块链中。

3. 部署合约

部署合约前，确保你有一个解锁的账户并且账户中有余额，因为部署合约得过程中会消耗以太币。输入`web3.fromWei(eth.getBalance(eth.accounts[0]),"ether")`可以查看账户余额。

解锁一个账户

> `personal.unlockAccount(eth.accounts[0])`

获得账户

> `primaryAddress = eth.accounts[0]`

定义一个`abi`（`abi`是个js的数组，否则不成功）

> `abi = [{ constant: false, inputs: [{ name: 'a', type: 'uint256' } ]}]`

创建智能合约

> `MyContract = eth.contract(abi)`

发送交易部署合约

> `contract = MyContract.new({from: primaryAddress, data:"0x6060604052602a8060106000396000f3606060405260e060020a6000350463c6888fa18114601a575b005b6007600435026060908152602090f3"})`

![](upload/attach/201902/93\_XVYDURZQE3XRFX8.png)

如果交易被pending，如图说明你的`miner`没有在挖矿

![](upload/attach/201902/93\_PAZXHAAMN2CPBXD.png)

> 启动一个矿工

> `miner.setEtherbase(eth.primaryAddress) //设定开采账户`

> `miner.start(8)`

> `eth.getBlock("pending", true).transactions`

> 这时候发现交易已经在区块中

> 不过会发现，交易还是pending，这是因为该交易区块没有人协助进行运算验证，这时候只需要再启动一个矿工就行了

> `miner.start(8)`

4. 与合约进行交互

> `Multiply7 = eth.contract(clientContract.info.abiDefinition);`

> `var myMultiply7 = Multiply7.at(contract.address);`

> `myMultiply7.multiply.call(3)`

> 或

> `myMultiply7.multiply.sendTransaction(3, {from: contract.address})`

### d. 使用`truffle`框架

1. `truffle init` (在新目录中) => 创建truffle项目目录结构，
2. 编写合约代码，保存到`contracts/YourContractName.sol`文件。例如`Ballot.sol`，此时要找到`migrations`文件夹，在`deploy_contracts.js`文件中添加`deployer.deploy(Ballot);`
3. `truffle compile` 编译合约代码。
4. 启动以太坊节点（例如在另一个终端里面运行`testrpc`）。
5. `truffle migrate`（在`truffle`项目目录中)。

#### 1. 编写单元测试

在`test`文件夹中新建`ballot.js`文件

```
contract('Ballot'，function(accounts)){
//accounts是所以账户得数值
it("获取投票权"，function(){
var meta = Ballot.deployed();
return meta.giveRightToVote(accounts[1]).then(function(b){
assert.equal(Boolean(b),true,"获取投票权失败");
});
});
}
```

在项目根目录下运行`truffle test`，你应该看到测试通过，如果使用自己构造的`ballot`对象，可以这样写：

```
contract('Ballot'，function(accounts)){
//accounts是所以账户得数值
it("获取投票权"，function(){
var proposals = [];
proposals.push("proposal0");
Ballot.new(proposals).then(function(meta){
return meta.giveRightToVote(accounts[1]).then(function(b){
assert.equal(Boolean(b),true,"获取投票权失败");
});
});
});
}
```

* 合约中发送以太币。`this`是合约实例的地址，以变接下来检查这个地址的余额（或者直接使用`this.balance`）

* 当你通过`web3.js`调用交易函数时（使用`web3.eth.sendTransaction`），交易并不会立即执行。事实上交易会被提交到矿工网络中，交易代码直到其中一位矿工产生一个新区块把交易记录进区块链之后才执行。因此你必须等交易进入区块链并且同步回本地节点之后才能验证交易执行的结果。用`testrpc`的时候可能看上去是实时的，因为测试环境很快，但是正式网络会比较慢。

* Gas. （译注：以太坊上的燃料，因为代码的执行必须消耗Gas。直译为汽油比较突兀，故保留原文做专有名词。）直到现在我们都没有涉及Gas的概念，因为在使用`testrpc`时通常不需要显式的设置。当你转向`geth`和正式网络时会需要。在交易函数调用中可以在`{ from: _, value: _, gas: _ }` 对象内设置Gas参数。`Web3.js`提供了`web3.eth.gasPrice`调用来获取当前Gas的价格，Solidity编译器也提供了一个参数让你可以从命令行获取合约的Gas开销概要：`solc --gas YouContract.sol`。

#### 2. 为合约创建一个界面

在`app`目录中，可以编写自己的html和js文件，js与智能合约的交互与单元测试基本一致，例如一个界面上有一个输入框和一个按钮，获得选民的投票权。

```
<!DOCTYPE html>
<html>
<head>
<title>Ballot App</title>
<link href='https://fonts.googleapis.com/css?family=Open+Sans:400,700' rel='stylesheet' type='text/css'>
<link href="./app.css" rel='stylesheet' type='text/css'>
<script src="./app.js"></script>
</head>
<body>
<h1>Ballot</h1>
<h2>Example Truffle Dapp</h2>
<br>
<h1>Send</h1>
<br><label for="amount">Account:</label><input type="text" id="account" placeholder="e.g., 0x453468394hdfg84858345348"></input>
<br><br><button id="getRightVote" onclick="getRight()">Get Right Vote</button>
<br><br>
<span id="status"></span>
</body>
</html>
```

`app.js`中的代码为
