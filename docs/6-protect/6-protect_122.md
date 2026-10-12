
# Rockey4

> 原文：`[www.pediy.com/kssd/pediy06/pediy6001.htm](https://www.pediy.com/kssd/pediy06/pediy6001.htm)`

大老的打狗教程第三篇(最终篇)如何解掉,`rockey4` 的狗加密的软件!希望对大家有所帮助!大老=`[DCG]=`

软件名:国内北京某著名婚纱摄影设计制作软件

保护 :北京飞天诚信公司公司坚石的狗(`rockey4`)

所用工具:`trw2000`, `wasm32`, `hiew688`

破解难度:难

破解人:大老

所属组织:`=BCG=` `=[DCG]=`

本人作品:文件加密狗检测工具 2.0

本人邮箱:`dalao@qdcnc.com`, `dalao@top86.com`

本人主页:`http://dalao2002.yeah.net`

本人论坛:`http://61.177.65.168/dalaobbs/cgi-bin/leoboard.cgi`

Oicq:`79234668`

此文献给所有爱好解密的朋友们!

我写的打狗教程这是第三篇!也是最后一篇了!这一篇我写两部分!其中第一部分是狗壳,第二部分是程序本身的解密了!

如果你看过我的第一篇 `HASP` 解密教程的话!会对你解 `ROCKEY` 狗有所帮助!

`ROCKEY` 狗和 `HASP` 狗读狗调用有些方面是很相似的!

我只是大体说一下破解的思路! 高手不要见笑呀!

希望对大家有所帮助!

## (第一部分) =狗壳=

我来讲讲 `rockey4` 的外壳!`rockey4` 的外壳做得不错!兼容性非常好!保护后有程序的每个段需要 4 组狗里的

返回数据来还原!遗憾的是这个外壳的花指令基本上没有!好了不说了!GO GO..

### (1)读狗部分

```
:004B1935 83C408                 add esp, 00000008
:004B1938 8D4C2408              lea ecx, dword ptr [esp+08]
:004B193C 8D542406              lea edx, dword ptr [esp+06]
:004B1940 8D442430              lea eax, dword ptr [esp+30]
:004B1944 6A00                  push 00000000
:004B1946 51                    push ecx
:004B1947 8B4E0A                mov ecx, dword ptr [esi+0A]
:004B194A 6A02                  push 00000002
:004B194C 52                    push edx
:004B194D 6A28                  push 00000028
:004B194F 50                    push eax
:004B1950 6800E410A4            push A410E400
:004B1955 51                    push ecx
:004B1956 FF9616050000          call dword ptr [esi+00000516]  ------>这里是 `DEVICEIOCONTROL` 函数 上面的是函数入口的参数 写底层仿真的话!上面有你需要的重要信息!
:004B195C 85C0                  test eax, eax
:004B195E 7509                  jne 004B1969
:004B1960 660DFFFF              or ax, FFFF
:004B1964 5E                    pop esi
:004B1965 83C454                add esp, 00000054
:004B1968 C3                    ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B195E(C)

:004B1969 668B442406            mov ax, word ptr [esp+06]   =========>这里是读狗返回的标志!没有狗是 3!有狗返回的是 0!
:004B196E 5E                    pop esi
:004B196F 83C454                add esp, 00000054
```

### (2)加密段数据解密部分

```
:004B12DC 8D55D4                 lea edx, dword ptr [ebp-2C]
:004B12DF 52                     push edx
:004B12E0 8D45E0                 lea eax, dword ptr [ebp-20]
:004B12E3 50                     push eax
:004B12E4 8D4DE8                 lea ecx, dword ptr [ebp-18]
:004B12E7 51                     push ecx
:004B12E8 8D55B0                 lea edx, dword ptr [ebp-50]
:004B12EB 52                     push edx
:004B12EC 8D45B8                 lea eax, dword ptr [ebp-48]
:004B12EF 50                     push eax
:004B12F0 8D4DD0                 lea ecx, dword ptr [ebp-30]
:004B12F3 51                     push ecx

* Possible Reference to Dialog: `DialogID_0066`, `CONTROL_ID:0008`, ""

:004B12F4 6A08                   push 000000008
:004B12F6 E8B5050000             call 004B18B0       ========>读狗,如果成功 `EAX=0`
:004B12FB 83C424                 add esp, 00000024
:004B12FE 25FFFF0000             and eax, 0000FFFF
:004B1303 85C0                   test eax, eax
:004B1305 7405                   je 004B130C       =====>成功 `JMP`
:004B1307 E900020000             jmp 004B150C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B1305(C)

:004B130C 668B55E8               mov dx, word ptr [ebp-18]  ======>注意返回的(重要数据 1)
:004B1310 668955BC               mov word ptr [ebp-44], dx
:004B1314 668B45E0               mov ax, word ptr [ebp-20]  ======>注意返回的(重要数据 2)
:004B1318 668945BE               mov word ptr [ebp-42], ax
:004B131C 668B4DD4               mov cx, word ptr [ebp-2C]  ======>注意返回的(重要数据 3)
:004B1320 66894DC0               mov word ptr [ebp-40], cx
:004B1324 668B55B4               mov dx, word ptr [ebp-4C]  ======>注意返回的(重要数据 4)
:004B1328 668955C2               mov word ptr [ebp-3E], dx
:004B132C 8B45FC                 mov eax, dword ptr [ebp-04]
:004B132F 8B4812                 mov ecx, dword ptr [eax+12]
:004B1332 8B55AC                 mov edx, dword ptr [ebp-54]
:004B1335 030A                   add ecx, dword ptr [edx]
:004B1337 894DF4                 mov dword ptr [ebp-0C], ecx
:004B133A C745D800000000         mov [ebp-28], 00000000
:004B1341 EB09                   jmp 004B134C

下面是数据还原解密部分

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B1379(U)

:004B1343 8B45D8                 mov eax, dword ptr [ebp-28]
:004B1346 83C001                 add eax, 00000001
:004B1349 8945D8                 mov dword ptr [ebp-28], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B1341(U)

:004B134C 8B4DAC                 mov ecx, dword ptr [ebp-54]
:004B134F 8B55D8                 mov edx, dword ptr [ebp-28]
:004B1352 3B5104                 cmp edx, dword ptr [ecx+04]  ========>判断是不是数据段解密完毕
:004B1355 7324                   jnb 004B137B                ========>如果是真则是数据还原解密完毕!下面的是还原算法!
:004B1357 8B45F4                 mov eax, dword ptr [ebp-0C]
:004B135A 0345D8                 add eax, dword ptr [ebp-28]
:004B135D 33C9                   xor ecx, ecx
:004B135F 8A08                   mov cl, byte ptr [eax]
:004B1361 8B45D8                 mov eax, dword ptr [ebp-28]
:004B1364 33D2                   xor edx, edx
:004B1366 F775F0                 div [ebp-10]
:004B1369 33C0                   xor eax, eax
:004B136B 8A4415BC               mov al, byte ptr [ebp+edx-44]
:004B136F 33C8                   xor ecx, eax
:004B1371 8B55F4                 mov edx, dword ptr [ebp-0C]
:004B1374 0355D8                 add edx, dword ptr [ebp-28]
:004B1377 880A                   mov byte ptr [edx], cl
:004B1379 EBC8                   jmp 004B1343
```

### (3)入口点

下面的代码就是外壳结尾部分

```
:004B14F3 8902                   mov dword ptr [edx], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004B14E0(C), :004B14EB(C)

:004B14F5 41                     inc ecx
:004B14F6 3B4B2E                 cmp ecx, dword ptr [ebx+2E]
:004B14F9 72DF                   jb 004B14DA

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B101D(C)

:004B14FB 8B83EA040000           mov eax, dword ptr [ebx+000004EA]  ===============>这里就是入口点的数据地址
:004B1501 034312                 add eax, dword ptr [ebx+12]
:004B1504 5F                     pop edi
:004B1505 5E                     pop esi
:004B1506 5B                     pop ebx
:004B1507 8BE5                   mov esp, ebp
:004B1509 5D                     pop ebp
:004B150A FFE0                   jmp eax        ========>如果有狗那么 `eax` 就是程序的入口点!
```

上面就是狗壳部分!我讲完了!希望讲的还不是很糟!如果你不明白我也没办法了!

## 第二部分 (程序本身的解密)

这个程序本身的加密做得很好!有很多处加密点!而且还用到了部分算法数据在程序当中使用!所以这东西解起来比较麻烦!

我在这就简单写写了!



### 代码段 1

```
:0047FD14 FF1504E44B00             Call dword ptr [004BE404]  ====================>这里读狗

:0047FD1A 85C0                    test eax, eax

:0047FD1C 750F                    jne 0047FD2D

:0047FD1E 66B8FFFF                mov ax, FFFF

:0047FD22 5D                      pop ebp

:0047FD23 5F                      pop edi

:0047FD24 5E                      pop esi

:0047FD25 5B                      pop ebx

:0047FD26 81C47C020000            add esp, 0000027C

:0047FD2C C3                      ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0047FD1C(C)

|

:0047FD2D 668B442412              mov ax, word ptr [esp+12]  ================>读狗后的返回标志!ax=0 就行了!

:0047FD32 5D                      pop ebp

:0047FD33 5F                      pop edi

:0047FD34 5E                      pop esi

:0047FD35 5B                      pop ebx

:0047FD36 81C47C020000            add esp, 0000027C

:0047FD3C C3                      ret
```

### 进展说明

这样搞完了！程序已经可以进入界面了！但是点重要功能！程序就非法操作了！

再来！经过跟踪！发现！

#### 代码段 2

```
(1)

:0044DEA4 68149D5200               push 00529D14

:0044DEA9 681C9E5200               push 00529E1C

:0044DEAE 681E9E5200               push 00529E1E

:0044DEB3 68209E5200               push 00529E20

:0044DEB8 68229E5200               push 00529E22

:0044DEBD 68149E5200               push 00529E14

:0044DEC2 68189E5200               push 00529E18

:0044DEC7 68249E5200               push 00529E24

:0044DECC 6A0E                     push 0000000E

:0044DECE C705189E520000000000     mov dword ptr [00529E18], 00000000

:0044DED8 A3149E5200               mov dword ptr [00529E14], eax

:0044DEDD E8FE1A0300               call 0047F9E0    ==========>这里读狗

:0044DEE2 83C424                   add esp, 00000024

:0044DEE5 66F7D8                   neg ax

:0044DEE8 1BC0                     sbb eax, eax

:0044DEEA F7D0                     not eax

:0044DEEC 662305229E5200           and ax, word ptr [00529E22] ==========>这里是返回的重要数据!

:0044DEF3 C3                      ret
```

还有这里

#### 代码段 3

```
:0044BE21 68149D5200               push 00529D14

:0044BE26 681C9E5200               push 00529E1C

:0044BE2B 681E9E5200               push 00529E1E

:0044BE30 68209E5200               push 00529E20

:0044BE35 68229E5200               push 00529E22

:0044BE3A 68149E5200               push 00529E14

:0044BE3F 68189E5200               push 00529E18

:0044BE44 68249E5200               push 00529E24

:0044BE49 6A08                     push 00000008

:0044BE4B C705149E520045970000     mov dword ptr [00529E14], 00009745

:0044BE55 E8863B0300               call 0047F9E0        ============>这里读狗!

:0044BE5A 83C424                   add esp, 00000024

:0044BE5D 6685C0                   test ax, ax

:0044BE60 7511                     jne 0044BE73        =======>有狗的话这里是不会跳转的!

:0044BE62 66A11E9E5200             mov ax, word ptr [00529E1E]  ===========>返回的重要数据!

:0044BE68 6689442444               mov word ptr [esp+44], ax

:0044BE6D 8B442444                 mov eax, dword ptr [esp+44]

:0044BE71 EB02                     jmp 0044BE75    =======>到正确的部分正常运行!

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044BE60(C)

|

:0044BE73 33C0                     xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044BE71(U)

|

:0044BE75 25FFFF0000               and eax, 0000FFFF  ========>下面的是算法部分如果返回的数据错误会导致程序非法操作!

:0044BE7A 8B7E04                   mov edi, dword ptr [esi+04]

:0044BE7D 2D44420000               sub eax, 00004244

:0044BE82 8B7608                   mov esi, dword ptr [esi+08]

:0044BE85 8BC8                     mov ecx, eax

:0044BE87 B8ABAAAA2A               mov eax, 2AAAAAAB

:0044BE8C F7E9                     imul ecx

:0044BE8E D1FA                     sar edx, 1
```

### 分析与总结

程序中有多处类似的代码！来监测加密狗！好了！就先说这个多了！解决其实很简单！只要有狗把重要数据部分得到！呵呵剩下的不用我多说了吧！

终于写完了！累！写东西好累呀！呵呵！

再来说两句加密狗现在发展的好快现在的加密狗已经发展到第 5 代了！代表产品（深思 4 和 `rockey5`）新的加密狗结合了传统加密锁技术和智能卡！硬件复制的难度好像更大了！软件本身破解的难度也提高了不少！如果加密者能结合新产品的特点灵活运用！软件本身的破解难度会变得异常的困难！当然了！如果加密者的加密方案比较简单或有漏洞软件还是可以破解的！谢谢大家看我罗嗦了这么多！希望以上写的对爱好解密的朋友！有所帮助！谢谢大家看完此文！ 如果你觉得写的还行请回个贴子！支持一下！谢谢！

如果要转载请保留完整

大老=`[DCG]`=`

`dalao@top86.com`

`http://dalao2002.yeah.net`

`2003-05-03`

凌晨 3:24 `4dd466d0e1e0cc123afbe504da2462ca`
