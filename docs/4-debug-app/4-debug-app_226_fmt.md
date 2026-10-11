
# Warcraft III 的一个 MapHack 的研究及一个 Anti-MapHack 方案

> 原文：[`www.pediy.com/kssd/pediy10/77492.html`](https://www.pediy.com/kssd/pediy10/77492.html)

其实我是想研究 `war3.exe` 的自我保护方式的，但是直接从游戏下手又找不到下手的地方，于是就来分析外挂，外挂里面一定有关于游戏的保护方式的信息。

这里使用的游戏版本是 `1.20e`

## `Wc3piderror120ae.exe` 的分析

`Wc3piderror120ae.exe` 是一个补丁，如果使用 `MapHack` 时出现 `pid` 错误时使用这个程序对 `game.dll` 进行补丁，即可以消除该错误。

### 修改的主过程：

代码:

```
00404D6B   .  FF15 90104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileOpen
00404D71   .  8B7E 34       mov     edi, dword ptr [esi+34]
00404D74   .  83FF 06       cmp     edi, 6
00404D77   .  72 06         jb      short 00404D7F
00404D79   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404D7F   >  6A 01         push    1
00404D81   .  8B46 48       mov     eax, dword ptr [esi+48]
00404D84   .  8B0CB8        mov     ecx, dword ptr [eax+edi*4]
00404D87   .  51            push    ecx                      ;  跟踪时会发现这里的偏移是 995E
00404D88   .  FF15 88104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileSeek
00404D8E   .  395D 0C       cmp     dword ptr [ebp+C], ebx
00404D91   .  8B7E 34       mov     edi, dword ptr [esi+34]
00404D94   .  75 15         jnz     short 00404DAB
00404D96   .  83FF 06       cmp     edi, 6
00404D99   .  72 06         jb      short 00404DA1
00404D9B   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404DA1   >  6A 01         push    1
00404DA3   .  8B56 64       mov     edx, dword ptr [esi+64]
00404DA6   .  03D7          add     edx, edi
00404DA8   .  52            push    edx
00404DA9   .  EB 16         jmp     short 00404DC1
00404DAB   >  83FF 06       cmp     edi, 6
00404DAE   .  72 06         jb      short 00404DB6
00404DB0   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404DB6   >  6A 01         push    1
00404DB8   .  8B86 80000000 mov     eax, dword ptr [esi+80]
00404DBE   .  03C7          add     eax, edi
00404DC0   .  50            push    eax
00404DC1   >  6A 01         push    1
00404DC3   .  FF15 0C104000 call    dword ptr [<&MSVBVM60.__vbaPut>;  MSVBVM60.__vbaPut3
00404DC9   .  8B4D DC       mov     ecx, dword ptr [ebp-24]
00404DCC   .  FF15 58104000 call    dword ptr [<&MSVBVM60.__vbaI2I>;  MSVBVM60.__vbaI2I4
00404DD2   .  50            push    eax
00404DD3   .  FF15 4C104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileClose
```

这段代码的任务是把文件中偏移 `0x995E` 处的数据修改为 `0xEB`

### 恢复的主过程

代码:

```
00404D6B   .  FF15 90104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileOpen
00404D71   .  8B7E 34       mov     edi, dword ptr [esi+34]
00404D74   .  83FF 06       cmp     edi, 6
00404D77   .  72 06         jb      short 00404D7F
00404D79   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404D7F   >  6A 01         push    1
00404D81   .  8B46 48       mov     eax, dword ptr [esi+48]
00404D84   .  8B0CB8        mov     ecx, dword ptr [eax+edi*4]
00404D87   .  51            push    ecx                      ;  跟踪时会发现这里的偏移是 995E
00404D88   .  FF15 88104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileSeek
00404D8E   .  395D 0C       cmp     dword ptr [ebp+C], ebx
00404D91   .  8B7E 34       mov     edi, dword ptr [esi+34]
00404D94   .  75 15         jnz     short 00404DAB
00404D96   .  83FF 06       cmp     edi, 6
00404D99   .  72 06         jb      short 00404DA1
00404D9B   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404DA1   >  6A 01         push    1
00404DA3   .  8B56 64       mov     edx, dword ptr [esi+64]
00404DA6   .  03D7          add     edx, edi
00404DA8   .  52            push    edx
00404DA9   .  EB 16         jmp     short 00404DC1
00404DAB   >  83FF 06       cmp     edi, 6
00404DAE   .  72 06         jb      short 00404DB6
00404DB0   .  FF15 54104000 call    dword ptr [<&MSVBVM60.__vbaGen>;  MSVBVM60.__vbaGenerateBoundsError
00404DB6   >  6A 01         push    1
00404DB8   .  8B86 80000000 mov     eax, dword ptr [esi+80]
00404DBE   .  03C7          add     eax, edi
00404DC0   .  50            push    eax
00404DC1   >  6A 01         push    1
00404DC3   .  FF15 0C104000 call    dword ptr [<&MSVBVM60.__vbaPut>;  MSVBVM60.__vbaPut3
00404DC9   .  8B4D DC       mov     ecx, dword ptr [ebp-24]
00404DCC   .  FF15 58104000 call    dword ptr [<&MSVBVM60.__vbaI2I>;  MSVBVM60.__vbaI2I4
00404DD2   .  50            push    eax
00404DD3   .  FF15 4C104000 call    dword ptr [<&MSVBVM60.__vbaFil>;  MSVBVM60.__vbaFileClose
```

这段代码的任务是把文件中偏移 `0x995E` 处的数据修改为 `0x74`

到 `Game.DLL` 中来找找被修改的代码是什么样子的，被修改的数据位于：

代码:



```
6F009880  /$  55                    push    ebp
6F009881  |.  8BEC                 mov     ebp, esp
6F009883  |.  81EC 20020000        sub     esp, 220
6F009889  |.  53                   push    ebx
6F00988A  |.  33DB                 xor     ebx, ebx
6F00988C  |.  56                   push    esi
6F00988D  |.  57                   push    edi
6F00988E  |.  885D F8              mov     byte ptr [ebp-8], bl
6F009891  |.  885D F9              mov     byte ptr [ebp-7], bl
6F009894  |.  885D FA              mov     byte ptr [ebp-6], bl
6F009897  |.  885D FB              mov     byte ptr [ebp-5], bl
6F00989A  |.  885D FC              mov     byte ptr [ebp-4], bl
6F00989D  |.  C645 FD 01           mov     byte ptr [ebp-3], 1
6F0098A1  |.  FF15 9050706F        call    dword ptr [<&KERNEL32.GetCurrent>; \GetCurrentProcess
6F0098A7  |.  68 6C7A7E6F          push    6F7E7A6C                     ; /FileName = "advapi32.dll"
6F0098AC  |.  8945 E0              mov     [local.8], eax               ; |
6F0098AF  |.  895D F4              mov     [local.3], ebx               ; |
6F0098B2  |.  33FF                 xor     edi, edi                     ; |
6F0098B4  |.  FF15 7850706F        call    dword ptr [<&KERNEL32.LoadLibrar>; \LoadLibraryA
6F0098BA  |.  8BF0                 mov     esi, eax
6F0098BC  |.  3BF3                 cmp     esi, ebx
6F0098BE  |.  0F84 C3000000        je      6F009987
6F0098C4  |.  68 507A7E6F          push    6F7E7A50                     ;  ASCII "AllocateAndInitializeSid"
6F0098C9  |.  8BD6                 mov     edx, esi
6F0098CB  |.  8D4D E8              lea     ecx, [local.6]
6F0098CE  |.  E8 CD000000          call    6F0099A0
6F0098D3  |.  85C0                 test    eax, eax
6F0098D5  |.  0F84 A5000000        je      6F009980
6F0098DB  |.  68 407A7E6F          push    6F7E7A40                     ;  ASCII "InitializeAcl"
6F0098E0  |.  8BD6                 mov     edx, esi
6F0098E2  |.  8D4D E4              lea     ecx, [local.7]
6F0098E5  |.  E8 B6000000          call    6F0099A0
6F0098EA  |.  85C0                 test    eax, eax
6F0098EC  |.  0F84 8E000000        je      6F009980
6F0098F2  |.  68 2C7A7E6F          push    6F7E7A2C                     ;  ASCII "AddAccessDeniedAce"
6F0098F7  |.  8BD6                 mov     edx, esi
6F0098F9  |.  8D4D F0              lea     ecx, [local.4]
6F0098FC  |.  E8 9F000000          call    6F0099A0
6F009901  |.  85C0                 test    eax, eax
6F009903  |.  74 7B                je      short 6F009980
6F009905  |.  68 1C7A7E6F          push    6F7E7A1C                     ;  ASCII "SetSecurityInfo"
6F00990A  |.  8BD6                 mov     edx, esi
6F00990C  |.  8D4D EC              lea     ecx, [local.5]
6F00990F  |.  E8 8C000000          call    6F0099A0
6F009914  |.  85C0                 test    eax, eax
6F009916  |.  74 68                je      short 6F009980
6F009918  |.  8D45 F4              lea     eax, [local.3]
6F00991B  |.  50                   push    eax
6F00991C  |.  53                   push    ebx
6F00991D  |.  53                   push    ebx
6F00991E  |.  53                   push    ebx
6F00991F  |.  53                   push    ebx
6F009920  |.  53                   push    ebx
6F009921  |.  53                   push    ebx
6F009922  |.  53                   push    ebx
6F009923  |.  53                   push    ebx
6F009924  |.  6A 01                push    1
6F009926  |.  8D4D F8              lea     ecx, [local.2]
6F009929  |.  51                   push    ecx
6F00992A  |.  FF55 E8              call    [local.6]
6F00992D  |.  85C0                 test    eax, eax
6F00992F  |.  74 4F                je      short 6F009980
6F009931  |.  6A 02                push    2
6F009933  |.  68 00020000          push    200
6F009938  |.  8D95 E0FDFFFF        lea     edx, [local.136]
6F00993E  |.  52                   push    edx
6F00993F  |.  FF55 E4              call    [local.7]
6F009942  |.  85C0                 test    eax, eax
6F009944  |.  74 3A                je      short 6F009980
6F009946  |.  8B45 F4              mov     eax, [local.3]
6F009949  |.  50                   push    eax
6F00994A  |.  68 FEFF1FF0          push    F01FFFFE
6F00994F  |.  6A 02                push    2
6F009951  |.  8D8D E0FDFFFF        lea     ecx, [local.136]
6F009957  |.  51                   push    ecx
6F009958  |.  FF55 F0              call    [local.4]
6F00995B  |.  85C0                 test    eax, eax
6F00995D  |.  74 21                je      short 6F009980
6F00995F  |.  8B45 E0              mov     eax, [local.8]
6F009962  |.  53                   push    ebx
6F009963  |.  8D95 E0FDFFFF        lea     edx, [local.136]
6F009969  |.  52                   push    edx
6F00996A  |.  53                   push    ebx
6F00996B  |.  53                   push    ebx
6F00996C  |.  68 04000080          push    80000004
6F009971  |.  6A 06                push    6
6F009973  |.  50                   push    eax
6F009974  |.  FF55 EC              call    [local.5]
6F009977  |.  85C0                 test    eax, eax
6F009979  |.  75 05                jnz     short 6F009980
6F00997B  |.  BF 01000000          mov     edi, 1
6F009980  |>  56                   push    esi                        ; /hLibModule
6F009981  |.  FF15 7450706F        call    dword ptr [<&KERNEL32.FreeLibrar>; \FreeLibrary
6F009987  |>  8B45 F4              mov     eax, [local.3]
6F00998A  |.  3BC3                 cmp     eax, ebx
6F00998C  |.  74 07                je      short 6F009995
6F00998E  |.  50                   push    eax                        ; /pSID
6F00998F  |.  FF15 0050706F        call    dword ptr [<&ADVAPI32.FreeSid>]  ; \FreeSid
6F009995  |>  8BC7                 mov     eax, edi
6F009997  |.  5F                   pop     edi
6F009998  |.  5E                   pop     esi
6F009999  |.  5B                   pop     ebx
6F00999A  |.  8BE5                 mov     esp, ebp
6F00999C  |.  5D                   pop     ebp
6F00999D  \.  C3                   retn
```

这段代码等价于：

```
BOOL getAddr(DWORD *pProc, HMODULE hModule, LPCSTR lpProcName)
{
  *pProc = GetProcAddress(hModule, lpProcName);
  if (nProc)
    return 1;
  else
    return 0;
}
BOOL fun()
{
  SID_IDENTIFIER_AUTHORITY IdentifierAuthority;
  SID sid = 0;
  DWORD nRet = 0;
  ACL acl;
  HANDLE hCurrentProcess = GetCurrentProcess();
  DWORD hLibModule = LoadLibraryA("advapi32.dll");
  LPVOID AllocateAndInitializeSid, InitializeAcl, AddAccessDeniedAce, SetSecurityInfo;

  IdentifierAuthority.value[0] = 1;
  IdentifierAuthority.value[1] = 0;
  IdentifierAuthority.value[2] = 0;
  IdentifierAuthority.value[3] = 0;
  IdentifierAuthority.value[4] = 0;
  IdentifierAuthority.value[5] = 0;
  if (temp)
  {
    if (getAddr(&AllocateAndInitializeSid, hLibModule, "AllocateAndInitializeSid") &&
      getAddr(&InitializeAcl, hLibModule, "InitializeAcl") &&
      getAddr(&AddAccessDeniedAce, hLibModule, "AddAccessDeniedAce") &&
      getAddr(&SetSecurityInfo, hLibModule, "SetSecurityInfo"))
    {
      if (AllocateAndInitializeSid(&IdentifierAuthority, 1, 0, 0, 0, 0, 0, 0, 0, 0, &sid) &&
          InitializeAcl(&acl, 0x200, 2) &&
          AddAccessDeniedAce(&acl, 2, 0xF01FFFFE, &sid) &&
          SetSecurityInfo(hCurrentProcess, 6, 0x80000004, NULL, NULL, &acl, NULL))
      {
        nRet = 1;
      }
    }
    FreeLibrary(hLibModule);
  }
  if (local3)
  {
    FreeSid(sid);
  }
  return nRet;
}
```

在这里就可以看出程序把 `0x995E` 处修改为 `EB` 的目的是跳过对 `SetSecurityInfo` 的调用，如此用 `OpenProcess` 打开游戏进程就不需要很高的权限了。

==================

### `mh120e5.exe` 的分析

程序全名为 `MapHack`，可以去掉游戏中的战争迷雾、可以看隐形单位、可以看对方英雄技能...

程序在初始化时执行了等价于以下代码的过程：

```
BOOL WINAPI dbgPrivilegeSet(PCHAR psName)
{
  HANDLE hToken;
  TOKEN_PRIVILEGES tp;
  BOOL fOK = FALSE;
  
  if (psName && OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES, &hToken))
  {
    if (LookupPrivilegeValue(NULL, psName, &(tp.Privileges->Luid)))
    {
      tp.Privileges->Attributes = SE_PRIVILEGE_ENABLED;
      tp.PrivilegeCount = 1;
      fOK = AdjustTokenPrivileges(hToken, FALSE, &tp, 0, NULL, NULL);
    }
  }
  
  return fOK;
}
init()
{
....
dbgPrivilegeSet("SeDebugPrivilege"); //提权
....
}
```

函数 `dbgPrivilegeSet` 的作用是改变当前进程的特权。`MapHack` 调用这个过程改变自己的特权，以获得足够的权限来打开游戏进程。

`MapHack` 实现其功能的代码在 `00414D90` 和 `00415D60` 两处的函数里，函数 `00414D90` 里根据游戏版本初始化了补丁数据，函数 `00415D60` 将补丁数据写入游戏进程。

分析 `00414D90` 来获取补丁数据是不切实际的，可以换个角度考虑－补丁数据是要调用 `WriteProcessMemory` 来写入目标程序的，而这个函数的参数里包含了补丁数据和补丁数组，我们只需要截获所有对这个函数的调用并记录参数即可。

用以下脚本实现收集补丁数据的功能：

```



### 脚本示例

```
\=============Script Begin================
var addr
var byte

eob lableb

lableb:
mov addr, esp
add addr, 8
mov addr, [addr]
mov byte, esp
add byte, 0c
mov byte, [byte]
mov byte, [byte]
and byte, FF
add byte, ";"
add addr, ", (LPVOID)(&var), 1, NULL);"
wrta "F:\\mh120e5\\out.txt", "var = 0x"+byte
wrta "F:\\mh120e5\\out.txt", "WriteProcessMemory(g\_hProcess, (LPVOID)0x"+addr

run
\=============Script End==================
```

我们可以用获取到的补丁数据来做一个自己的 `MapHack` 了。

### Anti-Maphack 原理与实现

`\=============anti-maphack ^_^`

想屏蔽掉 `maphack` 的功能只要每隔一段时间就把游戏数据恢复一下即可。使用之前获取补丁数据的方法同样可以获取恢复数据。

**恢复数据：**

```
while (1)
{
  if (检测到关键代码被修改)
    将恢复数据写回游戏内存，覆盖掉补丁数据
  Sleep(xxxx);
}
```

但是这个方法也并不是什么好方法，如果 `MH` 用更高的频率往游戏里写补丁数据的话就没辙了。(代码见 `anti-mh_1` 文件夹)

其实 `anti-maphack` 中心思想就是不让他把关键数据补丁掉，如果我们把关键数据移位了呢？那么他补丁的数据就会写到错误的地方，导致程序崩溃，这也算达到了我们的目的了。

我这里使用的方法是将一处 `MapHack` 要补丁代码拷贝到别的地方，然后从原来的地方跳转过去执行。如此，当 `MapHack` 补丁的时候就会把跳转语句覆盖掉，如此就会导致程序执行出错，也算是达到目的了。

这里选择 `6F4069F7` 处来做个演示，在 `6F704BAB` 有一段足够长的空闲内存。

从 `6F4069F7` 跳转到 `6F704BAB` 的指令：`E9 AF E1 2F 00`

原始指令：`66 8B 50 3C 66 89 55 E4`

从 `6F704BB3` 跳转到 `6F4069FF` 的指令：`E9 47 1E D0 FF`

(详细代码见 `anti-mh_2` 文件夹)

开启 `MapHack` 功能前运行 `anti-mh.exe`，执行完毕后进入游戏一切正常，然后开启 `MapHack` 功能，游戏崩溃，试验成功 :)

当然这个演示还是很简单的，如果 `MapHack` 想绕过这个防御只需要读 `6F4069F7` 处的代码，然后根据该处的代码定位到关键数据所在地。

不过我们可以设计一套算法来确定关键代码要移动到哪里去，然后设计一个 `locate` 函数来专门负责定位到这些代码，关键代码原来的地方就改成对 `locate` 的调用，然后加强对 `locate` 函数的保护，即可大大增加 `MapHack` 的制作难度![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

### 附件

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `1.rar` [请到论坛下载：[`bbs.pediy.com/showthread.php?t=77492`](http://bbs.pediy.com/showthread.php?t=77492)] |
