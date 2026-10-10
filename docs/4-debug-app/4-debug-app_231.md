

# 截获 QQ2010 明文聊天消息附源码,最新版本 QQ 可用

> 原文：[`www.pediy.com/kssd/pediy11/124169.html`](https://www.pediy.com/kssd/pediy11/124169.html)

声明：  
本次逆向不以破坏 QQ 功能为目的  
本次逆向不以耍流氓为目的  
纯属学习娱乐  
QQ 版本:QQ2010 1, 55, 1870, 0  
工具:OD VS2005  
目的:截获 QQ 明文消息  
定位到的关键代码段（位于 GF.dll 中）:  

代码:

```
3088F210   .  55            push    ebp
3088F211   .  8BEC          mov     ebp, esp
3088F213   .  8B45 10       mov     eax, dword ptr \[ebp+10\]
3088F216   .  8B4D 0C       mov     ecx, dword ptr \[ebp+C\]
3088F219   .  6A 00         push    0
3088F21B   ?  50            push    eax
3088F21C   ?  51            push    ecx
3088F21D   ?  8B4D 08       mov     ecx, dword ptr \[ebp+8\]
3088F220   .  83C1 04       add     ecx, 4
3088F223   ?  E8 28ACFFFF   call    30889E50

```

这部分就消息包和富文本框的交互,所以可以截获到很多东西..比如昵称 发送时间 消息内容 个性签名。  
编写代码做 Hook（额 代码写的超烂已经遭到无数人的鄙视了不要拍了...）:  
代码:

```
#include "stdafx.h"
DWORD dwQCMsg;
LPCWSTR lptext;
VOID QCMain()
{
  lptext = (LPCWSTR)dwQCMsg;

  MessageBox(NULL,lptext,L"QQ",MB\_OK);
  return;
}
VOID AutoSendRequest()
{

  \_\_asm{
        push    0
        push    eax
        push    ecx
        mov     dwQCMsg,ecx
        mov     ecx, dword ptr \[ebp+8\]
        add      ecx,4
        pushad
        call       QCMain
        popad
  }
  \_\_asm{
        mov     eax,0x3088F223
        jmp      eax
  }
}

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul\_reason\_for\_call,
                       LPVOID lpReserved
           )
{
  if (ul\_reason\_for\_call == DLL\_PROCESS\_ATTACH) {
    \_\_asm{
          mov eax,AutoSendRequest
          sub eax,0x3088f21d
          add eax,5
          mov ebx,0x3088f219
          mov word ptr \[ebx\], 0xe9
          mov  dword ptr \[ebx+1\],eax
          mov  dword ptr \[ebx+5\],0x9090909090
      }
    return TRUE;
}

```

注入 DLL 到 QQ 进程,就不写了 QQ 没有这个保护..  
处女文不喜勿拍...  
最后给个编译好的 DLL  
需要手动修改 GF.dll 的 Text 段的读写属性...

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || HookQQ.rar  
 |