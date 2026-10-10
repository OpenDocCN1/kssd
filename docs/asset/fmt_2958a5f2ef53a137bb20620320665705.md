
# 淘宝阿里旺旺 imageMan.dll ActiveX 漏洞真正原因是：栈溢出覆盖 SEH

> 原文：[`www.pediy.com/kssd/pediy12/132676.html`](https://www.pediy.com/kssd/pediy12/132676.html)

最近学习 ActiveX 漏洞。分析阿里旺旺的 `imageMan.dll` 漏洞。

死活跟不到真正发生问题的地方。

跟的情况就是

`widechatomultibyte` 转化 260 个字节

`215C720`

`215C820`

....

然后就到了一个循环的地方。循环干的事--取 `215C270` 一个字符 写入 `215C820` ，不停地循环

这里最终覆盖掉了 `SEH`，同时复制越界导致异常发生。最终导致被利用。

## 代码

```
03984053  |.  8B55 10       mov     edx, dword ptr \[ebp+10\]          ;  FDEA38E1 次数
03984056  |.  59            pop     ecx
03984057  |.  85D2          test    edx, edx
03984059  |.  74 3D         je      short 03984098
0398405B  |.  8B4D 0C       mov     ecx, dword ptr \[ebp+C\]           ;  str1
0398405E >|>  8A01          /mov     al, byte ptr \[ecx\]              ;  取 str 第一个字符
03984060  |.  4A            |dec     edx
03984061  |.  0FB6F0        |movzx   esi, al
03984064  |.  F686 C1AF9903>|test    byte ptr \[esi+399AFC1\], 4
0398406B >|.  8807          |mov     byte ptr \[edi\], al
0398406D  |.  74 13         |je      short 03984082
0398406F  |.  47            |inc     edi
03984070  |.  41            |inc     ecx
03984071  |.  85D2          |test    edx, edx
03984073  |.  74 19         |je      short 0398408E
03984075  |.  8A01          |mov     al, byte ptr \[ecx\]
03984077  |.  4A            |dec     edx
03984078  |.  8807          |mov     byte ptr \[edi\], al
0398407A  |.  47            |inc     edi
0398407B  |.  41            |inc     ecx
0398407C  |.  84C0          |test    al, al
0398407E  |.  74 14         |je      short 03984094
03984080  |.  EB 06         |jmp     short 03984088
03984082  |>  47            |inc     edi
03984083  |.  41            |inc     ecx
03984084  |.  84C0          |test    al, al
03984086  |.  74 10         |je      short 03984098
03984088  |>  85D2          |test    edx, edx
0398408A  |.^ 75 D2         \\jnz     short 0398405E

```

## OD 调试过程

用 `IE7.0` 载入 `exploit.html`，下断点 `bp DispCallFunc`。由于我电脑安装有迅雷，直接断在了 `MediaMon` 模块，下条件断点`[esp]!=2109921E` 即可，也可以在虚拟机干净环境调试。

中断在 `oleaut32.dll` 模块的 `DispCallFunc` 函数，向下查找 `call ecx`。`F4`，`F7` 进入 `imageMan.dll` 模块。

`F8` 单步，记录下发生异常的 `Call`。最终确定异常 `call` 顺序如下：

`call  0418C310      (imageMan.dll)`

|-----------`call  7C94A50E    (ntdll.dll)`

|------------`call   7C923247  (ntdll.dll)`

        |------------`call   7C923282  (ntdll.dll)`

            |---------------`call  ecx  (ntdll.dll)`

最终是 `ecx` 被污染导致溢出被利用。下面追踪 `ecx` 污染的原因。

### 1、call 0418C310 负责 copy 字符

`call 0418C310----------------call 04244027-------覆盖 SEH`

```
04244053  |.  8B55 10       mov     edx, dword ptr \[ebp+10\]          ;  FDEA38E1 次数  
04244056  |.  59            pop     ecx  
04244057  |.  85D2          test    edx, edx  
04244059  |.  74 3D         je      short 04244098  
0424405B  |.  8B4D 0C       mov     ecx, dword ptr \[ebp+C\]           ;  str  
0424405E  |>  8A01          /mov     al, byte ptr \[ecx\]              ;  取 str 第一个字符  
04244060  |.  4A            |dec     edx  
04244061  |.  0FB6F0        |movzx   esi, al  
04244064  |.  F686 C1AF2504>|test    byte ptr \[esi+425AFC1\], 4  
0424406B  |.  8807          |mov     byte ptr \[edi\], al            ;copy 至 edi  
0424406D  |.  74 13         |je      short 04244082  
0424406F  |.  47            |inc     edi  
04244070  |.  41            |inc     ecx  
04244071  |.  85D2          |test    edx, edx  
04244073  |.  74 19         |je      short 0424408E  
04244075  |.  8A01          |mov     al, byte ptr \[ecx\]  
04244077  |.  4A            |dec     edx  
04244078  |.  8807          |mov     byte ptr \[edi\], al  
0424407A  |.  47            |inc     edi  
0424407B  |.  41            |inc     ecx  
0424407C  |.  84C0          |test    al, al  
0424407E  |.  74 14         |je      short 04244094  
04244080  |.  EB 06         |jmp     short 04244088  
```

罪魁祸首就在这里。我们先看下内存以及 `SHE` 链。

`0235C720-0235C820` 共 260 字节 `WideCharToMultiByte`

`0235C824` 开始 copy

`SEH` 链

### 2、copy 越界异常，进入 ntdll.dll 异常处理

异常处理已经被覆盖了。

## IDA 定位

`0422AB7F- 04210000 + 10001000 -1000 = 1001AB7F`

```
int __stdcall sub_1001AB7F(int a1, LPCWSTR lpWideCharStr, int a3)
{
  int result; // eax@2
  const OLECHAR *v4; // eax@3
  CHAR FullPath; // [sp+118h] [bp-208h]@1
  char v6; // [sp+119h] [bp-207h]@1
  __int16 v7; // [sp+219h] [bp-107h]@1
  char v8; // [sp+21Bh] [bp-105h]@1
  _BYTE MultiByteStr[260]; // [sp+10h] [bp-310h]@1
  char *v10; // [sp+114h] [bp-20Ch]@1
  char Str; // [sp+21Ch] [bp-104h]@1
  char v12; // [sp+21Dh] [bp-103h]@1
  __int16 v13; // [sp+31Dh] [bp-3h]@1
  char v14; // [sp+31Fh] [bp-1h]@1
  FullPath = 0;
  memset(&v6, 0, 0x100u);
  v7 = 0;
  v8 = 0;
  WideCharToMultiByte(0, 0, lpWideCharStr, -1, &FullPath, 260, 0, 0);
  MultiByteStr[0] = 0;
  memset(&MultiByteStr[1], 0, 0x100u);
  *(_WORD *)&MultiByteStr[257] = 0;
  MultiByteStr[259] = 0;                                        // 查找路径中的\\
  v10 = strrchr(&FullPath, '\\\\');
  Str = 0;
  memset(&v12, 0, 0x100u);
  v13 = 0;
  v14 = 0;
  mbsnbcpy(&Str, &FullPath, v10 - &FullPath + 1);              //这玩意出事的。复制越界
  sub_100271FE(&Str);
  sub_10018BA1(&FullPath, (int)MultiByteStr);
  sub_1001BFE0(MultiByteStr);
  if ( a3 )
  {
    v4 = (const OLECHAR *)sub_1001C060();
    *(_DWORD *)a3 = SysAllocString(v4);
    sub_1001C040();
    result = 0;
  }
  else
  {
    sub_1001C040();
    result = 0;
  }
  return result;
}
```
