Mythril 是以太坊EVM字节码的安全分析工具。它检测以太坊、Hedera、Quorum、Vechain、Roostock、Tron 和其他兼容 EVM 的区块链构建的智能合约中的安全漏洞。它使用符号执行、SMT方案来分析检测智能合约代码中的各种安全漏洞。

[Mythril Github](https://github.com/ConsenSys/mythril)

## 安装

通过 Docker 获取：

```
$ docker pull mythril/myth
```

 从 Pypi 安装：

```
$ pip3 install mythril
```

目前，Mythril 支持 MacOS 和 Ubuntu，不支持 Windows。

## 用法

Mythril 支持 Solidity 源代码和合约地址的检测。

运行：

```
$ myth analyze <solidity-file>
```

或者：

```
$ myth analyze -a <contract-address>
```

## 实践

本文介绍在 Ubuntu 操作系统环境下的工具安装和检测分析过程。

### Docker 环境下安装

经笔者实践，采用非 Docker 方式安装时，安装过程报错。 所以本检测分析过程基于 Docker 环境。

所有的 Mythril 版本，从 v0.18.3 开始，都以 mythrl/myth 的名称作为 Docker images 发布到 Docker Hub。

安装 Docker CE 后，通过以下命令 pull 最新版本的 mythril/myth：

```
$ docker pull mythril/myth
```

使用 docker 运行mythrl/myth，就像你使用 myth 命令一样。

查看 mythril/myth 帮助的命令如下：

```
$ docker run mythril/myth --help
```

检测分析 Solidity 源代码的命令如下：

```
$ docker run -v $(pwd):/tmp mythril/myth analyze /tmp/contract.sol
```

### 用于检测的合约源码

**Roulette.sol**

```
// SPDX-License-Identifier: MIT
pragma solidity >=0.4.22;

contract Roulette {
    uint public pastBlockTime;
 
    // initially contract
    constructor() {}

    // receive function
    receive() external payable {}

    // fallback function used to make a bet
    fallback() external payable {
        require(msg.value == 1 ether); // must send 1 ether to play
        require(block.timestamp != pastBlockTime);  // only 1 transaction per block
        pastBlockTime = block.timestamp;
        if(block.timestamp % 15 == 0) { // winner
            payable(msg.sender).transfer(address(this).balance);
        }
    }
}
```

以上这个智能合约有一个时间戳依赖漏洞。时间戳依赖是指智能合约的执行依赖于当前区块的时间戳，如果时间戳不同，那么合约的执行结果也有差别。智能合约中取得时间戳只能依赖某个节点（矿工）来做到。这就是说，合约中取得的时间戳是由运行其代码的节点（矿工）的计算机本地时间决定的。理论上，这个本地时间是可以由矿工控制的。因此，如果在智能合约中不正确地使用区块时间戳，这将是非常危险的。

### 检测分析过程

接下来，让我们观察 Mythril 工具对这个合约的检测分析结果。

首先，我们把这个合约文件传递到 Ubuntu 主机的 ”/var/tmp/solidity_examples“ 目录。

然后，通过以下命令来对合约源码执行检测分析：

```
$ docker run -v $(pwd):/var/tmp/solidity_examples mythril/myth analyze /var/tmp/solidity_examples/Roulette.sol

```

耐心等待几分钟后（这个执行时间取决于主机的配置，大概在1分钟到几分钟不等），我们看到检测结果如下：

```
==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: Roulette
Function name: fallback
PC address: 70
Estimated Gas Usage: 918 - 1013
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
In file: /var/tmp/solidity_examples/Roulette.sol:16

imestamp != pastBlockTime);  // only 1 tr

--------------------
Initial State:

Account: [CREATOR], balance: 0xd, nonce:0, storage:{}
Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], calldata: , value: 0x0
Caller: [SOMEGUY], function: unknown, txdata: 0x01010101, value: 0xde0b6b3a7640000

==== Dependence on predictable environment variable ====
SWC ID: 116
Severity: Low
Contract: Roulette
Function name: fallback
PC address: 102
Estimated Gas Usage: 6134 - 26229
A control flow decision is made based on The block.timestamp environment variable.
The block.timestamp environment variable is used to determine a control flow decision. Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. Also keep in mind that attackers know hashes of earlier blocks. Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
--------------------
In file: /var/tmp/solidity_examples/Roulette.sol:18

p % 15 == 0) { // winner
            payable(msg.sender).transfer(address(this).balance);
        }
    }
}

--------------------
Initial State:

Account: [CREATOR], balance: 0x4000142021000000, nonce:0, storage:{}
Account: [ATTACKER], balance: 0x0, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], calldata: , value: 0x0
Caller: [SOMEGUY], function: unknown, txdata: 0x01010101, value: 0xde0b6b3a7640000

```

通过以上的检测分析结果，我们大致可以得出结论，Mythril 工具确实检测出了 Roulette.sol 这个合约中的漏洞。合约代码中的两个地方存在时间戳依赖漏洞。

让我们对检测结果做进一步的解释，其中：

SWC ID: 116 —— 表示该漏洞的分类编号，详见 [智能合约缺陷分类和测试用例](https://github.com/SecureSmartContract/SecurityLearningForSmartContract/tree/main/基础篇/以太坊的安全/智能合约的安全/智能合约缺陷分类和测试用例)

Severity: Low  —— 表示该漏洞的严重性程度

Contract: Roulette —— 表示检测的合约名称

Function name: fallback —— 表示发现漏洞的函数名称

PC address: 70 —— 这个输出是啥意思不详，有知道的给我留言哈

Estimated Gas Usage: 918 - 1013 —— 表示估算的 Gas 费用开销

接下来是对此漏洞的描述，让我们看一下翻译后的结果：

控制流的决定是基于区块的时间戳环境变量。block.timestamp 环境变量用于确定控制流决策。请注意，coinbase、gaslimit、block number 和 timestramp 等变量的值是可预测的，可以被恶意矿工操纵。还要记住，攻击者知道早期块的哈希值。请注意，不要使用任何这些环境变量作为随机性的来源，并且确保使用的这些变量对矿工是可信任的。

再接下来是漏洞代码在合约代码中的行号，并给出了代码段，以及合约的初始状态和交易序列。

*[未完待续...]*
