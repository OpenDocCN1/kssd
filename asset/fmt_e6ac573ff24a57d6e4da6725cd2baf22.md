
## 汇编代码分析

```asm
:0044E6B1 FF157C104000      Call dword ptr [0040107C]
                            <===注册成功!

:0044E6B7 8D5590           lea edx, dword ptr [ebp-70]

:0044E6BA 52               push edx

:0044E6BB 8D45A0           lea eax, dword ptr [ebp-60]

:0044E6BE 50               push eax

:0044E6BF 8D4DB0           lea ecx, dword ptr [ebp-50] 

...........（略） 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044E64E(C)

|

:0044EB65 C745FC16000000   mov [ebp-04], 00000016

:0044EB6C C7459804000280   mov [ebp-68], 80020004

:0044EB73 C745900A000000   mov [ebp-70], 0000000A

:0044EB7A C745A804000280   mov [ebp-58], 80020004

:0044EB81 C745A00A000000   mov [ebp-60], 0000000A

:0044EB88 C745B804000280   mov [ebp-48], 80020004

:0044EB8F C745B00A000000   mov [ebp-50], 0000000A

:0044EB96 C7458884074100   mov [ebp-78], 00410784

:0044EB9D C7458008000000   mov [ebp-80], 00000008

:0044EBA4 8D5580           lea edx, dword ptr [ebp-80]

:0044EBA7 8D4DC0           lea ecx, dword ptr [ebp-40] 

* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h

                            |

:0044EBAA FF15AC114000     Call dword ptr [004011AC]

:0044EBB0 8D4D90           lea ecx, dword ptr [ebp-70]

:0044EBB3 51               push ecx

:0044EBB4 8D55A0           lea edx, dword ptr [ebp-60]

:0044EBB7 52               push edx

:0044EBB8 8D45B0           lea eax, dword ptr [ebp-50]

:0044EBBB 50               push eax

:0044EBBC 6A00             push 00000000

:0044EBBE 8D4DC0           lea ecx, dword ptr [ebp-40]

:0044EBC1 51               push ecx 

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h

                            |

:0044EBC2 FF157C104000     Call dword ptr [0040107C]

                            <===注册失败!

:0044EBC8 8D5590           lea edx, dword ptr [ebp-70]

:0044EBCB 52               push edx

:0044EBCC 8D45A0           lea eax, dword ptr [ebp-60]

:0044EBCF 50               push eax

:0044EBD0 8D4DB0           lea ecx, dword ptr [ebp-50]

:0044EBD3 51               push ecx

:0044EBD4 8D55C0           lea edx, dword ptr [ebp-40]

:0044EBD7 52               push edx

:0044EBD8 6A04             push 00000004 

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h

                            |

:0044EBDA FF1528104000     Call dword ptr [00401028]

:0044EBE0 83C414           add esp, 00000014

:0044EBE3 C745FC17000000   mov [ebp-04], 00000017

:0044EBEA 8B4508           mov eax, dword ptr [ebp+08]

:0044EBED 8B08             mov ecx, dword ptr [eax]

:0044EBEF 8B5508           mov edx, dword ptr [ebp+08]

:0044EBF2 52               push edx

:0044EBF3 FF9110030000     call dword ptr [ecx+00000310]

:0044EBF9 50               push eax

:0044EBFA 8D45D0           lea eax, dword ptr [ebp-30]

:0044EBFD 50               push eax 
```

## 总结

这个软件的算法真是简单，只是涉及到浮点运算，很适合新手作熟悉浮点运算练手用。详细浮点运算见论坛精华合集。

注册码与用户名无关，只需满足`(n-1978)/983=机器码`这个表达式即可(`n` 为注册码)。

如我的注册码为:`4618407*983+1987=4539896059`

软件将注册信息保存在注册表的`HKEY_LOCAL_MACHINE\Software\hayuguang`下。

Cracked By ShenGe[BCG] 2003.7.31

`4dd466d0e1e0cc123afbe504da2462ca`
