# 智能合约缺陷分类和测试用例
原文链接：https://swcregistry.io/

下表包含SWC注册表的概述。每一行由SWC编号(ID)、缺陷标题、CWE父级和相关代码示例列表组成。“标识符”列链接到其SWC定义。“关系”列链接到CWE基类或类类型。
|标识符|缺陷标题|关系|测试用例|
|:-:|:-|:-|:-|
|SWC-105|不受保护的以太币取款|CWE-284: 访问控制不当|+ tokensalechallenge.sol<br/>+ rubixi.sol<br/>+ multiowned_not_vulnerable.sol<br/>+ multiowned_vulnerable.sol<br/>+ simple_ether_drain.sol<br/>+ wallet_01_ok.sol<br/>+ wallet_02_refund_nosub.sol<br/>+ wallet_03_wrong_constructor.sol<br/>+ wallet_04_confused_sign.sol|
|SWC-104|未检查的调用返回值|CWE-252: 未检查的返回值|+ unchecked_return_value.sol|
|SWC-103|浮动的编译器版本|CWE-664: 对资源生命周期的不适当控制|+ floating_pragma.sol<br/>+ floating_pragma_fixed.sol<br/>+  no_pragma.sol<br/>+ semver_floating_pragma.sol<br/>+ semver_floating_pragma_fixed.sol|
|SWC-102|过时的编译器版本|CWE-937: 使用含有已知安全漏洞的组件|+ version_0_4_13.sol|
|SWC-101|整数溢出和下溢|CWE-682: 不正确的计算|+ tokensalechallenge.sol<br/>+ integer_overflow_mapping_sym_1.sol<br/>+ integer_overflow_mapping_sym_1_fixed.sol<br/>+ integer_overflow_minimal.sol<br/>+ integer_overflow_minimal_fixed.sol<br/>+ integer_overflow_mul.sol<br/>+ integer_overflow_mul_fixed.sol<br/>+ integer_overflow_multitx_multifunc_feasible.sol<br/>+ integer_overflow_multitx_multifunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_feasible.sol<br/>+ integer_overflow_multitx_onefunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_infeasible.sol<br/>+ overflow_simple_add.sol<br/>+ overflow_simple_add_fixed.sol<br/>+ BECToken.sol|
|SWC-100|函数默认可见性|CWE-710: 没有正确的遵守编码标准|+ visibility_not_set.sol<br/>+ visibility_not_set_fixed.sol|

*[更新中......]*
