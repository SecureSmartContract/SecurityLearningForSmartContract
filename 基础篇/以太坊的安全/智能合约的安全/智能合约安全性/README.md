# 以太坊官方——智能合约安全性
原文链接：https://ethereum.org/en/developers/docs/smart-contracts/security/

以太坊智能合约非常灵活，既可以持有大量代币（通常超过10亿美元），又可以基于之前部署的智能合约代码运行不可变逻辑。虽然这创造了一个充满活力和创造性的生态系统，包含可信任的、相互关联的智能合约，但也是吸引攻击者利用智能合约漏洞和以太坊的意外行为牟利的完美生态系统。智能合约代码通常无法更改以修补安全漏洞，因此从智能合约中被盗的资产是无法追回的，且被盗资产极其难以追踪。由于智能合约问题而被盗或丢失的总价值很容易达到10亿美元。一些较大的由于智能合约而导致的损失包括：
+ [Parity钱包问题#1 - 3000万美元损失](https://www.coindesk.com/markets/2017/07/19/30-million-ether-reported-stolen-due-to-parity-wallet-breach/)
+ [Parity钱包问题#2 - 3亿美元锁定](https://www.theguardian.com/technology/2017/nov/08/cryptocurrency-300m-dollars-stolen-bug-ether)
+ [TheDAO被黑，360万ETH被盗！](https://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)

## 前置条件
这里将涵盖智能合约的安全性，所以在处理安全性之前，请确保您熟悉[智能合约](https://ethereum.org/en/developers/docs/smart-contracts/)

## 如何编写更安全的智能合约代码
在向主网启动任何代码之前，重要的是要采取充分的预防措施，以保护您的智能合约所受的任何有价值的东西。在本文中，我们将讨论一些特定的攻击，提供了解更多攻击类型的资源，并为您提供一些基本工具和最佳实践，以确保您的契约正确和安全地运行。

## 审计并不是灵丹妙药
几年前，用于编写、编译、测试和部署智能合约的工具还非常不成熟，导致许多项目随意编写 Solidity 代码，把代码扔到审计人员那里，让他们检查代码，以确保其安全运行。到2020年，支持编写 Solidity 的开发流程和工具将显著改善，利用这些最佳实践不仅可以确保您的项目更容易管理，而且是项目安全性的重要组成部分。在编写智能合约结束时进行审计，已不再是项目开发的唯一安全考虑。安全性在您编写第一行智能合约代码之前就开始了，安全性从正确的设计和开发过程开始。

## 智能合约开发过程
+ 所有代码存储在一个版本控制系统中，比如 git
+ 所有代码的修改都应该通过 Pull 请求
+ 所有的 Pull 请求都至少有一个审核人。如果你是一个单独的项目，考虑寻找另一个独立的作者和交易代码审查人
+ 使用以太坊开发环境，只需一个命令就可以编译、部署和运行一套针对你的代码的测试（参见：Truffle）
+ 已经通过如 Mythril 和 Slither 基本的代码分析工具运行了你的代码，最好在合并每个 Pull 请求之前，比较输出的差异
+ Solidity 不会发出任何编译器警告
+ 代码有好的文档

关于开发过程还有很多，但这些条目是一个很好的开始。更多条目和详细说明请参见 [DeFiSafety 提供的过程质量检查清单](https://docs.defisafety.com/master/process-quality-audit-process)。[DefiSafety](https://www.defisafety.com/) 是一个对公共以太坊 dApps 发布各种大型评论的非官方的公共服务。DeFiSafety 评级系统的一部分包括项目遵守过程质量检查清单的程度。通过遵循以下的过程：
+ 通过可复现的自动化测试，生成更安全的代码
+ 审计员将能够更有效地审查您的项目
+ 让新开发人员更容易上手
+ 允许开发人员快速迭代、测试，并获得修改的反馈
+ 项目代码回滚的可能性较低

## 攻击和漏洞
既然您正在使用高效的开发流程编写solididity代码，那么让我们看看一些常见的solididity漏洞，看看哪些地方可能出错。

### 重入攻击
重入攻击是开发智能合约时需要考虑的最大和最重要的安全问题之一。虽然EVM不能同时运行多个合约，但一个合约可以调用另一个合约来暂停合约的执行和内存状态，直到调用返回，此时代码仍会正常执行。这种暂停和重新启动可能会产生一个称为“重入攻击”的漏洞。

下面是一个容易被重入攻击的合约的简单版本：
```
// THIS CONTRACT HAS INTENTIONAL VULNERABILITY, DO NOT COPY
contract Victim {
    mapping (address => uint256) public balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
        balances[msg.sender] = 0;
    }
}
```

为了允许用户提取他们之前存储在合同中的ETH，这个功能
1. 读取用户的余额
2. 发送用户的余额
3. 将他们的余额重置为0，这样他们就不能再次提取余额了

如果合约被一个普通的帐户调用（比如你自己的Metamask帐户），这个函数会像预期的那样：msg.sender.call.value() 只是发送你的帐户 ETH。然而，智能合约也能调用其他合约。如果一个自定义的恶意合约调用了 withdraw()， msg.sender.call.value() 不仅会发送一定量的 ETH，还会隐式调用该合约来开始执行代码。想象一下这个恶意合约：
```
contract Attacker {
    function beginAttack() external payable {
        Victim(VICTIM_ADDRESS).deposit.value(1 ether)();
        Victim(VICTIM_ADDRESS).withdraw();
    }

    function() external payable {
        if (gasleft() > 40000) {
            Victim(VICTIM_ADDRESS).withdraw();
        }
    }
}
```

调用 attack.beginattack() 将开始一个循环，看起来像：
```
0) 攻击者账户调用了带有 1 个 ETH 的 attack.beginattack()
0) attack.beginattack() 向受害者存入 1 个 ETH
  1) 攻击者 → Victim.withdraw()
  1) 受害者读取余额 (msg.sender)
  1) 受害者向攻击者发送 ETH (执行默认函数)
    2) 攻击者 → Victim.withdraw()
    2) 受害者读取余额 (msg.sender)
    2) 受害者向攻击者发送 ETH (执行默认函数)
      3) 攻击者 → Victim.withdraw()
      3) 受害者读取余额 (msg.sender)
      3) 受害者向攻击者发送 ETH (执行默认函数)
        4) 攻击者最终因账户没有足够的 gas 费而停止调用
      3) balances[msg.sender] = 0;
    2) balances[msg.sender] = 0; (已经是0了)
  1) balances[msg.sender] = 0; (已经是0了)
```

攻击者账户使用 1 个 ETH 调用 Attacker.beginAttack 函数重入攻击受害者账户，取走比它提供的更多的 ETH（从其他用户的余额中取走，导致受害者余额减少）。

### 如何处理重入攻击（错误的方法）
一种简单的防止重入攻击的方法是通过阻止任何智能合约与你的代码进行交互。当你搜索 stackoverflow，你会发现这段代码有大量的赞：
```
function isContract(address addr) internal returns (bool) {
  uint size;
  assembly { size := extcodesize(addr) }
  return size > 0;
}
```

这似乎是有道理的：合约有代码，如果调用者有代码，就不允许存入。让我们添加它：
```
// THIS CONTRACT HAS INTENTIONAL VULNERABILITY, DO NOT COPY
contract ContractCheckVictim {
    mapping (address => uint256) public balances;

    function isContract(address addr) internal returns (bool) {
        uint size;
        assembly { size := extcodesize(addr) }
        return size > 0;
    }

    function deposit() external payable {
        require(!isContract(msg.sender)); // <- NEW LINE
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
        balances[msg.sender] = 0;
    }
}
```

现在，为了存入ETH，你的地址必须没有智能合约代码。然而，这很容易被以下攻击者合约击败：
```
contract ContractCheckAttacker {
    constructor() public payable {
        ContractCheckVictim(VICTIM_ADDRESS).deposit(1 ether); // <- New line
    }

    function beginAttack() external payable {
        ContractCheckVictim(VICTIM_ADDRESS).withdraw();
    }

    function() external payable {
        if (gasleft() > 40000) {
            Victim(VICTIM_ADDRESS).withdraw();
        }
   }
}
```

第一次攻击是对合约逻辑的攻击，而这次是对以太坊合约部署行为的攻击。在部署过程中，合约尚未返回已在其地址部署完成上的代码，但在此过程中保留了完全的 EVM 控制。

从技术上来说，防止智能合约调用你的代码是可能的，使用下面这行代码：
```
require(tx.origin == msg.sender)
```

然而，这仍然不是一个好的解决方案。以太坊最令人兴奋的一个方面是它的可组合性，智能合约相互集成和构建。通过使用上面的代码，您限制了项目的有用性。

### 如何处理重入攻击（正确的方法）
通过简单地切换存储更新和外部调用的顺序，我们可以防止启用攻击的可重入性条件。尽管可能调用提款，但攻击者不会从中受益，因为余额存储已经被设置为0。
```
contract NoLongerAVictim {
    function withdraw() external {
        uint256 amount = balances[msg.sender];
        balances[msg.sender] = 0;
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success);
    }
}
```

上面的代码遵循“检查-效果-交互”设计模式，这有助于防止重入。你可以在这里[阅读更多关于检查-效果-交互的信息](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html)

### 如何处理重入攻击（核心选项）
任何时候，当你将ETH发送到一个不可信的地址，或者与一个未知的合约（例如调用 transfer() 到用户提供的令牌地址）进行交互时，你就有了重入的可能性。**通过设计既不发送 ETH 也不调用不可信合约的合约，可以防止可重入的可能性！**

## 更多攻击类型
上述攻击类型涵盖了智能合约编码问题（重入）和以太坊的古怪之处（在合约地址可用代码之前，在合约构造函数内运行代码）。还有很多很多的攻击类型需要注意，比如:
+ 前台运行
+ ETH 发送拒绝
+ 整数上溢/下溢

延伸阅读：
+ [Consensys智能合约已知攻击](https://consensys.github.io/smart-contract-best-practices/attacks/) - 一个最重要的漏洞的非常易读的解释，并有大量示例代码。
+ [SWC Registry](https://swcregistry.io/docs/SWC-128) - 适用于以太坊和智能合约的CWE推荐列表。

## 安全工具
虽然理解以太坊的安全基础知识和聘请专业的审计公司来检查您的代码是不可替代的，但有许多工具可以帮助检查您的代码中的潜在问题。

### 智能合约安全性
**Slither - Solidity 静态分析框架，用 Python 3 编写。**
+ [Github](https://github.com/crytic/slither)

**MythX - 以太坊智能合约安全分析 API。**
+ [mythx.io](https://mythx.io/)
+ [相关文档](https://docs.mythx.io/)

**Mythril - EVM 字节码安全分析工具。**
+ [mythril](https://github.com/ConsenSys/mythril)
+ [相关文档](https://mythril-classic.readthedocs.io/en/master/about.html)

**Manticore - 在智能合约和二进制文件上使用符号执行工具的命令行接口。**
+ [Github](https://github.com/trailofbits/manticore)
+ [相关文档](https://github.com/trailofbits/manticore/wiki)

**Securify - 用于以太坊智能合约的安全扫描器。**
+ [securify.chainsecurity.com](https://securify.chainsecurity.com/)
+ [Discord](https://discord.com/)

**ERC20 验证器 - 用于检查合同是否符合 ERC20 标准的验证工具。**
+ [erc20-verifier.openzeppelin.com](https://erc20-verifier.openzeppelin.com/)
+ [论坛](https://forum.openzeppelin.com/t/online-erc20-contract-verifier/1575)

### 形式化验证
**形式化验证的信息**
+ [智能合约的形式化验证是如何工作的](https://runtimeverification.com/blog/how-formal-verification-of-smart-contracts-works/) *July 20, 2018 - Brian Marick*
+ [形式化验证如何确保智能合约的完美无缺](https://media.consensys.net/how-formal-verification-can-ensure-flawless-smart-contracts-cbda8ad99bd1?gi=3b3d600f1f88) *Jan 29, 2018 - Bernard Mueller*

### 使用工具
两个最流行的智能合同安全分析工具是：
+ [Slither](https://github.com/crytic/slither) 来自 [Trail of Bits](https://www.trailofbits.com/) （托管版本：[Crytic](https://www.crytic.io/)）
+ [Mythril](https://github.com/ConsenSys/mythril) 来自 [ConsenSys](https://consensys.net/) （托管版本：[MythX](https://mythx.io/)）

两者都是分析代码和报告问题的有用工具。每个都有一个[商业]托管版本，但也可以免费在本地运行。下面是如何运行 Slither 的一个快速示例，它可以在 Docker 映像 trailofbits/eth-security-toolbox 中获得。如果你还没有安装，你需要[安装 Docker](https://docs.docker.com/get-docker/)。
```
$ mkdir test-slither
$ curl https://gist.githubusercontent.com/epheph/460e6ff4f02c4ac582794a41e1f103bf/raw/9e761af793d4414c39370f063a46a3f71686b579/gistfile1.txt > bad-contract.sol
$ docker run -v `pwd`:/share  -it --rm trailofbits/eth-security-toolbox
docker$ cd /share
docker$ solc-select 0.5.11
docker$ slither bad-contract.sol
```

将生成以下输出：
```
ethsec@1435b241ca60:/share$ slither bad-contract.sol
INFO:Detectors:
Reentrancy in Victim.withdraw() (bad-contract.sol#11-16):
    External calls:
    - (success) = msg.sender.call.value(amount)() (bad-contract.sol#13)
    State variables written after the call(s):
    - balances[msg.sender] = 0 (bad-contract.sol#15)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities
INFO:Detectors:
Low level call in Victim.withdraw() (bad-contract.sol#11-16):
    - (success) = msg.sender.call.value(amount)() (bad-contract.sol#13)
Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-calls
INFO:Slither:bad-contract.sol analyzed (1 contracts with 46 detectors), 2 result(s) found
INFO:Slither:Use https://crytic.io/ to get access to additional detectors and GitHub integration
```

Slither 已经在这里确定了重入的可能性，确定了问题可能发生的关键线路，并为我们提供了有关问题的更多细节的链接：

&nbsp;&nbsp;&nbsp;&nbsp;参考：[https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities](https://github.com/crytic/slither/wiki/Detector-Documentation#reentrancy-vulnerabilities)

允许您快速了解代码的潜在问题。和所有自动化测试工具一样，Slither 也不是完美的，它会在报告方面出错。它可以警告潜在的重入，即使没有可利用的漏洞。通常，在代码更改之间检查 Slither 输出的差异非常有启示性，有助于发现比等到项目代码完成之前引入的漏洞早得多的漏洞。

## 延伸阅读
**智能合约安全最佳实践指南**
+ [consensys.github.io/smart-contract-best-practices/](consensys.github.io/smart-contract-best-practices/)
+ [Github](https://github.com/ConsenSys/smart-contract-best-practices/)
+ [安全建议和最佳实践的聚合集合](https://github.com/guylando/KnowledgeLists/blob/master/EthereumSmartContracts.md)

**智能合约安全验证标准（SCSVS）**
+ [securing.github.io/SCSVS/](https://securing.github.io/SCSVS/)

*还有哪些社区资源帮助过你？请编辑这个页面并添加它！*

## 相关教程
+ [安全开发流程](https://ethereum.org/en/developers/tutorials/secure-development-workflow/)
+ [如何使用 Slither 找到智能合同漏洞](https://ethereum.org/en/developers/tutorials/how-to-use-slither-to-find-smart-contract-bugs/)
+ [如何使用 Manticore 找到智能合同的漏洞](https://ethereum.org/en/developers/tutorials/how-to-use-manticore-to-find-smart-contract-bugs/)
+ [安全指引](https://ethereum.org/en/developers/tutorials/smart-contract-security-guidelines/)
+ [令牌安全](https://ethereum.org/en/developers/tutorials/token-integration-checklist/)
