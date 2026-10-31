
# [分享]EncryptPE V2.2007.4.11 非S方式 跳过注册的脚本

///////////////////////////////////////////////////////////////////////////////////

// FileName    :  skip\_reg2\_1.txt

// Comment     :  EncryptPE V2.2007.4.11  非S方式 ---- skip\_reg

// Environment :  WinXP SP2,OllyDbg V1.10, OllyScript

// Author      :  d\_h

// Date        :  2008-1-23

// WebSite     :  7d2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0

///////////////////////////////////////////////////////////////////////////////////

```
var WPM

var data1

var data2

var data3

var data4

// 先用OD调试Explorer.exe，然后LifeOD注入Explorer.exe，使用skip\_reg2\_2跳过REG

start1:

    cmp $VERSION, "1.48"

    jb version

    gpa "WriteProcessMemory","kernel32.dll"

    mov WPM, $RESULT

    add WPM , 2

//    bphws WPM ,"x"  //    奇怪，这里用硬件断点行不通

    bpcnd WPM , "[ESP]>=7121000"

next:

    esto

    cmp eip , WPM

    jne next

    mov data1,[esp]

    cmp data1,71121000

    jb next

    cmp data1,7125F000

    ja next

    mov data2,[esp+8]

    cmp data2,71121000

    jb next

    cmp data3,7125F000

    ja next

    mov data3,[esp+0C]

    cmp data2,data3

    jne next

    mov data4,[esp+10]

    cmp data4,0

    je next

    mov [esp+10],0

    log data1

    log data2

    jmp next

version:

    msg "插件版本过低"

    ret
```

///////////////////////////////////////////////////////////////////////////////////

// FileName    :  skip\_reg2\_2.txt

// Comment     :  EncryptPE V2.2007.4.11 非S方式 ---- skip\_reg

// Environment :  WinXP SP2,LifeDbg V1.4, OllyScript 1.65.2

// Author      :  d\_h

// Date        :  2008-1-23

// WebSite     :  50aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0

///////////////////////////////////////////////////////////////////////////////////

```
var patch1

var patch2

var patch3

var p11

var p12

var p13

var p14

var p21

var p22

// 注意：须设置忽略0EEDFADE异常

// 先用OD调试Explorer.exe，需配合使用skip\_reg2\_1脚本，防止Patch被覆盖

Start:

    cmp $VERSION, "1.48"

    jb version

    gpa "IsDebuggerPresent","kernel32.dll"

    bp $RESULT

    esto

    bc $RESULT

next:

    mov  patch1 ,7120350A

    mov [patch1],#7400#   //#7407#

    mov  patch2 ,71201642

    mov [patch2],#EB5E#   //#745E#

    mov  patch3, 711F9561

    mov [patch3],#7400#   //#7406#

    mov  p11, 711FF5F5

    mov [p11],#EB7E#     //#757E#

    mov  p12, 711F41F4

    mov [p12],#750E#     //#7507#

    mov  p13, 71201F8E

    mov [p13],#7400#     //#740F#

    mov  p21, 71201F9D

    mov [p21],#EB0B#     //#750B#

    esto

    jmp next

version:

    msg "插件版本过低"

    ret
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- `1.rar` [javascript:void(0);]
  （1.82kb，220次下载）
