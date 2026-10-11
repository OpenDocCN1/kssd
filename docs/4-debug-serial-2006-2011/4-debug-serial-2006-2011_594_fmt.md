
# U 盘加密软件的破解

> 原文：[`www.pediy.com/kssd/pediy12/128852.html`](https://www.pediy.com/kssd/pediy12/128852.html)

## 文章信息

- **【文章标题】**: U 盘加密软件的破解  
- **【文章作者】**: `Thomasyzh`  
- **【作者邮箱】**: `machinesy@gmail.com`  
- **【软件名称】**: U 盘超级加密  
- **【软件大小】**: 944  
- **【下载地址】**: 自己搜索下载  
- **【加壳方式】**: `ASProtect 1.2x - 1.3x`  
- **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

很久没发技术上的一些文章了。因为看了下，很多的学习和建树在 `pediy` 上超越我的文章实在太多了，所以一直没有发什么技术性的文章。今年没有太多的机会去研究很底层的东西，大多时候是转向应用了。所以更加多的时候是在学习别人如何很方便的很高效的完成一个任务或者工作。所以没有什么沉淀，也就不好意思发出来了。也就一直没发。

前段时间帮一个人做了一个外包。500 元破解了一个软件，然后改改界面，再写个注册机。用了几个小时帮他怎了，后来他也没继续联系我。引导引导新手吧，就把这个发出做一篇初级破文。

通常不是很恶心的壳，我现在都是带壳调试。其实和本生调试是没有什么区别的。到内存后，都是解开了的。选择用 `od`，`start process` 然后勾取 `hide od` 就 OK。

我没有去逆算法，其实去逆算法也不困难。承包方的需求是，制作注册机，然后修改界面。完美的做法是完全修复和脱去壳，完全替换点资源和软件界面，然后把注册机写出来。我的做法是，暴力破解掉软件功能部分，然后外挂软件功能部分，动态替换掉软件资源。

非专业脱壳人事，会尽量靠近我这样的做法。做 `pe` 修复如果对壳的解密流程不是很熟悉，那么做起来会想很多。

### 1 以试用版进去后观察程序行为

发现点部分功能，会有 `MessageBoxW` 提示 所以 `bp user32!MessageBoxW`，这里我给大家一个温馨提示 IDA 能识别很多静态库函数，而 OD 不能。你可以选择把内存 dump 出来，用 `ida` 和 `od` 对着看，这样更加方便你操作。

点功能：全盘解密 然后 `Ctrl+F9` 一路往上。因为会遇到一个模态对话框，你再把那个模态对话框点了，就能来到代码这里。

```
0046EF19  |.  56            push    esi 
0046EF1A  |.  0F87 C4000000 ja      0046EFE4 
0046EF20  |.  FF2485 EFEF46>jmp     dword ptr [eax*4+46EFEF] 
0046EF27  |>  FF55 14       call    dword ptr [ebp+14]               ;  UDE.0040E3B0; Case 38 ('8') of switch 0046EF13 
0046EF2A  |.  E9 B1000000   jmp     0046EFE0 
0046EF2F  |>  FF55 14       call    dword ptr [ebp+14]               ;  Case 39 ('9') of switch 0046EF13 
0046EF32  |.  E9 A7000000   jmp     0046EFDE 
0046EF37  |>  FF75 0C       push    dword ptr [ebp+C]                ;  Case 3A (':') of switch 0046EF13 
0046EF3A  |>  FF55 14       call    dword ptr [ebp+14]
```

这里是个 switch case 的地方。一般 switch case 都是处理一些固定的流程的逻辑分支，大多书时候也是一个逻辑开始的地方。

所以从这里下去能看见大多数的功能。

F7，跟入 Function

```
0040E3B0  |> /55            push    ebp 
0040E3B1  |. |8BEC          mov     ebp, esp 
0040E3B3  |. |83E4 F8       and     esp, FFFFFFF8 
0040E3B6  |. |6A FF        push    -1 
0040E3B8  |. |68 FD4E4A00  push    004A4EFD 
0040E3BD  |. |64:A1 0000000>mov     eax, dword ptr fs:[0] 
0040E3C3  |. |50           push    eax 
0040E3C4  |. |81EC 58030000 sub     esp, 358 
0040E3CA  |. |A1 18C64C00  mov     eax, dword ptr [4CC618] 
0040E3CF  |. |33C4         xor     eax, esp 
0040E3D1  |. |898424 500300>mov     dword ptr [esp+350], eax 
0040E3D8  |. |53           push    ebx 
0040E3D9  |. |56           push    esi 
0040E3DA  |. |57           push    edi 
0040E3DB  |. |A1 18C64C00  mov     eax, dword ptr [4CC618] 
0040E3E0  |. |33C4         xor     eax, esp 
0040E3E2  |. |50           push    eax 
0040E3E3  |. |8D8424 680300>lea     eax, dword ptr [esp+368] 
0040E3EA  |. |64:A3 0000000>mov     dword ptr fs:[0], eax 
0040E3F0  |. |33F6         xor     esi, esi 
0040E3F2  |. |8D4424 10    lea     eax, dword ptr [esp+10] 
0040E3F6  |. |8BF9         mov     edi, ecx 
0040E3F8  |. |897424 10    mov     dword ptr [esp+10], esi 
0040E3FC  |. |E8 1FAEFFFF  call    00409220                    ;这个函数就是功能验证函数 
0040E401  |. |3BC6         cmp     eax, esi 
0040E403  |. |0F84 96020000 je      0040E69F 
0040E409  |. |837C24 10 58  cmp     dword ptr [esp+10], 58
```

嗯，是的，那个函数就是功能函数，至于如何发现的，我通常是比较关注分支的不同跑法，根据外部环境。实际上很容易分析出。

然后进去读这个函数。

```
seg002:00409220  56                                        push    esi 
seg002:00409221  8B F0                                     mov     esi, eax 
seg002:00409223  E8 28 01 00 00                            call    sub_409350 
seg002:00409228  85 C0                                     test    eax, eax 
seg002:0040922A  74 07                                     jz      short loc_409233 
seg002:0040922C  E8 2F FF FF FF                            call    sub_409160 
seg002:00409231  5E                                        pop     esi 
seg002:00409232  C3                                        retn 

seg002:00409233          ; ------------------------------------------------------------------------- 

seg002:00409233 

seg002:00409233          loc_409233:                       ; CODE XREF: Check_Fun+Aj 
seg002:00409233  52                                        push    edx 
seg002:00409234  57                                        push    edi 
seg002:00409235  83 CA 49                                  or      edx, 49h 
seg002:00409238  83 EA 45                                  sub     edx, 45h 
seg002:0040923B  03 D5                                     add     edx, ebp 
seg002:0040923D  33 D3                                     xor     edx, ebx 
seg002:0040923F  8D 94 39 8D 2A 41 00                       lea     edx, loc_412A8D[ecx+edi] 
seg002:00409246  2B D7                                     sub     edx, edi 
seg002:00409248  2B D1                                     sub     edx, ecx 
seg002:0040924A  FF 32                                     push    dword ptr [edx] 
seg002:0040924C  BA 76 41 44 00                            mov     edx, (offset loc_444171+5) 
seg002:00409251  03 54 24 38                                add     edx, [esp+10h+arg_24] 
seg002:00409255  5A                                        pop     edx 
seg002:00409256  0F B6 FA                                    movzx   edi, dl 
seg002:00409259  33 54 24 08                                  xor     edx, [esp+0Ch+var_4] 
seg002:0040925D  BA 2A 47 47 00                              mov     edx, (offset loc_474729+1) 
seg002:00409262  81 EA 86 1B C0 BE                             sub     edx, 0BEC01B86h 
seg002:00409268  8D 54 4B 4A                                  lea     edx, [ebx+ecx*2+4Ah] 
seg002:0040926C  8D 94 39 50 9F E5 8A                         lea     edx, [ecx+edi-751A60B0h] 
seg002:00409273  2B D7                                     sub     edx, edi 
seg002:00409275  2B D1                                     sub     edx, ecx 
seg002:00409277  8D BC 0F C8 5F 1A 75                          lea     edi, [edi+ecx+751A5FC8h] 
seg002:0040927E  2B F9                                     sub     edi, ecx 
seg002:00409280  03 FA                                     add     edi, edx 
seg002:00409282  0B FF                                     or      edi, edi 
seg002:00409284  90                                        nop 
seg002:00409285  E9 1F 00 00 00                              jmp     loc_4092A9 

seg002:0040928A          ; ------------------------------------------------------------------------- 

seg002:0040928A  81 C5 7E AE A4 09                              add     ebp, 9A4AE7Eh 
seg002:00409290  BE AE E6 45 00                                mov     esi, offset loc_45E6AE
```



### 汇编代码段
```asm
seg002:00409295 83 CE 73                                                                 or      esi, 73h
seg002:00409298 81 C7 10 B5 87 C3                                                     add     edi, 0C387B510h
seg002:0040929E 68 1C 37 5C BC                                                      push    0BC5C371Ch
seg002:004092A3 52                                                                  push    edx
seg002:004092A4 E9 00 00 00 00                                                      jmp     $+5
seg002:004092A9
seg002:004092A9                                                                    loc_4092A9:                   ; CODE XREF: Check_Fun+65j
seg002:004092A9 5F                                                                  pop     edi
seg002:004092AA 5A                                                                  pop     edx
seg002:004092AB E9 93 00 00 00                                                      jmp     loc_409343
```

这个函数 call 了两个函数，进去看函数 1 个一个读。

### 子函数 `sub_409350`
首先进入读函数 `sub_409350`
```asm
seg002:00409350 53                                                                  push    ebx
seg002:00409351 56                                                                  push    esi
seg002:00409352 1B DF                                                               sbb     ebx, edi        ; nop
seg002:00409354 8D 5C 35 BD                                                         lea     ebx, [ebp+esi-43h] ; nop
seg002:00409358 8D 99 04 66 41 00                                                   lea     ebx, loc_416604[ecx]
seg002:0040935E 2B D9                                                               sub     ebx, ecx
seg002:00409360 FF 33                                                               push    dword ptr [ebx]
seg002:00409362 83 DB 19                                                             sbb     ebx, 19h        ; nop
seg002:00409365 BB 2A 1C 44 00                                                      mov     ebx, offset loc_441C2A ; nop
seg002:0040936A 5B                                                                  pop     ebx
seg002:0040936B 0F B6 F3                                                            movzx   esi, bl
seg002:0040936E 03 5C 24 38                                                         add     ebx, [esp+8+arg_2C] ; nop
seg002:00409372 BB 46 88 42 00                                                      mov     ebx, (offset loc_428845+1) ; nop
seg002:00409377 68 EF 0F D2 30                                                      push    30D20FEFh
seg002:0040937C 8D 1C 0B                                                             lea     ebx, [ebx+ecx]   ; nop
seg002:0040937F 5B                                                                  pop     ebx
seg002:00409380 8D B6 29 EF 2D CF                                                   lea     esi, [esi-30D210D7h]
seg002:00409386 03 F3                                                               add     esi, ebx
seg002:00409388 85 F6                                                               test    esi, esi
seg002:0040938A 0F 85 2D 00 00 00                                                   jnz     loc_4093BD      ; nop
seg002:00409390 C1 C6 E7                                                             rol     esi, 0E7h       ; nop
seg002:00409393 BE 06 DC 40 00                                                      mov     esi, (offset loc_40DC01+5) ; nop
seg002:00409398 33 F6                                                               xor     esi, esi
seg002:0040939A 8D 44 47 65                                                         lea     eax, [edi+eax*2+65h] ; nop
seg002:0040939E B8 B2 C7 46 00                                                      mov     eax, (offset loc_46C7B0+2) ; nop
seg002:004093A3 B8 AA 7F 49 00                                                      mov     eax, (offset loc_497FA7+3) ; nop
seg002:004093A8 81 D8 C6 00 3F 15                                                    sbb     eax, 153F00C6h  ; nop
seg002:004093AE 8D 44 0E 1B                                                         lea     eax, [esi+ecx+1Bh]
seg002:004093B2 2B C1                                                               sub     eax, ecx
seg002:004093B4 83 E8 1B                                                             sub     eax, 1Bh
seg002:004093B7 40                                                                  inc     eax
seg002:004093B8 E9 10 00 00 00                                                      jmp     loc_4093CD
seg002:004093BD                        ; ---------------------------------------------------------------------------
seg002:004093BD
seg002:004093BD                                                                    loc_4093BD:                   ; CODE XREF: sub_409350+3Aj
seg002:004093BD B8 02 D3 48 00                                                      mov     eax, (offset loc_48D301+1) ; nop
seg002:004093C2 33 44 24 28                                                         xor     eax, [esp+8+arg_1C] ; nop
seg002:004093C6 33 C0                                                               xor     eax, eax
seg002:004093C8 E9 00 00 00 00                                                      jmp     $+5
seg002:004093CD
seg002:004093CD                                                                    loc_4093CD:                   ; CODE XREF: sub_409350+68j
seg002:004093CD 5E                                                                  pop     esi
seg002:004093CE 5B                                                                  pop     ebx
seg002:004093CF E9 8C 00 00 00                                                      jmp     locret_409460
```

其实它就是加了点花，我写了下把那些该 `nop` 的 `nop` 掉，要不看到恶心。

然后真相就出来了只需要对 `0x4d1cfc` 写 `0x58`,以及对 `0x4d1cec` 写非 `0x2c`,软件就有全功能了。OK。破解功能到这里就完成了。去花后同学们就明白了。

### 注册程序源码
然后我们要写一个注册程序和修改程序算法。

我是直接外挂了程序的注册部分实现自己的注册。全代码如下。

```c
#include "OhYe.h"
#include "md5.h"
#include <tchar.h>

Environ   allEnvir;
char    szFileName[MAX_PATH];

int WINAPI DllMain(_In_ void * _HDllHandle, _In_ unsigned _Reason, _In_opt_ void * _Reserved)
{
    if (_Reason == DLL_PROCESS_ATTACH)
    {
        memset(szFileName,0,MAX_PATH);
        GetModuleFileNameA(NULL,szFileName,MAX_PATH);
        DWORD dwThreadID = 0;
        CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)WorkThread,NULL,NULL,&dwThreadID);
    }
    return TRUE;
}

int WINAPI WorkThread(LPARAM lParam)
{
    InitEnviron(allEnvir);
    Sleep(1000);
    InlineHook(0x407539,(DWORD)SetStrInput,5);
    ChageRes();
    //MessageBoxA(NULL,"debug","debug",MB_OK);
    DWORD* addr1 =(DWORD*) 0x4d2718;
    DWORD* addr2 = (DWORD*)(*(addr1)+0x24);
    //MessageBoxW(NULL,(LPCTSTR)(*(addr2)),(LPCTSTR)(*(addr2)),MB_OK);
    WCHAR *wszDiskData = (WCHAR*)(*(addr2));
    USES_CONVERSION;
    char szoutmd5[33];
    memset(szoutmd5,0,33);
    char* pstring =  W2A(wszDiskData);
    pstring[0] = 'U';
    pstring[1] = 'E';
    pstring[3] = pstring[3]+1;
    pstring[4] = pstring[4]+3;
    pstring[5] = pstring[5]+2;
    char* pstrmd5 = MD5String(pstring,szoutmd5);
    char szBuffer[MAX_PATH];
    GetPrivateProfileStringA("KEY","register","yeallright",szBuffer,MAX_PATH,allEnvir.szIniFileName);
    if (stricmp(szBuffer,pstrmd5)!=0)
    {
        MessageBoxA(NULL,"注册码错误，请从新联系 QQ：516674080 注册使用","注册码错误，请从新联系 QQ：516674080 注册使用",MB_OK);
        //ExitProcess(0);
        return 0;
    }
    DWORD* dwWirteAddr = 0;
    while(TRUE)
    {
        dwWirteAddr = (DWORD*)0x4d1cec;
        *(dwWirteAddr) = 0x2D;
        dwWirteAddr = (DWORD*)0x4d1cfc;
        *(dwWirteAddr) = 0x58;
        VIRTUALIZER_END
    }
    return 0;
}

WCHAR* pwstr;
```



# 代码实现与逻辑分析

以下代码片段展示了 `SetStrInput` 函数及资源处理函数 `ChageRes` 的实现。

```cpp
__declspec(naked) void SetStrInput()
{
  __asm
  {
    pushad
    pushfd
    mov pwstr,ecx
  }
  WritePrivateProfileStringW(_T("KEY"),_T("register"),pwstr,allEnvir.wscIniFileName);
  __asm
  {
    popfd
    popad
    mov eax,0x40753e
    push eax
    mov eax,0x407900
    jmp eax
  }
}

void add()
{
  return;
}

void ChageRes()
{
   HANDLE hFile = CreateFileA(allEnvir.szResFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
   SetFilePointer(hFile,0xd4000,NULL,FILE_BEGIN);
   DWORD dwRead = 0;
   DWORD dwReadLen = 0x10;
   DWORD dwWriteAddr = 0x4d4000;
   char szBuffer[0x11];
   for(int i = 0;i<0x4cff0;i=i+0x10)
   {
     memset(szBuffer,0,0x11);
     ReadFile(hFile,szBuffer,dwReadLen,&dwRead,NULL);    
     GetLastError();
     memcpy((void*)dwWriteAddr,szBuffer,0x10);
     dwWriteAddr=dwWriteAddr +0x10;
   }
}
```

## 注册挂接分析

代码写得比较乱，但核心逻辑如下。在注册部分，直接挂接程序的 `0x00407539` 地址：

```asm
00407530  |.  894C24 18     mov     dword ptr [esp+18], ecx
00407534  |.  E8 11470600   call    0046BC4A
00407539  |.  E8 C2030000   call    00407900                       ;  getstring
0040753E  |.  8BDF          mov     ebx, edi
00407540  |.  E8 3B030000   call    00407880
```

将其替换为上述 `SetStrInput` 函数。这个函数的本意是复用原有的注册框，将用户输入的注册码提取出来，然后写入到一个配置文件中。

## 验证逻辑

验证部分的代码如下：

```cpp
DWORD* addr1 =(DWORD*) 0x4d2718;
DWORD* addr2 = (DWORD*)(*(addr1)+0x24);
//MessageBoxW(NULL,(LPCTSTR)(*(addr2)),(LPCTSTR)(*(addr2)),MB_OK);
WCHAR *wszDiskData = (WCHAR*)(*(addr2));
USES_CONVERSION;
char szoutmd5[33];
memset(szoutmd5,0,33);
char* pstring =  W2A(wszDiskData);
pstring[0] = 'U';
pstring[1] = 'E';
pstring[3] = pstring[3]+1;
pstring[4] = pstring[4]+3;
pstring[5] = pstring[5]+2;
char* pstrmd5 = MD5String(pstring,szoutmd5);
char szBuffer[MAX_PATH];
GetPrivateProfileStringA("KEY","register","yeallright",szBuffer,MAX_PATH,allEnvir.szIniFileName);
if (stricmp(szBuffer,pstrmd5)!=0)
{
  MessageBoxA(NULL,"注册码错误，请从新联系 QQ：516674080 注册使用","注册码错误，请从新联系 QQ：516674080 注册使用",MB_OK);
  //ExitProcess(0);
  return 0;
}
```

这段代码是验证核心。首先通过分析程序验证逻辑中取硬件特征码的固定偏移量：

```cpp
DWORD* addr1 =(DWORD*) 0x4d2718;
DWORD* addr2 = (DWORD*)(*(addr1)+0x24);
```

此处获取的地址即为机器码。经过处理后，与配置文件中取出的注册码进行 `stricmp` 比较即可。（是否有更简单的方法？能否不写代码就生成注册过程？还是觉得这样太麻烦了。）

## 资源替换策略

接下来是处理资源。由于程序加了壳且未完全脱壳，不能在静态阶段替换资源（因为静态时资源尚未解密）。

因此，策略是“能猥琐就猥琐”，直接在程序动态解密完成后，替换内存中的资源。将其替换为使用 `eXeScope` 修改后的资源。代码如下：

```cpp
void ChageRes()
{
   HANDLE hFile = CreateFileA(allEnvir.szResFileName,GENERIC_READ,0,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
   SetFilePointer(hFile,0xd4000,NULL,FILE_BEGIN);
   DWORD dwRead = 0;
   DWORD dwReadLen = 0x10;
   DWORD dwWriteAddr = 0x4d4000;
   char szBuffer[0x11];
   for(int i = 0;i<0x4cff0;i=i+0x10)
   {
     memset(szBuffer,0,0x11);
     ReadFile(hFile,szBuffer,dwReadLen,&dwRead,NULL);    
     GetLastError();
     memcpy((void*)dwWriteAddr,szBuffer,0x10);
     dwWriteAddr=dwWriteAddr +0x10;
   }
}
```

其中 `0x4d4000` 是资源节，文件的 `0xd4000` 也是资源节。基本上资源部分的替换使用 `eXeScope` 即可完成。

另外，将该 `dll` 感染到目标 `pe` 中即可。直接使用 `LoadPE` 就完成了此功能。

--------------------------------------------------------------------------------
## 经验总结

字就码完了。感觉没啥可说的。近来全走的是猥琐路子，没沉淀。最近在做虚拟机还原的一些积累，还做得不好，都还在学习阶段。完了一起分享起。

--------------------------------------------------------------------------------
【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2011 年 02 月 01 日 1:50:33

[`bbs.pediy.com/attachment.php?...1&d=1296497066`](http://bbs.pediy.com/attachment.php?attachmentid=53895&stc=1&d=1296497066)

[`bbs.pediy.com/attachment.php?...1&d=1296497153`](http://bbs.pediy.com/attachment.php?attachmentid=53896&stc=1&d=1296497153)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `OhYe.rar`
|
