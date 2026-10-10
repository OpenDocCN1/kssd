

# VM 初探 讲述 VM 原理

> 原文：[`www.pediy.com/kssd/pediy11/124732.html`](https://www.pediy.com/kssd/pediy11/124732.html)

小弟最近看了点 vm   下面是一些心得体会  跟大家分享  
通俗的讲 vm 就是自己设计一套指令集  和一套解析这个虚拟指令集的引擎   把原本的 386 代码翻译成你自己的虚拟指令  然后调用你的虚拟引擎来执行这些虚拟指令。。。  
这样说不知道合适不？   欢迎一起参与讨论  
本文的目的是告诉大家虚拟机并不可怕    其实我们每个人都设计一套虚拟指令   这将是 Cracker 的噩梦。  
这里感谢  jackozoo 版主提供的一个 vm  CrackMe 源码 作为我的参考资料  
感谢 老罗的代码着色器  
  
下面是最简单的一个 vm 例子  只虚拟了  push 和 call 指令  

代码:

```
/\*
    VM 初探
    作者:blueapplez
    QQ：1242721xx
    Email:blueapple1987@163.com
    http://bbs.pediy.com/
\*/

#include "stdafx.h"
#include "windows.h"

/\* 下面是虚拟指令 我们只模拟了 2 条指令 \*/

//push 0x12345678  push 一个 4 字节的数
#define vPushData    0x10

//call 0x12345678  call 一个 4 字节的地址
#define vCall        0x12

//结束符
#define vEnd        0xff

//一个字符串
char \*str = "Hello World";

/\*
    这是我们构造的虚拟指令, 数据还不  在 mian 里面我们进行了修改
    push 0
    push 0
    push offset str  ;把字符串的地址入栈
    push 0
    call MessageBoxA ;
\*/
BYTE bVmData\[\] = {    vPushData,    0x00, 0x00, 0x00,0x00, 
                    vPushData,    0x00, 0x00, 0x00,0x00,
                    vPushData,    0x00, 0x00, 0x00, 0x00,
                    vPushData,    0x00, 0x00, 0x00,0x00,
                    vCall,    0x00, 0x00, 0x00, 0x00,
                    vEnd};

//这就是简单的虚拟引擎了
\_declspec(naked) void  VM(PVOID pvmData)
{
    \_\_asm
    {
        //取 vCode 地址放入 ecx
        mov ecx, dword ptr ss:\[esp+4\]
\_\_vstart:
        //取第一个字节到 al 中
        mov al, byte ptr ds:\[ecx\]
        cmp al, vPushData
        je    \_\_vPushData
        cmp al, vCall
        je    \_\_vCall
        cmp al, vEnd
        je \_\_vEnd
        int 3
\_\_vPushData:
        inc ecx
        mov edx, dword ptr ds:\[ecx\]
        push edx
        add ecx, 4
        jmp \_\_vstart
\_\_vCall:
        inc ecx
        mov edx, dword ptr ds:\[ecx\]
        call edx
        add ecx, 4
        jmp \_\_vstart
\_\_vEnd:
        ret
    }
}

int main(int argc, char\* argv\[\])
{
    //修改虚拟指令的数据
    \*(DWORD \*)(bVmData+10 + 1) = (DWORD)str;
    \*(DWORD \*)(bVmData+20 + 1) = (DWORD)MessageBoxA;
    
    //执行虚拟指令
    VM(bVmData);
    return 0;
}

```