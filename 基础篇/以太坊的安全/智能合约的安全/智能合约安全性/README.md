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



### 如何处理重入攻击（错误的方法）


### 如何处理重入攻击（正确的方法）


## 更多攻击类型

## 安全工具

## 延伸阅读

## 相关教程
