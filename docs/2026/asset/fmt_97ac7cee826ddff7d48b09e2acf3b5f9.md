
# 简单算法――暴风共享软件管理器I V1.0

标 题:简单算法——暴风共享软件管理器I V1.0

发信人:fly

时 间:2003/04/11 10:07am

详细信息:

下载页面：`http://www.skycn.com/soft/10315.html`

软件大小：378 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 杂类工具

应用平台：Win9x/NT/2000/XP

加入时间：2003-01-05 09:01:25

下载次数：383

推荐等级：\*\*\*

开 发 商：`http://www.380000.com/`

## 软件简介

《暴风共享软件管理器I》是一款专业的共享软件管理工具，它能帮助你方便地管理你的共享软件。《暴风共享软件管理器I》利用“接口式动态链接库注册码自动生成系统”可以自动用你提供的算法算出相应的注册码；可以画出各产品销售额、利润、销售量的统计图形；采用适合中国共享软件销售方式的定单式管理风格。《暴风共享软件管理器I》必将成为你管理共享软件的好帮手。

## 软件限制

30天试用。

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

`TRW2000娃娃修改版`、`Ollydbg1.09`、`PEiD`、`W32Dasm 10修改版`

―――――――――――――――――――――――――――――――――

## 过程

`暴风共享软件管理器I.exe` 无壳。Visual C++ 6.0编写。

呵呵，分析完了后看到newlaos兄写的《奇门遁甲演义V6.3》，发觉算法很相似，再看看软件的开发公司，哦，是一家的，“共享”了一套注册算法。看来函数图像大师、鼠到擒来 等等同门软件也是差不多了。

虽然注册码很长，但算法基本的流程是一样的，变换了参数而得到其它几组注册码，所以我只是记录了第一组的算法过程。

用户名：`fly`

试炼码：`12345-67890-ABCDE-FGHIJ-KLMNO`

反汇编，看看参考，很容易就能找到下面的核心。

―――――――――――――――――――――――――――――――――

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408C13(C)
|
:00408C21 8D44242C               lea eax, dword ptr [esp+2C]
:00408C25 6A1E                    push 0000001E
:00408C27 50                      push eax
:00408C28 8D8E0C010000            lea ecx, dword ptr [esi+0000010C]
:00408C2E E88D350000              call 0040C1C0
:00408C33 8D4C240C                lea ecx, dword ptr [esp+0C]
:00408C37 6A1E                    push 0000001E
:00408C39 51                      push ecx
:00408C3A 8D8E1C010000            lea ecx, dword ptr [esi+0000011C]
:00408C40 E87B350000              call 0040C1C0
:00408C45 8D7C242C                lea edi, dword ptr [esp+2C]
:00408C49 83C9FF                  or ecx, FFFFFFFF
:00408C4C 33C0                    xor eax, eax
:00408C4E F2                      repnz
:00408C4F AE                      scasb
:00408C50 F7D1                    not ecx
:00408C52 49                      dec ecx
:00408C53 7511                    jne 00408C66
             ====>填用户名了吗？
:00408C55 6A10                    push 00000010
* Possible StringData Ref from Data Obj ->"错误"
|
:00408C57 680CC24200              push 0042C20C
* Possible StringData Ref from Data Obj ->"没有用户名！"
|
:00408C5C 68F0C34200              push 0042C3F0
:00408C61 E983000000              jmp 00408CE9
*
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408C53(C)
|
:00408C66 8D7C240C                lea edi, dword ptr [esp+0C]
:00408C6A 83C9FF                  or ecx, FFFFFFFF
:00408C6D 33C0                    xor eax, eax
:00408C6F F2                      repnz
:00408C70 AE                      scasb
:00408C71 F7D1                    not ecx
:00408C73 49                      dec ecx
:00408C74 7512                    jne 00408C88
                       ====>填注册码了吗？
:00408C76 8B460C                  mov eax, dword ptr [esi+0C]
:00408C79 6A10                    push 00000010
* Possible StringData Ref from Data Obj ->"错误"
|
:00408C7B 680CC24200              push 0042C20C
* Possible StringData Ref from Data Obj ->"没有注册码！"
|
:00408C80 68E0C34200              push 0042C3E0
:00408C85 50                      push eax
:00408C86 EB65                    jmp 00408CED
*
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408C74(C)
|
:00408C88 8B8E08010000            mov ecx, dword ptr [esi+00000108]
:00408C8E E8DD84FFFF              call 00401170
:00408C93 84C0                    test al, al
:00408C95 7412                    je 00408CA9
                   ====>注册过了吗？呵呵，挺逗。
:00408C97 8B4E0C                  mov ecx, dword ptr [esi+0C]
:00408C9A 6A40                    push 00000040
* Possible StringData Ref from Data Obj ->"你已经注册过了。"
|
:00408C9C 6898C34200              push 0042C398
* Possible StringData Ref from Data Obj ->"你已经注册过了。"
|
:00408CA1 6898C34200              push 0042C398
:00408CA6 51                      push ecx
:00408CA7 EB44                    jmp 00408CED
*
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408C95(C)
|
:00408CA9 8B8E08010000            mov ecx, dword ptr [esi+00000108]
:00408CAF 8D54240C                lea edx, dword ptr [esp+0C]
               ====>EDX=12345-67890-ABCDE-FGHIJ-KLMNO
:00408CB3 8D44242C                lea eax, dword ptr [esp+2C]
               ====>EAX=fly 用户名
:00408CB7 52                      push edx
:00408CB8 50                      push eax
:00408CB9 E88286FFFF              call 00401340
:00408CBE 8B8E08010000            mov ecx, dword ptr [esi+00000108]
:00408CC4 E8A784FFFF              call 00401170
   ====>关键CALL！进入！
:00408CC9 84C0                    test al, al
:00408CCB 6A40                    push 00000040
:00408CCD 7410                    je 00408CDF
 ====>跳则OVER！
:00408CCF 8B4E0C                  mov ecx, dword ptr [esi+0C]
* Possible StringData Ref from Data Obj ->"成功"
   ====>呵呵，胜利女神！
:00408CD2 68D8C34200              push 0042C3D8
* Possible StringData Ref from Data Obj ->"注册将在重启后生效！"
|
:00408CD7 68C0C34200              push 0042C3C0
:00408CDC 51                      push ecx
:00408CDD EB0E                    jmp 00408CED
*
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408CCD(C)
|
* Possible StringData Ref from Data Obj ->"失败"
|
:00408CDF 68B8C34200              push 0042C3B8
* Possible StringData Ref from Data Obj ->"非法注册码"
     ====>BAD BOY！
:00408CE4 68ACC34200              push 0042C3AC
```

―――――――――――――――――――――――――――――――――

进入关键CALL：`408CC4`，`call 00401170`

```asm
* Referenced by a CALL at Addresses:
|:00403D92   , :00408B93   , :00408C8E
  , :00408CC4

……
……省 略…… ……
:00401223 8A4C2425                mov cl, byte ptr [esp+25]
:00401227 B02D                    mov al, 2D
                 ====>AL=2D 即：-
:00401229 3AC8                    cmp cl, al
             ====>比较注册码第6个字符是否是 -
:0040122B 7572                    jne 0040129F
:0040122D 3844242B                cmp byte ptr [esp+2B], al
                     ====>比较注册码第12个字符是否是 -
:00401231 756C                    jne 0040129F
:00401233 38442431                cmp byte ptr [esp+31], al
           ====>比较注册码第18个字符是否是 -
:00401237 7566                    jne 0040129F
:00401239 38442437                cmp byte ptr [esp+37], al
                     ====>比较注册码第24个字符是否是 -
:0040123D 7560                    jne 0040129F
:0040123F 33FF                    xor edi, edi
:00401241 8D742422                lea esi, dword ptr [esp+22]
*
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401294(C)
|
:00401245 8D4C2418                lea ecx, dword ptr [esp+18]
:00401249 8D542440                lea edx, dword ptr [esp+40]
                     ====>EDX=fly
:0040124D 51                      push ecx
:0040124E 57                      push edi
:0040124F 52                      push edx
:00401250 8BCD                    mov ecx, ebp
:00401252 E859000000              call 004012B0
     ====>算法CALL！进入！
       ====>下面是逐位比较！有一处不同就OVER了！
:00401257 8A46FE                  mov al, byte ptr [esi-02]
         ====>[esi-02]=12345
:0040125A 8A4C2418                mov cl, byte ptr [esp+18]
             ====>[esp+18]=1E9TT
第一个大循环得出：1E9TT
第二个大循环得出：5GDGG
第三个大循环得出：72WW8
第四个大循环得出：72WR9
第五个大循环得出：11MGG
:0040125E 3AC1                    cmp al, cl
:00401260 753D                    jne 0040129F
:00401262 8A4EFF                  mov cl, byte ptr [esi-01]
:00401265 8A442419                mov al, byte ptr [esp+19]
:00401269 3AC8                    cmp cl, al
:0040126B 7532                    jne 0040129F
:0040126D 8A16                    mov dl, byte ptr [esi]
:0040126F 8A44241A                mov al, byte ptr [esp+1A]
:00401273 3AD0                    cmp dl, al
:00401275 7528                    jne 0040129F
:00401277 8A4601                  mov al, byte ptr [esi+01]
:0040127A 8A4C241B                mov cl, byte ptr [esp+1B]
:0040127E 3AC1                    cmp al, cl
:00401280 751D                    jne 0040129F
:00401282 8A4E02                  mov cl, byte ptr [esi+02]
:00401285 8A44241C                mov al, byte ptr [esp+1C]
:00401289 3AC8                    cmp cl, al
:0040128B 7512                    jne 0040129F
:0040128D 47                      inc edi
:0040128E 83C606                  add esi, 00000006
:00401291 83FF05                  cmp edi, 00000005
:00401294 7CAF                    jl 00401245
:00408C96
```
