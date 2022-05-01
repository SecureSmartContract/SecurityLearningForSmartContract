# 智能合约缺陷分类和测试用例
原文链接：https://swcregistry.io/

下表包含SWC注册表的概述。每一行由SWC编号(ID)、缺陷标题和相关代码示例列表组成。“标识符”列链接到其SWC定义。
|ID|缺陷标题|测试用例|
|:-:|:-|:-|
|SWC-112|Delegatecall调用漏洞|+ proxy.sol<br/>+ proxy_fixed.sol<br/>+ proxy_pattern_false_positive.sol|
|SWC-111|使用已弃用的Solidity函数|+ deprecated_simple.sol<br/>+ deprecated_simple_fixed.sol|
|SWC-110|不合规的断言|+ assert_constructor.sol<br/>+ assert_minimal.sol<br/>+ assert_multitx_1.sol<br/>+ assert_multitx_2.sol|
|SWC-109|未初始化的存储指针|+ crypto_roulette.sol<br/>+ crypto_roulette_fixed.sol|
|SWC-108|状态变量默认可见性|+ storage.sol|
|SWC-107|重入攻击|+ modifier_reentrancy.sol<br/>+ modifier_reentrancy_fixed.sol<br/>+ simple_dao.sol<br/>+ simple_dao_fixed.sol|
|[SWC-106](SWC-106.md)|无保护的SELFDESTRUCT指令|+ WalletLibrary.sol<br/>+ simple_suicide.sol<br/>+ suicide_multitx_feasible.sol<br/>+ suicide_multitx_infeasible.sol|
|[SWC-105](SWC-105.md)|不受保护的以太币取款|+ tokensalechallenge.sol<br/>+ rubixi.sol<br/>+ multiowned_not_vulnerable.sol<br/>+ multiowned_vulnerable.sol<br/>+ simple_ether_drain.sol<br/>+ wallet_01_ok.sol<br/>+ wallet_02_refund_nosub.sol<br/>+ wallet_03_wrong_constructor.sol<br/>+ wallet_04_confused_sign.sol|
|[SWC-104](SWC-104.md)|未检查的调用返回值|+ unchecked_return_value.sol|
|[SWC-103](SWC-103.md)|浮动的编译器版本|+ floating_pragma.sol<br/>+ floating_pragma_fixed.sol<br/>+ no_pragma.sol<br/>+ semver_floating_pragma.sol<br/>+ semver_floating_pragma_fixed.sol|
|[SWC-102](SWC-102.md)|过时的编译器版本|+ version_0_4_13.sol|
|[SWC-101](SWC-101.md)|整数溢出和下溢|+ tokensalechallenge.sol<br/>+ integer_overflow_mapping_sym_1.sol<br/>+ integer_overflow_mapping_sym_1_fixed.sol<br/>+ integer_overflow_minimal.sol<br/>+ integer_overflow_minimal_fixed.sol<br/>+ integer_overflow_mul.sol<br/>+ integer_overflow_mul_fixed.sol<br/>+ integer_overflow_multitx_multifunc_feasible.sol<br/>+ integer_overflow_multitx_multifunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_feasible.sol<br/>+ integer_overflow_multitx_onefunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_infeasible.sol<br/>+ overflow_simple_add.sol<br/>+ overflow_simple_add_fixed.sol<br/>+ BECToken.sol|
|[SWC-100](SWC-100.md)|函数默认可见性|+ visibility_not_set.sol<br/>+ visibility_not_set_fixed.sol|

*[更新中......]*
