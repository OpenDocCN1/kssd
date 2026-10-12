
### 汇编代码分析

```
* Reference To: MFC42.Ordinal:039C, Ord:039Ch

                    |
:004184F0 E8097E0000        Call 004202FE
:004184F5 8D4DE4            lea ecx, dword ptr [ebp-1C]
:004184F8 C645FC04          mov [ebp-04], 04

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                    |
:004184FC E8517B0000        Call 00420052
:00418501 8D4DE0            lea ecx, dword ptr [ebp-20]
:00418504 C645FC03          mov [ebp-04], 03

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                    |
:00418508 E8457B0000        Call 00420052
:0041850D 8D4360            lea eax, dword ptr [ebx+60]
:00418510 50                push eax
:00418511 8D45DC            lea eax, dword ptr [ebp-24]

* Possible StringData Ref from Data Obj ->"http://www.wjmshome.com/register/"  看到了吗!要上网去效验的啊!

                    |
:00418514 68AC214300        push 004321AC
:00418519 50                push eax

* Reference To: MFC42.Ordinal:039E, Ord:039Eh

                    |
:0041851A E8F17D0000        Call 00420310
:0041851F 56                push esi
:00418520 50                push eax
:00418521 8D45E8            lea eax, dword ptr [ebp-18]
:00418524 C645FC05          mov [ebp-04], 05
:00418528 50                push eax

* Reference To: MFC42.Ordinal:039C, Ord:039Ch

                    |
:00418529 E8D07D0000        Call 004202FE
:0041852E 8D4DDC            lea ecx, dword ptr [ebp-24]
:00418531 C645FC07          mov [ebp-04], 07

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                    |
:00418535 E8187B0000        Call 00420052
:0041853A 8BCF              mov ecx, edi
:0041853C E89D27FFFF        call 0040ACDE
:00418541 51                push ecx
:00418542 8D45EC            lea eax, dword ptr [ebp-14]
:00418545 8BCC              mov ecx, esp
:00418547 8965D8            mov dword ptr [ebp-28], esp
:0041854A 50                push eax

* Reference To: MFC42.Ordinal:0217, Ord:0217h

                    |
:0041854B E81E7D0000        Call 0042026E
:00418550 51                push ecx
:00418551 8D45E8            lea eax, dword ptr [ebp-18]
:00418554 8BCC              mov ecx, esp
:00418556 8965D4            mov dword ptr [ebp-2C], esp
:00418559 50                push eax
:0041855A C645FC08          mov [ebp-04], 08

* Reference To: MFC42.Ordinal:0217, Ord:0217h

                    |
:0041855E E80B7D0000        Call 0042026E
:00418563 8BCF              mov ecx, edi
:00418565 C645FC07          mov [ebp-04], 07
:00418569 E8E025FFFF        call 0040AB4E
:0041856E 33F6              xor esi, esi
:00418570 3BC6              cmp eax, esi
:00418572 7511              jne 00418585
:00418574 56                push esi
:00418575 56                push esi

* Possible StringData Ref from Data Obj ->"无法接入 Internet 注册数据，请与开发商联系"

                    |
:00418576 6880214300        push 00432180

* Reference To: MFC42.Ordinal:04B0, Ord:04B0h

                    |
:0041857B E8A87D0000        Call 00420328
:00418580 E93B010000        jmp 004186C0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00418572(C)
|
:00418585 8D4DC4            lea ecx, dword ptr [ebp-3C]

* Reference To: MFC42.Ordinal:0162, Ord:0162h

                    |
:00418588 E8417D0000        Call 004202CE
:0041858D 56                push esi
:0041858E 6800800000        push 00008000
:00418593 FF75EC            push [ebp-14]
:00418596 8D4DC4            lea ecx, dword ptr [ebp-3C]
:00418599 C645FC09          mov [ebp-04], 09

* Reference To: MFC42.Ordinal:1442, Ord:1442h

                    |
:0041859D E81A7D0000        Call 004202BC
:004185A2 85C0              test eax, eax
:004185A4 7511              jne 004185B7
:004185A6 56                push esi
:004185A7 56                push esi

* Possible StringData Ref from Data Obj ->"无法读取已经下载的 Internet 注册数据"

                    |
:004185A8 685C214300        push 0043215C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00418639(U), :00418693(U)
|
* Reference To: MFC42.Ordinal:04B0, Ord:04B0h

                    |
:004185AD E8767D0000        Call 00420328
:004185B2 E9FD000000        jmp 004186B4
```

注册在网上比较，如果错误一定提示所以在关键字符中找找有没有关于注册码的提示！哈!!!!!

```
==================================================================================
:0041867F E8CE790000        Call 00420052
:00418684 807DF300          cmp byte ptr [ebp-0D], 00---------->为 0 则跳,一定要跳
:00418688 740E              je 00418698
:0041868A 33F6              xor esi, esi
:0041868C 56                push esi
:0041868D 56                push esi

* Possible StringData Ref from Data Obj ->"注册码输入错误"

                    |
:0041868E 6828214300        push 00432128
:00418693 E915FFFFFF        jmp 004185AD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00418688(C)
|
:00418698 807DC201          cmp byte ptr [ebp-3E], 01---------->如果返回的值是 1 就正确
:0041869C 754B              jne 004186E9----------------------->不相等则跳
:0041869E 6A40              push 00000040

* Possible StringData Ref from Data Obj ->"感谢您的注册"

                    |
:004186A0 6818214300        push 00432118

* Possible StringData Ref from Data Obj ->"你已是******的正式注册用户并可免费注册以后的"
                                      ->"所有版本,
如果公开注册码将会被取消注册用户资?

                                      ->"?"

                    |
:004186A5 68B8204300        push 004320B8
==================================================================================
```

到这里我们知道我们找到的关键 `CALL` 该返回什么,有知道在哪里返回---改啊!还等什么啊!

爆破之后你会发现,软件重启之后就已经注册成功了,没有重启效验看来作者对他的注册码保护方式很有信心啊!

注册名和注册码的变形值保存在注册表里:

小弟对密码学不是很懂但是软件是一个用 `RSA` 为文件加密的软件所以小弟猜这个变型是 `RSA` 的(没有验证啊)

注册码和用户名没有关系,但是保存在 `JIM.DAT` 中的我们输入的注册码和保存在注册表里的变型之后的数值有关啊!

`HKEY_CURRENT_USER\Software\文件密使\Register`

`RegisterName="powerboy"`

`RegisterCodeSanLieZhi=fc 01 c9 5b eb f6 14 68 5e b0 70 82 5b 77 4b 56`

注册码保存在 `JIM.DAT` 里看这个文件的最下面,是我们输入的注册码.

到这里应该清楚了,原来软件注册成功之后只是把 `JIM.DAT` 里的注册码与注册表里的注册码变形进行比较.

所以注册码和用户名无关,注册成功就是把我们输入的注册码变形保存在注册表里. `4dd466d0e1e0cc123afbe504da2462ca`
