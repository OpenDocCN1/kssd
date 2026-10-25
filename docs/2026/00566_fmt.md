
# [原创]利用 windbg 脚本动态调试代码

利用脚本可以**自动设置断点**，并**暂停之后处理事务**是核心，不用一对一检查。

## 1. 编写测试代码

先编写代码 `windbgTest.cpp`，编译后一定要保留 `windbgTest.pdb` 文件，代码如下：

```cpp
#include "stdafx.h"

int main(int argc, char* argv[])
{
        char buf[][5]={
                {"buf1"},
                {"buf2"},
                {"buf3"}};
        for(int i=0;i<3;i++)
        {
                printf("%s\n",buf[i]); //要检查输出内容
        }
        return 0;
}
```

## 2. 编写 windbg 脚本

编写 windbg 脚本 `SRCIPT_WINDBGTEST_DEBUG.TXT`，内容如下：

```text
$$===================================================================
$$    $$><D:\!Prog\winDBG\windbgTest\SRCIPT_WINDBGTEST_DEBUG.txt
$$    测试结束后 .logclose
$$===================================================================
.logopen D:\!Prog\winDBG\windbgTest\windbgTest.log
bp windbgTest!printf "da edx;g"  <----暂停后要做的事务
```

## 3. 运行调试

打开 windbg，用户模式打开 `windbgTest.exe`，命令窗口中输入 `$$><D:\!Prog\winDBG\windbgTest\SRCIPT_WINDBGTEST_DEBUG.txt`，然后正常运行就可以自动记录 `printf` 函数中的内容。最后不要忘记**关闭日志**文件，在 windbg 命令窗口中输入 `.logclose`。

```text
CommandLine: D:\!Prog\winDBG\windbgTest\Debug\windbgTest.exe

Symbol search path is: D:\WinDDK\Symbols_Full
SRV*D:\WinDDK\Symbols_Dynamic\* 479K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3`.

Executable search path is:

ModLoad: 00400000 00426000   windbgTest.exe
ModLoad: 7c920000 7c9b6000   ntdll.dll
ModLoad: 7c800000 7c91e000   C:\WINDOWS\system32\kernel32.dll

(f30.598): Break instruction exception - code 80000003 (first chance)
eax=00241eb4 ebx=7ffd6000 ecx=00000001 edx=00000002 esi=00241f48 edi=00241eb4
eip=7c92120e esp=0012fb20 ebp=0012fc94 iopl=0         nv up ei pl nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202

ntdll!DbgBreakPoint:
7c92120e cc              int     3

0:000> $$><D:\!Prog\winDBG\windbgTest\SRCIPT_WINDBGTEST_DEBUG.txt <---手动输入
Opened log file 'D:\!Prog\winDBG\windbgTest\windbgTest.log'
*** WARNING: Unable to verify checksum for windbgTest.exe

0:000> g                     <---自动记录printf的内容
0012ff70  "buf1"
0012ff75  "buf2"
0012ff7a  "buf3"

eax=00000000 ebx=00000000 ecx=7c800000 edx=7c99e120 esi=7c92de6e edi=00000000
eip=7c92e514 esp=0012fe54 ebp=0012ff50 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

ntdll!KiFastSystemCallRet:
7c92e514 c3              ret

0:000> .logclose
Closing open log file D:\!Prog\winDBG\windbgTest\windbgTest.log
```

## 4. 验证结果

打开 log 文件验证结果：

```text
Opened log file 'D:\!Prog\winDBG\windbgTest\windbgTest.log'
*** WARNING: Unable to verify checksum for windbgTest.exe

0:000> g
0012ff70  "buf1"
0012ff75  "buf2"
0012ff7a  "buf3"

eax=00000000 ebx=00000000 ecx=7c800000 edx=7c99e120 esi=7c92de6e edi=00000000
eip=7c92e514 esp=0012fe54 ebp=0012ff50 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246

ntdll!KiFastSystemCallRet:
7c92e514 c3              ret

0:000> .logclose
Closing open log file D:\!Prog\winDBG\windbgTest\windbgTest.log
```

下载测试代码包括脚本 `31eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6A6L8r3g2K6i4K6u0W2j5$3&6T1L8r3!0Y4M7#2)9J5k6h3y4G2L8g2)9J5c8X3&6D9M7$3!0X3N6q4)9J5c8Y4N6A6L8X3c8T1k6#2c8W2M7%4c8Q4x3X3g2J5j5i4t1`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
