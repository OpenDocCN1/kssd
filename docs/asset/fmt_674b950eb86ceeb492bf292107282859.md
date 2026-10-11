
# 小小的 SD 2.90.40 调试器检测

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-826.htm`](https://www.pediy.com/kssd/pediy08/pediy8-826.htm)

今天有幸看了一下`safedisc`的调试器检查，发现颇多。按顺序小列如下，希望大侠们不要耻笑我。

`ZwQueryInformationProcess`和`ZwQuerySystemInformation`后面的值是我在无调试器的情况下运行再附加得到的。

SD 版本：`2.90.40`

- `IsDebuggerPresent`
- 9x:`\\.\SICE`
  - `\\.\SIWVID`
- NT:`\\.\NTICE`
- NT:`ZwQueryInformationProcess(GetCurrentProcess(),7,,,)=0?`
- NT:`stxt371`完整性检查
- 9x:`stxt371`完整性检查
- 9x:`Int3`
- NT:`Kernel32`模块函数首字节`CC`检查
- 9x:未知
- NT:`\\.\SecDrv`
- NT:`int1`
- 9x:`GetKeyboardType(0)`:返回键盘类型
  - `=7`(日文键盘):`GetKeyboardType(1)`:返回键盘子类型
    - `>0D07 Or <0D00:int68`
- NT:`ZwQuerySystemInformation(SystemDebuggerInfo,,,,)=824f0000`(传说中的`DebugPort`检查？)

呵呵，除了这些还有很多在`.sys`里的
