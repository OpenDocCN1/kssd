
# 腾博条形码生产-管理系统

> 原文：[`www.pediy.com/kssd/pediy03/forum1096.htm`](https://www.pediy.com/kssd/pediy03/forum1096.htm)

作者：`xy2000[BCG]`

腾博条形码生产者-管理系统

下载地址：`http://www.csdn.net/soft/openfile.asp?kind=1&id=11414`

工具：`W32DSM`（大老专版），`TRW2000`

```
* Possible StringData Ref from Data Obj ->"注册号："
                              |
:0040B263 BACA7E5000                mov edx, 00507ECA
:0040B268 8D45CC                   lea eax, dword ptr [ebp-34]
:0040B26B E880600E00                call 004F12F0
:0040B270 FF461C                   inc [esi+1C]

* Possible StringData Ref from Data Obj ->"请输入注册号"
                              |
:0040B273 BABD7E5000                mov edx, 00507EBD
:0040B278 8B08                      mov ecx, dword ptr [eax]
:0040B27A 8D45D0                   lea eax, dword ptr [ebp-30]
:0040B27D 51                        push ecx
:0040B27E 66C746107400             mov [esi+10], 0074
:0040B284 E867600E00                call 004F12F0
:0040B289 FF461C                   inc [esi+1C]
:0040B28C 8B00                      mov eax, dword ptr [eax]
:0040B28E 8D4DFC                   lea ecx, dword ptr [ebp-04]
:0040B291 5A                        pop edx
:0040B292 E8C5D30900                call 004A865C
:0040B297 50                        push eax
:0040B298 FF4E1C                   dec [esi+1C]
:0040B29B 8D45CC                   lea eax, dword ptr [ebp-34]
:0040B29E BA02000000                mov edx, 00000002
:0040B2A3 E8A8610E00                call 004F1450
:0040B2A8 FF4E1C                   dec [esi+1C]
:0040B2AB 8D45D0                   lea eax, dword ptr [ebp-30]
:0040B2AE BA02000000                mov edx, 00000002
:0040B2B3 E898610E00                call 004F1450
:0040B2B8 59                        pop ecx
:0040B2B9 84C9                      test cl, cl
:0040B2BB 0F845CFFFFFF              je 0040B21D
:0040B2C1 66C746108000             mov [esi+10], 0080

* Possible StringData Ref from Data Obj ->"Tb1765431Xm" 大家看看这是什么，注册码是明文，哈哈
                              |
:0040B2C7 BAD37E5000                mov edx, 00507ED3
:0040B2CC 8D45C8                   lea eax, dword ptr [ebp-38] 这里下 d edx  也可看到注册码
:0040B2CF E81C600E00                call 004F12F0
:0040B2D4 FF461C                   inc [esi+1C]
:0040B2D7 8D55C8                   lea edx, dword ptr [ebp-38]
:0040B2DA 8D45FC                   lea eax, dword ptr [ebp-04]
:0040B2DD E852620E00                call 004F1534
:0040B2E2 50                        push eax
:0040B2E3 FF4E1C                   dec [esi+1C]
:0040B2E6 8D45C8                   lea eax, dword ptr [ebp-38]
:0040B2E9 BA02000000                mov edx, 00000002
:0040B2EE E85D610E00                call 004F1450
:0040B2F3 59                        pop ecx
:0040B2F4 84C9                      test cl, cl
:0040B2F6 741D                      je 0040B315
:0040B2F8 A1BC2F5100                mov eax, dword ptr [00512FBC]
:0040B2FD 6A00                      push 00000000  4dd466d0e1e0cc123afbe504da2462ca
```
