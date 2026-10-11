
# 新春大奉献\_9 城名将三国的分析

> 原文：[`www.pediy.com/kssd/pediy11/107013.html`](https://www.pediy.com/kssd/pediy11/107013.html)

**【文章标题】:** 新春大奉献\_9 城名将三国的分析

**【文章作者】:** `thomasyzh`

**【作者邮箱】:** `machinesy@gmail.com`

**【作者 QQ 号】:** `516674080`

**【软件名称】:** \_9 城名将三国的分析

**【下载地址】:** 自己搜索下载

**【保护方式】:** `HS`

**【编写语言】:** `c++`

**【软件介绍】:** 第 9 城市的一个游戏

**【作者声明】:** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

传闻写外挂很赚钱。于是我搞了搞。之前没有灰色产业链的商业积累。身边的朋友也越来越少。基本上外挂这块由于牵扯到钞票的关系，所以很少有人讨论。

其实我一开始的主要目的是想赚钱。也没打算发出来。但是觉得自己现在的编程技术和水平想吃这个票子确实来得有点太累了。所以痛下决心，再花半年打基础。至少啃完 `vmp`。然后把写脚本，写插件那些能力再提高一翻翻。再来吃这个票子。

首先向在背后默默鼓励我的商务朋友表示歉意。东西我还是会慢慢完成。就当作练习。只是觉得，忙活了半天没让你们赚到钱实在有点不好意思。

### 1. `hs` 保护

我打 log 分析出了第九城 `wof` 的 `hs` 只需要调其 `hs` 的 7 号函数：

```c
HMODULE hEhsvc = NULL;
hEhsvc = (HMODULE)GetModuleHandleA("EHSvc.dll");
if (hEhsvc == INVALID_HANDLE_VALUE)
{
  printf("<<<<<<<<<<<<<<<get EHSvc.dll handle error<<<<<<<<<<<<<\r\n");
}
else
{
  UnStallAntiDebug UnStall = NULL;
  UnStall = (UnStallAntiDebug)GetProcAddress(hEhsvc,"7");
  if (UnStall == NULL)
  {
    printf("<<<<<<<<<<<<<<<get EHSvc.dll get fun 7 Error<<<<<<<<<<<<<\r\n");
    MessageBoxA(NULL,"abc","abc",MB_OK);
  }
  else
  {
    UnStall();
  }
}
```

流程如上代码。

执行后就卸载掉保护了。

### 2. 还原封包

#### 2.1

非动作封包还原。

因为动作封包在包上有个 `index` 所以我把这个游戏的封包分成非动作封包和动作封包。具体哪个 `index` 的意义，我还没有分析。

打开商店的封包明文：

```c
BYTE OpenShop[] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};
```

这个包完整的说，不因而该算打开的全明文。世纪其头部有个经过另外一个函数算出来的效验值。

打开商店的全代码：

```c
RtlZeroMemory(&g_packCmd,sizeof(PACK_CMD));
BYTE OpenShop[] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};
g_packCmd.dwCmdAddr = (DWORD)OpenShop;
g_packCmd.dwCmdLen = 0x0c;
g_packCmd.dwEncodeTableLen = 0x10;
if (dwEncodeTable!=0)
{
  g_packCmd.dwEncodeTable = dwEncodeTable;
}
else
{
  MessageBoxA(NULL,"Dot get table","Dot get table",MB_OK);
  return;
}
__asm
{
  mov    eax,g_packCmd.dwEncodeTableLen
  push   eax
  mov    eax,g_packCmd.dwEncodeTable
  sub    eax,0x10
  push   eax
  mov    esi,eax
  mov    eax,edi
  mov    eax,g_packCmd.dwCmdLen
  push   eax
  mov    eax,g_packCmd.dwCmdAddr
  push   eax
  mov    eax,Package_Data
  call   eax
  add    esp,0x10
  xor    eax,eax
  mov    edi,esi
table_change:
  add byte ptr [eax+edi],al
  add eax,1
  cmp eax,0x10
  jl  table_change
}
BYTE SendEncodeTable[0x10] = {0};
DWORD *pdwpointer = (DWORD*)SendEncodeTable;
*pdwpointer = (DWORD)dwHead+0x10;
memcpy((SendEncodeTable+4),(void*)g_packCmd.dwCmdAddr,0x0c);
send(g_send,(char*)SendEncodeTable,0x10,0);
```

这个代码中起重函数 `Package_Data` 是风暴的加密函数。加密方式我就不多说了。是垃圾。

给出封包加密函数的汇编代码：

```asm
_Package_Data   proc    near
  mov    eax,dword ptr [esp+08h]
  xor    ecx, ecx
  xor    edx, edx
  test   eax, eax
  jbe    short FLA_1
  push   ebx
  push   ebp
  mov    ebp, dword ptr [esp+14h]
  push   esi
  mov    esi, dword ptr [esp+10h]
  push   edi
  mov    edi, dword ptr [esp+20h]
  lea    esp, dword ptr [esp]

FLA_3:
  mov    bl, byte ptr [ecx+ebp]
  xor    byte ptr [edx+esi], bl
  add    ecx, 1
  cmp    ecx, edi
  jb     short FLA_2
  xor    ecx, ecx

FLA_2:
  add    edx, 1
  cmp    edx, eax
  jb     short FLA_3
  pop    edi
  pop    esi
  pop    ebp
  pop    ebx

FLA_1:
  retn
_Package_Data  endp
```

具体数据结构，请自己分析分析。

算封包效验的函数代码：

```asm
_Package_Head  proc    near
   movzx   edx,word ptr [ecx]
   xor     eax,eax
   test    dx,dx
   jbe     short FLA_1
   movzx   edx, dx
   push    esi
   mov     edi, edi

FLA_2:
   movzx   esi, byte ptr [ecx]
   add     eax, esi
   add     ecx, 1
   sub     edx, 1
   jnz     short FLA_2
   pop     esi

FLA_1:
   retn
_Package_Head endp
```

具体我也不说了。这个在有 `index` 的风暴就要自己去算下效验。比如说攻击。

下边我贴出一个攻击的发包代码：

```c
BYTE AttackPack[] = {0xaf,0x05,0x00,0x00,0x10,0x00,0x17,0x08,0x0a,0xa1,0x34,0x00,0x00,0x21,0x24,0x60,0xff,0xff,0xff,0xff};
g_packCmd.dwCmdAddr = (DWORD)AttackPack;
g_packCmd.dwCmdLen = 0x14;
g_packCmd.dwEncodeTableLen = 0x10;
if (g_packIndex == 0)
{
  MessageBoxA(NULL,"packIndex error","packIndex error",MB_OK);
  return;
}
if (dwEncodeTable!=0)
{
  g_packCmd.dwEncodeTable = dwEncodeTable;
}
else
{
  MessageBoxA(NULL,"Dot get table","Dot get table",MB_OK);
  return;
}
__asm
{
  lea  ecx,AttackPack
  mov  eax,g_packIndex
  mov  eax,dword ptr [eax+0x04]
  mov  dword ptr [ecx+0x09],eax
  add  ecx,4
  mov  eax,Package_Head
  call eax
  lea  ecx,AttackPack
  mov  dword ptr [ecx],eax
}
__asm
{
  mov    eax,g_packCmd.dwEncodeTableLen
  push   eax
  mov    eax,g_packCmd.dwEncodeTable
  push   eax
  mov    esi,eax
  mov    edi,eax
  mov    eax,0x14
  push   eax
  mov    eax,g_packCmd.dwCmdAddr
  push   eax
  mov    eax,Package_Data
  call   eax
  add    esp,0x10
  xor    eax,eax
  mov    edi,esi
change_table:
  add byte ptr [eax+edi],al
  add eax,1
  cmp eax,0x10
  jl  change_table
}
BYTE attackPakc[0x18] = {0};
DWORD* dwordPointer = (DWORD*)attackPakc;
*dwordPointer = (DWORD)dwHead+0x18;
memcpy((attackPakc+0x04),(void*)g_packCmd.dwCmdAddr,0x14);
send(g_send,(char*)attackPakc,0x18,0);
//index+14
__asm
{
  mov eax,g_packIndex
  mov ebx,dword ptr [eax+0x04]
  add ebx,0x14
  mov dword ptr [eax+0x04],ebx
}
return;
```

具体过程就是：

1. 取 `index`
2. 算效验
3. 加密
4. 加个包长度头
5. 发包

到这里，有心的人可以作脱机挂了。只是还由很多的过程要分析。只能说还原了封包。重要的还是分析后边的协议。不过分析协议不复杂。

### 3. 内挂

下边谈内挂。

我一开是想做变态的全屏秒杀挂。难免要找一些数据结构。

具体我分析到如下结果了。

其主程序很多时候使用一片内存作为参数传递，那我们就适时的在一些特定的点动态的修改其内存。实现修改攻击等数据。

我找到了攻击坐标。

实现了全屏打。但是一旦攻击一下。其就会切换验证模式。然后就失效了。总的来说，就是可以在单人模式下。攻击一次。

我的修改和我的修改的地方的特征地址大概如下代码。具体游戏版本可能有更新。不过按照我的思路，修改那片内存，一样可以实现很多功能。不过一定要在适当的时候修改。

代码如下。特征码在注释片断。

```asm
//attack x
/*
005CF942  |.  51           push    ecx
005CF943  |.  8B4C24 78    mov     ecx, dword ptr [esp+78]
005CF947  |.  50           push    eax
005CF948  |.  51           push    ecx
005CF949  |.  8BCF         mov     ecx, edi
005CF94B  |.  E8 B071E6FF  call    00436B00
005CF950  |.  8B5424 3C    mov     edx, dword ptr [esp+3C]
005CF954  |.  52           push    edx
005CF955  |.  894424 54    mov     dword ptr [esp+54], eax
005CF959  |.  C78424 C4000>mov     dword ptr [esp+C4], -1
005CF964  |.  E8 A7F4FFFF  call    005CEE10
005CF969  |.  8BF8         mov     edi, eax
005CF96B  |.  83C4 04      add     esp, 4
*/
/*
if 0x7ff73fa0
```
