
# 破解 Quick Screen Recorder

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-318.htm`](https://www.pediy.com/kssd/pediy07/pediy7-318.htm)

## 破解信息

- 【破解作者】 `yijun`
- 【作者邮箱】 `yijun8354@sina.com`
- 【使用工具】 `OD`, `PEID`
- 【破解平台】 `WinXP`
- 【软件名称】 `Quick Screen Recorder`
- 【下载地址】 天空
- 【软件简介】 `Etrusoft Quick Screen Recorder` is a tool used for recording screen activity into standard `AVI` video files. If you move the cursor, launch a new program, type some text, click a few buttons, or select some menus -- anything that you see on your screen -- `Quick Screen Recorder` will be able to record all these and allow you to play them back later on.
- 【软件大小】 `540K`
- 【加壳方式】 无
- 【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

`PEID` 查壳知该软件无壳，`Microsoft Visual C++ 6.0` 编写。`OD` 载入很容易来到以下地方：

```
00404C69    55             push ebp   //在此下断
00404C6A    56             push esi
00404C6B    57             push edi
00404C6C    8BE9           mov ebp,ecx
00404C6E    6A 01          push 1
00404C70    E8 7C910200    call qsr.0042DDF1                 ; 取用户名
00404C75    8D7D 60        lea edi,dword ptr ss:[ebp+60]
00404C78    8BCF           mov ecx,edi
00404C7A    E8 FA3E0200    call qsr.00428B79                 ; 取注册码
00404C7F    8BCF           mov ecx,edi
00404C81    E8 A73E0200    call qsr.00428B2D
00404C86    8D75 64        lea esi,dword ptr ss:[ebp+64]
00404C89    8BCE           mov ecx,esi
00404C8B    E8 E93E0200    call qsr.00428B79                 ; 用户名送 EAX
00404C90    8BCE           mov ecx,esi
00404C92    E8 963E0200    call qsr.00428B2D
00404C97    8B06           mov eax,dword ptr ds:[esi]
00404C99    8378 F8 02     cmp dword ptr ds:[eax-8],2
00404C9D    7D 26          jge short qsr.00404CC5
00404C9F    6A 40          push 40
00404CA1    68 3CE54500    push qsr.0045E53C                 ; ASCII "Quick Screen Recorder"
00404CA6    68 24E54500    push qsr.0045E524                 ; ASCII "Please input your name."
00404CAB    8BCD           mov ecx,ebp
00404CAD    E8 C3840200    call qsr.0042D175
00404CB2    8B4C24 1C      mov ecx,dword ptr ss:[esp+1C]
00404CB6    64:890D 0000000>mov dword ptr fs:[0],ecx
00404CBD    5F             pop edi
00404CBE    5E             pop esi
00404CBF    5D             pop ebp
00404CC0    5B             pop ebx
00404CC1    83C4 18        add esp,18
00404CC4    C3             retn
00404CC5    8A45 5C        mov al,byte ptr ss:[ebp+5C]
00404CC8    84C0           test al,al
00404CCA    0F85 1C010000  jnz qsr.00404DEC
00404CD0    8D4C24 14      lea ecx,dword ptr ss:[esp+14]
00404CD4    E8 87E3FFFF    call qsr.00403060
00404CD9    51             push ecx
00404CDA    C74424 28 00000>mov dword ptr ss:[esp+28],0
00404CE2    8BCC           mov ecx,esp
00404CE4    896424 14      mov dword ptr ss:[esp+14],esp
00404CE8    56             push esi
00404CE9    E8 12A50200    call qsr.0042F200                 ; 测试用户名是否是 0
00404CEE    51             push ecx
00404CEF    C64424 2C 01   mov byte ptr ss:[esp+2C],1
00404CF4    8BCC           mov ecx,esp
00404CF6    896424 20      mov dword ptr ss:[esp+20],esp
00404CFA    57             push edi
00404CFB    E8 00A50200    call qsr.0042F200                 ; 测试注册码是否是 0
00404D00    8D4C24 1C      lea ecx,dword ptr ss:[esp+1C]
00404D04    C64424 2C 00   mov byte ptr ss:[esp+2C],0
00404D09    E8 92E3FFFF    call qsr.004030A0                 ; 关键 CALL，跟进~~~~~~~~~~~
00404D0E    84C0           test al,al                         ; 刚才那两处都等就注册成功，此时 AL=1~~~
00404D10    75 37          jnz short qsr.00404D49             ; AL=1 就注册成功~~~~
00404D12    6A 40          push 40
00404D14    68 3CE54500    push qsr.0045E53C                 ; ASCII "Quick Screen Recorder"
00404D19    68 DCE44500    push qsr.0045E4DC                 ; ASCII "Sorry, your registration key is wrong. Please check it and try again."
00404D1E    8BCD           mov ecx,ebp
00404D20    E8 50840200    call qsr.0042D175
00404D25    8D4C24 14      lea ecx,dword ptr ss:[esp+14]
00404D29    C74424 24 FFFFF>mov dword ptr ss:[esp+24],-1
00404D31    E8 5AE3FFFF    call qsr.00403090
00404D36    8B4C24 1C      mov ecx,dword ptr ss:[esp+1C]
00404D3A    64:890D 0000000>mov dword ptr fs:[0],ecx
00404D41    5F             pop edi
00404D42    5E             pop esi
00404D43    5D             pop ebp
00404D44    5B             pop ebx
00404D45    83C4 18        add esp,18
00404D48    C3             retn
```

跟进 `00404D09` 处 `CALL` 来到：

```
004030A0    6A FF          push -1        //一路 F8 下去~~~~
004030A2    68 C88F4400    push qsr.00448FC8
004030A7    64:A1 00000000  mov eax,dword ptr fs:[0]
004030AD    50             push eax
004030AE    64:8925 0000000>mov dword ptr fs:[0],esp
004030B5    83EC 24        sub esp,24
004030B8    53             push ebx
004030B9    55             push ebp
004030BA    56             push esi
004030BB    57             push edi
004030BC    68 28E24500    push qsr.0045E228                 ; ASCII "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"
004030C1    8D4C24 24      lea ecx,dword ptr ss:[esp+24]
004030C5    C74424 40 01000>mov dword ptr ss:[esp+40],1
004030CD    E8 27C40200    call qsr.0042F4F9
004030D2    A1 38F44500    mov eax,dword ptr ds:[45F438]
004030D7    894424 2C      mov dword ptr ss:[esp+2C],eax
004030DB    8D4C24 48      lea ecx,dword ptr ss:[esp+48]
004030DF    C64424 3C 03   mov byte ptr ss:[esp+3C],3
004030E4    E8 905A0200    call qsr.00428B79
004030E9    8D4C24 48      lea ecx,dword ptr ss:[esp+48]
004030ED    E8 3B5A0200    call qsr.00428B2D
004030F2    8D4C24 44      lea ecx,dword ptr ss:[esp+44]
004030F6    E8 7E5A0200    call qsr.00428B79
004030FB    8D4C24 44      lea ecx,dword ptr ss:[esp+44]
004030FF    E8 295A0200    call qsr.00428B2D
00403104    8B4C24 48      mov ecx,dword ptr ss:[esp+48]
00403108    8B41 F8        mov eax,dword ptr ds:[ecx-8]
0040310B    83F8 02        cmp eax,2
0040310E    0F8C 4F030000  jl qsr.00403463                   ; 用户名小于 2 就跳
00403114    8B5424 44      mov edx,dword ptr ss:[esp+44]
00403118    837A F8 18     cmp dword ptr ds:[edx-8],18
0040311C    0F85 41030000  jnz qsr.00403463                  ; 注册码不等于 18（16 进制）就跳
00403122    8D4424 30      lea eax,dword ptr ss:[esp+30]
00403126    6A 01          push 1
00403128    50             push eax
00403129    8D4C24 50      lea ecx,dword ptr ss:[esp+50]
0040312D    E8 C4550200    call qsr.004286F6
00403132    8B00           mov eax,dword ptr ds:[eax]
00403134    8D4C24 20      lea ecx,dword ptr ss:[esp+20]
00403138    50             push eax
00403139    C64424 40 04   mov byte ptr ss:[esp+40],4
0040313E    E8 4D560200    call qsr.00428790
00403143    8D4C24 30      lea ecx,dword ptr ss:[esp+30]
00403147    8BF0           mov esi,eax
00403149    C64424 3C 03   mov byte ptr ss:[esp+3C],3
0040314E    E8 38C30200    call qsr.0042F48B
00403153    8D46 0A        lea eax,dword ptr ds:[esi+A]
00403156    B9 3E000000    mov ecx,3E
0040315B    99             cdq
0040315C    F7F9           idiv ecx
0040315E    6A 01          push 1
```
