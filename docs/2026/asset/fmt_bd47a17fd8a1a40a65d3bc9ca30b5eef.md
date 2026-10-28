
> 注意, 因为`VirtualAllocEx`返回的是虚拟地址, 默认情况下`CreateRemoteThread`函数的`lpStartAddress`参数使用该地址是没问题的. 但是**若注入器是32位而被注入程序是64位**, 则可能导致`CreateRemoteThread`失败而返回NULL. 参考: [createremotethread-returns-null-while-trying-to-use-it-to-inject-dll](https://bbs.kanxue.com/elink@e7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3y4U0l9$3z5o6M7@1y4e0S2Q4x3V1k6U0M7X3g2S2N6r3g2J5k6h3#2G2N6r3g2@1K9s2u0W2j5h3c8Q4x3X3c8J5k6i4c8#2M7X3&6K6i4K6u0V1L8Y4g2D9L8q4)9J5k6s2N6Z5K9h3I4W2i4K6u0V1N6s2u0&6K9h3&6Y4i4K6u0V1N6r3!0Q4x3X3c8#2M7$3g2Q4x3X3c8A6N6q4)9J5k6s2c8G2i4K6u0V1K9h3&6B7k6h3y4@1i4K6u0V1k6r3I4D9)

# 反射式注入(reflective injection)

## 概要

- 没有使用`LoadLibrary`函数注入dll.
- 需由注入器自行解析PE文件:
  - 将dll头部(包括`DOS头`, `PE头`, `区块表`)逐字节写入新开辟的内存.
  - 按重定位表的信息手动重定位
  - 修复导入函数表: 使用`LdrLoadDll`得到shellcode需要的库的内存地址, 用`LdrGetProcedureAddress`得到要导入的函数的内存地址, 然后将这些地址填入导入表.

## 流程

- 将自己实现的LoadLibrary功能函数保存为shellcode.
  - 注: 在shellcode中使用的系统api都要事先通过`GetProcAddress`获取(使用`GetModuleHandleA`获取模块句柄, 传入的模块名不用后缀), 并**作为参数传给shellcode**.
  - 注: 需要审查注入器保存的shellcode是否是真实的函数体. 调试发现在vs2019中, 按默认选项编译**得到的函数地址处是一条跳转到实际函数体的jmp指令**. 因此`需要使用jmp指令的操作数计算实际函数地址`. 如下为获取一个shellcode函数的代码:

    |  |  |
    | --- | --- |
    | 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30 | `// shellcode函数`  `void` `shellcodeFunc(PMY_PARAMS pParams) {`  `// pParams保存LdrLoadDll等系统api的内存地址`  `// 用NtAllocateVirtualMemory在目标进程中开辟一块内存(需指定PAGE_EXECUTE_READWRITE权限)`  `// 将dll的文件内容写入开辟的内存`  `// 修复导入表; 重定位`  `// 执行dll的入口函数DLLMain`  `}`    `DWORD` `size = 0, ssss=0;`    `// 获取jmp指令后的双字操作数(即jmp的目的地址偏移)`  `DWORD`* jmpAddr = (`DWORD`*) ((`BYTE`*) shellcodeFunc + 1);    `// 加5得到jmp指令的下一条指令地址, 然后加上jmp的目的地址偏移, 得到函数体的实际起始地址`  `WORD`* Memx0 = (`WORD`*) ((`BYTE`*) shellcodeFunc + 5 + *jmpAddr);  `LONG_PTR`* Memx = (`LONG_PTR`*) Memx0;    `// 用0xCCCCCCCCCCCCCCCC作为函数体结束识别标识.`  `while` `(*Memx != 0xCCCCCCCCCCCCCCCC) {`  `Memx++;`  `size += 8;`  `}`    `// 将shellcode写入文件`  `HANDLE` `hFile = CreateFile(LOADECODE, GENERIC_ALL, FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, CREATE_ALWAYS, NULL, NULL);`  `if` `(hFile) {`  `WriteFile(hFile, Memx0, size, &ssss, NULL);`  `CloseHandle(hFile);`  `}` |

  - 20230810:发现可以通过修改编译和链接选项, 让编译的程序不会多一个跳转表.
    - 编译时带上`/GL`选项(`C/C++` -> `优化` -> `全程序优化`, 选择`是 (/GL)`).
    - 链接时带上`/INCREMENTAL:NO`选项(`链接器` -> `常规` -> `启用增量链接`, 选`否 (/INCREMENTAL:NO)`). 在[微软对这个链接选项的说明文档](https://bbs.kanxue.com/elink@8d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3W2F1j5%4u0W2L8h3g2F1N6r3q4D9i4K6u0V1L8r3W2F1K9#2)9J5k6r3W2F1j5%4u0W2L8h3g2F1N6r3q4D9L8s2W2Q4x3@1k6$3K9h3g2%4i4K6y4p5L8i4y4$3j5#2)9J5k6o6p5%4x3l9`.`.)中, 有如下一句:
      > May contain jump thunks to handle relocation of functions to new addresses.
      也就是说`/INCREMENTAL`选项会让程序多一张跳转表, 以处理函数的重定位. 在Debug模式下就会默认开启此选项.
  - 注: 用windbg调试远程线程, 发现远程线程中`出现地址访问冲突`:
    - 原因1: 出问题的地方试图读取`__security_cookie`
      - 解决: 编译时关闭`/GS`选项, 禁用栈保护.
    - 原因2: 远程线程中试图调用一些不可用的函数, 包括`__CheckForDebuggerJustMyCode`, `_RTC_CheckStackVars`
      - 解决: 禁用`/JMC`选项, `/RTC`选项, 其他的如果是必要使用的动态库函数, 则需要用`LoadLibrary`和`GetProcAddress`获取, 且要确保目标进程已载入相应dll.
  - 注: **shellcode函数中不要用字符串常量, 因为这些字符串总是存在于注入器进程的堆栈上, 不会随着shellcode一起注入到目标进程中, 这样一来shellcode在运行时无从获取这些字符串常量**. 在shellcode中最好是通过传参的方式获取需要用到的字符串常量.

- 在目标进程中开辟新内存, 依次写入:
  - 要注入的dll的文件内容.
  - shellcode.
  - 传给shellcode的参数, 主要是shellcode需要的如下系统api的函数地址.
    - `LdrLoadDll`: 获取注入的dll依赖的dll的内存地址.
    - `LdrGetProcedureAddress`: 获取注入的dll需导入的函数的内存地址.
    - `RtlInitAnsiString` `RtlAnsiStringToUnicodeString` `RtlFreeUnicodeString`: 用以配合上述两个函数, 得到导入函数的内存地址.
    - `NtAllocateVirtualMemory`: 分配内存空间, 以写入要注入的dll的文件内容.

- 创建远程线程, 执行shellcode, 由shellcode载入dll.

## 优点

- 仅枚举进程模块列表并不能发现注入的dll.

![反射式dll注入后目标进程的内存](https://bbs.kanxue.com/upload/attach/202208/873582_WA5PQQ93QAD3H2R.png)

# 镂空(hollowing)

## 概要

- 两种方式:
  - 镂空已有进程模块: 直接修改进程中已有模块的代码节, 注入恶意代码.
  - 先注入后镂空: 注入一个合法dll(拥有合法签名), 然后修改dll入口点处代码为自己想执行的代码.
- 下面只讲先注入dll后镂空的方法.

## 流程

- 首先, 如普通的dll注入, `CreateRemoteThread`创建远程线程, 执行`LoadLibrary`注入一个dll, 不同的是**注入到进程的是一个合法dll(比如system32目录下的dll)**.
- `EnumProcessModules`枚举进程模块, `GetModuleBaseNameA`得到每个模块的名称, 从而找到注入的dll.
- 注入器进程中分配0x1000的内存空间(可用`malloc`或`HeapAlloc`), 然后将找到的dll的PE头部内容读进来.
- 由PE头中的optional头得到目标dll的入口地址, 加上枚举模块时得到的dll的基址, 得到实际dll入口地址, 并用`WriteProcessMemory`向该地址写入shellcode.
  - 注: 通过windbg调试发现, **注入入口地址不一定能成功执行shellcode, 因为DllMain函数可能多次执行.** 如果只想执行一次shellcode, `可把shellcode及其写在dll的末尾对齐空间.`
  - 如同反射式dll注入, 生成shellcode的方法也是在注入器中定义shellcode函数并获取其机器码.
- 创建远程线程, 并以写入shellcode的地址为线程执行地址.

## 优点

- 无需将恶意dll保存在磁盘中, 可躲避静态查杀.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2023-8-11 14:06

被wx_Niatruc编辑

，原因： 修改错误表述

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)
