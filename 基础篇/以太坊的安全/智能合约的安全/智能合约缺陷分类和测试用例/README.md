# 智能合约缺陷分类和测试用例
原文链接：https://swcregistry.io/

下表包含SWC注册表的概述。每一行由SWC标识符(ID)、缺陷标题、CWE父级和相关代码示例列表组成。ID和Test Cases列中的链接链接到各自的SWC定义。“关系”列中的链接链接到CWE基类或类类型。
|ID|Title|Relationships|Test Cases|
|:-:|:-|:-|:-|
|SWC-101|整数溢出和下溢|CWE-682: 不正确的计算|+ tokensalechallenge.sol<br/>+ integer_overflow_mapping_sym_1.sol<br/>+ integer_overflow_mapping_sym_1_fixed.sol<br/>+ integer_overflow_minimal.sol<br/>+ integer_overflow_minimal_fixed.sol<br/>+ integer_overflow_mul.sol<br/>+ integer_overflow_mul_fixed.sol<br/>+ integer_overflow_multitx_multifunc_feasible.sol<br/>+ integer_overflow_multitx_multifunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_feasible.sol<br/>+ integer_overflow_multitx_onefunc_feasible_fixed.sol<br/>+ integer_overflow_multitx_onefunc_infeasible.sol<br/>+ overflow_simple_add.sol<br/>+ overflow_simple_add_fixed.sol<br/>+ BECToken.sol|
|SWC-100|函数默认可见性|CWE-710: 没有正确的遵守编码标准|+ visibility_not_set.sol<br/>+ visibility_not_set_fixed.sol|

*[更新中......]*
