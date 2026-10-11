
# 万能五笔 2002

> 原文：[`www.pediy.com/kssd/pediy06/pediy6026.htm`](https://www.pediy.com/kssd/pediy06/pediy6026.htm)

`万能五笔 2002` 这个软件好像已经有人分析了,不过我没看到!我是昨天下午从作者主页下载的,拿回来后就开始分析了,如果一定要说我是抄回来的我也没办法! (不过下面的东西绝对是我自己在家花了一晚时间整理的,绝对没有抄袭!!!) 

第一次做这么详细的分析,可能有很多地方错误,请大哥哥\大姐姐指正和支持! 

**【软件简介】**: 本系统的目的在于提供一种万能的多元汉字输入法，通过这种输入法，可以克服现有技术的缺点，以多重（多元）汉字编码代替传统的单一编码。具体地说，就是要设计并实现一种包含多种互不冲突、相辅相成、相互取长补短的汉字编码方案的万能汉字输入法，即：在一种汉字编码输入状态下，对任一个汉字或词组短语，同时存在多种编码输入途径，从而提供更便利、更高效的编码输入。

**【下载页面】**: [`http://www.tt98.com/wnwb2002/wnwb_rjxz.htm`](http://www.tt98.com/wnwb2002/wnwb_rjxz.htm)

**【软件大小】**: `1556K`

**【软件限制】**: 试用 `200` 次...还有就是不知道了!呵呵...

**【作者声明】**: 只是个人兴趣,读者看后所做出的一切事情与我无关,我也不负责,希望读者能三思再后行!

**【破解工具】**: `w32Dasm` `TRW2K` `PEid`

---

**【过 程】**: 先用 `w32Dasm` 反它,根据参考很快就找到关键!下断 `bpx 40EB98` `F5` 返回,输入假的注册码 `7878787878787878` 一定要大于 `16` 位,按 `OK` 拦下来到下面:

```
:0040EB98 8B742474                 mov esi, dword ptr [esp+74]

* Reference To: USER32.GetDlgItemTextA, Ord:0104h
                        |
:0040EB9C 8B3D40524200             mov edi, dword ptr [00425240]
:0040EBA2 8D4C243C                 lea ecx, dword ptr [esp+3C]
:0040EBA6 6A32                     push 00000032
:0040EBA8 51                       push ecx
:0040EBA9 68F7030000               push 000003F7
:0040EBAE 56                       push esi
:0040EBAF FFD7                     call edi
:0040EBB1 8D542428                 lea edx, dword ptr [esp+28]
:0040EBB5 6A11                     push 00000011
:0040EBB7 52                       push edx
:0040EBB8 68F8030000               push 000003F8
:0040EBBD 56                       push esi
:0040EBBE FFD7                     call edi
:0040EBC0 6A04                     push 00000004
:0040EBC2 56                       push esi

* Reference To: USER32.ShowWindow, Ord:026Ah
                        |
:0040EBC3 FF15E8514200             Call dword ptr [004251E8]
:0040EBC9 8D7C243C                 lea edi, dword ptr [esp+3C] \\\取得发布网站的名称
:0040EBCD 83C9FF                   or ecx, FFFFFFFF
:0040EBD0 33C0                     xor eax, eax
:0040EBD2 F2                       repnz                               \\\这个是什么用的就不清楚了！
:0040EBD3 AE                       scasb                               \\\这个是什么用的就不清楚了！
:0040EBD4 F7D1                     not ecx                             \\\取反
:0040EBD6 49                       dec ecx                             \\\减一
:0040EBD7 83F904                   cmp ecx, 00000004                   \\\?!比较发布网站是否小于 4 个字节,我真搞不懂,那个筐里的东西是灰色的根本填不了东西可,就是小于四个字节也不关我事啊!!晕哦...
:0040EBDA 7342                     jnb 0040EC1E                        \\\小于就不跳,死掉!不小于就跳到下面!
:0040EBDC A1F08F4300               mov eax, dword ptr [00438FF0]
:0040EBE1 6A10                     push 00000010
:0040EBE3 85C0                     test eax, eax

* Possible StringData Ref from Data Obj ->"Error"
                        |
:0040EBE5 6800804200               push 00428000
:0040EBEA 7519                     jne 0040EC05

* Possible StringData Ref from Data Obj ->"发布网站名不能小于 4 个字节!"
                        |
:0040EBEC 686CA74200               push 0042A76C
:0040EBF1 56                       push esi

--------------------------------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EBDA(C)
|
:0040EC1E 8D7C2428                 lea edi, dword ptr [esp+28]   \\\从 0040EBDA 跳到这里!EDI 里是我输入的注册码!
:0040EC22 83C9FF                   or ecx, FFFFFFFF
:0040EC25 33C0                     xor eax, eax
:0040EC27 F2                       repnz                         \\\?????????
:0040EC28 AE                       scasb                         \\\?????????
:0040EC29 F7D1                     not ecx                       \\\取反
:0040EC2B 49                       dec ecx                       \\\减一
:0040EC2C 83F910                   cmp ecx, 00000010             \\\比较注册码是否大于 16 位!不明白的是怎么取得注册码的位数的!可能是我太菜了...
:0040EC2F 7442                     je 0040EC73                   \\\大于就跳到下面,否则就死掉!
:0040EC31 A1F08F4300               mov eax, dword ptr [00438FF0]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EC91(C)
|
:0040EC36 85C0                     test eax, eax
:0040EC38 6A10                     push 00000010

* Possible StringData Ref from Data Obj ->"Error"
                        |
:0040EC3A 6800804200               push 00428000
:0040EC3F 7519                     jne 0040EC5A

* Possible StringData Ref from Data Obj ->"解码失败!"
                        |
:0040EC41 6844A74200               push 0042A744
:0040EC46 56                       push esi

------------------------------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EC2F(C)
|
:0040EC73 8D442428                 lea eax, dword ptr [esp+28]   \\\从 0040EC2F 跳到这里!EAX 装的是我输入的注册码!
:0040EC77 6800994300               push 00439900                 \\\这里是机器码压入堆栈!
:0040EC7C 8D4C2440                 lea ecx, dword ptr [esp+40]   \\\ECX 装的是发布网站 WWW.TT98.COM!
:0040EC80 50                       push eax                       \\\我输入的注册码压栈
:0040EC81 51                       push ecx                       \\\[www.TT98.COM 压栈](http://www.TT98.COM 压栈)
:0040EC82 E8D925FFFF               call 00401260                 \\\关键 CALL F8 追进去!好眼熟啊,经典比较!!
:0040EC87 83C40C                   add esp, 0000000C
:0040EC8A 85C0                     test eax, eax                 \\\注册码是否正确
:0040EC8C A1F08F4300               mov eax, dword ptr [00438FF0]
:0040EC91 74A3                     je 0040EC36                   \\\不正确就跳回去死掉!!!
:0040EC93 85C0                     test eax, eax
:0040EC95 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Success"
                        |
:0040EC97 68909F4200               push 00429F90
:0040EC9C 7507                     jne 0040ECA5

* Possible StringData Ref from Data Obj ->"解码成功，感谢您对国产共享软件的支持!"
                        |
:0040EC9E 6810A74200               push 0042A710
:0040ECA3 EB05                     jmp 0040ECAA

------------------------------------------------------------------
```

上面 `F8` 到这里:

```
:00401260 83EC14                 sub esp, 00000014
:00401263 53                       push ebx
:00401264 55                       push ebp
:00401265 8B6C2420                 mov ebp, dword ptr [esp+20]   \\\这里是 www.TT98.com
:00401269 56                       push esi
:0040126A 57                       push edi
:0040126B C644242000               mov [esp+20], 00

* Possible StringData Ref from Data Obj ->"[www.TT98.net"](http://www.TT98.net")
                        |
:00401270 BE40794200               mov esi, 00427940             \\\这里是 www.TT98.net
:00401275 8BC5                     mov eax, ebp                  \\\这里是 www.TT98.com

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401299(C)
|
:00401277 8A10                     mov dl, byte ptr [eax]         \\\把 www.TT98.com 的第一个字符的 ASCII 码放入 EDX 的低位!
:00401279 8A1E                     mov bl, byte ptr [esi]         \\\把 www.TT98.net 的第一个字符的 ASCII 码放入 EBX 的低位!
:0040127B 8ACA                     mov cl, dl
:0040127D 3AD3                     cmp dl, bl                    \\\比较
:0040127F 751E                     jne 0040129F                  \\\不等于就跳下去死
:00401281 84C9                     test cl, cl                   \\\是否为零
:00401283 7416                     je 0040129B                   \\\是就跳下去死
:00401285 8A5001                   mov dl, byte ptr [eax+01]     \\\把 www.TT98.com 的第二个字符的 ASCII 码放入 EDX 的低位!
:00401288 8A5E01                   mov bl, byte ptr [esi+01]     \\\把 www.TT98.net 的第二个字符的 ASCII 码放入 EBX 的低位!
:0040128B 8ACA                     mov cl, dl
```



```
:0040128D 3AD3 cmp dl, bl \\比较
:0040128F 750E jne 0040129F \\不等于就跳下去死
:00401291 83C002 add eax, 00000002 \\EAX 加二之后就变成 w.TT98.com
:00401294 83C602 add esi, 00000002 \\ESI 加二之后就变成 w.TT98.net
:00401297 84C9 test cl, cl \\是否取完
:00401299 75DC jne 00401277 \\没有取完就跳上去继续↑

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401283(C)

:0040129B 33C0 xor eax, eax \\清零
:0040129D EB05 jmp 004012A4 \\跳去死!

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040127F(C), :0040128F(C)

:0040129F 1BC0 sbb eax, eax
:004012A1 83D8FF sbb eax, FFFFFFFF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040129D(U)

:004012A4 85C0 test eax, eax
:004012A6 750D jne 004012B5 \\这里不跳的话就死得好惨...
:004012A8 5F pop edi
:004012A9 5E pop esi
:004012AA 5D pop ebp
:004012AB B801000000 mov eax, 00000001
:004012B0 5B pop ebx
:004012B1 83C414 add esp, 00000014
:004012B4 C3 ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004012A6(C)

:004012B5 8BFD mov edi, ebp \\这里是发布网站的名称 www.TT98.com
:004012B7 83C9FF or ecx, FFFFFFFF
:004012BA 33C0 xor eax, eax
:004012BC F2 repnz
:004012BD AE scasb
:004012BE F7D1 not ecx
:004012C0 49 dec ecx
:004012C1 83F904 cmp ecx, 00000004 \\发布网站得名称是否大于四个字节!它是怎么取得发布网站的字节的我搞不懂,可能是我太菜了吧!
:004012C4 0F82C0000000 jb 0040138A \\不大于就跳去死掉!
:004012CA 8B5C242C mov ebx, dword ptr [esp+2C] \\我输入的注册码
:004012CE 83C9FF or ecx, FFFFFFFF
:004012D1 8BFB mov edi, ebx \\我输入的注册码
:004012D3 F2 repnz
:004012D4 AE scasb
:004012D5 F7D1 not ecx
:004012D7 49 dec ecx
:004012D8 83F910 cmp ecx, 00000010 \\比较注册码是否大于 16 位!它是怎么取得我输入注册码的字节的我搞不懂,可能是我太菜了吧!
:004012DB 0F85A9000000 jne 0040138A \\不大于得话就跳去死!
:004012E1 8B542430 mov edx, dword ptr [esp+30]
:004012E5 83C9FF or ecx, FFFFFFFF
:004012E8 8BFA mov edi, edx
:004012EA F2 repnz
:004012EB AE scasb
:004012EC F7D1 not ecx
:004012EE 2BF9 sub edi, ecx
:004012F0 8BC1 mov eax, ecx
:004012F2 8BF7 mov esi, edi
:004012F4 8BFD mov edi, ebp
:004012F6 C1E902 shr ecx, 02
:004012F9 F3 repz
:004012FA A5 movsd
:004012FB 8BC8 mov ecx, eax
:004012FD 33C0 xor eax, eax
:004012FF 83E103 and ecx, 00000003
:00401302 F3 repz
:00401303 A4 movsb
:00401304 8BFA mov edi, edx
:00401306 83C9FF or ecx, FFFFFFFF
:00401309 F2 repnz
:0040130A AE scasb
:0040130B F7D1 not ecx
:0040130D 2BF9 sub edi, ecx
:0040130F 8BC1 mov eax, ecx
:00401311 8BF7 mov esi, edi
:00401313 8944242C mov dword ptr [esp+2C], eax
:00401317 8BFD mov edi, ebp
:00401319 83C9FF or ecx, FFFFFFFF
:0040131C 33C0 xor eax, eax
:0040131E F2 repnz
:0040131F AE scasb
:00401320 8B44242C mov eax, dword ptr [esp+2C]
:00401324 4F dec edi
:00401325 8BC8 mov ecx, eax
:00401327 C1E902 shr ecx, 02
:0040132A F3 repz
:0040132B A5 movsd
:0040132C 8BC8 mov ecx, eax
:0040132E 8B442428 mov eax, dword ptr [esp+28]
:00401332 83E103 and ecx, 00000003
:00401335 F3 repz
:00401336 A4 movsb
:00401337 8D4C2410 lea ecx, dword ptr [esp+10]
:0040133B 51 push ecx \\这个不知道是什么.可能是 C 盘或者是密匙!
:0040133C 50 push eax \\这里是两个机器码连在一起,也是注册标志!
:0040133D 52 push edx \\这里是机器码
:0040133E E8ADFEFFFF call 004011F0 \\这个 CALL 就是注册算法!
\\想知道算法的话就追进去!
\\我追了一会儿,头晕了就没追了!
\\其实是我的汇编太差,看得不太明白!
\\但有一些地方还是看得懂的,一会总结分析!
:00401343 83C40C add esp, 0000000C
:00401346 8D742410 lea esi, dword ptr [esp+10] \\这里就是真的注册码了!
:0040134A 8BC3 mov eax, ebx \\我输入的假注册码!下面是比较!

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040136E(C)

:0040134C 8A10 mov dl, byte ptr [eax] \\假码的第一位的 ASCII 码放入 EDX 的低位!
:0040134E 8A1E mov bl, byte ptr [esi] \\真码的第一位的 ASCII 码放入 EBX 的低位!
:00401350 8ACA mov cl, dl \\CL=DL
:00401352 3AD3 cmp dl, bl \\比较
:00401354 751E jne 00401374 \\不等于就跳去死!
:00401356 84C9 test cl, cl \\注册码第一位是否为零!
:00401358 7416 je 00401370 \\是零的话就跳去死!
:0040135A 8A5001 mov dl, byte ptr [eax+01] \\假码的第二位的 ASCII 码放入 EDX 的低位!
:0040135D 8A5E01 mov bl, byte ptr [esi+01] \\真码的第二位的 ASCII 码放入 EBX 的低位!
:00401360 8ACA mov cl, dl \\CL=DL
:00401362 3AD3 cmp dl, bl \\比较第二位是否相等!
:00401364 750E jne 00401374 \\不相等就跳去死!
:00401366 83C002 add eax, 00000002 \\假码加二(这里和上面 00401291 道理相同)
:00401369 83C602 add esi, 00000002 \\真码加二(这里和上面 00401292 道理相同)
:0040136C 84C9 test cl, cl \\注册码第二位是否为零!
:0040136E 75DC jne 0040134C \\不为零的话就跳回去继续比较!

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401358(C)

:00401370 33C0 xor eax, eax \\清零
:00401372 EB05 jmp 00401379 \\跳去死!

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401354(C), :00401364(C)

:00401374 1BC0 sbb eax, eax
:00401376 83D8FF sbb eax, FFFFFFFF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401372(U)

:00401379 85C0 test eax, eax
:0040137B 750D jne 0040138A
:0040137D 5F pop edi
```



### 汇编代码片段

```
:0040137E 5E                           pop esi  
:0040137F 5D                           pop ebp  
:00401380 B801000000                   mov eax, 00000001  
:00401385 5B                           pop ebx  
:00401386 83C414                     add esp, 00000014  
:00401389 C3                           ret
```

### 内存注册机信息

- 中断地址：`40134A`
- 中断次数：`1`
- 第一字节：`8B`
- 指令长度：`2`
- 内存方式：寄存器 `ESI`
- 注册码格式：`XXXX-XXXXXX-XXXXX`

### 注册信息修改

注册信息保存在同目录下的 `wnb.ini` 文件的最下面那一行：

```
serialno=XXXX-XXXXXX-XXXX
```

把这一行删掉就又变成未注册版本了。

### 机器码分析

曾追进：

```
:0040133E E8ADFEFFFF                   call 004011F0
```

后发现程序是把机器码（我的机器码是 `527901190`）的 ASCII 的和 `X` 与某个数 `Y` 运算后得出一个数 `Z`，`Z` 的十进制就十注册码的前面部分！再把 `Z` 这个数的十进制乘以二就十后面部分，中间部分是怎么来的我就真的搞不懂了！唉........

我的机器码的 ASCII 和是 `1D2`，再与某个数运算后得出 `CFC`。

`CFC` 转为十进制是 `3324`。

我的注册码是 `3324-594401-6648`。

整个分析就到此结束了！谢谢您的观赏！

2003.05.11 晚于清远

`4dd466d0e1e0cc123afbe504da2462ca`
