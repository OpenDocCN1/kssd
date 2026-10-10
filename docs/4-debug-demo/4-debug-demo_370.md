

# 功能限制的程序

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-51.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-51.htm)

**功能限制的程序 **　【习题】 

**1、习题一** chap6-1-5-01 菜单选项呈灰色 大小：163K 易

 

 1、习题一 答案         

| 这个程序是利用函数 EnableMenuItem 来使菜单变灰的，用 W32DASM 打开程序，查找 EnableMenuItem，你会发现这部分是重点：       
|   
:004329E4 8B7E58                  mov edi, dword ptr \[esi+58\]   
:004329E7 85FF                    test edi, edi   
:004329E9 7427                    je 00432A12<---如是零则跳过 EnableMenuItem   
:004329EB F6462002                test \[esi+20\], 02   
:004329EF 7521                    jne 00432A12   
:004329F1 33C0                    xor eax, eax   
:004329F3 8AC3                    mov al, bl   
:004329F5 8B0485D4374400          mov eax, dword ptr \[4\*eax+004437D4\]   
:004329FC 83C800                  or eax, 00000000   
:004329FF 50                      push eax   
:00432A00 0FB74644                movzx eax, word ptr \[esi+44\]   
:00432A04 50                      push eax   
:00432A05 8BC7                    mov eax, edi   
:00432A07 E824F4FFFF              call 00431E30   
:00432A0C 50                      push eax   
  
\* Reference To: user32.EnableMenuItem, Ord:0000h   
                                  |   
:00432A0D E89A36FDFF              Call 004060AC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004329E9(C), :004329EF(C)   
|   
:00432A12 33D2                    xor edx, edx   
:00432A14 8BC6                    mov eax, esi   
:00432A16 8B08                    mov ecx, dword ptr \[eax\]   
:00432A18 FF5134                  call \[ecx+34\]   
  
  
这有很多办法可使其跳转，但在这里选用另一法，你在 SOFTICE 下跟踪会发现在:004329E7 一行，EBX 是 0,因此将：   
004329E7 85FF        test edi, edi   
改成：                  test ebx, edi   
菜单功能恢复正常。  |
| --- |

  　

 

 4dd466d0e1e0cc123afbe504da2462ca