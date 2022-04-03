# 以太坊智能合约安全指引
原文链接：https://blog.openzeppelin.com/onward-with-ethereum-smart-contract-security-97a827e47702/

如果你是以太坊开发的新手，我建议你在继续之前阅读我们的[《以太坊智能合约指南》](https://medium.com/zeppelin-blog/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05#.6dob381ks)。

学习以太坊智能合约的安全性是非常困难的。很少有好的指南和汇编，如 [Consensys](https://github.com/ConsenSys/smart-contract-best-practices) 智能合约实践或 [Solidity 安全考量](https://docs.soliditylang.org/en/latest/security-considerations.html)。但是如果不自己写下来，这些概念是很难记住和消化的。

我将尝试一种稍微不同的方法。我将解释一些建议的策略，以提高智能合约的安全性，并展示不遵守这些策略会导致问题的例子。我还将向您展示可以用来保护智能合约的示例。希望，这将有助于强化记忆，帮助我们思考。

闲话少说，让我们来看看这些最佳实践：

## 尽可能早的发现失败
一个简单而强大的编程实践是尽可能迅速地让失败发生。大声说出来。让我们看一个合约的例子：
```
// UNSAFE , DO NOT USE!

contract BadFailEarly {
  uint constant DEFAULT_SALARY = 50000;
  mapping(string => uint) nameToSalary;

  function getSalary(string name) constant returns (uint) {
    if (bytes(name).length != 0 && nameToSalary[name] != 0) {
      return nameToSalary[name];
    } else {
      return DEFAULT_SALARY;
    }
  }
}
```

我们希望避免合约失败，或者在不稳定或不一致的状态下继续执行。函数 `getSalary` 在返回存储的工资之前检查条件，这是一件好事。问题是，如果这些条件不满足，则返回一个默认值。这可能会对调用者隐藏错误。这是一种极端情况，但这种编程很常见，通常是因为我们担心出错会破坏应用。事实是，我们越早失败，就越容易发现问题。如果我们隐藏错误，它们会传播到其他部分，并导致难以跟踪的不一致。更正确的方法是：
```
contract GoodFailEarly {

  mapping(string => uint256) nameToSalary;

  function getSalary(string name) constant returns (uint256) {
    if(bytes(name).length == 0) throw;
    if(nameToSalary[name] == 0) throw;

    return nameToSalary[name];
  }
}
```

这个版本还展示了另一种理想的编程模式，即分离先决条件并使每个条件单独失败。注意，其中一些检查（特别是那些依赖于内部状态的检查）可以通过函数修饰符来实现。

## 推荐 pull 付款而不是 push 付款
每一次 Ether 转移都意味着潜在的执行。接收地址可以实现一个可以抛出错误的 Fallback 函数。因此，我们绝不应该相信 send 调用会毫无错误地执行。一个解决方案是：我们的合约应该偏向于 pull 付款而不是 push 付款。

让我们来看看下面这个竞价函数：
```
// UNSAFE , DO NOT USE!
contract BadPushPayments {

  address highestBidder;
  uint256 highestBid;

  function bid() {
    if(msg.value < highestBid) throw;
    if (highestBidder != 0) {
      // return bid to previous winner
      if (!highestBidder.send(highestBid)) {
        throw;
      }
    }
    highestBidder = msg.sender;
    highestBid = msg.value;
  }
}
```

请注意，合约调用 `send` 函数并检查其返回值，这似乎是合理的。但是它在函数的中间调用 `send`，这是不安全的。为什么？记住，如上所述，send 可以触发另一个合约的执行。

想象一下，有人从一个地址出价，但每次有人给它寄钱时，这个地址就会抛出一个错误。如果有人出价高于这个价格，会发生什么？`send` 调用将总是失败，冒泡并使 `bid` 抛出一个异常。以错误结束的函数调用将保持状态不变（所做的任何更改都将回滚）。这意味着没有其他人可以投标，合约就被破坏了。

最简单的解决方案是将支付分成不同的功能，让用户独立于合约逻辑的其余部分请求(pull)资金：
```
contract GoodPullPayments {
  address highestBidder;
  uint highestBid;

  mapping(address => uint) refunds;

  function bid() external {
    if (msg.value < highestBid) throw;

    if (highestBidder != 0) {
      refunds[highestBidder] += highestBid;
    }

    highestBidder = msg.sender;
    highestBid = msg.value;
  }

 function withdrawBid() external {
    uint refund = refunds[msg.sender];
    refunds[msg.sender] = 0;
    if (!msg.sender.send(refund)) {
      refunds[msg.sender] = refund;
    }
  }
}
```

这一次，我们使用一个 `mapping` 来存储每个中标者的退款值，并提供一个函数来提取他们的资金。在 `send` 调用出现问题的情况下，只有该出价方受到影响。这是一个简单的模式，可以解决许多其他问题，如重入攻击，所以请记住：当发送 ETH 时，建议 pull 付款而非 push 付款。

## 组织功能顺序：条件、动作、交互

*[未完待续...]*



