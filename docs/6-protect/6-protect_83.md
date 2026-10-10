

# HackShiled4.3.0.5 log 文件加密算法

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-299.htm`](https://www.pediy.com/kssd/pediy09/pediy09-299.htm)

> 算法很简单，汇编代码如下  
> 
> 代码:
> 
> * * *
> 
> `int _declspec(naked) _cdecl HSEncLog(unsigned char *pOutBuf, int nOutlen, unsigned char *pInBuf, int nInlen, int key)   {       _asm       {       mov     edx, dword ptr [esp+4]       push    ebx       test    edx, edx       push    esi       je L054       mov     esi, dword ptr [esp+14h]       test    esi, esi       je L054       mov     ecx, dword ptr [esp+10h]       test    ecx, ecx       jle L054       mov     ebx, dword ptr [esp+18h]       test    ebx, ebx       jle L054       push    ebp       mov     ebp, ecx       push    edi       xor     eax, eax       mov     edi, edx       shr     ecx, 2       rep     stos dword ptr es:[edi]       mov     ecx, ebp       and     ecx, 3       test    ebx, ebx       rep     stos byte ptr es:[edi]       jle L048       mov     ecx, dword ptr [esp+24h]       sub     esi, edx       mov     edi, ebx   L029:       mov     al, byte ptr [esi+edx]       xor     ebx, ebx       mov     bl, ch       xor     al, bl       mov     byte ptr [edx], al       movzx   ax, al       add     eax, ecx       lea     ecx, dword ptr [eax+eax*2]       shl     ecx, 4       sub     ecx, eax       lea     ecx, dword ptr [ecx+ecx*2]       lea     ecx, dword ptr [ecx+ecx*4]       lea     ecx, dword ptr [ecx+ecx*8]       lea     eax, dword ptr [eax+ecx*2]       mov     ecx, 58BFh       sub     ecx, eax       inc     edx       dec     edi       jnz L029   L048:       pop     edi       pop     ebp       pop     esi       xor     eax, eax       pop     ebx       retn   L054:       pop     esi       or      eax, 0FFFFFFFFh       pop     ebx       retn       }   }` 
> 
> * * *
> 
> 逆成 c 代码  
>   
> 代码:
> 
> * * *
> 
> `int HSEncLog(unsigned char * pOutBuf, DWORD nOutLen, unsigned char * pInBuf, DWORD nInLen, WORD key)   {     if (pInBuf == NULL || nOutLen == 0 || pInBuf == NULL || nInLen == 0 || key == 0)     {       return -1;     }        ZeroMemory(pOutBuf, nInLen);          for (DWORD i = 0; i < nInLen; i ++)     {       pOutBuf[i] = pInBuf[i] ^ (key >> 8) & 0xFF;       key = 0x58BF - (pOutBuf[i] + key) * 0x3193;     }     return 0;   }` 
> 
> * * *
> 
> 解密代码稍微改动下就 ok 了。  
> 有研究 HackShiled 的共同交流下，手头版本都是加了 Themida 的，有些代码被 vm 过了，哪位手头有稍旧点版本的能否发我一份？  
> 另外有哪位保存了娃娃发过的那一小段代码的也 pm 一份吧![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

 4dd466d0e1e0cc123afbe504da2462ca