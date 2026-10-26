
# [原创]新春大奉献_9城名将三国的分析

- 【文章标题】: 新春大奉献_9城名将三国的分析
- 【文章作者】: thomasyzh
- 【作者邮箱】: machinesy@gmail.com
- 【作者QQ号】: 516674080
- 【软件名称】: _9城名将三国的分析
- 【下载地址】: 自己搜索下载
- 【保护方式】: HS
- 【编写语言】: c++
- 【软件介绍】: 第9城市的一个游戏
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

传闻写外挂很赚钱.于是我搞了搞.之前没有灰色产业链的商业积累.身边的朋友也越来越少.基本上外挂这块由于牵扯到钞票的关系,所以很少有人讨论.

其实我一开始的主要目的是像赚钱.也没打算发出来.但是觉得自己现在的编程技术和水平想吃这个票子确实来的有点太累了所以痛下决心,再花半年打基础.至少怎完vmp.然后把写脚本,写插件那些能力再提高一翻翻.再来吃这个票子.

首先向在背后默默鼓励我的商务朋友表示歉意.东西我还是会慢慢完成.就当作练习.只是觉得,忙活了半天没让你们赚到钱实在有点不好意思.

谈技术了:

### 1. hs保护:

我打log分析出了第九城wof的hs只需要调其hs的7号函数

```
HMODULE hEhsvc = NULL;
hEhsvc = (HMODULE)GetModuleHandleA("EHSvc.dll");
if (hEhsvc == INVALID_HANDLE_VALUE)
{
    printf("<<<<<<<<<<<<<<<get EHSvc.dll handle error<<<<<<<<<<<\r\n");
}
else
{
    UnStallAntiDebug  UnStall = NULL;
    UnStall = (UnStallAntiDebug)GetProcAddress( hEhsvc,"7");
    if (UnStall == NULL)
    {
        printf("<<<<<<<<<<<<<<<get EHSvc.dll get fun 7 Error<<<<<<<<<<<\r\n");
        MessageBoxA(NULL,"abc","abc",MB_OK);
    }
    else
    {
        UnStall();
    }
}
```

流程如上代码.

执行后就卸载掉保护了.

### 2. 还原封包

#### 2.1 非动作封包还原.

因为动作封包在包上有个index所以我把这个游戏的封包分成非动作封包和动作封包.具体哪个index的意义,我还没有分析.

打开商店的封包明文

```
BYTE   OpenShop[] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};--这个包完整的说,不因
该算打开的全明文.世纪其头部有个经过另外一个函数算出来的效验值.
```

打开商店的全代码:

```
RtlZeroMemory(&g_packCmd,sizeof(PACK_CMD));
BYTE   OpenShop[] = {0x1b,0x00,0x00,0x00,0x08,0x00,0x01,0x12,0x00,0x00,0x00,0x00};
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
    mov                eax,g_packCmd.dwEncodeTableLen
    push        eax
    mov                eax,g_packCmd.dwEncodeTable
    sub     eax,0x10
    push    eax
    mov     esi,eax
    mov     eax,edi
    mov     eax,g_packCmd.dwCmdLen
    push    eax
    mov                eax,g_packCmd.dwCmdAddr
    push    eax
    mov     eax,Package_Data
    call        eax
    add                esp,0x10
    xor                eax,eax
    mov     edi,esi
table_change:
    add byte ptr [eax+edi],al
    add eax,1
    cmp eax,0x10
    jl  table_change
}
BYTE  SendEncodeTable[0x10] = {0};
DWORD *pdwpointer = (DWORD*)SendEncodeTable;
*pdwpointer = (DWORD)dwHead+0x10;
memcpy((SendEncodeTable+4),(void*)g_packCmd.dwCmdAddr,0x0c);
send(g_send,(char*)SendEncodeTable,0x10,0);
```

这个代码中起重函数`Package_Data`是风暴的加密函数.加密方式我就不多说了.是垃圾.

给出封包加密函数的汇编代码

```
_Package_Data   proc    near
                mov                eax,dword ptr [esp+08h]
                xor     ecx, ecx
                xor     edx, edx
                test    eax, eax
                jbe     short FLA_1
                push    ebx
                push    ebp
                mov     ebp, dword ptr [esp+14h]
                push    esi
                mov     esi, dword ptr [esp+10h]
                push    edi
                mov     edi, dword ptr [esp+20h]
                lea     esp, dword ptr [esp]
FLA_3:
                mov     bl, byte ptr [ecx+ebp]
                xor     byte ptr [edx+esi], bl
                add     ecx, 1
                cmp     ecx, edi
                jb      short FLA_2
                xor     ecx, ecx
FLA_2:
                add     edx, 1
                cmp     edx, eax
                jb      short FLA_3
                pop     edi
                pop     esi
                pop     ebp
                pop     ebx
FLA_1:
                retn
_Package_Data  endp
```

具体数据结构,请自己分析分析.

算封包效验的函数代码

```
_Package_Head  proc    near
                movzx   edx,word ptr [ecx]
                xor           eax,eax
                test                dx,dx
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

具体我也不说了.这个在有index的风暴就要自己去算下效验.比如说攻击.

下边我贴出一个攻击的发包代码

```
BYTE   AttackPack[] = {0xaf,0x05,0x00,0x00,0x10,0x00,0x17,0x08,0x0a,0xa1,0x34,0x00,0x00,0x21,0x24,0x60,0xff,0xff,0xff,0xff};
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
    mov                eax,g_packCmd.dwEncodeTableLen
    push        eax
    mov                eax,g_packCmd.dwEncodeTable
    push    eax
    mov     esi,eax
    mov     edi,eax
    mov     eax,0x14
    push    eax
    mov     eax,g_packCmd.dwCmdAddr
    push    eax
    mov                eax,Package_Data
    call        eax
    add                esp,0x10
    xor     eax,eax
    mov     edi,esi
    change_table:
    add byte ptr [eax+edi],al
    add eax,1
    cmp eax,0x10
    jl  change_table
}
BYTE attackPakc[0x18] = {0};
```
