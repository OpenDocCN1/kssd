
# 超级通讯王(SuperPIM)注册算法分析+汇编注册机

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-438.htm`](https://www.pediy.com/kssd/pediy07/pediy7-438.htm)

【破文标题】 超级通讯王(SuperPIM)注册算法分析+汇编注册机

【破文作者】 snake

【软件名称】 超级通讯王(SuperPIM) `2.00.1002`

【下载地址】 [`yncnc.onlinedown.net/soft/20052.htm`](http://yncnc.onlinedown.net/soft/20052.htm)

【软件简介】 超级通讯王是一款为中国人精心设计、绝佳的个人信息管理(PIM)软件，包括网络电子名片（全球独创，酷！）、通讯录、手机短信、网上即时聊天、日常记事、万年历、信息速查等功能。★网络电子名片是 `2.0` 版开始新增的独创功能，能使日常的名片交流变得前所未有的简单方便！

【调试环境】 `Win2000`、`PEiD`、`C32Asm`、`Ollydbg`

【作者声明】 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

\----------------------------------------------------------------------------------------------

## 破解过程

用 `PEiD` 查壳，无壳，`Microsoft Visual C++ 7.0` 编写。

运行程序，输入注册信息

序列号：`1EHQVLDGL23PN1L`

注册码：`78787878`

注册后提示：对不起，注册码不正确，注册失败！

用 `C32Asm` 载入程序，搜索该字符串，查得引用地址为 `004300B1`，参考点。

`Ollydbg` 载入主程序，`Ctrl+G`，`004300B1`，来到该处，向上找到 `0042FF80`，在这下断点

`F9` 运行程序，输入注册码，确定，程序被断下

```
0042FF80    .  6A FF           push -1
0042FF82    .  68 98555000     push SuperPIM.00505598                    ;  SE 句柄安装
0042FF87    .  64:A1 00000000  mov eax,dword ptr fs:[0]
0042FF8D    .  50             push eax
0042FF8E    .  64:8925 000000> mov dword ptr fs:[0],esp
0042FF95    .  83EC 0C        sub esp,0C
0042FF98    .  53             push ebx
0042FF99    .  56             push esi
0042FF9A    .  57             push edi
0042FF9B    .  33DB           xor ebx,ebx
0042FF9D    .  6A 01          push 1
0042FF9F    .  8BF1           mov esi,ecx
0042FFA1    .  895C24 14      mov dword ptr ss:[esp+14],ebx
0042FFA5    .  E8 6FB90900    call SuperPIM.004CB919
0042FFAA    .  8D4424 10      lea eax,dword ptr ss:[esp+10]
0042FFAE    .  50             push eax
0042FFAF    .  E8 2C150600    call SuperPIM.004914E0                    ;  算法 call1，由硬盘参数生成机器码
                                ;  再生成 15 位注册码并转换为 20 位字符串，即注册码
                                ;  再转换为 40 位字符串与假码生成的 40 位字符串比较
0042FFB4    .  83C4 04        add esp,4
0042FFB7    .  8B4E 74        mov ecx,dword ptr ds:[esi+74]           ;  假码
0042FFBA    .  8B41 F4        mov eax,dword ptr ds:[ecx-C]            ;  假码长度
0042FFBD    .  85C0           test eax,eax
0042FFBF    .  8D7E 74        lea edi,dword ptr ds:[esi+74]
0042FFC2    .  895C24 20      mov dword ptr ss:[esp+20],ebx
0042FFC6    .  74 2A          je short SuperPIM.0042FFF2
0042FFC8    .  6A 28          push 28
0042FFCA    .  8D5424 18      lea edx,dword ptr ss:[esp+18]
0042FFCE    .  57             push edi
0042FFCF    .  52             push edx
0042FFD0    .  E8 0B480500    call SuperPIM.004847E0     ;  将假注册码转换为 40 位字符串
0042FFD5    .  8B4C24 1C      mov ecx,dword ptr ss:[esp+1C]
0042FFD9    .  8B00          mov eax,dword ptr ds:[eax]
0042FFDB    .  51             push ecx
0042FFDC    .  50             push eax
0042FFDD    .  BB 01000000    mov ebx,1
0042FFE2    .  E8 DF730800    call SuperPIM.004B73C6     ;  比较两字符串 call
0042FFE7    .  83C4 14        add esp,14
0042FFEA    .  85C0           test eax,eax
0042FFEC    .  885C24 0F      mov byte ptr ss:[esp+F],bl
0042FFF0    .  74 05          je short SuperPIM.0042FFF7
0042FFF2    >  C64424 0F 00   mov byte ptr ss:[esp+F],0
0042FFF7    >  F6C3 01        test bl,1
0042FFFA    .  74 1E          je short SuperPIM.0043001A
0042FFFC    .  8B4424 14      mov eax,dword ptr ss:[esp+14]
00430000    .  83C0 F0        add eax,-10
00430003    .  8D50 0C        lea edx,dword ptr ds:[eax+C]
00430006    .  83C9 FF        or ecx,FFFFFFFF
00430009    .  F0:0FC10A      lock xadd dword ptr ds:[edx],ecx
0043000D    .  49             dec ecx
0043000E    .  85C9           test ecx,ecx
00430010    .  7F 08          jg short SuperPIM.0043001A
00430012    .  8B08          mov ecx,dword ptr ds:[eax]
00430014    .  8B11          mov edx,dword ptr ds:[ecx]
00430016    .  50             push eax
00430017    .  FF52 04        call dword ptr ds:[edx+4]
0043001A    >  8A4424 0F      mov al,byte ptr ss:[esp+F]
0043001E    .  84C0          test al,al
00430020    .  8D4424 14      lea eax,dword ptr ss:[esp+14]
00430024    .  0F84 87000000  je SuperPIM.004300B1          ;  关键跳，不跳则为正式版。爆破点
0043002A    .  6A 71          push 71                         ;  提示注册成功
0043002C    .  50             push eax
0043002D    .  E8 9E120600    call SuperPIM.004912D0
00430032    .  83C4 08        add esp,8
00430035    .  8B00          mov eax,dword ptr ds:[eax]
00430037    .  6A 00          push 0
00430039    .  6A 00          push 0
0043003B    .  50             push eax
0043003C    .  8BCE          mov ecx,esi
0043003E    .  C64424 2C 01   mov byte ptr ss:[esp+2C],1
00430043    .  E8 69B50900    call SuperPIM.004CB5B1
00430048    .  8B4424 14      mov eax,dword ptr ss:[esp+14]
0043004C    .  83C0 F0        add eax,-10
0043004F    .  C64424 20 00   mov byte ptr ss:[esp+20],0
00430054    .  8D48 0C        lea ecx,dword ptr ds:[eax+C]
00430057    .  83CA FF        or edx,FFFFFFFF
0043005A    .  F0:0FC111      lock xadd dword ptr ds:[ecx],edx
0043005E    .  4A             dec edx
0043005F    .  85D2          test edx,edx
00430061    .  7F 08          jg short SuperPIM.0043006B
00430063    .  8B08          mov ecx,dword ptr ds:[eax]
00430065    .  8B11          mov edx,dword ptr ds:[ecx]
00430067    .  50             push eax
00430068    .  FF52 04        call dword ptr ds:[edx+4]
0043006B    >  E8 E1C40B00    call SuperPIM.004EC551
00430070    .  8B40 04        mov eax,dword ptr ds:[eax+4]
00430073    .  6A 01          push 1
00430075    .  57             push edi
00430076    .  C780 70040000 > mov dword ptr ds:[eax+470],1
00430080    .  E8 4B460500    call SuperPIM.004846D0
00430085    .  8B3F          mov edi,dword ptr ds:[edi]
00430087    .  8B5F F4        mov ebx,dword ptr ds:[edi-C]
0043008A    .  83C4 08        add esp,8
0043008D    .  E8 BFC40B00    call SuperPIM.004EC551
00430092    .  8B40 04        mov eax,dword ptr ds:[eax+4]
00430095    .  53             push ebx                      ; /Arg4
00430096    .  57             push edi                      ; |Arg3
00430097    .  68 68985100    push SuperPIM.00519868        ; |Arg2 = 00519868 ASCII "rc"
0043009C    .  68 9A1B5100    push SuperPIM.00511B9A        ; |Arg1 = 00511B9A
004300A1    .  8BC8          mov ecx,eax                   ; |
004300A3    .  E8 FC8D0A00    call SuperPIM.004D8EA4        ; \\SuperPIM.004D8EA4
004300A8    .  8BCE          mov ecx,esi
004300AA    .  E8 60FC0900    call SuperPIM.004CFD0F
004300AF    .  EB 41          jmp short SuperPIM.004300F2
004300B1    >  6A 72          push 72                         ;  提示注册失败
004300B3    .  50             push eax
004300B4    .  E8 17120600    call SuperPIM.004912D0
004300B9    .  83C4 08        add esp,8
004300BC    .  8B00          mov eax,dword ptr ds:[eax]
004300BE    .  6A 00          push 0
004300C0    .  6A 00          push 0
004300C2    .  50             push eax
004300C3    .  8BCE          mov ecx,esi
004300C5    .  C64424 2C 02   mov byte ptr ss:[esp+2C],2
004300CA    .  E8 E2B40900    call SuperPIM.004CB5B1
004300CF    .  8B4424 14      mov eax,dword ptr ss:[esp+14]
004300D3    .  83C0 F0        add eax,-10
004300D6    .  C64424 20 00   mov byte ptr ss:[esp+20],0
004300DB    .  8D48 0C        lea ecx,dword ptr ds:[eax+C]
004300DE    .  83CA FF        or edx,FFFFFFFF
004300E1    .  F0:0FC111      lock xadd dword ptr ds:[ecx],edx
004300E5    .  4A             dec edx
004300E6    .  85D2          test edx,edx
```



004300E8 . 7F 08 jg short SuperPIM.004300F2
004300EA . 8B08 mov ecx,dword ptr ds:[eax]
004300EC . 8B11 mov edx,dword ptr ds:[ecx]
004300EE . 50 push eax
004300EF . FF52 04 call dword ptr ds:[edx+4]
004300F2 > 8B4424 10 mov eax,dword ptr ss:[esp+10]
004300F6 . 83C0 F0 add eax,-10
004300F9 . C74424 20 FFFF>mov dword ptr ss:[esp+20],-1
00430101 . 8D48 0C lea ecx,dword ptr ds:[eax+C]
00430104 . 83CA FF or edx,FFFFFFFF
00430107 . F0:0FC111 lock xadd dword ptr ds:[ecx],edx
0043010B . 4A dec edx
0043010C . 85D2 test edx,edx
0043010E . 5F pop edi
0043010F . 5E pop esi
00430110 . 5B pop ebx
00430111 . 7F 08 jg short SuperPIM.0043011B
00430113 . 8B08 mov ecx,dword ptr ds:[eax]
00430115 . 8B11 mov edx,dword ptr ds:[ecx]
00430117 . 50 push eax
00430118 . FF52 04 call dword ptr ds:[edx+4]
0043011B > 8B4C24 0C mov ecx,dword ptr ss:[esp+C]
0043011F . 64:890D 000000>mov dword ptr fs:[0],ecx
00430126 . 83C4 18 add esp,18
00430129 . C3 retn

========================= 跟进 0042FFAF E8 2C150600 call SuperPIM.004914E0 =========================

004914E0 /$ 6A FF push -1
004914E2 |. 68 20CC5000 push SuperPIM.0050CC20 ; SE 句柄安装
004914E7 |. 64:A1 00000000 mov eax,dword ptr fs:[0]
004914ED |. 50 push eax
004914EE |. 64:8925 000000>mov dword ptr fs:[0],esp
004914F5 |. 83EC 08 sub esp,8
004914F8 |. 8D4424 04 lea eax,dword ptr ss:[esp+4]
004914FC |. 56 push esi
004914FD |. 50 push eax
004914FE |. C74424 0C 0000>mov dword ptr ss:[esp+C],0
00491506 |. E8 65FeFFFF call SuperPIM.00491370 ; 通过获取硬盘参数算出 15 位机器码
0049150B |. 6A 14 push 14
0049150D |. 8D4C24 10 lea ecx,dword ptr ss:[esp+10]
00491511 |. 51 push ecx
00491512 |. 8D5424 10 lea edx,dword ptr ss:[esp+10]
00491516 |. 52 push edx
00491517 |. C74424 24 0000>mov dword ptr ss:[esp+24],0
0049151F |. E8 BC32FFFF call SuperPIM.004847E0 ; 算法 call2，把 15 位机器码转换为 20 位字符串
; 即为注册码
00491524 |. 8B7424 2C mov esi,dword ptr ss:[esp+2C]
00491528 |. 6A 28 push 28
0049152A |. 8D4424 18 lea eax,dword ptr ss:[esp+18]
0049152E |. 50 push eax
0049152F |. 56 push esi
00491530 |. C64424 30 01 mov byte ptr ss:[esp+30],1
00491535 |. E8 A632FFFF call SuperPIM.004847E0 ; 把 20 位字符串转换为 40 位字符串用于后面的验证
0049153A |. 8B4424 20 mov eax,dword ptr ss:[esp+20]
0049153E |. 83C0 F0 add eax,-10
00491541 |. 83C4 1C add esp,1C
00491544 |. C64424 14 00 mov byte ptr ss:[esp+14],0
00491549 |. 8D48 0C lea ecx,dword ptr ds:[eax+C]
0049154C |. 83CA FF or edx,FFFFFFFF
0049154F |. F0:0FC111 lock xadd dword ptr ds:[ecx],edx
00491553 |. 4A dec edx
00491554 |. 85D2 test edx,edx
00491556 |. 7F 08 jg short SuperPIM.00491560
00491558 |. 8B08 mov ecx,dword ptr ds:[eax]
0049155A |. 8B11 mov edx,dword ptr ds:[ecx]
0049155C |. 50 push eax
0049155D |. FF52 04 call dword ptr ds:[edx+4]
00491560 |> 8B4424 08 mov eax,dword ptr ss:[esp+8]
00491564 |. 83C0 F0 add eax,-10
00491567 |. C74424 14 FFFF>mov dword ptr ss:[esp+14],-1
0049156F |. 8D48 0C lea ecx,dword ptr ds:[eax+C]
00491572 |. 83CA FF or edx,FFFFFFFF
00491575 |. F0:0FC111 lock xadd dword ptr ds:[ecx],edx
00491579 |. 4A dec edx
0049157A |. 85D2 test edx,edx
0049157C |. 7F 08 jg short SuperPIM.00491586
0049157E |. 8B08 mov ecx,dword ptr ds:[eax]
00491580 |. 8B11 mov edx,dword ptr ds:[ecx]
00491582 |. 50 push eax
00491583 |. FF52 04 call dword ptr ds:[edx+4]
00491586 |> 8B4C24 0C mov ecx,dword ptr ss:[esp+C]
0049158A |. 8BC6 mov eax,esi
0049158C |. 5E pop esi
0049158D |. 64:890D 000000>mov dword ptr fs:[0],ecx
00491594 |. 83C4 14 add esp,14
00491597 \\. C3 retn

========================= 跟进 0049151F E8 BC32FFFF call SuperPIM.004847E0 =========================

004847E0 |$ 6A FF push -1
004847E2 \\. 68 B8BB5000 push SuperPIM.0050BBB8 ; SE 句柄安装
004847E7 . 64:A1 00000000 mov eax,dword ptr fs:[0]
004847ED . 50 push eax
004847EE . 64:8925 000000>mov dword ptr fs:[0],esp
004847F5 . 83EC 14 sub esp,14
004847F8 . 56 push esi
004847F9 . 57 push edi
004847FA . 33FF xor edi,edi
004847FC . 897C24 18 mov dword ptr ss:[esp+18],edi
00484800 . E8 A5570400 call SuperPIM.004C9FAA
00484805 . 8B10 mov edx,dword ptr ds:[eax]
00484807 . 8BC8 mov ecx,eax
00484809 . FF52 0C call dword ptr ds:[edx+C]
......(省略部分)
004848AC . 8BC3 mov eax,ebx
004848AE . BA 01000000 mov edx,1
004848B3 . F0:0FC111 lock xadd dword ptr ds:[ecx],edx
004848B7 . E9 5C010000 jmp SuperPIM.00484A18
004848BC > 8B7C24 14 mov edi,dword ptr ss:[esp+14] ; edi=[12f5b4]计数器
004848C0 . 8B6C24 3C mov ebp,dword ptr ss:[esp+3C] ; ebp=[12f5dc]=14 常数
004848C4 . 8B4C24 1C mov ecx,dword ptr ss:[esp+1C] ; ecx=[12febc]=f 机器码长度
004848C8 > 8BC7 mov eax,edi
004848CA . 99 cdq
004848CB . F7F9 idiv ecx
004848CD . 85D2 test edx,edx
004848CF . 0F8C 06010000 jl SuperPIM.004849DB
004848D5 . 8B4424 38 mov eax,dword ptr ss:[esp+38]
004848D9 . 8B00 mov eax,dword ptr ds:[eax] ; eax="1EHQVLDGL23PN1L"
004848DB . 3B50 F4 cmp edx,dword ptr ds:[eax-C]
004848DE . 0F8F F7000000 jg SuperPIM.004849DB
004848E4 . 8A0C10 mov cl,byte ptr ds:[eax+edx]
004848E7 . 884C24 20 mov byte ptr ss:[esp+20],cl
004848EB . 8B5424 20 mov edx,dword ptr ss:[esp+20]
004848EF . 57 push edi
004848F0 . 52 push edx
004848F1 . E8 FAFBFFFF call SuperPIM.004844F0 ; 算法 call3，返回 eax 值
004848F6 . 33D2 xor edx,edx
004848F8 . B9 2B000000 mov ecx,2B
004848FD . F7F1 div ecx
004848FF . 83C4 08 add esp,8
00484902 . 8BDA mov ebx,edx
00484904 . 80C3 30 add bl,30
00484907 . 80FB 39 cmp bl,39
0048490A . 7E 08 jle short SuperPIM.00484914
0048490C . 80FB 41 cmp bl,41
0048490F . 7D 03 jge short SuperPIM.00484914
00484911 . 80C3 F6 add bl,0F6
00484914 > 3BFD cmp edi,ebp
00484916 . 7D 44 jge short SuperPIM.0048495C
00484918 . 8B46 FC mov eax,dword ptr ds:[esi-4] ; eax=[01635b74]
0048491B . 8B6E F4 mov ebp,dword ptr ds:[esi-C] ; ebp=[01635b70]
0048491E . B9 01000000 mov ecx,1
00484923 . 2BC8 sub ecx,eax
```
004300E8 . 7F 08 jg short SuperPIM.004300F2
004300EA . 8B08 mov ecx,dword ptr ds:[eax]
004300EC . 8B11 mov edx,dword ptr ds:[ecx]
004300EE . 50 push eax
004300EF . FF52 04 call dword ptr ds:[edx+4]
004300F2 > 8B4424 10 mov eax,dword ptr ss:[esp+10]
004300F6 . 83C0 F0 add eax,-10
004300F9 . C74424 20 FFFF>mov dword ptr ss:[esp+20],-1
00430101 . 8D48 0C lea ecx,dword ptr ds:[eax+C]
00430104 . 83CA FF or edx,FFFFFFFF
00430107 . F0:0FC111 lock xadd dword ptr ds:[ecx],edx
0043010B . 4A dec edx
0043010C . 85D2 test edx,edx
0043010E . 5F pop edi
0043010F . 5E pop esi
00430110 . 5B pop ebx
00430111 . 7F 08 jg short SuperPIM.0043011B
00430113 . 8B08 mov ecx,dword ptr ds:[eax]
00430115 . 8B11 mov edx,dword ptr ds:[ecx]
00430117 . 50 push eax
00430118 . FF52 04 call dword ptr ds:[edx+4]
0043011B > 8B4C24 0C mov ecx,dword ptr ss:[esp+C]
0043011F . 64:890D 000000>mov dword ptr fs:[0],ecx
00430126 . 83C4 18 add esp,18
00430129 . C3 retn

========================= 跟进 0042FFAF E8 2C150600 call SuperPIM.004914E0 =========================

004914E0 /$ 6A FF push -1
004914E2 |. 68 20CC5000 push SuperPIM.0050CC20 ; SE 句柄安装
004914E7 |. 64:A1 00000000 mov eax,dword ptr fs:[0]
004914ED |. 50 push eax
004914EE |. 64:8925 000000>mov dword ptr fs:[0],esp
004914F5 |. 83EC 08 sub esp,8
004914F8 |. 8D4424 04 lea eax,dword ptr ss:[esp+4]
004914FC |. 56 push esi
004914FD |. 50 push eax
004914FE |. C74424 0C 0000>mov dword ptr ss:[esp+C],0
00491506 |. E8 65FeFFFF call SuperPIM.00491370 ; 通过获取硬盘参数算出 15 位机器码
0049150B |. 6A 14 push 14
0049150D |. 8D4C24 10 lea ecx,dword ptr ss:[esp+10]
00491511 |. 51 push ecx
00491512 |. 8D5424 10 lea edx,dword ptr ss:[esp+10]
00491516 |. 52 push edx
00491517 |. C74424 24 0000>mov dword ptr ss:[esp+24],0
0049151F |. E8 BC32FFFF call SuperPIM.004847E0 ; 算法 call2，把 15 位机器码转换为 20 位字符串
; 即为注册码
00491524 |. 8B7424 2C mov esi,dword ptr ss:[esp+2C]
00491528 |. 6A 28 push 28
0049152A |. 8D4424 18 lea eax,dword ptr ss:[esp+18]
0049152E |. 50 push eax
0049152F |. 56 push esi
00491530 |. C64424 30 01 mov byte ptr ss:[esp+30],1
00491535 |. E8 A632FFFF call SuperPIM.004847E0 ; 把 20 位字符串转换为 40 位字符串用于后面的验证
0049153A |. 8B4424 20 mov eax,dword ptr ss:[esp+20]
0049153E |. 83C0 F0 add eax,-10
00491541 |. 83C4 1C add esp,1C
00491544 |. C64424 14 00 mov byte ptr ss:[esp+14],0
00491549 |. 8D48 0C lea ecx,dword ptr ds:[eax+C]
0049154C |. 83CA FF or edx,FFFFFFFF
0049154F |. F0:0FC111 lock xadd dword ptr ds:[ecx],edx
00491553 |. 4A dec edx
00491554 |. 85D2 test edx,edx
00491556 |. 7F 08 jg short SuperPIM.00491560
00491558 |. 8B08 mov ecx,dword ptr ds:[eax]
0049155A |. 8B11 mov edx,dword ptr ds:[ecx]
0049155C |. 50 push eax
0049155D |. FF52 04 call dword ptr ds:[edx+4]
00491560 |> 8B4424 08 mov eax,dword ptr ss:[esp+8]
00491564 |. 83C0 F0 add eax,-10
00491567 |. C74424 14 FFFF>mov dword ptr ss:[esp+14],-1
0049156F |. 8D48 0C lea ecx,dword ptr ds:[eax+C]
00491572 |. 83CA FF or edx,FFFFFFFF
00491575 |. F0:0FC111 lock xadd dword ptr ds:[ecx],edx
00491579 |. 4A dec edx
0049157A |. 85D2 test edx,edx
0049157C |. 7F 08 jg short SuperPIM.00491586
0049157E |. 8B08 mov ecx,dword ptr ds:[eax]
00491580 |. 8B11 mov edx,dword ptr ds:[ecx]
00491582 |. 50 push eax
00491583 |. FF52 04 call dword ptr ds:[edx+4]
00491586 |> 8B4C24 0C mov ecx,dword ptr ss:[esp+C]
0049158A |. 8BC6 mov eax,esi
0049158C |. 5E pop esi
0049158D |. 64:890D 000000>mov dword ptr fs:[0],ecx
00491594 |. 83C4 14 add esp,14
00491597 \\. C3 retn

========================= 跟进 0049151F E8 BC32FFFF call SuperPIM.004847E0 =========================

004847E0 |$ 6A FF push -1
004847E2 \\. 68 B8BB5000 push SuperPIM.0050BBB8 ; SE 句柄安装
004847E7 . 64:A1 00000000 mov eax,dword ptr fs:[0]
004847ED . 50 push eax
004847EE . 64:8925 000000>mov dword ptr fs:[0],esp
004847F5 . 83EC 14 sub esp,14
004847F8 . 56 push esi
004847F9 . 57 push edi
004847FA . 33FF xor edi,edi
004847FC . 897C24 18 mov dword ptr ss:[esp+18],edi
00484800 . E8 A5570400 call SuperPIM.004C9FAA
00484805 . 8B10 mov edx,dword ptr ds:[eax]
00484807 . 8BC8 mov ecx,eax
00484809 . FF52 0C call dword ptr ds:[edx+C]
......(省略部分)
004848AC . 8BC3 mov eax,ebx
004848AE . BA 01000000 mov edx,1
004848B3 . F0:0FC111 lock xadd dword ptr ds:[ecx],edx
004848B7 . E9 5C010000 jmp SuperPIM.00484A18
004848BC > 8B7C24 14 mov edi,dword ptr ss:[esp+14] ; edi=[12f5b4]计数器
004848C0 . 8B6C24 3C mov ebp,dword ptr ss:[esp+3C] ; ebp=[12f5dc]=14 常数
004848C4 . 8B4C24 1C mov ecx,dword ptr ss:[esp+1C] ; ecx=[12febc]=f 机器码长度
004848C8 > 8BC7 mov eax,edi
004848CA . 99 cdq
004848CB . F7F9 idiv ecx
004848CD . 85D2 test edx,edx
004848CF . 0F8C 06010000 jl SuperPIM.004849DB
004848D5 . 8B4424 38 mov eax,dword ptr ss:[esp+38]
004848D9 . 8B00 mov eax,dword ptr ds:[eax] ; eax="1EHQVLDGL23PN1L"
004848DB . 3B50 F4 cmp edx,dword ptr ds:[eax-C]
004848DE . 0F8F F7000000 jg SuperPIM.004849DB
004848E4 . 8A0C10 mov cl,byte ptr ds:[eax+edx]
004848E7 . 884C24 20 mov byte ptr ss:[esp+20],cl
004848EB . 8B5424 20 mov edx,dword ptr ss:[esp+20]
004848EF . 57 push edi
004848F0 . 52 push edx
004848F1 . E8 FAFBFFFF call SuperPIM.004844F0 ; 算法 call3，返回 eax 值
004848F6 . 33D2 xor edx,edx
004848F8 . B9 2B000000 mov ecx,2B
004848FD . F7F1 div ecx
004848FF . 83C4 08 add esp,8
00484902 . 8BDA mov ebx,edx
00484904 . 80C3 30 add bl,30
00484907 . 80FB 39 cmp bl,39
0048490A . 7E 08 jle short SuperPIM.00484914
0048490C . 80FB 41 cmp bl,41
0048490F . 7D 03 jge short SuperPIM.00484914
00484911 . 80C3 F6 add bl,0F6
00484914 > 3BFD cmp edi,ebp
00484916 . 7D 44 jge short SuperPIM.0048495C
00484918 . 8B46 FC mov eax,dword ptr ds:[esi-4] ; eax=[01635b74]
0048491B . 8B6E F4 mov ebp,dword ptr ds:[esi-C] ; ebp=[01635b70]
0048491E . B9 01000000 mov ecx,1
00484923 . 2BC8 sub ecx,eax
```



```
00484925    .  8B46 F8        mov eax,dword ptr ds:[esi-8]                ;  eax=[01635b78]  

00484928    .  8D7D 01        lea edi,dword ptr ss:[ebp+1]                ;  edi=ebp+1  

0048492B    .  2BC7          sub eax,edi  

0048492D    .  0BC1          or eax,ecx  

0048492F    .  7D 0E          jge short SuperPIM.0048493F  

00484931    .  57             push edi  

00484932    .  8D4C24 14     lea ecx,dword ptr ss:[esp+14]  

00484936    .  E8 95D2F7FF    call SuperPIM.00401BD0  

0048493B    .  8B7424 10     mov esi,dword ptr ss:[esp+10]  

0048493F    >  85FF          test edi,edi  

00484941    .  881C2E        mov byte ptr ds:[esi+ebp],bl                ;  [1635b88]=bl 存储转换后字符  

00484944    .  0F8C 91000000  jl SuperPIM.004849DB  

0048494A    .  3B7E F8        cmp edi,dword ptr ds:[esi-8]  

0048494D    .  0F8F 88000000  jg SuperPIM.004849DB  

00484953    .  897E F4        mov dword ptr ds:[esi-C],edi  

00484956    .  C60437 00     mov byte ptr ds:[edi+esi],0  

0048495A    .  EB 65          jmp short SuperPIM.004849C1  

0048495C    >  8BC7          mov eax,edi  

0048495E    .  99             cdq  

0048495F    .  F7FD          idiv ebp  

00484961    .  8BFA          mov edi,edx  

00484963    .  85FF          test edi,edi  

00484965    .  7C 74          jl short SuperPIM.004849DB  

00484967    .  8B6E F4        mov ebp,dword ptr ds:[esi-C]  

0048496A    .  3BFD          cmp edi,ebp  

0048496C    .  7F 6D          jg short SuperPIM.004849DB  

0048496E    .  0FBE1437      movsx edx,byte ptr ds:[edi+esi]  

00484972    .  0FBEC3        movsx eax,bl  

00484975    .  8D4402 A0     lea eax,dword ptr ds:[edx+eax-60]  

00484979    .  99             cdq  

0048497A    .  B9 2B000000    mov ecx,2B  

0048497F    .  F7F9          idiv ecx  

00484981    .  8BDA          mov ebx,edx  

00484983    .  80C3 30        add bl,30  

00484986    .  80FB 39        cmp bl,39  

00484989    .  7E 08          jle short SuperPIM.00484993  

0048498B    .  80FB 41        cmp bl,41  

0048498E    .  7D 03          jge short SuperPIM.00484993  

00484990    .  80C3 F6        add bl,0F6  

00484993    >  3BFD          cmp edi,ebp  

00484995    .  7D 44          jge short SuperPIM.004849DB  

00484997    .  837E FC 01     cmp dword ptr ds:[esi-4],1  

0048499B    .  7E 11          jle short SuperPIM.004849AE  

0048499D    .  8B56 F4        mov edx,dword ptr ds:[esi-C]  

004849A0    .  52             push edx  

004849A1    .  8D4C24 14     lea ecx,dword ptr ss:[esp+14]  

004849A5    .  E8 86CDF7FF    call SuperPIM.00401730  

004849AA    .  8B7424 10     mov esi,dword ptr ss:[esp+10]  

004849AE    >  85ED          test ebp,ebp  

004849B0    .  881C37        mov byte ptr ds:[edi+esi],bl  

004849B3    .  7C 26          jl short SuperPIM.004849DB  

004849B5    .  3B6E F8        cmp ebp,dword ptr ds:[esi-8]  

004849B8    .  7F 21          jg short SuperPIM.004849DB  

004849BA    .  896E F4        mov dword ptr ds:[esi-C],ebp  

004849BD    .  C6042E 00     mov byte ptr ds:[esi+ebp],0  

004849C1    >  8B4424 14     mov eax,dword ptr ss:[esp+14]               ;  eax=[12f5b4]计数器  

004849C5    .  8B4C24 18     mov ecx,dword ptr ss:[esp+18]               ;  ecx=[12f5b8]=14 常数  

004849C9    .  40             inc eax  

004849CA    .  3BC1          cmp eax,ecx  

004849CC    .  894424 14     mov dword ptr ss:[esp+14],eax  

004849D0    .^ 0F8C E6FEFFFF  jl SuperPIM.004848BC  

004849D6    .^ E9 B0FEFFFF    jmp SuperPIM.0048488B                       ;  跳向返回  

\========================= 跟进 004848F1    .  E8 FABFFFF    call SuperPIM.004844F0   =========================  

004844F0   /$  51             push ecx  

004844F1   |.  8B4424 0C     mov eax,dword ptr ss:[esp+C]                ;  eax=[12f59c]取字符序号  

004844F5   |.  53             push ebx  

004844F6   |.  55             push ebp  

004844F7   |.  56             push esi  

004844F8   |.  8BF0          mov esi,eax                                   ;  esi=eax  

004844FA   |.  0FAFF0        imul esi,eax  

004844FD   |.  6BF6 07       imul esi,esi,7  

00484500   |.  0FB65C24 14   movzx ebx,byte ptr ss:[esp+14]              ;  ebx=[12f598]取字符  

00484505   |.  83C6 05       add esi,5  

00484508   |.  0FAFF0        imul esi,eax  

0048450B   |.  83C6 17       add esi,17  

0048450E   |.  0FAFF0        imul esi,eax  

00484511   |.  8D68 05       lea ebp,dword ptr ds:[eax+5]                ;  ebp=eax+5  

00484514   |.  8BC5          mov eax,ebp  

00484516   |.  85C0          test eax,eax  

00484518   |.  57             push edi  

00484519   |.  895C24 10     mov dword ptr ss:[esp+10],ebx               ;  [12f590]转存字符  

0048451D   |.  897424 18     mov dword ptr ss:[esp+18],esi               ;  [12f598]存储 esi 值  

00484521   |.  74 5B          je short SuperPIM.0048457E  

00484523   |.  83F8 20       cmp eax,20  

00484526   |.  7C 0C          jl short SuperPIM.00484534  

00484528   |.  25 1F000080    and eax,8000001F  

0048452D   |.  79 05          jns short SuperPIM.00484534  

0048452F   |.  48             dec eax  

00484530   |.  83C8 E0        or eax,FFFFFFE0  

00484533   |.  40             inc eax  

00484534   |>  85C0          test eax,eax  

00484536   |.  74 46          je short SuperPIM.0048457E  

00484538   |.  894424 1C     mov dword ptr ss:[esp+1C],eax               ;  [12f59c]存 eax 值，做计数器用  

0048453C   |.  8D6424 00     lea esp,dword ptr ss:[esp]  

00484540   |>  33C0          /xor eax,eax  

00484542   |.  8A4424 1B     |mov al,byte ptr ss:[esp+1B]                ;  al=[12f59b]存储的 esi 值按字节处理  

00484546   |.  8D4C24 18     |lea ecx,dword ptr ss:[esp+18]  

0048454A   |.  BF 04000000    |mov edi,4  

0048454F   |.  25 80000000    |and eax,80  

00484554   |.  8BF0          |mov esi,eax  

00484556   |>  8A01          |/mov al,byte ptr ds:[ecx]                  ;  al=[12f598]  

00484558   |.  33D2          ||xor edx,edx  

0048455A   |.  8AD0          ||mov dl,al  

0048455C   |.  D0E0          ||shl al,1  

0048455E   |.  8801          ||mov byte ptr ds:[ecx],al                  ;  [12f598]=al  

00484560   |.  81E2 80000000 ||and edx,80  

00484566   |.  85F6          ||test esi,esi  

00484568   |.  74 04          ||je short SuperPIM.0048456E  

0048456A   |.  0C 01          ||or al,1  

0048456C   |.  8801          ||mov byte ptr ds:[ecx],al  

0048456E   |>  41             ||inc ecx  

0048456F   |.  4F             ||dec edi  

00484570   |.  8BF2          ||mov esi,edx  

00484572   |.^ 75 E2          |\jnz short SuperPIM.00484556  

00484574   |.  FF4C24 1C     |dec dword ptr ss:[esp+1C]                  ;  [12f59c]计数器  

00484578   |.^ 75 C6          \\jnz short SuperPIM.00484540  

0048457A   |.  8B7424 18     mov esi,dword ptr ss:[esp+18]               ;  esi=[12f598]变换后的值  

0048457E   |>  8BC5          mov eax,ebp  

00484580   |.  85C0          test eax,eax  

00484582   |.  74 5B          je short SuperPIM.004845DF  

00484584   |.  83F8 20       cmp eax,20  

00484587   |.  7C 0C          jl short SuperPIM.00484595  

00484589   |.  25 1F000080    and eax,8000001F  

0048458E   |.  79 05          jns short SuperPIM.00484595  

00484590   |.  48             dec eax  

00484591   |.  83C8 E0        or eax,FFFFFFE0  

00484594   |.  40             inc eax  

00484595   |>  85C0          test eax,eax  

00484597   |.  74 46          je short SuperPIM.004845DF  

00484599   |.  894424 18     mov dword ptr ss:[esp+18],eax               ;  [12f598]=eax 做计数器  

0048459D   |.  EB 04          jmp short SuperPIM.004845A3  

0048459F   |>  8B5C24 10     /mov ebx,dword ptr ss:[esp+10]  

004845A3   |>  83E3 01        and ebx,1  

004845A6   |.  8BFB          |mov edi,ebx  

004845A8   |.  8D5424 13     |lea edx,dword ptr ss:[esp+13]  

004845AC   |.  BD 04000000    |mov ebp,4  
```



004845B1 `|> 8A0A` `mov cl, byte ptr ds:[edx]` ; cl=[12f593] 转存字符按字节处理

004845B3 `|. 33C0` `xor eax, eax`

004845B5 `|. 8AC1` `mov al, cl`

004845B7 `|. D0E9` `shr cl, 1`

004845B9 `|. 880A` `mov byte ptr ds:[edx], cl`

004845BB `|. 83E0 01` `and eax, 1`

004845BE `|. 85FF` `test edi, edi`

004845C0 `|. 74 05` `je short SuperPIM.004845C7`

004845C2 `|. 80C9 80` `or cl, 80`

004845C5 `|. 880A` `mov byte ptr ds:[edx], cl` ; [12f593]=cl

004845C7 `|> 4A` `dec edx`

004845C8 `|. 4D` `dec ebp`

004845C9 `|. 8BF8` `mov edi, eax`

004845CB `|. ^ 75 E4` `jnz short SuperPIM.004845B1`

004845CD `|. FF4C24 18` `dec dword ptr ss:[esp+18]` ; [12f598] 计数器

004845D1 `|. ^ 75 CC` `jnz short SuperPIM.0048459F`

004845D3 `|. 337424 10` `xor esi, dword ptr ss:[esp+10]` ; esi ^ [12f590] 变换后的值

004845D7 `|. 5F` `pop edi`

004845D8 `|. 8BC6` `mov eax, esi` ; 返回 eax 值

004845DA `|. 5E` `pop esi`

004845DB `|. 5D` `pop ebp`

004845DC `|. 5B` `pop ebx`

004845DD `|. 59` `pop ecx`

004845DE `|. C3` `retn`

004845DF `|> \ 33F3` `xor esi, ebx`

004845E1 `|. 5F` `pop edi`

004845E2 `|. 8BC6` `mov eax, esi`

004845E4 `|. 5E` `pop esi`

004845E5 `|. 5D` `pop ebp`

004845E6 `|. 5B` `pop ebx`

004845E7 `|. 59` `pop ecx`

004845E8 `\. C3` `retn`

---------------------------------------------------------------------------------------------------------

### 【汇编注册机算法部分源码】

```asm
.code

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;函数功能：对字符中指定位置的字符进行运算并返回 eax 值
;函数参数：
;  lpChar：指针，待运算字符的地址
;  dwNum： 字符所在字符串中的序数
;返回值：eax
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CharConvert  proc  lpChar:DWORD, dwNum:DWORD
    local  @szChar1[4]:BYTE, @szChar2[4]:BYTE, @Num:DWORD

    mov  ecx, lpChar
    mov  eax, DWORD ptr [ecx]
    mov  DWORD ptr [@szChar1], eax

    mov  eax, dwNum
    mov  @Num, eax
    mov  esi, eax
    imul  esi, eax
    imul  esi, esi, 7

    movzx  ebx, BYTE ptr [@szChar1]
    add  esi, 5
    imul  esi, eax
    add  esi, 17h
    imul  esi, eax
    add  eax, 5
    push  eax

    test  eax, eax
    mov  DWORD ptr [@szChar2], esi
    je  @1
    cmp  eax, 20h
    jl  @2
    and  eax, 8000001fh
    jns  @2
    dec  eax
    or  eax, 0ffffffe0h
    inc  eax

  @2:
    test  eax, eax
    je  @1
    mov  @Num, eax

  @5:
    xor  eax, eax
    mov  al, BYTE ptr [@szChar2+3]
    lea  ecx, @szChar2
    mov  edi, 4
    and  eax, 80h
    mov  esi, eax

  @4:
    mov  al, BYTE ptr [ecx]
    xor  edx, edx
    mov  dl, al
    shl  al, 1
    mov  BYTE ptr [ecx], al
    and  edx, 80h
    test  esi, esi
    je  @3
    or  al, 1
    mov  BYTE ptr [ecx], al

  @3:
    inc  ecx
    dec  edi
    mov  esi, edx
    jnz  @4
    dec  @Num
    jnz  @5
    mov  esi, DWORD ptr [@szChar2]

  @1:
    pop  eax
    test  eax, eax
    je  @6
    cmp  eax, 20h
    jl  @7
    and  eax, 8000001fh
    jns  @7
    dec  eax
    or  eax, 0ffffffe0h
    inc  eax

  @7:
    test  eax, eax
    je  @6
    mov  @Num, eax
    jmp  @8

  @11:
    mov  ebx, DWORD ptr [@szChar1]

  @8:
    and  ebx, 1
    mov  edi, ebx
    lea  edx, @szChar1
    add  edx, 3
    mov  ch, 4

  @10:
    mov  cl, BYTE ptr [edx]
    xor  eax, eax
    mov  al, cl
    shr  cl, 1
    mov  BYTE ptr [edx], cl
    and  eax, 1
    test  edi, edi
    je  @9
    or  cl, 80h
    mov  BYTE ptr [edx], cl

  @9:
    dec  edx
    dec  ch
    mov  edi, eax
    jnz  @10
    dec  @Num
    jnz  @11
    xor  esi, DWORD ptr [@szChar1]
    mov  eax, esi
    ret

  @6:
    xor  esi, ebx
    mov  eax, esi
    ret

CharConvert  endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;算法函数
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

GetRegKey  proc  hDlg:DWORD
  local  szRegName[32]:BYTE, szRegNum[64]:BYTE, nLen:DWORD, Char:DWORD

  pushad

  invoke  RtlZeroMemory, addr szRegName, sizeof szRegName
  invoke  RtlZeroMemory, addr szRegNum, sizeof szRegNum
  invoke  GetDlgItemText, hDlg, IDC_NAME, addr szRegName, sizeof szRegName

  .if  !eax
    invoke  SetDlgItemText, hDlg, IDC_REG, addr szTextErr
  .else
    mov  nLen, eax
    xor  edi, edi

  @1:
    mov  ecx, nLen
    mov  edx, 14h
    push  edx
    mov  eax, edi
    push  edi
    cdq
    idiv  ecx
    lea  eax, szRegName
    movzx  edx, BYTE ptr [eax+edx]
    mov  Char, edx
    invoke  CharConvert, addr Char, edi

    xor  edx, edx
    mov  ecx, 2bh
    div  ecx
    mov  ebx, edx
    add  bl, 30h
    cmp  bl, 39h
    jle  @2
    cmp  bl, 41h
    jge  @2
    add  bl, 0f6h

  @2:
    pop  edi
    mov  BYTE ptr [szRegNum+edi], bl
    pop  edx
    inc  edi
    cmp  edi, edx
    jl  @1

    invoke  SetDlgItemText, hDlg, IDC_REG, addr szRegNum

  .endif

  popad
  ret

GetRegKey  endp
```

### 【版权声明】

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

---------------------------------------------------------------------------------------------------------

`4dd466d0e1e0cc123afbe504da2462ca`
