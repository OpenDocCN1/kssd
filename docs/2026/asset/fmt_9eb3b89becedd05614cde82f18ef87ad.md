
# 晕倒，永远也没有合法的……说`DISKdata.exe`

## 软件信息

- 软件名称：`DISKdata.exe`
- 下载地址：`5cfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8A6k6$3q4D9L8r3g2J5P5g2)9J5k6h3y4G2L8g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3k6r3c8K6k6i4c8#2M7q4)9J5k6h3g2^5k6b7`.`
- 软件功能：不太明白，没用过
- 编译语言：Borland Delphi 4.0 - 5.0
- 破解作者：`wofan[OCN][PYG]`

## 关键地址

`00491738=DISKdata.00491738` (ASCII "The license is invalid! Be sure to enter the "Licensed To" and the "License Key" information exactly as supplied in your license.")

`Jumps from 004911E5, 00491248, 004912CA`

## 破解分析

软件启动时有Nag延时，`OD`载入，先找个断点，这个软件有未注册成功的提示，见上面！

注册码要22位！！！！当然开始并不知道，所以折腾了一些时间，并且因为打雷闪电，又要上班，并未在我预定的6.1儿童节完成。

输入：

```
Licensed to:wofan
License key:1234123412341234563478
```

（试验码1）\*\*\*\*\*\*暂定为22位,如果选取的注册码在运算中出现不可见字符，就增加了分析的难度。

在断点在`F2`断下后，点击`Set`按钮，断下：

```
0049115A    .  E8 61F5F9FF       call DISKdata.004306C0      \*\*\*\*\*\*\*\*\*\*取得注册码长度0x16
0049115F    .  8B45 F0           mov eax,dword ptr ss:[ebp-10]\*\*\*\*\*\*\*\*\*假注册码1234123412341234563478
00491162    .  8D55 F4           lea edx,dword ptr ss:[ebp-C]
00491165    .  E8 AA74F7FF       call DISKdata.00408614
0049116A    .  8B45 F4           mov eax,dword ptr ss:[ebp-C]\*\*\*\*\*\*\*\*\*假注册码地地址
0049116D    .  8D4D FC           lea ecx,dword ptr ss:[ebp-4]
00491170    .  BA 20164900       mov edx,DISKdata.00491620              ;  ASCII "DISKdata"
00491175    .  E8 4E03FEFF       call DISKdata.004714C8      \*\*\*\*\*\*\*\*\*第一处按F7跟进！！！！！
0049117A    .  84C0              test al,al                    \*\*\*\*\*\*这里是第一处判断，不能通过就会弹出未成功注册的信息：The license is invalid!……
0049117C    .  0F84 C7030000     je DISKdata.00491549
00491182    .  8D45 E8           lea eax,dword ptr ss:[ebp-18]
00491185    .  50                push eax
00491186    .  B9 02000000       mov ecx,2                     \*\*\*ECX=2
0049118B    .  BA 05000000       mov edx,5                     \*\*\*EDX=5
00491190    .  8B45 FC           mov eax,dword ptr ss:[ebp-4]   \*\*\*\*\*\*新字串3:V}AvUfU{p（上面如果不跳，这里开始效验）
00491193    .  E8 102FF7FF       call DISKdata.004040A8         \*\*\*\*\*\*取新字串3的第五六位：Stack ss:[0012F67C]=00DBA290, (ASCII "vU")
00491198    .  8B4D E8           mov ecx,dword ptr ss:[ebp-18]
0049119B    .  8D45 EC           lea eax,dword ptr ss:[ebp-14]
0049119E    .  BA 34164900       mov edx,DISKdata.00491634
004911A3    .  E8 442DF7FF       call DISKdata.00403EEC          \*\*\*\*\*Stack ss:[0012F680]=00DBA2A0, (ASCII "$vU")  在前面加上一个$
004911A8    .  8B45 EC           mov eax,dword ptr ss:[ebp-14]
004911AB    .  E8 C475F7FF       call DISKdata.00408774          \*\*\*\*\*开始效验之一，不成功会返回：“''is not a valid integer value”\*\*\*\*\*\*\*
004911B0    .  8BF0              mov esi,eax                    ××××以修改后的注册码12341234050012345634B2(试验码2），会使ESI＝EAX=AA×××××××
004911B2    .  8D55 DC           lea edx,dword ptr ss:[ebp-24]
004911B5    .  8B83 48040000     mov eax,dword ptr ds:[ebx+448]
004911BB    .  E8 00F5F9FF       call DISKdata.004306C0         ××××注册名：wofan
004911C0    .  8B45 DC           mov eax,dword ptr ss:[ebp-24]
004911C3    .  8D55 E0           lea edx,dword ptr ss:[ebp-20]
004911C6    .  E8 0572F7FF       call DISKdata.004083D0         ××××开始处理注册名：wofan，注册名大写化
004911CB    .  8B45 E0           mov eax,dword ptr ss:[ebp-20]   ××××WOFAN
004911CE    .  8D55 E4           lea edx,dword ptr ss:[ebp-1C]
004911D1    .  E8 3E74F7FF       call DISKdata.00408614
004911D6    .  8B45 E4           mov eax,dword ptr ss:[ebp-1C]
004911D9    .  E8 F604FEFF       call DISKdata.004716D4        ×××处理注册名之二，得到注册名大写化后ASCII累加值，（保留低八位）7B送EAX
004911DE    .  25 FF000000       and eax,0FF                    ×××EAX=7B
004911E3    .  3BF0              cmp esi,eax                    ×××ESI=AA，EAX=7B
004911E5    .  0F85 52030000     jnz DISKdata.0049153D           ×××可不能跳呀！原来新字串3，的第五，六位是7B
```

## 注册码推导过程一

```
××××××××××××××××××××××××××××××
通过注册名wofan得到7B，
37(7) xor 64(d)=53
42(B) xor 61(a)=23
注册码的9-12位就是5323
××××××××××××××××××××××××××××××
```

```
004911EB    .  8D45 D4           lea eax,dword ptr ss:[ebp-2C]
004911EE    .  50                push eax
004911EF    .  8D55 CC           lea edx,dword ptr ss:[ebp-34]
004911F2    .  8B83 48040000     mov eax,dword ptr ds:[ebx+448]
004911F8    .  E8 C3F4F9FF       call DISKdata.004306C0           ×××注册名长度5
004911FD    .  8B45 CC           mov eax,dword ptr ss:[ebp-34]   ×××注册名wofan
00491200    .  8D55 D0           lea edx,dword ptr ss:[ebp-30]
00491203    .  E8 0C74F7FF       call DISKdata.00408614           ×××注册名wofan
00491208    .  8B45 D0           mov eax,dword ptr ss:[ebp-30]
0049120B    .  B9 02000000       mov ecx,2                        ×××ECX=2
00491210    .  BA 01000000       mov edx,1                         ×××EDX=1
00491215    .  E8 8E2EF7FF       call DISKdata.004040A8            ×××取wofan的前两位wo
0049121A    .  8B45 D4           mov eax,dword ptr ss:[ebp-2C]
0049121D    .  8D55 D8           lea edx,dword ptr ss:[ebp-28]
00491220    .  E8 AB71F7FF       call DISKdata.004083D0            ×××大写化成为WO
00491225    .  8B45 D8           mov eax,dword ptr ss:[ebp-28]
00491228    .  50                push eax                          ×××Push
00491229    .  8D45 C8           lea eax,dword ptr ss:[ebp-38]
0049122C    .  50                push eax
0049122D    .  B9 02000000       mov ecx,2                       ×××ECX=2
00491232    .  BA 01000000       mov edx,1                       ×××EDX=1
00491237    .  8B45 FC           mov eax,dword ptr ss:[ebp-4]
0049123A    .  E8 692EF7FF       call DISKdata.004040A8           ×××取新字串3的前两位，应该记得是 "V}"
0049123F    .  8B55 C8           mov edx,dword ptr ss:[ebp-38]
00491242    .  58                pop eax                         ×××pop,使EAX得到WO
00491243    .  E8 682DF7FF       call DISKdata.00403FB0           ×××比较
00491248    .  0F85 EF020000     jnz DISKdata.0049153D             ×××当然要相等才行！
```

```
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
可见注册码前四位应该是：1306
44(D) xor 57(W)=13
49(I) xor 4F(O)=06
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

```
0049124E    .  8D45 C0           lea eax,dword ptr ss:[ebp-40]
00491251    .  50                push eax
00491252    .  8D55 B8           lea edx,dword ptr ss:[ebp-48]
00491255    .  8B83 48040000     mov eax,dword ptr ds:[ebx+448]
0049125B    .  E8 60F4F9FF       call DISKdata.004306C0
00491260    .  8B45 B8           mov eax,dword ptr ss:[ebp-48]
00491263    .  8D55 BC           lea edx,dword ptr ss:[ebp-44]
00491266    .  E8 A973F7FF       call DISKdata.00408614
0049126B    .  8B45 BC           mov eax,dword ptr ss:[ebp-44]
0049126E    .  E8 2D2CF7FF       call DISKdata.00403EA0         ×××注册名长度5送EAX
00491273    .  48                dec eax                        ×××EAX=EAX-1＝4（用它确定，从注册名后取两位）
00491274    .  50                push eax
00491275    .  8D55 B0           lea edx,dword ptr ss:[ebp-50]
00491278    .  8B83 48040000     mov eax,dword ptr ds:[ebx+448]
0049127E    .  E8 3DF4F9FF       call DISKdata.004306C0
00491283    .  8B45 B0           mov eax,dword ptr ss:[ebp-50]
00491286    .  8D55 B4           lea edx,dword ptr ss:[ebp-4C]
00491289    .  E8 8673F7FF       call DISKdata.00408614
0049128E    .  8B45 B4           mov eax,dword ptr ss:[ebp-4C]      ×××注册名wofan
00491291    .  B9 02000000       mov ecx,2                          ×××ECX=2
00491296    .  5A                pop edx                            ×××Pop，使EDX=4
00491297    .  E8 0C2EF7FF       call DISKdata.004040A8              ×××取注册名的后两位an
0049129C    .  8B45 C0           mov eax,dword ptr ss:[ebp-40]
0049129F    .  8D55 C4           lea edx,dword ptr ss:[ebp-3C]
004912A2    .  E8 2971F7FF       call DISKdata.004083D0            ×××大写化变成AN
004912A7    .  8B45 C4           mov eax,dword ptr ss:[ebp-3C]
004912AA    .  50                push eax
004912AB    .  8D45 AC           lea eax,dword ptr ss:[ebp-54]
004912AE    .  50                push eax                          ×××Push
```
