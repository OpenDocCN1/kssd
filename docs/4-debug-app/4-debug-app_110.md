

# 某游戏密码加密算法逆向

> 原文：[`www.pediy.com/kssd/pediy10/102174.html`](https://www.pediy.com/kssd/pediy10/102174.html)

某游戏密码加密算法逆向输入一位密码，通过跟 timegettime 得到的时间 密码的 ascii 码 加密 得到一个 word 数，然后在用 3des 算法加密生成 64 位的字符，  
每输入一次密码，在原加密基础上继续加密，当密码输入完成之后，登录游戏  
64 字节密码在通过 3des 解密 然后跟 IP 地址 xor 之后发送密码，太疯狂了。  
为了防止木马盗号，竟然用这样的手段加密密码。  
  
不过想资号还是很容易的，在输入密码的地方直接 hook，得到他寄存器里面的密码就可以了  
  
\_declspec(naked)  MyHooKGetPassFun()  
{  
        \_asm  
        {  
                pushad  
                        pushfd  
                        mov     bl, byte ptr \[esp+0x20\]              ; 读取密码  
        }  
//得到密码之后当然是保存密码了，代码就不写了   
        \_asm  
        {  
                popfd  
                        popad  //下面恢复 hook  
                        mov     bl, byte ptr \[esp+0x20\]   
                        mov     eax, dword ptr \[edi\]  
                        jmp g\_GetPointReturnAddress  
        }  
}  
  
写个 dll 在 (DLL\_PROCESS\_ATTACH 下面加入以下代码  
CodeLen = 0x6;  
OldAddress = g\_BaseAddress + JMP\_GETPASS\_OFFSET;  
g\_GetPassReturnAddress = OldAddress + CodeLen;  
pGetPassHook = new CInlineHook((DWORD)MyHooKGetPassFun, OldAddress, g\_PatchCode, CodeLen);  
  
下面是类  
#include "StdAfx.h"  
#include ".\\inlinehook.h"  
  
CInlineHook::CInlineHook(DWORD NewAddress, DWORD OldAddress, BYTE\*PatchCode, int CodeLen)  
{  
        m\_NewAddress = NewAddress;  
        m\_OldAddress = OldAddress;  
        memcpy(m\_PatchCode, PatchCode, CodeLen);   
        m\_CodeLen = CodeLen;  
  
        m\_pJmpCodeOffset = (DWORD \*)(m\_PatchCode+1);  
        m\_Ishook = FALSE;  
  
        HooK();  
}  
  
CInlineHook::~CInlineHook(void)  
{  
        UnHooK();  
}  
  
BOOL CInlineHook::HooK(void)  
{  
        \*m\_pJmpCodeOffset = m\_NewAddress - m\_OldAddress - 0x5;//跳转偏移量  
  
        memcpy(m\_OldCode, (BYTE \*)m\_OldAddress, m\_CodeLen);   //保存原来的指令  
  
        DWORD dwOldProtect;  
        VirtualProtect((LPVOID)m\_OldAddress, m\_CodeLen, PAGE\_READWRITE, &dwOldProtect);  
  
        memcpy((BYTE \*)m\_OldAddress, m\_PatchCode, m\_CodeLen); //更改指令,跳向补丁代码 HooKFun  
  
        VirtualProtect((LPVOID)m\_OldAddress, m\_CodeLen, dwOldProtect, 0);  
        m\_Ishook = TRUE;  
  
        return TRUE;  
}  
  
BOOL CInlineHook::UnHooK(void)  
{  
        if (m\_Ishook)  
        {  
                DWORD dwOldProtect;  
                VirtualProtect((LPVOID)m\_OldAddress, m\_CodeLen, PAGE\_READWRITE, &dwOldProtect);  
  
                memcpy((BYTE \*)m\_OldAddress, m\_OldCode, m\_CodeLen);  //还原,原来的指令  
  
                VirtualProtect((LPVOID)m\_OldAddress, m\_CodeLen, dwOldProtect, 0);  
  
                m\_Ishook = FALSE;  
                return TRUE;  
        }  
  
        return FALSE;  
}  
  
  
  
下面是具体加密分析  
  
00583BC0    83EC 08             sub     esp, 8  
00583BC3    53                  push    ebx  
00583BC4    56                  push    esi  
00583BC5    57                  push    edi  
00583BC6    8BF9                mov     edi, ecx  
00583BC8    90                  nop                                        ; 01F37457  
00583BC9    E8 81125976         call    winmm.timeGetTime  
00583BCE    50                  push    eax  
00583BCF    E8 A6613100         call    00899D7A                           ; 没作用，纯粹为了迷惑你  
00583BD4    8B7F 1C             mov     edi, dword ptr \[edi+1C\]            ; 加密表的指针  
00583BD7    8A5C24 20           mov     bl, byte ptr \[esp+20\]              ; 读取密码  
00583BDB    8B07                mov     eax, dword ptr \[edi\]  
00583BDD    83C4 04             add     esp, 4  
00583BE0    80C3 02             add     bl, 2                              ; 密码的 16 进制+2  
00583BE3    33F6                xor     esi, esi  
00583BE5    3BC7                cmp     eax, edi  
00583BE7    74 1A               je      short 00583C03  
00583BE9    8DA424 00000000     lea     esp, dword ptr \[esp\]  
00583BF0    83FE 08             cmp     esi, 8  
00583BF3    73 0E               jnb     short 00583C03  
00583BF5    8A48 08             mov     cl, byte ptr \[eax+8\]               ; 读取加密表的数据  
00583BF8    8B00                mov     eax, dword ptr \[eax\]  
00583BFA    884C34 0C           mov     byte ptr \[esp+esi+C\], cl           ; 逐字节存入，共八字节  
00583BFE    46                  inc     esi  
00583BFF    3BC7                cmp     eax, edi  
00583C01  ^ 75 ED               jnz     short 00583BF0  
00583C03    E8 7F613100         call    00899D87                           ; 计算时间  
00583C08    99                  cdq  
00583C09    B9 07000000         mov     ecx, 7  
00583C0E    F7F9                idiv    ecx  
00583C10    8B7424 18           mov     esi, dword ptr \[esp+18\]            ; 读取密码写入指针  
00583C14    8A4E 01             mov     cl, byte ptr \[esi+1\]               ; 读取密码第二个字节  
00583C17    8AC3                mov     al, bl  
00583C19    C0E0 04             shl     al, 4  
00583C1C    C0E9 04             shr     cl, 4  
00583C1F    0AC8                or      cl, al  
00583C21    8A46 01             mov     al, byte ptr \[esi+1\]               ; 写入到密码  
00583C24    C0EB 04             shr     bl, 4  
00583C27    C0E0 04             shl     al, 4  
00583C2A    0AC3                or      al, bl  
00583C2C    8806                mov     byte ptr \[esi\], al                 ; 写入到密码  
00583C2E    0FB6FA              movzx   edi, dl  
00583C31    8A5C3C 0C           mov     bl, byte ptr \[esp+edi+C\]  
00583C35    885C24 1C           mov     byte ptr \[esp+1C\], bl              ; 输入的字符串 ascii  
00583C39    32D9                xor     bl, cl  
00583C3B    324424 1C           xor     al, byte ptr \[esp+1C\]              ; 输入的字符串 ascii 码  
00583C3F    24 0F               and     al, 0F  
00583C41    C0E2 04             shl     dl, 4  
00583C44    0AC2                or      al, dl  
00583C46    5F                  pop     edi  
00583C47    885E 01             mov     byte ptr \[esi+1\], bl               ; 写入前面一位  
00583C4A    8806                mov     byte ptr \[esi\], al                 ; 写入后一位  
00583C4C    5E                  pop     esi  
00583C4D    5B                  pop     ebx  
00583C4E    83C4 08             add     esp, 8  
00583C51    C2 0800             retn    8  
  
  
00899D87    E8 C0740000         call    008A124C                           ; 又是迷惑你的代码  
00899D8C    8B48 14             mov     ecx, dword ptr \[eax+14\]            ; 读取前面用 timegettime 得到的时间  
00899D8F    69C9 FD430300       imul    ecx, ecx, 343FD  
00899D95    81C1 C39E2600       add     ecx, 269EC3  
00899D9B    8948 14             mov     dword ptr \[eax+14\], ecx  
00899D9E    8BC1                mov     eax, ecx  
00899DA0    C1E8 10             shr     eax, 10  
00899DA3    25 FF7F0000         and     eax, 7FFF  
00899DA8    C3                  retn  
  
  
  
  
每输入一位密码得到时间，然后通过 timegettime 得到的时间进行计算，得到两位数的密码串  
然后在通过变形 3des 加密得到一个 64 位的密码  
  
  
  
直接内连汇编  
  
const BYTE Pass\[8\] = {0xD2, 0x29, 0xB6, 0x8D, 0x0E, 0xF2, 0x78, 0xB2};  
//aa:=TimeGetTime();  
DWORD aa=0x01077BB0;  
WORD bb=0x0000;  
BYTE cc=0x61;  
\_\_asm  
        {  
                pushad  
                mov ecx,aa  
                imul ecx,ecx,0x343fd  
                add ecx,0x269ec3  
                mov eax,ecx  
                shr eax,0x10  
                and eax,0x7fff  
                mov ecx,0x7  
                idiv ecx  
                lea esi,bb  
                mov     cl, byte ptr \[esi+0x1\]  
                mov     bl, cc  
                add     bl, 2  
                mov     al, bl  
                shl     al, 0x4  
                shr     cl, 0x4  
                or      cl, al  
                mov     al, byte ptr \[esi+0x1\]  
                shr     bl, 0x4  
                shl     al, 0x4  
                or      al, bl  
                mov     byte ptr \[esi\], al  
                movzx   edi, dl  
                lea     ebp,Pass  
                mov     bl, byte ptr \[ebp+edi\]  //查表  
                mov     byte ptr \[esp+0x1C\], bl  
                xor     bl, cl  
                xor     al, byte ptr \[esp+0x1C\]  
                and     al, 0x0F  
                shl     dl, 0x4  
                or      al, dl  
                mov     byte ptr \[esi+0x1\], bl  
                mov     byte ptr \[esi\], al  
                popad  
        }  
char  ss\[20\];  
sprintf(ss,"%x",bb);  
  
  
  
下面是 3des   
  
3des   表生成代码  
通过固定的 key   
0x7E, 0x40, 0x3E, 0x6C, 0x44, 0x4F, 0x06, 0x74, 0x34, 0x94, 0xA4, 0x98, 0x06, 0x4D, 0xC2, 0x4D  
生成  
  
00583B5F    52                  push    edx  
00583B60    8D5424 0C           lea     edx, dword ptr \[esp+C\]  
00583B64    52                  push    edx  
00583B65    FF10                call    dword ptr \[eax\]                    ; 生成表的  
  
这里就不注释了，3des 表生成代码，没经过变形  
008777D0    8B4424 08           mov     eax, dword ptr \[esp+8\]  
008777D4    33D2                xor     edx, edx  
008777D6    894424 08           mov     dword ptr \[esp+8\], eax  
008777DA    8B4424 04           mov     eax, dword ptr \[esp+4\]  
008777DE    8A70 01             mov     dh, byte ptr \[eax+1\]  
008777E1    8D48 01             lea     ecx, dword ptr \[eax+1\]  
008777E4    53                  push    ebx  
008777E5    55                  push    ebp  
008777E6    56                  push    esi  
008777E7    0FB630              movzx   esi, byte ptr \[eax\]  
008777EA    0FB641 01           movzx   eax, byte ptr \[ecx+1\]  
008777EE    0BF2                or      esi, edx  
008777F0    41                  inc     ecx  
008777F1    0FB651 01           movzx   edx, byte ptr \[ecx+1\]  
008777F5    C1E0 10             shl     eax, 10  
008777F8    0BF0                or      esi, eax  
008777FA    41                  inc     ecx  
008777FB    0FB641 01           movzx   eax, byte ptr \[ecx+1\]  
008777FF    C1E2 18             shl     edx, 18  
00877802    0BF2                or      esi, edx  
00877804    41                  inc     ecx  
00877805    41                  inc     ecx  
00877806    33D2                xor     edx, edx  
00877808    8A31                mov     dh, byte ptr \[ecx\]  
0087780A    57                  push    edi  
0087780B    0BC2                or      eax, edx  
0087780D    41                  inc     ecx  
0087780E    33D2                xor     edx, edx  
00877810    8A71 01             mov     dh, byte ptr \[ecx+1\]  
00877813    8A11                mov     dl, byte ptr \[ecx\]  
00877815    C1E2 10             shl     edx, 10  
00877818    0BC2                or      eax, edx  
0087781A    8BC8                mov     ecx, eax  
0087781C    C1E9 04             shr     ecx, 4  
0087781F    33CE                xor     ecx, esi  
00877821    81E1 0F0F0F0F       and     ecx, 0F0F0F0F  
00877827    33F1                xor     esi, ecx  
00877829    C1E1 04             shl     ecx, 4  
0087782C    33C1                xor     eax, ecx  
0087782E    8BCE                mov     ecx, esi  
00877830    C1E1 12             shl     ecx, 12  
00877833    33CE                xor     ecx, esi  
00877835    81E1 0000CCCC       and     ecx, CCCC0000  
0087783B    8BD1                mov     edx, ecx  
0087783D    C1EA 12             shr     edx, 12  
00877840    33D1                xor     edx, ecx  
00877842    8BC8                mov     ecx, eax  
00877844    C1E1 12             shl     ecx, 12  
00877847    33C8                xor     ecx, eax  
00877849    81E1 0000CCCC       and     ecx, CCCC0000  
0087784F    33F2                xor     esi, edx  
00877851    8BD1                mov     edx, ecx  
00877853    C1EA 12             shr     edx, 12  
00877856    33D1                xor     edx, ecx  
00877858    33C2                xor     eax, edx  
0087785A    8BC8                mov     ecx, eax  
0087785C    D1E9                shr     ecx, 1  
0087785E    33CE                xor     ecx, esi  
00877860    81E1 55555555       and     ecx, 55555555  
00877866    33F1                xor     esi, ecx  
00877868    03C9                add     ecx, ecx  
0087786A    33C1                xor     eax, ecx  
0087786C    8BCE                mov     ecx, esi  
0087786E    C1E9 08             shr     ecx, 8  
00877871    33C8                xor     ecx, eax  
00877873    81E1 FF00FF00       and     ecx, 0FF00FF  
00877879    33C1                xor     eax, ecx  
0087787B    C1E1 08             shl     ecx, 8  
0087787E    33F1                xor     esi, ecx  
00877880    8BC8                mov     ecx, eax  
00877882    D1E9                shr     ecx, 1  
00877884    33CE                xor     ecx, esi  
00877886    81E1 55555555       and     ecx, 55555555  
0087788C    8D1409              lea     edx, dword ptr \[ecx+ecx\]  
0087788F    33C2                xor     eax, edx  
00877891    33F1                xor     esi, ecx  
00877893    8BE8                mov     ebp, eax  
00877895    C1ED 0C             shr     ebp, 0C  
00877898    8BCE                mov     ecx, esi  
0087789A    81E5 F00F0000       and     ebp, 0FF0  
008778A0    81E1 0F0000F0       and     ecx, F000000F  
008778A6    0BE9                or      ebp, ecx  
008778A8    8BD0                mov     edx, eax  
008778AA    81E2 FF000000       and     edx, 0FF  
008778B0    C1ED 04             shr     ebp, 4  
008778B3    C1E2 10             shl     edx, 10  
008778B6    0BEA                or      ebp, edx  
008778B8    25 00FF0000         and     eax, 0FF00  
008778BD    0BE8                or      ebp, eax  
008778BF    81E6 FFFFFF0F       and     esi, 0FFFFFFF  
008778C5    C74424 14 E8DE9E00  mov     dword ptr \[esp+14\], 009EDEE8  
008778CD    8D49 00             lea     ecx, dword ptr \[ecx\]  
008778D0    8B4424 14           mov     eax, dword ptr \[esp+14\]  
008778D4    8338 00             cmp     dword ptr \[eax\], 0  
008778D7    74 16               je      short 008778EF  
008778D9    8BCE                mov     ecx, esi  
008778DB    8BD5                mov     edx, ebp  
008778DD    C1E1 1A             shl     ecx, 1A  
008778E0    C1EE 02             shr     esi, 2  
008778E3    C1E2 1A             shl     edx, 1A  
008778E6    C1ED 02             shr     ebp, 2  
008778E9    0BF1                or      esi, ecx  
008778EB    0BEA                or      ebp, edx  
008778ED    EB 12               jmp     short 00877901  
008778EF    8BC6                mov     eax, esi  
008778F1    8BCD                mov     ecx, ebp  
008778F3    C1E0 1B             shl     eax, 1B  
008778F6    D1EE                shr     esi, 1  
008778F8    C1E1 1B             shl     ecx, 1B  
008778FB    D1ED                shr     ebp, 1  
008778FD    0BF0                or      esi, eax  
008778FF    0BE9                or      ebp, ecx  
00877901    81E6 FFFFFF0F       and     esi, 0FFFFFFF  
00877907    8BC6                mov     eax, esi  
00877909    D1E8                shr     eax, 1  
0087790B    8BD0                mov     edx, eax  
0087790D    81E2 00000007       and     edx, 7000000  
00877913    8BF8                mov     edi, eax  
00877915    8BCE                mov     ecx, esi  
00877917    81E1 0000C000       and     ecx, 0C00000  
0087791D    0BD1                or      edx, ecx  
0087791F    D1EA                shr     edx, 1  
00877921    25 000F0000         and     eax, 0F00  
00877926    8BCE                mov     ecx, esi  
00877928    81E1 00001000       and     ecx, 100000  
0087792E    0BD1                or      edx, ecx  
00877930    C1EA 14             shr     edx, 14  
00877933    81E7 00000600       and     edi, 60000  
00877939    8BCE                mov     ecx, esi  
0087793B    81E1 00E00100       and     ecx, 1E000  
00877941    0BCF                or      ecx, edi  
00877943    8B3C95 70EC9400     mov     edi, dword ptr \[edx\*4+94EC70\]  
0087794A    C1E9 0D             shr     ecx, 0D  
0087794D    8B1C8D 70EB9400     mov     ebx, dword ptr \[ecx\*4+94EB70\]  
00877954    8BD6                mov     edx, esi  
00877956    81E2 C0000000       and     edx, 0C0  
0087795C    0BC2                or      eax, edx  
0087795E    C1E8 06             shr     eax, 6  
00877961    0BFB                or      edi, ebx  
00877963    8B1C85 70EA9400     mov     ebx, dword ptr \[eax\*4+94EA70\]  
0087796A    81E5 FFFFFF0F       and     ebp, 0FFFFFFF  
00877970    8BC6                mov     eax, esi  
00877972    83E0 3F             and     eax, 3F  
00877975    8B0C85 70E99400     mov     ecx, dword ptr \[eax\*4+94E970\]  
0087797C    0BFB                or      edi, ebx  
0087797E    0BF9                or      edi, ecx  
00877980    8BC5                mov     eax, ebp  
00877982    D1E8                shr     eax, 1  
00877984    8BC8                mov     ecx, eax  
00877986    81E1 001E0000       and     ecx, 1E00  
0087798C    8BD5                mov     edx, ebp  
0087798E    81E2 80010000       and     edx, 180  
00877994    0BCA                or      ecx, edx  
00877996    25 00000006         and     eax, 6000000  
0087799B    8BD5                mov     edx, ebp  
0087799D    81E2 0000E001       and     edx, 1E00000  
008779A3    0BC2                or      eax, edx  
008779A5    C1E8 15             shr     eax, 15  
008779A8    8B1485 70F09400     mov     edx, dword ptr \[eax\*4+94F070\]  
008779AF    C1E9 07             shr     ecx, 7  
008779B2    8B1C8D 70EE9400     mov     ebx, dword ptr \[ecx\*4+94EE70\]  
008779B9    0BDA                or      ebx, edx  
008779BB    8BC5                mov     eax, ebp  
008779BD    C1E8 0F             shr     eax, 0F  
008779C0    83E0 3F             and     eax, 3F  
008779C3    0B1C85 70EF9400     or      ebx, dword ptr \[eax\*4+94EF70\]  
008779CA    8BCD                mov     ecx, ebp  
008779CC    83E1 3F             and     ecx, 3F  
008779CF    0B1C8D 70ED9400     or      ebx, dword ptr \[ecx\*4+94ED70\]  
008779D6    8BD7                mov     edx, edi  
008779D8    8BC3                mov     eax, ebx  
008779DA    81E2 FFFF0000       and     edx, 0FFFF  
008779E0    C1E0 10             shl     eax, 10  
008779E3    0BD0                or      edx, eax  
008779E5    6A 1E               push    1E  
008779E7    52                  push    edx  
008779E8    E8 8E7B0200         call    0089F57B  
008779ED    8B4C24 20           mov     ecx, dword ptr \[esp+20\]  
008779F1    C1EF 10             shr     edi, 10  
008779F4    81E3 0000FFFF       and     ebx, FFFF0000  
008779FA    8901                mov     dword ptr \[ecx\], eax  
008779FC    83C1 04             add     ecx, 4  
008779FF    6A 1A               push    1A  
00877A01    0BFB                or      edi, ebx  
00877A03    57                  push    edi  
00877A04    894C24 28           mov     dword ptr \[esp+28\], ecx  
00877A08    E8 6E7B0200         call    0089F57B  
00877A0D    8B4C24 28           mov     ecx, dword ptr \[esp+28\]  
00877A11    83C4 10             add     esp, 10  
00877A14    8901                mov     dword ptr \[ecx\], eax  
00877A16    83C1 04             add     ecx, 4  
00877A19    8B4424 14           mov     eax, dword ptr \[esp+14\]  
00877A1D    83C0 04             add     eax, 4  
00877A20    3D 28DF9E00         cmp     eax, 009EDF28  
00877A25    894C24 18           mov     dword ptr \[esp+18\], ecx  
00877A29    894424 14           mov     dword ptr \[esp+14\], eax  
00877A2D  ^ 0F8C 9DFEFFFF       jl      008778D0  
00877A33    5F                  pop     edi  
00877A34    5E                  pop     esi  
00877A35    5D                  pop     ebp  
00877A36    5B                  pop     ebx  
00877A37    C3                  retn  
  
逆向出的代码  
  
var  
  c, d, t, s, t2, i: dword;  
begin  
  c:= KeyB^\[0\] or (KeyB^\[1\] shl 8) or (KeyB^\[2\] shl 16) or (KeyB^\[3\] shl 24);  
  d:= KeyB^\[4\] or (KeyB^\[5\] shl 8) or (KeyB^\[6\] shl 16) or (KeyB^\[7\] shl 24);  
  perm\_op(d,c,t,4,$0f0f0f0f);  
  hperm\_op(c,t,dword(-2),$cccc0000);  
  hperm\_op(d,t,dword(-2),$cccc0000);  
  perm\_op(d,c,t,1,$55555555);  
  perm\_op(c,d,t,8,$00ff00ff);  
  perm\_op(d,c,t,1,$55555555);  
  d:= ((d and $ff) shl 16) or (d and $ff00) or ((d and $ff0000) shr 16) or  
        ((c and $f0000000) shr 4);  
  c:= c and $fffffff;  
  for i:= 0 to 15 do  
  begin  
    if shifts2\[i\]<> 0 then  
    begin  
      c:= ((c shr 2) or (c shl 26));  
      d:= ((d shr 2) or (d shl 26));  
    end  
    else  
    begin  
      c:= ((c shr 1) or (c shl 27));  
      d:= ((d shr 1) or (d shl 27));  
    end;  
    c:= c and $fffffff;  
    d:= d and $fffffff;  
    s:= des\_skb\[0,c and $3f\] or  
        des\_skb\[1,((c shr  6) and $03) or ((c shr  7) and $3c)\] or  
        des\_skb\[2,((c shr 13) and $0f) or ((c shr 14) and $30)\] or  
        des\_skb\[3,((c shr 20) and $01) or ((c shr 21) and $06) or ((c shr 22) and $38)\];  
    t:= des\_skb\[4,d and $3f\] or  
        des\_skb\[5,((d shr  7) and $03) or ((d shr  8) and $3c)\] or  
        des\_skb\[6, (d shr 15) and $3f                         \] or  
        des\_skb\[7,((d shr 21) and $0f) or ((d shr 22) and $30)\];  
    t2:= ((t shl 16) or (s and $ffff));  
    KeyData^\[(i shl 1)\]:= ((t2 shl 2) or (t2 shr 30));  
    t2:= ((s shr 16) or (t and $ffff0000));  
    KeyData^\[(i shl 1)+1\]:= ((t2 shl 6) or (t2 shr 26));  
  end;  
end;  
  
3des 变形加密算法  
  
00583B97    8B01                mov     eax, dword ptr \[ecx\]  
00583B99    55                  push    ebp                                ; 加密/解密标识  
00583B9A    8D5424 1C           lea     edx, dword ptr \[esp+1C\]  
00583B9E    52                  push    edx                                ; key  
00583B9F    56                  push    esi                                ; 需要加密的数据  
00583BA0    FF50 04             call    dword ptr \[eax+4\]                  ; zhengtu.00876B40  
  
00877A40    53                  push    ebx  
00877A41    55                  push    ebp  
00877A42    56                  push    esi  
00877A43    57                  push    edi  
00877A44    8B4C24 14           mov     ecx, dword ptr \[esp+14\]  
00877A48    8B01                mov     eax, dword ptr \[ecx\]  
00877A4A    8B71 04             mov     esi, dword ptr \[ecx+4\]  
00877A4D    8BCE                mov     ecx, esi  
00877A4F    C1E9 04             shr     ecx, 4  
00877A52    33C8                xor     ecx, eax  
00877A54    81E1 0F0F0F0F       and     ecx, 0F0F0F0F  
00877A5A    33C1                xor     eax, ecx  
00877A5C    C1E1 04             shl     ecx, 4  
00877A5F    33F1                xor     esi, ecx  
00877A61    8BC8                mov     ecx, eax  
00877A63    C1E9 10             shr     ecx, 10  
00877A66    33CE                xor     ecx, esi  
00877A68    81E1 FFFF0000       and     ecx, 0FFFF  
00877A6E    33F1                xor     esi, ecx  
00877A70    C1E1 10             shl     ecx, 10  
00877A73    33C1                xor     eax, ecx  
00877A75    8BCE                mov     ecx, esi  
00877A77    C1E9 02             shr     ecx, 2  
00877A7A    33C8                xor     ecx, eax  
00877A7C    81E1 33333333       and     ecx, 33333333  
00877A82    33C1                xor     eax, ecx  
00877A84    C1E1 02             shl     ecx, 2  
00877A87    33F1                xor     esi, ecx  
00877A89    8BC8                mov     ecx, eax  
00877A8B    C1E9 08             shr     ecx, 8  
00877A8E    33CE                xor     ecx, esi  
00877A90    81E1 FF00FF00       and     ecx, 0FF00FF  
00877A96    33F1                xor     esi, ecx  
00877A98    C1E1 08             shl     ecx, 8  
00877A9B    33C1                xor     eax, ecx  
00877A9D    8BFE                mov     edi, esi  
00877A9F    D1EF                shr     edi, 1  
00877AA1    33F8                xor     edi, eax  
00877AA3    81E7 55555555       and     edi, 55555555  
00877AA9    8BD7                mov     edx, edi  
00877AAB    33D0                xor     edx, eax  
00877AAD    6A 1D               push    1D  
00877AAF    52                  push    edx  
00877AB0    E8 C67A0200         call    0089F57B  
00877AB5    8BD8                mov     ebx, eax  
00877AB7    8D043F              lea     eax, dword ptr \[edi+edi\]  
00877ABA    33C6                xor     eax, esi  
00877ABC    6A 1D               push    1D  
00877ABE    50                  push    eax  
00877ABF    E8 B77A0200         call    0089F57B  
00877AC4    8B7424 2C           mov     esi, dword ptr \[esp+2C\]  
00877AC8    8BF8                mov     edi, eax  
00877ACA    8B4424 30           mov     eax, dword ptr \[esp+30\]  
00877ACE    83C4 10             add     esp, 10  
00877AD1    85C0                test    eax, eax  
00877AD3    8B4424 18           mov     eax, dword ptr \[esp+18\]  
00877AD7    0F84 52020000       je      00877D2F  
00877ADD    83C0 08             add     eax, 8  
00877AE0    894424 18           mov     dword ptr \[esp+18\], eax  
00877AE4    C74424 20 04000000  mov     dword ptr \[esp+20\], 4  
00877AEC    EB 04               jmp     short 00877AF2  
00877AEE    8B4424 18           mov     eax, dword ptr \[esp+18\]  
00877AF2    8B68 F8             mov     ebp, dword ptr \[eax-8\]  
00877AF5    8B40 FC             mov     eax, dword ptr \[eax-4\]  
00877AF8    33C3                xor     eax, ebx  
00877AFA    6A 04               push    4  
00877AFC    50                  push    eax  
00877AFD    33EB                xor     ebp, ebx  
00877AFF    E8 777A0200         call    0089F57B  
00877B04    8BC8                mov     ecx, eax  
00877B06    C1E9 12             shr     ecx, 12  
00877B09    83E1 3F             and     ecx, 3F  
00877B0C    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877B13    8BD5                mov     edx, ebp  
00877B15    C1EA 12             shr     edx, 12  
00877B18    83E2 3F             and     edx, 3F  
00877B1B    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877B22    8BD0                mov     edx, eax  
00877B24    C1EA 0A             shr     edx, 0A  
00877B27    83E2 3F             and     edx, 3F  
00877B2A    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877B31    8BD5                mov     edx, ebp  
00877B33    C1EA 0A             shr     edx, 0A  
00877B36    83E2 3F             and     edx, 3F  
00877B39    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877B40    8BD0                mov     edx, eax  
00877B42    C1E8 1A             shr     eax, 1A  
00877B45    C1EA 02             shr     edx, 2  
00877B48    83E2 3F             and     edx, 3F  
00877B4B    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877B52    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877B59    8BC5                mov     eax, ebp  
00877B5B    C1E8 1A             shr     eax, 1A  
00877B5E    33CA                xor     ecx, edx  
00877B60    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877B67    C1ED 02             shr     ebp, 2  
00877B6A    83E5 3F             and     ebp, 3F  
00877B6D    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877B70    33CA                xor     ecx, edx  
00877B72    33C8                xor     ecx, eax  
00877B74    8B4424 20           mov     eax, dword ptr \[esp+20\]  
00877B78    8B28                mov     ebp, dword ptr \[eax\]  
00877B7A    8B40 04             mov     eax, dword ptr \[eax+4\]  
00877B7D    33F9                xor     edi, ecx  
00877B7F    33C7                xor     eax, edi  
00877B81    6A 04               push    4  
00877B83    50                  push    eax  
00877B84    33EF                xor     ebp, edi  
00877B86    E8 F0790200         call    0089F57B  
00877B8B    8BC8                mov     ecx, eax  
00877B8D    C1E9 12             shr     ecx, 12  
00877B90    83E1 3F             and     ecx, 3F  
00877B93    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877B9A    8BD5                mov     edx, ebp  
00877B9C    C1EA 12             shr     edx, 12  
00877B9F    83E2 3F             and     edx, 3F  
00877BA2    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877BA9    8BD0                mov     edx, eax  
00877BAB    C1EA 0A             shr     edx, 0A  
00877BAE    83E2 3F             and     edx, 3F  
00877BB1    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877BB8    8BD5                mov     edx, ebp  
00877BBA    C1EA 0A             shr     edx, 0A  
00877BBD    83E2 3F             and     edx, 3F  
00877BC0    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877BC7    8BD0                mov     edx, eax  
00877BC9    C1EA 02             shr     edx, 2  
00877BCC    C1E8 1A             shr     eax, 1A  
00877BCF    83E2 3F             and     edx, 3F  
00877BD2    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877BD9    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877BE0    8BC5                mov     eax, ebp  
00877BE2    C1E8 1A             shr     eax, 1A  
00877BE5    33CA                xor     ecx, edx  
00877BE7    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877BEE    C1ED 02             shr     ebp, 2  
00877BF1    83E5 3F             and     ebp, 3F  
00877BF4    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877BF7    33CA                xor     ecx, edx  
00877BF9    33C8                xor     ecx, eax  
00877BFB    8B4424 28           mov     eax, dword ptr \[esp+28\]  
00877BFF    8B68 08             mov     ebp, dword ptr \[eax+8\]  
00877C02    33D9                xor     ebx, ecx  
00877C04    8B40 0C             mov     eax, dword ptr \[eax+C\]  
00877C07    33C3                xor     eax, ebx  
00877C09    6A 04               push    4  
00877C0B    50                  push    eax  
00877C0C    33EB                xor     ebp, ebx  
00877C0E    E8 68790200         call    0089F57B  
00877C13    8BC8                mov     ecx, eax  
00877C15    C1E9 12             shr     ecx, 12  
00877C18    83E1 3F             and     ecx, 3F  
00877C1B    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877C22    8BD5                mov     edx, ebp  
00877C24    C1EA 12             shr     edx, 12  
00877C27    83E2 3F             and     edx, 3F  
00877C2A    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877C31    8BD0                mov     edx, eax  
00877C33    C1EA 0A             shr     edx, 0A  
00877C36    83E2 3F             and     edx, 3F  
00877C39    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877C40    8BD5                mov     edx, ebp  
00877C42    C1EA 0A             shr     edx, 0A  
00877C45    83E2 3F             and     edx, 3F  
00877C48    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877C4F    8BD0                mov     edx, eax  
00877C51    C1E8 1A             shr     eax, 1A  
00877C54    C1EA 02             shr     edx, 2  
00877C57    83E2 3F             and     edx, 3F  
00877C5A    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877C61    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877C68    8BC5                mov     eax, ebp  
00877C6A    C1E8 1A             shr     eax, 1A  
00877C6D    33CA                xor     ecx, edx  
00877C6F    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877C76    C1ED 02             shr     ebp, 2  
00877C79    83E5 3F             and     ebp, 3F  
00877C7C    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877C7F    33CA                xor     ecx, edx  
00877C81    33C8                xor     ecx, eax  
00877C83    8B4424 30           mov     eax, dword ptr \[esp+30\]  
00877C87    8B68 10             mov     ebp, dword ptr \[eax+10\]  
00877C8A    8B40 14             mov     eax, dword ptr \[eax+14\]  
00877C8D    33F9                xor     edi, ecx  
00877C8F    33C7                xor     eax, edi  
00877C91    6A 04               push    4  
00877C93    50                  push    eax  
00877C94    33EF                xor     ebp, edi  
00877C96    E8 E0780200         call    0089F57B  
00877C9B    8BC8                mov     ecx, eax  
00877C9D    C1E9 12             shr     ecx, 12  
00877CA0    83E1 3F             and     ecx, 3F  
00877CA3    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877CAA    8BD5                mov     edx, ebp  
00877CAC    C1EA 12             shr     edx, 12  
00877CAF    83E2 3F             and     edx, 3F  
00877CB2    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877CB9    8BD0                mov     edx, eax  
00877CBB    C1EA 0A             shr     edx, 0A  
00877CBE    83E2 3F             and     edx, 3F  
00877CC1    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877CC8    8BD5                mov     edx, ebp  
00877CCA    C1EA 0A             shr     edx, 0A  
00877CCD    83E2 3F             and     edx, 3F  
00877CD0    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877CD7    8BD0                mov     edx, eax  
00877CD9    C1EA 02             shr     edx, 2  
00877CDC    83E2 3F             and     edx, 3F  
00877CDF    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877CE6    C1E8 1A             shr     eax, 1A  
00877CE9    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877CF0    8BC5                mov     eax, ebp  
00877CF2    33CA                xor     ecx, edx  
00877CF4    C1E8 1A             shr     eax, 1A  
00877CF7    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877CFE    C1ED 02             shr     ebp, 2  
00877D01    83E5 3F             and     ebp, 3F  
00877D04    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877D07    33CA                xor     ecx, edx  
00877D09    33C8                xor     ecx, eax  
00877D0B    33D9                xor     ebx, ecx  
00877D0D    8B4C24 38           mov     ecx, dword ptr \[esp+38\]  
00877D11    83C4 20             add     esp, 20  
00877D14    83C1 20             add     ecx, 20  
00877D17    8B4424 20           mov     eax, dword ptr \[esp+20\]  
00877D1B    48                  dec     eax  
00877D1C    894C24 18           mov     dword ptr \[esp+18\], ecx  
00877D20    894424 20           mov     dword ptr \[esp+20\], eax  
00877D24  ^ 0F85 C4FDFFFF       jnz     00877AEE  
00877D2A    E9 4D020000         jmp     00877F7C  
00877D2F    83C0 70             add     eax, 70  
00877D32    894424 18           mov     dword ptr \[esp+18\], eax  
00877D36    C74424 20 04000000  mov     dword ptr \[esp+20\], 4  
00877D3E    EB 04               jmp     short 00877D44  
00877D40    8B4424 18           mov     eax, dword ptr \[esp+18\]  
00877D44    8B68 08             mov     ebp, dword ptr \[eax+8\]  
00877D47    8B40 0C             mov     eax, dword ptr \[eax+C\]  
00877D4A    33C3                xor     eax, ebx  
00877D4C    6A 04               push    4  
00877D4E    50                  push    eax  
00877D4F    33EB                xor     ebp, ebx  
00877D51    E8 25780200         call    0089F57B  
00877D56    8BC8                mov     ecx, eax  
00877D58    C1E9 12             shr     ecx, 12  
00877D5B    83E1 3F             and     ecx, 3F  
00877D5E    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877D65    8BD5                mov     edx, ebp  
00877D67    C1EA 12             shr     edx, 12  
00877D6A    83E2 3F             and     edx, 3F  
00877D6D    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877D74    8BD0                mov     edx, eax  
00877D76    C1EA 0A             shr     edx, 0A  
00877D79    83E2 3F             and     edx, 3F  
00877D7C    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877D83    8BD5                mov     edx, ebp  
00877D85    C1EA 0A             shr     edx, 0A  
00877D88    83E2 3F             and     edx, 3F  
00877D8B    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877D92    8BD0                mov     edx, eax  
00877D94    C1E8 1A             shr     eax, 1A  
00877D97    C1EA 02             shr     edx, 2  
00877D9A    83E2 3F             and     edx, 3F  
00877D9D    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877DA4    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877DAB    8BC5                mov     eax, ebp  
00877DAD    C1E8 1A             shr     eax, 1A  
00877DB0    33CA                xor     ecx, edx  
00877DB2    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877DB9    C1ED 02             shr     ebp, 2  
00877DBC    83E5 3F             and     ebp, 3F  
00877DBF    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877DC2    33CA                xor     ecx, edx  
00877DC4    33C8                xor     ecx, eax  
00877DC6    8B4424 20           mov     eax, dword ptr \[esp+20\]  
00877DCA    8B28                mov     ebp, dword ptr \[eax\]  
00877DCC    8B40 04             mov     eax, dword ptr \[eax+4\]  
00877DCF    33F9                xor     edi, ecx  
00877DD1    33C7                xor     eax, edi  
00877DD3    6A 04               push    4  
00877DD5    50                  push    eax  
00877DD6    33EF                xor     ebp, edi  
00877DD8    E8 9E770200         call    0089F57B  
00877DDD    8BC8                mov     ecx, eax  
00877DDF    C1E9 12             shr     ecx, 12  
00877DE2    83E1 3F             and     ecx, 3F  
00877DE5    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877DEC    8BD5                mov     edx, ebp  
00877DEE    C1EA 12             shr     edx, 12  
00877DF1    83E2 3F             and     edx, 3F  
00877DF4    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877DFB    8BD0                mov     edx, eax  
00877DFD    C1EA 0A             shr     edx, 0A  
00877E00    83E2 3F             and     edx, 3F  
00877E03    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877E0A    8BD5                mov     edx, ebp  
00877E0C    C1EA 0A             shr     edx, 0A  
00877E0F    83E2 3F             and     edx, 3F  
00877E12    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877E19    8BD0                mov     edx, eax  
00877E1B    C1EA 02             shr     edx, 2  
00877E1E    C1E8 1A             shr     eax, 1A  
00877E21    83E2 3F             and     edx, 3F  
00877E24    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877E2B    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877E32    8BC5                mov     eax, ebp  
00877E34    C1E8 1A             shr     eax, 1A  
00877E37    33CA                xor     ecx, edx  
00877E39    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877E40    C1ED 02             shr     ebp, 2  
00877E43    83E5 3F             and     ebp, 3F  
00877E46    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877E49    33CA                xor     ecx, edx  
00877E4B    33C8                xor     ecx, eax  
00877E4D    8B4424 28           mov     eax, dword ptr \[esp+28\]  
00877E51    8B68 F8             mov     ebp, dword ptr \[eax-8\]  
00877E54    33D9                xor     ebx, ecx  
00877E56    8B40 FC             mov     eax, dword ptr \[eax-4\]  
00877E59    33C3                xor     eax, ebx  
00877E5B    6A 04               push    4  
00877E5D    50                  push    eax  
00877E5E    33EB                xor     ebp, ebx  
00877E60    E8 16770200         call    0089F57B  
00877E65    8BC8                mov     ecx, eax  
00877E67    C1E9 12             shr     ecx, 12  
00877E6A    83E1 3F             and     ecx, 3F  
00877E6D    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877E74    8BD5                mov     edx, ebp  
00877E76    C1EA 12             shr     edx, 12  
00877E79    83E2 3F             and     edx, 3F  
00877E7C    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877E83    8BD0                mov     edx, eax  
00877E85    C1EA 0A             shr     edx, 0A  
00877E88    83E2 3F             and     edx, 3F  
00877E8B    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877E92    8BD5                mov     edx, ebp  
00877E94    C1EA 0A             shr     edx, 0A  
00877E97    83E2 3F             and     edx, 3F  
00877E9A    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877EA1    8BD0                mov     edx, eax  
00877EA3    C1E8 1A             shr     eax, 1A  
00877EA6    C1EA 02             shr     edx, 2  
00877EA9    83E2 3F             and     edx, 3F  
00877EAC    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877EB3    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877EBA    8BC5                mov     eax, ebp  
00877EBC    C1E8 1A             shr     eax, 1A  
00877EBF    33CA                xor     ecx, edx  
00877EC1    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877EC8    C1ED 02             shr     ebp, 2  
00877ECB    83E5 3F             and     ebp, 3F  
00877ECE    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877ED1    33CA                xor     ecx, edx  
00877ED3    33C8                xor     ecx, eax  
00877ED5    8B4424 30           mov     eax, dword ptr \[esp+30\]  
00877ED9    8B68 F0             mov     ebp, dword ptr \[eax-10\]  
00877EDC    8B40 F4             mov     eax, dword ptr \[eax-C\]  
00877EDF    33F9                xor     edi, ecx  
00877EE1    33C7                xor     eax, edi  
00877EE3    6A 04               push    4  
00877EE5    50                  push    eax  
00877EE6    33EF                xor     ebp, edi  
00877EE8    E8 8E760200         call    0089F57B  
00877EED    8BC8                mov     ecx, eax  
00877EEF    C1E9 12             shr     ecx, 12  
00877EF2    83E1 3F             and     ecx, 3F  
00877EF5    8B8C8E 00050000     mov     ecx, dword ptr \[esi+ecx\*4+500\]  
00877EFC    8BD5                mov     edx, ebp  
00877EFE    C1EA 12             shr     edx, 12  
00877F01    83E2 3F             and     edx, 3F  
00877F04    338C96 00040000     xor     ecx, dword ptr \[esi+edx\*4+400\]  
00877F0B    8BD0                mov     edx, eax  
00877F0D    C1EA 0A             shr     edx, 0A  
00877F10    83E2 3F             and     edx, 3F  
00877F13    338C96 00030000     xor     ecx, dword ptr \[esi+edx\*4+300\]  
00877F1A    8BD5                mov     edx, ebp  
00877F1C    C1EA 0A             shr     edx, 0A  
00877F1F    83E2 3F             and     edx, 3F  
00877F22    338C96 00020000     xor     ecx, dword ptr \[esi+edx\*4+200\]  
00877F29    8BD0                mov     edx, eax  
00877F2B    C1EA 02             shr     edx, 2  
00877F2E    83E2 3F             and     edx, 3F  
00877F31    338C96 00010000     xor     ecx, dword ptr \[esi+edx\*4+100\]  
00877F38    C1E8 1A             shr     eax, 1A  
00877F3B    8B9486 00070000     mov     edx, dword ptr \[esi+eax\*4+700\]  
00877F42    8BC5                mov     eax, ebp  
00877F44    33CA                xor     ecx, edx  
00877F46    C1E8 1A             shr     eax, 1A  
00877F49    8B9486 00060000     mov     edx, dword ptr \[esi+eax\*4+600\]  
00877F50    C1ED 02             shr     ebp, 2  
00877F53    83E5 3F             and     ebp, 3F  
00877F56    8B04AE              mov     eax, dword ptr \[esi+ebp\*4\]  
00877F59    33CA                xor     ecx, edx  
00877F5B    33C8                xor     ecx, eax  
00877F5D    33D9                xor     ebx, ecx  
00877F5F    8B4C24 38           mov     ecx, dword ptr \[esp+38\]  
00877F63    83C4 20             add     esp, 20  
00877F66    83E9 20             sub     ecx, 20  
00877F69    8B4424 20           mov     eax, dword ptr \[esp+20\]  
00877F6D    48                  dec     eax  
00877F6E    894C24 18           mov     dword ptr \[esp+18\], ecx  
00877F72    894424 20           mov     dword ptr \[esp+20\], eax  
00877F76  ^ 0F85 C4FDFFFF       jnz     00877D40  
00877F7C    6A 03               push    3  
00877F7E    57                  push    edi  
00877F7F    E8 F7750200         call    0089F57B  
00877F84    6A 03               push    3  
00877F86    53                  push    ebx  
00877F87    8BF0                mov     esi, eax  
00877F89    E8 ED750200         call    0089F57B  
00877F8E    8BC8                mov     ecx, eax  
00877F90    D1E9                shr     ecx, 1  
00877F92    33CE                xor     ecx, esi  
00877F94    81E1 55555555       and     ecx, 55555555  
00877F9A    33F1                xor     esi, ecx  
00877F9C    03C9                add     ecx, ecx  
00877F9E    33C1                xor     eax, ecx  
00877FA0    8BCE                mov     ecx, esi  
00877FA2    C1E9 08             shr     ecx, 8  
00877FA5    33C8                xor     ecx, eax  
00877FA7    81E1 FF00FF00       and     ecx, 0FF00FF  
00877FAD    33C1                xor     eax, ecx  
00877FAF    C1E1 08             shl     ecx, 8  
00877FB2    33F1                xor     esi, ecx  
00877FB4    8BC8                mov     ecx, eax  
00877FB6    C1E9 02             shr     ecx, 2  
00877FB9    33CE                xor     ecx, esi  
00877FBB    81E1 33333333       and     ecx, 33333333  
00877FC1    33F1                xor     esi, ecx  
00877FC3    8D148D 00000000     lea     edx, dword ptr \[ecx\*4\]  
00877FCA    8BCE                mov     ecx, esi  
00877FCC    C1E9 10             shr     ecx, 10  
00877FCF    33C2                xor     eax, edx  
00877FD1    8B5424 24           mov     edx, dword ptr \[esp+24\]  
00877FD5    33C8                xor     ecx, eax  
00877FD7    81E1 FFFF0000       and     ecx, 0FFFF  
00877FDD    33C1                xor     eax, ecx  
00877FDF    C1E1 10             shl     ecx, 10  
00877FE2    33F1                xor     esi, ecx  
00877FE4    8BC8                mov     ecx, eax  
00877FE6    C1E9 04             shr     ecx, 4  
00877FE9    33CE                xor     ecx, esi  
00877FEB    81E1 0F0F0F0F       and     ecx, 0F0F0F0F  
00877FF1    8BF9                mov     edi, ecx  
00877FF3    83C4 10             add     esp, 10  
00877FF6    33FE                xor     edi, esi  
00877FF8    893A                mov     dword ptr \[edx\], edi  
00877FFA    5F                  pop     edi  
00877FFB    5E                  pop     esi  
00877FFC    C1E1 04             shl     ecx, 4  
00877FFF    33C8                xor     ecx, eax  
00878001    5D                  pop     ebp  
00878002    894A 04             mov     dword ptr \[edx+4\], ecx  
00878005    5B                  pop     ebx  
00878006    C3                  retn  
  
工作量太大直接内连  
  
procedure  \_\_0089F57B();StdCall;  
begin  
        asm  
                        and     dword ptr \[esp+$8\], $1F  
                        mov     edx, dword ptr \[esp+$4\]  
                        push    $20  
                        pop     ecx  
                        sub     ecx, dword ptr \[esp+$8\]  
                        mov     eax, edx  
                        shl     eax, cl  
                        mov     ecx, dword ptr \[esp+$8\]  
                        shr     edx, cl  
                        or      eax, edx  
                        retn  
        end;  
end;  
  
procedure   \_\_00877A40();StdCall;  
begin  
        asm  
                        push    ebx  
                        push    ebp  
                        push    esi  
                        push    edi  
                        mov     ecx, dword ptr \[esp+$14\]  
                        mov     eax, dword ptr \[ecx\]  
                        mov     esi, dword ptr \[ecx+$4\]  
                        mov     ecx, esi  
                        shr     ecx, $4  
                        xor     ecx, eax  
                        and     ecx, $0F0F0F0F  
                        xor     eax, ecx  
                        shl     ecx, $4  
                        xor     esi, ecx  
                        mov     ecx, eax  
                        shr     ecx, $10  
                        xor     ecx, esi  
                        and     ecx, $0FFFF  
                        xor     esi, ecx  
                        shl     ecx, $10  
                        xor     eax, ecx  
                        mov     ecx, esi  
                        shr     ecx, $2  
                        xor     ecx, eax  
                        and     ecx, $33333333  
                        xor     eax, ecx  
                        shl     ecx, $2  
                        xor     esi, ecx  
                        mov     ecx, eax  
                        shr     ecx, $8  
                        xor     ecx, esi  
                        and     ecx, $0FF00FF  
                        xor     esi, ecx  
                        shl     ecx, $8  
                        xor     eax, ecx  
                        mov     edi, esi  
                        shr     edi, $1  
                        xor     edi, eax  
                        and     edi, $55555555  
                        mov     edx, edi  
                        xor     edx, eax  
                        push    $1D  
                        push    edx  
                        call    \_\_0089F57B  
                        mov     ebx, eax  
                        lea     eax, dword ptr \[edi+edi\]  
                        xor     eax, esi  
                        push    $1D  
                        push    eax  
                        call    \_\_0089F57B  
                        mov     esi, dword ptr\[esp+$2c\]  
                        mov     edi, eax  
                        mov     eax, dword ptr \[esp+$30\]  
                        add     esp, $10  
                        test    eax, eax  
                        mov     eax, dword ptr \[esp+$18\]    //key 地址  
                        je @L230  
                        add     eax, $8  
                        mov     dword ptr \[esp+$18\], eax  
                        mov     dword ptr \[esp+$20\], $4  
                        jmp @L062  
@L061:  
                        mov     eax, dword ptr \[esp+$18\]  
@L062:  
                        mov     ebp, dword ptr \[eax-$8\]  
                        mov     eax, dword ptr \[eax-$4\]  
                        xor     eax, ebx  
                        push    $4  
                        push    eax  
                        xor     ebp, ebx  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     eax, $1A  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$20\]  
                        mov     ebp, dword ptr \[eax\]  
                        mov     eax, dword ptr \[eax+$4\]  
                        xor     edi, ecx  
                        xor     eax, edi  
                        push    $4  
                        push    eax  
                        xor     ebp, edi  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     edx, $2  
                        shr     eax, $1A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$28\]  
                        mov     ebp, dword ptr \[eax+$8\]  
                        xor     ebx, ecx  
                        mov     eax, dword ptr \[eax+$C\]  
                        xor     eax, ebx  
                        push    $4  
                        push    eax  
                        xor     ebp, ebx  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     eax, $1A  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$30\]  
                        mov     ebp, dword ptr \[eax+$10\]  
                        mov     eax, dword ptr \[eax+$14\]  
                        xor     edi, ecx  
                        xor     eax, edi  
                        push    $4  
                        push    eax  
                        xor     ebp, edi  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        shr     eax, $1A  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        xor     ecx, edx  
                        shr     eax, $1A  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        xor     ebx, ecx  
                        mov     ecx, dword ptr \[esp+$38\]  
                        add     esp, $20  
                        add     ecx, $20  
                        mov     eax, dword ptr \[esp+$20\]  
                        dec     eax  
                        mov     dword ptr \[esp+$18\], ecx  
                        mov     dword ptr \[esp+$20\], eax  
                        jnz @L061  
                        jmp @L402  
@L230:  
                        add     eax, $70  
                        mov     dword ptr \[esp+$18\], eax  
                        mov     dword ptr \[esp+$20\], $4  
                        jmp @L235  
@L234:  
                        mov     eax, dword ptr \[esp+$18\]  
@L235:  
                        mov     ebp, dword ptr \[eax+$8\]  
                        mov     eax, dword ptr \[eax+$C\]  
                        xor     eax, ebx  
                        push    $4  
                        push    eax  
                        xor     ebp, ebx  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     eax, $1A  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$20\]  
                        mov     ebp, dword ptr \[eax\]  
                        mov     eax, dword ptr \[eax+$4\]  
                        xor     edi, ecx  
                        xor     eax, edi  
                        push    $4  
                        push    eax  
                        xor     ebp, edi  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     edx, $2  
                        shr     eax, $1A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$28\]  
                        mov     ebp, dword ptr \[eax-$8\]  
                        xor     ebx, ecx  
                        mov     eax, dword ptr \[eax-$4\]  
                        xor     eax, ebx  
                        push    $4  
                        push    eax  
                        xor     ebp, ebx  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     eax, $1A  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        shr     eax, $1A  
                        xor     ecx, edx  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        mov     eax, dword ptr \[esp+$30\]  
                        mov     ebp, dword ptr \[eax-$10\]  
                        mov     eax, dword ptr \[eax-$C\]  
                        xor     edi, ecx  
                        xor     eax, edi  
                        push    $4  
                        push    eax  
                        xor     ebp, edi  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $12  
                        and     ecx, $3F  
                        mov     ecx, dword ptr \[esi+ecx\*$4+$500\]  
                        mov     edx, ebp  
                        shr     edx, $12  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$400\]  
                        mov     edx, eax  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$300\]  
                        mov     edx, ebp  
                        shr     edx, $0A  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$200\]  
                        mov     edx, eax  
                        shr     edx, $2  
                        and     edx, $3F  
                        xor     ecx, dword ptr \[esi+edx\*$4+$100\]  
                        shr     eax, $1A  
                        mov     edx, dword ptr \[esi+eax\*$4+$700\]  
                        mov     eax, ebp  
                        xor     ecx, edx  
                        shr     eax, $1A  
                        mov     edx, dword ptr \[esi+eax\*$4+$600\]  
                        shr     ebp, $2  
                        and     ebp, $3F  
                        mov     eax, dword ptr \[esi+ebp\*$4\]  
                        xor     ecx, edx  
                        xor     ecx, eax  
                        xor     ebx, ecx  
                        mov     ecx, dword ptr \[esp+$38\]  
                        add     esp, $20  
                        sub     ecx, $20  
                        mov     eax, dword ptr \[esp+$20\]  
                        dec     eax  
                        mov     dword ptr \[esp+$18\], ecx  
                        mov     dword ptr \[esp+$20\], eax  
                        jnz @L234  
@L402:  
                        push    $3  
                        push    edi  
                        call    \_\_0089F57B  
                        push    $3  
                        push    ebx  
                        mov     esi, eax  
                        call    \_\_0089F57B  
                        mov     ecx, eax  
                        shr     ecx, $1  
                        xor     ecx, esi  
                        and     ecx, $55555555  
                        xor     esi, ecx  
                        add     ecx, ecx  
                        xor     eax, ecx  
                        mov     ecx, esi  
                        shr     ecx, $8  
                        xor     ecx, eax  
                        and     ecx, $0FF00FF  
                        xor     eax, ecx  
                        shl     ecx, $8  
                        xor     esi, ecx  
                        mov     ecx, eax  
                        shr     ecx, $2  
                        xor     ecx, esi  
                        and     ecx, $33333333  
                        xor     esi, ecx  
                        lea     edx, dword ptr \[ecx\*$4\]  
                        mov     ecx, esi  
                        shr     ecx, $10  
                        xor     eax, edx  
                        mov     edx, dword ptr \[esp+$24\]  
                        xor     ecx, eax  
                        and     ecx, $0FFFF  
                        xor     eax, ecx  
                        shl     ecx, $10  
                        xor     esi, ecx  
                        mov     ecx, eax  
                        shr     ecx, $4  
                        xor     ecx, esi  
                        and     ecx, $0F0F0F0F  
                        mov     edi, ecx  
                        add     esp, $10  
                        xor     edi, esi  
                        mov     dword ptr \[edx\], edi  
                        pop     edi  
                        pop     esi  
                        shl     ecx, $4  
                        xor     ecx, eax  
                        pop     ebp  
                        mov     dword ptr \[edx+$4\], ecx  
                        pop     ebx  
                        retn  
  
  end;  
end;  
  
  
procedure \_\_008777D0();StdCall;  
begin  
        asm  
                        mov     eax, dword ptr \[esp+$08\]  
                        xor     edx, edx  
                        mov     dword ptr \[esp+$08\], eax  
                        mov     eax, dword ptr \[esp+$04\]  
                        mov     dh, byte ptr \[eax+$01\]  
                        lea     ecx, dword ptr \[eax+$01\]  
                        push    ebx  
                        push    ebp  
                        push    esi  
                        movzx   esi, byte ptr \[eax\]  
                        movzx   eax, byte ptr \[ecx+$01\]  
                        or      esi, edx  
                        inc     ecx  
                        movzx   edx, byte ptr \[ecx+$01\]  
                        shl     eax, $10  
                        or      esi, eax  
                        inc     ecx  
                        movzx   eax, byte ptr \[ecx+$01\]  
                        shl     edx, $18  
                        or      esi, edx  
                        inc     ecx  
                        inc     ecx  
                        xor     edx, edx  
                        mov     dh, byte ptr \[ecx\]  
                        push    edi  
                        or      eax, edx  
                        inc     ecx  
                        xor     edx, edx  
                        mov     dh, byte ptr \[ecx+$01\]  
                        mov     dl, byte ptr \[ecx\]  
                        shl     edx, $10  
                        or      eax, edx  
                        mov     ecx, eax  
                        shr     ecx, $04  
                        xor     ecx, esi  
                        and     ecx, $0F0F0F0F  
                        xor     esi, ecx  
                        shl     ecx, $04  
                        xor     eax, ecx  
                        mov     ecx, esi  
                        shl     ecx, $12  
                        xor     ecx, esi  
                        and     ecx, $CCCC0000  
                        mov     edx, ecx  
                        shr     edx, $12  
                        xor     edx, ecx  
                        mov     ecx, eax  
                        shl     ecx, $12  
                        xor     ecx, eax  
                        and     ecx, $CCCC0000  
                        xor     esi, edx  
                        mov     edx, ecx  
                        shr     edx, $12  
                        xor     edx, ecx  
                        xor     eax, edx  
                        mov     ecx, eax  
                        shr     ecx, $01  
                        xor     ecx, esi  
                        and     ecx, $55555555  
                        xor     esi, ecx  
                        add     ecx, ecx  
                        xor     eax, ecx  
                        mov     ecx, esi  
                        shr     ecx, $08  
                        xor     ecx, eax  
                        and     ecx, $0FF00FF  
                        xor     eax, ecx  
                        shl     ecx, $08  
                        xor     esi, ecx  
                        mov     ecx, eax  
                        shr     ecx, $01  
                        xor     ecx, esi  
                        and     ecx, $55555555  
                        lea     edx, dword ptr \[ecx+ecx\]  
                        xor     eax, edx  
                        xor     esi, ecx  
                        mov     ebp, eax  
                        shr     ebp, $0C  
                        mov     ecx, esi  
                        and     ebp, $0FF0  
                        and     ecx, $F000000F  
                        or      ebp, ecx  
                        mov     edx, eax  
                        and     edx, $0FF  
                        shr     ebp, $04  
                        shl     edx, $10  
                        or      ebp, edx  
                        and     eax, $0FF00  
                        or      ebp, eax  
                        and     esi, $0FFFFFFF  
                        push        ecx  
                        lea                ecx,TABLE\_\_009EDEE8  
                        mov     dword ptr \[esp+$18\], ecx  
                        pop                ecx  
                        lea     ecx, dword ptr \[ecx\]  
@L092:  
                        mov     eax, dword ptr \[esp+$14\]  
                        cmp     dword ptr \[eax\], 0  
                        je @L104  
                        mov     ecx, esi  
                        mov     edx, ebp  
                        shl     ecx, $1A  
                        shr     esi, $02  
                        shl     edx, $1A  
                        shr     ebp, $02  
                        or      esi, ecx  
                        or      ebp, edx  
                        jmp @L112  
@L104:  
                        mov     eax, esi  
                        mov     ecx, ebp  
                        shl     eax, $1B  
                        shr     esi, $01  
                        shl     ecx, $1B  
                        shr     ebp, $01  
                        or      esi, eax  
                        or      ebp, ecx  
@L112:  
                        and     esi, $0FFFFFFF  
                        mov     eax, esi  
                        shr     eax, $01  
                        mov     edx, eax  
                        and     edx, $7000000  
                        mov     edi, eax  
                        mov     ecx, esi  
                        and     ecx, $0C00000  
                        or      edx, ecx  
                        shr     edx, $01  
                        and     eax, $0F00  
                        mov     ecx, esi  
                        and     ecx, $100000  
                        or      edx, ecx  
                        shr     edx, $14  
                        and     edi, $60000  
                        mov     ecx, esi  
                        and     ecx, $1E000  
                        or      ecx, edi  
                        mov     edi, dword ptr \[edx\*4+TABLE\_\_0094EC70\]  
                        shr     ecx, $0D  
                        mov     ebx, dword ptr \[ecx\*4+TABLE\_\_0094EB70\]  
                        mov     edx, esi  
                        and     edx, $0C0  
                        or      eax, edx  
                        shr     eax, $06  
                        or      edi, ebx  
                        mov     ebx, dword ptr \[eax\*4+TABLE\_\_0094EA70\]  
                        and     ebp, $0FFFFFFF  
                        mov     eax, esi  
                        and     eax, $3F  
                        mov     ecx, dword ptr \[eax\*4+TABLE\_\_0094E970\]  
                        or      edi, ebx  
                        or      edi, ecx  
                        mov     eax, ebp  
                        shr     eax, $01  
                        mov     ecx, eax  
                        and     ecx, $1E00  
                        mov     edx, ebp  
                        and     edx, $180  
                        or      ecx, edx  
                        and     eax, $6000000  
                        mov     edx, ebp  
                        and     edx, $1E00000  
                        or      eax, edx  
                        shr     eax, $15  
                        mov     edx, dword ptr \[eax\*4+TABLE\_\_0094F070\]  
                        shr     ecx, $7  
                        mov     ebx, dword ptr \[ecx\*4+TABLE\_\_0094EE70\]  
                        or      ebx, edx  
                        mov     eax, ebp  
                        shr     eax, $0F  
                        and     eax, $3F  
                        or      ebx, dword ptr \[eax\*4+TABLE\_\_0094EF70\]  
                        mov     ecx, ebp  
                        and     ecx, $3F  
                        or      ebx, dword ptr \[ecx\*4+TABLE\_\_0094ED70\]  
                        mov     edx, edi  
                        mov     eax, ebx  
                        and     edx, $0FFFF  
                        shl     eax, $10  
                        or      edx, eax  
                        push    $1E  
                        push    edx  
                        call    \_\_0089F57B  
                        mov     ecx, dword ptr \[esp+$20\]  
                        shr     edi, $10  
                        and     ebx, $FFFF0000  
                        mov     dword ptr \[ecx\], eax  
                        add     ecx, $04  
                        push    $1A  
                        or      edi, ebx  
                        push    edi  
                        mov     dword ptr \[esp+$28\], ecx  
                        call    \_\_0089F57B  
                        mov     ecx, dword ptr \[esp+$28\]  
                        add     esp, $10  
                        mov     dword ptr \[ecx\], eax  
                        add     ecx, $04  
                        mov     eax, dword ptr \[esp+$14\]  
                        add     eax, $04  
                        push        ecx  
                        lea                ecx,TABLE\_\_009EDEE8  
                        add                ecx,$40  
                        cmp     eax,ecx  
                        pop                ecx  
                        mov     dword ptr \[esp+$18\], ecx  
                        mov     dword ptr \[esp+$14\], eax  
                        jl @L092  
                        pop     edi  
                        pop     esi  
                        pop     ebp  
                        pop     ebx  
                        retn  
        end;  
end;