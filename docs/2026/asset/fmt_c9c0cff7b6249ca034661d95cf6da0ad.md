
```
010B37B0  00 00 00 00 24 CA 40 CC 3B FF EC 00 00 00 00 00  ....$蔃??....

010B37C0  00 00 00 00 10 6D 1C B9 68 E4 53 21 00 00 00 00  ....m筯銼!....

010B37D0  00 00 00 00 40 56 FF 07 BA 1E CC AF 04 00 00 00  ....@V?摊...

010B37E0  00 00 00 00 00 21 E8 1F 29 52 B4 B8 A8 00 00 00  .....!?)R锤?..

010B37F0  00 00 00 00 00 A4 A4 7C C8 8D 5B F9 B9 17 00 00  .....い|葝[..

010B3800  00 00 00 00 00 10 27 87 31 F0 DF 10 27 56 03 00  .....'?疬'V.

010B3810  00 00 00 00 00 40 7E 01 F7 C6 7D 5F 7E 1D 78 00  .....@~髌}\_~x.

010B3820  00 00 00 00 00 00 C1 35 BC FA AF 6D C5 25 E4 10  ......?贱痬??
```

用计算器简单算一下前几项，就知道这是`0x24`的各次幂，也就是10进制36的各次幂。

想到注册码的范围从0-9，A-Z，正好是36个。于是，这个乘法无非是把一个25位的36进制数(注册码)转换成一个32位的16进制数(也就是2进制的128位)。

转换之后，来到这里：

```
01092437  |.  8D9F C8010000  lea ebx, [edi+1C8]
0109243D  |.  8B33           mov esi, [ebx]        ; 取出一个DWORD
0109243F  |.  56             push esi
01092440  |.  E8 36FBFFFF    call RMCPRO.01091F7B    ; 计算其二进制表示中1的各数
01092445  |.  8AC8           mov cl, al
01092447  |.  D3CE           ror esi, cl
01092449  |.  8933           mov [ebx], esi
0109244B  |.  8D9F CC010000  lea ebx, [edi+1CC]
01092451  |.  8B33           mov esi, [ebx]
01092453  |.  56             push esi
01092454  |.  E8 22FBFFFF    call RMCPRO.01091F7B
01092459  |.  8AC8           mov cl, al
0109245B  |.  D3CE           ror esi, cl
0109245D  |.  8933           mov [ebx], esi
0109245F  |.  8D9F D0010000  lea ebx, [edi+1D0]
01092465  |.  8B33           mov esi, [ebx]
01092467  |.  56             push esi
01092468  |.  E8 0EFBFFFF    call RMCPRO.01091F7B
0109246D  |.  8AC8           mov cl, al
0109246F  |.  D3CE           ror esi, cl
01092471  |.  81C7 D4010000  add edi, 1D4
01092477  |.  8933           mov [ebx], esi
01092479  |.  8B37           mov esi, [edi]
0109247B  |.  56             push esi
0109247C  |.  E8 FAFAFFFF    call RMCPRO.01091F7B
01092481  |.  8AC8           mov cl, al
01092483  |.  83C4 10        add esp, 10
01092486  |.  D3CE           ror esi, cl
01092488  |.  33C0           xor eax, eax
0109248A  |.  40             inc eax
0109248B  |.  8937           mov [edi], esi
```

把128位的结果分成4个`dword`，计算其中每一个的"1"的各数，然后循环右移相应的位数。

接着，再把这个结果跟前面用户名和email相加的那个指比较，如果相等，那就成功了。

算法：

```
A=特征字串+用户名+email
B=注册码(36进制→16进制)→分组ror
```

如果`A=B`，那就成功啦。

逆算法非常简单。只要把`A`分组`rol`，然后转换成36进制就OK了。

例如前面得到的结果：

```
A=F2 35 0E 35 30 50 20 32 2E 78 78 00 00 00 00 00
```

分组`rol`之后：

```
0E 35 F2 35 32 30 50 20 07 E0 82 87 00 00 00 00
```

再转换成36进制:(`0-9=0-9，10-35=A-Z`)

`IVEBX5I0SFNIGAEKD240000000`

注意这个是逆序的。这就是注册码了。

另一个update模块，非常地类似。

```
00436FAE  |.  FF75 10        push [arg.3]        ; 注册码
00436FB1  |.  8B10           mov edx, [eax]
00436FB3  |.  FF75 0C        push [arg.2]        ; email
00436FB6  |.  8BC8           mov ecx, eax
00436FB8  |.  FF75 08        push [arg.1]        ; 用户名
00436FBB  |.  FF52 10        call near [edx+10]    ; 进入RMCUpdater.dll
```

这里还是一样的：

```
100016A1  |.  E8 F7FDFFFF    call RMCUpdat.1000149D
100016A6  |.  E8 7BFEFFFF    call RMCUpdat.10001526
100016AB  |.  85C0           test eax, eax
100016AD  |.  75 0A          jnz short RMCUpdat.100016B9
```

唯一的不同就是，特征字符串变成了：

```
1002AF08  52 4D 43 55 70 64 20 32 2E 78 78 00 00 00 00 00  RMCUpd 2.xx.....
```

那就重新计算一下注册码吧：

`2JEELTNFLDF4GAEKD240000000`

测试成功，可以升级了。

【版权声明】   本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#加密算法](https://bbs.kanxue.com/forum-4-1-5.htm)
