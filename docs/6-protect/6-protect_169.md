
# AutoCAD 加载的 ARX

> 原文：[`www.pediy.com/kssd/pediy04/kanxue126.htm`](https://www.pediy.com/kssd/pediy04/kanxue126.htm)

某 AutoCAD 加载的 ARX 程序，Sense3 的狗， 既然解狗就要找“狗”，查找“狗”，共两处，改掉之后就 OK 了！

难道这也叫解狗！

```
Exported fn(): acrxEntryPoint - Ord:0002h

:1C05CF00 8B442404                    mov eax, dword ptr [esp+04]

:1C05CF04 48                          dec eax

:1C05CF05 83F804                      cmp eax, 00000004

:1C05CF08 0F878C000000                ja 1C05CF9A

:1C05CF0E FF2485A0CF051C              jmp dword ptr [4*eax+1C05CFA0]

:1C05CF15 8B442408                    mov eax, dword ptr [esp+08]

:1C05CF19 50                          push eax

* Reference To: ACAD.acrxUnlockApplication, Ord:0D5Bh

:1C05CF1A E8BDC00800                  Call 1C0E8FDC

:1C05CF1F 83C404                      add esp, 00000004

:1C05CF22 E8C9AAFEFF                  call 1C0479F0

:1C05CF27 85C0                        test eax, eax

:1C05CF29 7505                        jne 1C05CF30

:1C05CF2B E8C0AAFEFF                  call 1C0479F0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C05CF29(C)

:1C05CF30 E84BFEFFFF                  call 1C05CD80

:1C05CF35 E8A6A8FEFF                  call 1C0477E0

:1C05CF3A E8019CFEFF                  call 1C046B40

:1C05CF3F A388E00F1C                  mov dword ptr [1C0FE088], eax

:1C05CF44 85C0                        test eax, eax

:1C05CF46 7521                        jne 1C05CF69    ×××××改 7521 为 7421，启动时跳过检查狗×××

:1C05CF48 6A00                        push 00000000

:1C05CF4A 6A04                        push 00000004

:1C05CF4C E82FAAFEFF                  call 1C047980

:1C05CF51 83C408                      add esp, 00000008

:1C05CF54 E807ABFEFF                  call 1C047A60

* Possible StringData Ref from Data Obj ->"

软件出现致命错误,请检查加密狗是否正确!"

:1C05CF59 6888560F1C                  push 1C0F5688

* Reference To: ACAD.acrx_abort, Ord:0D5Dh

:1C05CF5E E8CBBF0800                  Call 1C0E8F2E

:1C05CF63 83C404                      add esp, 00000004

:1C05CF66 33C0                        xor eax, eax

:1C05CF68 C3                          ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C04674A(C)

:1C046753 66C7442404AF07              mov [esp+04], 07AF

:1C04675A 66C74424060700              mov [esp+06], 0007

:1C046761 66C74424081A00              mov [esp+08], 001A

:1C046768 66C7442402FFFF              mov [esp+02], FFFF

:1C04676F 8D442400                    lea eax, dword ptr [esp]

:1C046773 50                          push eax

:1C046774 E8C7290A00                  call 1C0E9140

:1C046779 66837C240001                cmp word ptr [esp], 0001

:1C04677F 1BC0                        sbb eax, eax

:1C046781 83E002                      and eax, 00000002

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1C046751(U)

:1C046784 83F801                      cmp eax, 00000001

:1C046787 7418                        je 1C0467A1      ××74 改为 75 跳过××

:1C046789 83F802                      cmp eax, 00000002

:1C04678C 7413                        je 1C0467A1      ××74 改为 75 跳过 ××

* Possible StringData Ref from Data Obj ->"01 加密狗不存在!"××此处为每条命令执行时检查加密狗××

:1C04678E 68A03C0F1C                  push 1C0F3CA0

* Reference To: ACAD.ads_printf, Ord:0DC6h

:1C046793 E88E260A00                  Call 1C0E8E26

:1C046798 83C404                      add esp, 00000004

:1C04679B 33C0                        xor eax, eax

:1C04679D 83C460                      add esp, 00000060

:1C0467A0 C3                          ret
```
4dd466d0e1e0cc123afbe504da2462ca
