
# [求助]OD条件断点多条件链接符号 “”&&“”

`[esp+c]!=7&&[esp+c]!=8&&[esp+c]!=9` 这个 `&&` 在OD上是连接符的意思还是代表逻辑运算符？如果是逻辑运算符 `[esp+8]=="EnumProcesses"&&[esp+8]=="EnumProcessesModules"&&[esp+8]=="GetModuleBaseNameA"`，OD反调试的下这个几函数断点断不下来，反而反过来 `[esp+8]=="EnumProcesses"||[esp+8]=="EnumProcessesModules"||[esp+8]=="GetModuleBaseNameA"`，能断下2个函数，虽然没能断下三个，所以请问下前辈，OD `&&` 链接符 实际上是不是逻辑运算符？

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
