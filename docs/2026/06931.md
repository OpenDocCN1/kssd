
# 木马克星5.33.60破解过程 (9千字)

老是听朋友说现在的网页很多木马（特别是一种叫网络神偷的软件能穿过很多防火墙进入我们的系统进行盗窃），只要你看了它的网页就会中招，所以担心的情况下就马上到华军的网页上下了一个叫“木马克星”的软件，但是要注册码的，反正最近心情一般，就打算拿这个软件来试试，增加一下信心！

## 软件介绍

- 软件名称：`iparmor`
- 整理日期：2002.3.27
- 最新版本：5.33.60
- 文件大小：1392KB
- 软件授权：共享软件
- 使用平台：Win9x/Me/NT/2000
- 发布公司：`http://www.luosoft.com/`

## 软件简介

可以查杀5021种国际木马，112种电子邮件木马，保证查杀冰河类文件关联木马，oicq类寄生木马，icmp类幽灵木马，网络神偷类反弹木马。内置木马防火墙，任何黑客试图与本机建立连接，都需要`Iparmor`确认，不仅可以查杀木马，更可以查黑客。

下载地址：`http://gwbn.onlinedown.net/down/iparmor.exe`

破解工具：`trw2000`（ver:1.23）、`PW32Dasm9b.exe`

破解作者：zwlzwl

信箱：`tsgmx@21cn.com`

## 破解过程

首先用`PW32Dasm9b.exe`把`Iparmor.exe`进行反编译。

在字串参考中找到“软件已经被成功注册”的字符串，再向上能看到下面的代码，大家请跟我一起来分析一下这些代码：

```
:004F6C39 8B8024020000            mov eax, dword ptr [eax+00000224]
:004F6C3F 2D6E020000              sub eax, 0000026E
:004F6C44 E86B31F1FF              call 00409DB4
:004F6C49 8B95ECFEFFFF            mov edx, dword ptr [ebp+FFFFFEEC]
:004F6C4F 8B45FC                  mov eax, dword ptr [ebp-04]
:004F6C52 E861DFF0FF              call 00404BB8〈--因为下面的是关键，所以说明注册码是在这里比较的，所以我们进入去做深入研究
:004F6C57 0F858F000000            jne 004F6CEC〈---关键的跳，不能跳，一跳就出错
:004F6C5D C6050C12510001          mov byte ptr [0051120C], 01
:004F6C64 33D2                    xor edx, edx
:004F6C66 8B83E0020000            mov eax, dword ptr [ebx+000002E0]
:004F6C6C E8830BF4FF              call 004377F4
\* Possible StringData Ref from Code Obj ->"Reistered successfully."
:004F6C71 BA3C774F00              mov edx, 004F773C
:004F6C76 8B83E8020000            mov eax, dword ptr [ebx+000002E8]
:004F6C7C E8670DF4FF              call 004379E8
:004F6C81 803F00                  cmp byte ptr [edi], 00
:004F6C84 7410                    je 004F6C96
\* Possible StringData Ref from Code Obj ->"软件已经被 成功注册."
:004F6C86 BA5C774F00              mov edx, 004F775C
:004F6C8B 8B83E8020000            mov eax, dword ptr [ebx+000002E8]
:004F6C91 E8520DF4FF              call 004379E8
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:004F6C84(C)
:004F6C96 33D2                    xor edx, edx
```

进入了 `:004F6C52 E861DFF0FF call 00404BB8`，我们能看到下面代码，大家请看我的分析：

```
:00404BB8 53                      push ebx
:00404BB9 56                      push esi
:00404BBA 57                      push edi
:00404BBB 89C6                    mov esi, eax〈--使用ESI做EAX的影象，[EAX]里放着假的注册码
:00404BBD 89D7                    mov edi, edx〈--使用EDI做EDX的影象，[EDX]里放着真的注册码
:00404BBF 39D0                    cmp eax, edx〈--比较真假注册码的地址是不是一样
:00404BC1 0F848F000000            je 00404C56〈--如相同就跳到不能注册里去
:00404BC7 85F6                    test esi, esi〈--测试假注册码的地址是不是0（也就是是不是存在）
:00404BC9 7468                    je 00404C33〈--如等于0（不存在）就跳到不能注册里去
:00404BCB 85FF                    test edi, edi〈--测试假注册码的地址是不是0（也就是是不是存在）
:00404BCD 746B                    je 00404C3A〈--如等于0（不存在）就跳到不能注册里去
:00404BCF 8B46FC                  mov eax, dword ptr [esi-04]〈--取得假注册码的位数
:00404BD2 8B57FC                  mov edx, dword ptr [edi-04]〈--取得真注册码的位数
:00404BD5 29D0                    sub eax, edx〈--真假注册码的位数相减
:00404BD7 7702                    ja 00404BDB〈--假注册码的位数大过真的注册码，就跳到不能注册
:00404BD9 01C2                    add edx, eax〈--把假注册码的位数放到EDX中去
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00404BD7(C)
:00404BDB 52                      push edx〈----把假注册码的位数保存起来
:00404BDC C1EA02                  shr edx, 02〈--把位数值（2进制）右移动2位
:00404BDF 7426                    je 00404C07〈--如假注册码的位数小于不大于2就跳到不能注册那里
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00404BFD(C)
:00404BE1 8B0E                    mov ecx, dword ptr [esi]〈--取出假注册码的ASCII码转换的前4位
:00404BE3 8B1F                    mov ebx, dword ptr [edi]〈--取出真注册码的ASCII码转换的前4位
:00404BE5 39D9                    cmp ecx, ebx〈--比较这4个ASCII码
:00404BE7 7558                    jne 00404C41〈--不相等就跳到不能注册
:00404BE9 4A                      dec edx〈--注册码位数计数器减1
:00404BEA 7415                    je 00404C01〈--如计数器等于0就跳到不能注册
:00404BEC 8B4E04                  mov ecx, dword ptr [esi+04]〈--取出假注册码的ASCII码转换的5-8位
:00404BEF 8B5F04                  mov ebx, dword ptr [edi+04]〈--取出真注册码的ASCII码转换的5-8位
:00404BF2 39D9                    cmp ecx, ebx〈--比较这4个ASCII码
:00404BF4 754B                    jne 00404C41〈--不相等就跳到不能注册
:00404BF6 83C608                  add esi, 00000008〈--把地址指针指向第9个假ASCII码
:00404BF9 83C708                  add edi, 00000008〈--把地址指针指向第9个真ASCII码
:00404BFC 4A                      dec edx〈--注册码位数计数器再次减1
:00404BFD 75E2                    jne 00404BE1〈--不等于0就继续跳回上面去进行每次4个ASCII码比较的流程里面去
:00404BFF EB06                    jmp 00404C07〈--如计数器等于0就跳到每次1个ASCII码比较的流程里面去（因为如是正确的注册码现在只剩下9-8=1位ASCII码了）
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00404BEA(C)
:00404C01 83C604                  add esi, 00000004
:00404C04 83C704                  add edi, 00000004
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: |:00404BDF(C), :00404BFF(U)
:00404C07 5A                      pop edx〈--取回假注册码的位数
:00404C08 83E203                  and edx, 00000003〈--截取2进制表达式的最后两位数放到EDX中
:00404C0B 7422                    je 00404C2F〈--如最后两位都是0就跳到不能注册里去
:00404C0D 8B0E                    mov ecx, dword ptr [esi]〈--取出最后一位假ASCII码放到CL中
:00404C0F 8B1F                    mov ebx, dword ptr [edi]〈--取出最后一位假ASCII码放到BL中
:00404C11 38D9                    cmp cl, bl〈--比较最后一位ASCII码
:00404C13 7541                    jne 00404C56〈--不相等就跳到不能注册里去
:00404C15 4A                      dec edx〈--最后两位数 的 位数计数器 减一
:00404C16 7417                    je 00404C2F〈--如位数计数器等于0就比较完毕，跳到正确注册里去
:00404C18 38FD                    cmp ch, bh
:00404C1A 753A                    jne 00404C56
:00404C1C 4A                      dec edx
:00404C1D 7410                    je 00404C2F
:00404C1F 81E30000FF00            and ebx, 00FF0000
:00404C25 81E10000FF00            and ecx, 00FF0000
:00404C2B 39D9                    cmp ecx, ebx
:00404C2D 7527                    jne 00404C56
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: |:00404C0B(C), :00404C16(C), :00404C1D(C)
:00404C2F 01C0                    add eax, eax
:00404C31 EB23                    jmp 00404C56
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00404BC9(C)
:00404C33 8B57FC                  mov edx, dword ptr [edi-04]
:00404C36 29D0                    sub eax, edx
:00404C38 EB1C                    jmp 00404C56
\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:00404BCD(C)
:00404C3A 8B46FC                  mov eax, dword ptr [esi-04]
:00404C3D 29D0                    sub eax, edx
:00404C3F EB15                    jmp 00404C56
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: |:00404BE7(C), :00404BF4(C)
:00404C41 5A                      pop edx
:00404C42 38D9                    cmp cl, bl
:00404C44 7510                    jne 00404C56
:00404C46 38FD                    cmp ch, bh
:00404C48 750C                    jne 00404C56
:00404C4A C1E910                  shr ecx, 10
:00404C4D C1EB10                  shr ebx, 10
:00404C50 38D9                    cmp cl, bl
:00404C52 7502                    jne 00404C56
:00404C54 38FD                    cmp ch, bh
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: |:00404BC1(C), :00404C13(C), :00404C1A(C), :00404C2D(C), :00404C31(U) |:00404C38(U), :00404C3F(U), :00404C44(C), :00404C48(C), :00404C52(C)
:00404C56 5F                      pop edi
:00404C57 5E                      pop esi
:00404C58 5B                      pop ebx
:00404C59 C3                      ret
```

所以我们从上面分析可知道在：

```
:00404BBB 89C6                    mov esi, eax〈--按下D EAX能看到假的注册码
:00404BBD 89D7                    mov edi, edx〈--按下D EDX能看到真的注册码
```

我的为：

```
name:zyx
code:381039972
```

因为我写这篇文章的对象是那些和我一样是菜鸟的朋友看的，所以就尽量详细一点，希望各位高手不要看到我的文章说我写得太烦琐了。

本篇文章只可做交流技术使用，不要用做商业用途。希望大家转载时能保持完整！如有什么地方写得不好，或不对的地方，欢迎大家来信指出。我的信箱：`tsgmx@21cn.com`
