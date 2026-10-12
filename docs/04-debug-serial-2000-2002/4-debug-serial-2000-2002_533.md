
# webeasymail

> 原文：`www.pediy.com/kssd/pediy03/forum1198.htm`

`webeasymail` 的简单破解过程

```
:00415C96 B8FC260000           mov eax, 000026FC

:00415C9B E8E09B0300           call 0044F880

:00415CA0 55                    push ebp

:00415CA1 56                    push esi

:00415CA2 57                    push edi

:00415CA3 894C240C               mov dword ptr [esp+0C], ecx

:00415CA7 E894FDFFFF             call 00415A40

:00415CAC 85C0                   test eax, eax

:00415CAE 7518                   jne 00415CC8  <-75 改 EB(在 win98 和 winme 就能运行)

:00415CB0 6A10                   push 00000010

* Possible Reference to Dialog: 

                              |

:00415CB2 6860324900             push 00493260

* Possible StringData Ref from Data Obj ->"您的操作系统不是 WinNT 或且 Win2000" 

                              |

:00415CB7 683C324900             push 0049323C

:00415CBC 50                    push eax

__________________________________________________________

:0040558D E831140600           call 004669C3

:00405592 A13C6C4900           mov eax, dword ptr [00496C3C]

:00405597 89442404             mov dword ptr [esp+04], eax

:0040559B C744241800000000     mov [esp+18], 00000000

:004055A3 8944240C             mov dword ptr [esp+0C], eax

:004055A7 8B8D3C040000         mov ecx, dword ptr [ebp+0000043C]<-你的序列号

:004055AD 8D853C040000         lea eax, dword ptr [ebp+0000043C]<-序列号地址

:004055B3 C644241801           mov [esp+18], 01

:004055B8 8B51F8               mov edx, dword ptr [ecx-08]

:004055BB 85D2                 test edx, edx

:004055BD 0F84D5010000         je 00405798

:004055C3 8B9538040000         mov edx, dword ptr [ebp+00000438]

:004055C9 8B4AF8               mov ecx, dword ptr [edx-08] <-你输入的注册码

:004055CC 85C9                 test ecx, ecx

:004055CE 0F84C4010000         je 00405798

:004055D4 83F90A               cmp ecx, 0000000A

:004055D7 0F8EBB010000         jle 00405798

:004055DD 50                    push eax

:004055DE 8D4C2408             lea ecx, dword ptr [esp+08]

:004055E2 E866EB0500           call 0046414D

:004055E7 8D442404             lea eax, dword ptr [esp+04]

:004055EB 50                    push eax

:004055EC E84F460200           call 00429C40

:004055F1 8B8538040000         mov eax, dword ptr [ebp+00000438]

:004055F7 8B4C2408             mov ecx, dword ptr [esp+08]

:004055FB 50                    push eax

:004055FC 51                    push ecx<-真正的注册码^^^(哇，kao32 位耶太长了吧，呵呵)

:004055FD E88E180500           call 00456E90
```

注：成功注册后，在根目录下生成`SN.INI`

附：我的`SN.INI` `2`<-代表无限用户(1<-代表 250 用户)

`81721b50a99d1aa07d5bef07c6b095a0`

我的特征码：`2-00-90-D0-0D-17-9F`

已修改和脱壳的主文件下载地址：`http://crackbaby.y365.com/esyamail.zip`

`CRACKCODE2000` 做个注册机我想应该没问题 (不知道能否显示 32 位？) `4dd466d0e1e0cc123afbe504da2462ca`
