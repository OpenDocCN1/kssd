
```asm
                 <---比较注册码第18位

00488388  JNZ     SHORT LEAPFTP.004883A3

-------------------------------------------

0048838A  MOV     EAX, ECX

                 <---EAX=0x30C,3部分累加值的总和

0048838C  MOV     ECX, 1A

00488391  CDQ

00488392  IDIV    ECX

00488394  ADD     EDX, 41

                 <---EDX=41,对应字符为"A"

00488397  MOV     EAX, [LOCAL.1]

0048839A  CMP     DL, BYTE PTR DS:[EAX+12]

                 <---比较注册码第19位

0048839D  JNZ     SHORT LEAPFTP.004883A3

★★★★★★★★★

0048839F  MOV     BYTE PTR SS:[EBP-5], 1

                 <---置标志位1,程序中应该有校验位

004883A3  XOR     EAX, EAX

                 <---EAX=0!Bad

Boy! :(

004883A5  POP     EDX

004883A6  POP     ECX

004883A7  POP     ECX

004883A8  MOV     DWORD PTR FS:[EAX], EDX

004883AB  PUSH    LEAPFTP.004883C0

004883B0  LEA     EAX, [LOCAL.1]

004883B3  CALL    LEAPFTP.00403CF4

004883B8  RETN
```

由此得到我的正确注册码为:ABC1-EF2G-I3JK-NANA

### 总结

注册码与用户名无关!注册码必须为19位,其中第5,10,15位为"-",注册码的第4,8,12位必须为数字,其余位为大写字母.假设注册码的第1至4位的Hex值累加值为`a`,第6至9位的Hex值累加值为`b`,第11至14位的Hex值累加值为`c`,则注册码的后4位分别为:

第16位---->`Asc(a mod 0x1A+41)`

第17位---->`Asc(b mod 0x1A+41)`

第18位---->`Asc(c mod 0x1A+41)`

第19位---->`Asc[(a+b+c) mod 0x1A+41)]`

软件注册成功后将注册信息保存在注册表的`HKEY_CURRENT_USER\Software\LeapWare\Registry\LeapFTP`下

一组可用注册码:

用户名:ShenGe[BCG]

注册码:ABC1-EF2G-I3JK-NANA

Cracked By ShenGe[BCG]
