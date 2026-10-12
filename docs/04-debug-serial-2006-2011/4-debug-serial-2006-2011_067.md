
# HyperCalendar 2.52

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-24.htm`](https://www.pediy.com/kssd/pediy08/pediy8-24.htm)

**【破文标题】** `HyperCalendar 2.52 注册算法分析+汇编注册机`

**【破文作者】** `snake`

**【软件名称】** `HyperCalendar 2.52`

**【下载地址】** [`yncnc.onlinedown.net/soft/11274.htm`](http://yncnc.onlinedown.net/soft/11274.htm)

**【软件简介】** 日历软件，多媒体日历，支持各种媒体的播放，各种图片的显示!

**【调试工具】** `Windows 2000+SP4`、`PEiD`、`Ollydbg`

**【作者声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

### 算法总结

以本机 ID `"HC267726978-28175866804502"` 为例说明

1. 将 ID 与几组固定字符串连接组成新串

```
HyperCalendar 2HC267726978-28175866804502dDFsewrEeRWfdsg#@$SFF^%J&^\*hgj\*&(JIUL\_fWWFFGG
4534o-=iqew.,nva;gf453242134saf29050327%$&$%&@$#%#!%!#@\_+#$%#@#%#RWEQFSADdsfg$#$DS2AF#
$&^%$$##@@#GF><MM<<:fdgdsewf:::@dd#$^DS@#$$%&DSGA\[\]\]\[\]{}reT{{ess###sf|~~@!#\`\`degfsdfVC
NM<KJFHDYMGNsr354375uhyfgdsbnhgnfwe9257453242fdljpewrpwtew345767i8u)\*\*^#%!#@\_+#$%#@#%#
RWEQSADdsfg$#$DS2fdsg#^DSDF@fhlpoyui90932sag87dsslkl34vo3245dsfa#^#%$#@#@!dg$#%5443535
234|~~@!#\`\`degfsdfVCNM<KJFHDYMGNsr354375uhyfgdsbnhgnfwe9257
```

2. 计算组成新串的 MD5 值：`1F318CA01CB0CC44A91353B92B88D888`

3. 根据新串的 MD5 值生成数据表

```
00128D28  31 1F A0 8B B0 1B 44 CC 13 A9 B9 52 88 2A 88 D7
00128D38  37 40 98 61 52 89 A5 26 55 72 AF 11 3E 10 17 63
00128D48  12 31 4D A4 E4 4A 23 AA 20 5E C6 7C 80 2E C3 6E
00128D58  95 9A 54 C9 BC 46 F9 40 5D 8C DD 00 62 86 48 25
00128D68  8D A8 81 78 18 F3 01 BA 0C BB 4A C4 35 91 92 2B
00128D78  E6 03 74 31 76 03 88 19 22 95 57 6A 51 25 F1 1A
00128D88  06 E8 33 EC 2A 11 D4 44
```

4. 依次取用户 ID 字符的 ASCII 值计算生成新字符
5. 新字符与取的用户 ID 字符前面的子串组成新字符串
6. 判断新字符串是否要变换
7. 新字符串与上面生成的数据表进行运算、变换
8. 取结果的首字节计算后得到注册码的一位
9. 循环直到取完用户 ID 的字符，并生成每一位注册码
10. 计算用户 ID 所有字符的 ASCII 码和运算后得注册码最后一位

最终结果

```
UserID: HC267726978-28175866804502
RegCode:6AR65O6K7V65IQ62Q98981D88M4
```

有些具体过程语言表达不清，见后面分析

### 破解过程

#### 一、程序脱壳

用 `PEiD` 查壳，为 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`，`OD` 载入手动脱之。（略）

脱壳后查为 `Microsoft Visual C++ 7.0`

#### 二、算法分析

`OD` 载入脱壳后的程序，运行并输入假码，查找提示的错误的信息分析后可在下处设断

```asm
004089A9     8B6C24 1C         mov ebp,dword ptr ss:[esp+1C]                 ; 取注册码
004089AD     68 B2294600       push x.004629B2
004089B2     55                push ebp
004089B3     E8 723F0300       call x.0043C92A                               ; 判断注册码是否为空
004089B8     83C4 08           add esp,8
004089BB     85C0              test eax,eax
004089BD     75 21             jnz short x.004089E0
004089BF     90                nop
004089C0     90                nop
004089C1     57                push edi
004089C2     51                push ecx
004089C3     8BCC              mov ecx,esp
004089C5     896424 1C         mov dword ptr ss:[esp+1C],esp
004089C9     68 743D4600       push x.00463D74                               ; ASCII "Invalid Registration Code."
004089CE     E8 BDB7FFFF       call x.00404190
004089D3     E8 38670100       call x.0041F110
004089D8     83C4 08           add esp,8
004089DB     E9 B2010000       jmp x.00408B92
004089E0     90                nop
004089E1     90                nop
004089E2     90                nop
004089E3     68 C42C4600       push x.00462CC4                               ; ASCII "HyperCalendar 2"
004089E8     8D4C24 18         lea ecx,dword ptr ss:[esp+18]
004089EC     E8 9FB7FFFF       call x.00404190
004089F1     51                push ecx
004089F2     8BD4              mov edx,esp
004089F4     896424 24         mov dword ptr ss:[esp+24],esp
004089F8     8D4C24 10         lea ecx,dword ptr ss:[esp+10]
004089FC     51                push ecx
004089FD     50                push eax
004089FE     52                push edx
004089FF     C68424 B8090000 0>mov byte ptr ss:[esp+9B8],3
00408A07     E8 54C5FFFF       call x.00404F60                               ; 上面的字符串与注册码连接
00408A0C     8B4424 1C         mov eax,dword ptr ss:[esp+1C]
00408A10     83C4 08           add esp,8
00408A13     83C0 F0           add eax,-10
00408A16     896424 20         mov dword ptr ss:[esp+20],esp
00408A1A     8BF4              mov esi,esp
00408A1C     50                push eax
00408A1D     C68424 B4090000 0>mov byte ptr ss:[esp+9B4],4
00408A25     E8 A6AEFFFF       call x.004038D0
00408A2A     8D5424 1C         lea edx,dword ptr ss:[esp+1C]
00408A2E     83C4 04           add esp,4
00408A31     83C0 10           add eax,10
00408A34     52                push edx
00408A35     8906              mov dword ptr ds:[esi],eax
00408A37     C68424 B4090000 0>mov byte ptr ss:[esp+9B4],3
00408A3F     E8 AC4C0200       call x.0042D6F0                               ; 算法 call，跟进
00408A44     83C4 0C           add esp,0C
00408A47     8B4424 14         mov eax,dword ptr ss:[esp+14]
00408A4B     83C0 F0           add eax,-10
00408A4E     C68424 A8090000 0>mov byte ptr ss:[esp+9A8],6
00408A56     8D48 0C           lea ecx,dword ptr ds:[eax+C]
00408A59     83CA FF           or edx,FFFFFFFF
00408A5C     F0:0FC111         lock xadd dword ptr ds:[ecx],edx
00408A60     4A                dec edx
00408A61     85D2              test edx,edx
00408A63     7F 08             jg short x.00408A6D
00408A65     8B08              mov ecx,dword ptr ds:[eax]
00408A67     8B11              mov edx,dword ptr ds:[ecx]
00408A69     50                push eax
00408A6A     FF52 04           call dword ptr ds:[edx+4]
00408A6D     90                nop
00408A6E     90                nop
00408A6F     90                nop
00408A70     8B4424 10         mov eax,dword ptr ss:[esp+10]
00408A74     55                push ebp                                     ; 假码入栈
00408A75     50                push eax                                     ; 真码入栈
00408A76     E8 AF3E0300       call x.0043C92A                               ; 验证
00408A7B     83C4 08           add esp,8
00408A7E     85C0              test eax,eax
00408A80     0F85 C9000000     jnz x.00408B4F                                ; 关键跳转
00408A86     90                nop
00408A87     90                nop
00408A88     90                nop
00408A89     6A 01             push 1
00408A8B     E8 C0580200       call x.0042E350
00408A90     83C4 04           add esp,4
00408A93     90                nop
00408A94     90                nop
00408A95     90                nop
00408A96     57                push edi
00408A97     51                push ecx
00408A98     8BCC              mov ecx,esp
00408A9A     896424 20         mov dword ptr ss:[esp+20],esp
00408A9E     68 503D4600       push x.00463D50                               ; ASCII "Register Successfully! Thank you."
00408AA3     E8 E8B6FFFF       call x.00404190
00408AA8     E8 63660100       call x.0041F110
00408AAD     83C4 08           add esp,8
......
```

\============== 跟进 `00408A3F call x.0042D6F0` ===============

```asm
0042D73F     6A 40             push 40
0042D741     68 80784600       push x.00467880                               ; ASCII "dDFsewrEeRWfdsg#@$SFF^%J&^\*hgj\*&(JIUL\_fWWFFGG4534o-=iqew.,nva;gf"
0042D746     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D74A     E8 1165FDFF       call x.00403C60                               ; 与上面字符串相连
0042D74F     90                nop
0042D750     90                nop
0042D751     90                nop
0042D752     90                nop
0042D753     6A 20             push 20
0042D755     68 5C784600       push x.0046785C                               ; ASCII "453242134saf29050327%$&$%&@$#%#!"
0042D75A     8D4C24 54         lea ecx,dword ptr ss:[esp+54]
0042D75E     E8 FD64FDFF       call x.00403C60                               ; 与上面字符串相连
0042D763     90                nop
0042D764     90                nop
0042D765     90                nop
0042D766     6A 20             push 20
```



```
0042D768     68 38784600       push x.00467838                      ; ASCII "%!#@\_+#$%#@#%#RWEQFSADdsfg$#$DS2"  

0042D76D     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D771     E8 EA64FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D776     90                nop  

0042D777     90                nop  

0042D778     90                nop  

0042D779     90                nop  

0042D77A     90                nop  

0042D77B     90                nop  

0042D77C     6A 21             push 21  

0042D77E     68 14784600       push x.00467814                      ; ASCII "AF#$&^%$$##@@#GF><MM<<:fdgdsewf::"  

0042D783     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D787     E8 D464FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D78C     90                nop  

0042D78D     90                nop  

0042D78E     90                nop  

0042D78F     6A 1F             push 1F  

0042D791     68 F4774600       push x.004677F4                      ; ASCII ":@dd#$^DS@#$$%&DSGA\[\]\]\[\]{}reT{{"  

0042D796     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D79A     E8 C164FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D79F     90                nop  

0042D7A0     90                nop  

0042D7A1     90                nop  

0042D7A2     90                nop  

0042D7A3     90                nop  

0042D7A4     90                nop  

0042D7A5     6A 40             push 40  

0042D7A7     68 B0774600       push x.004677B0                      ; ASCII "ess###sf|~~@!#\`\`degfsdfVCNM<KJFHDYMGNsr354375uhyfgdsbnhgnfwe9257"  

0042D7AC     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D7B0     E8 AB64FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D7B5     90                nop  

0042D7B6     90                nop  

0042D7B7     90                nop  

0042D7B8     90                nop  

0042D7B9     6A 22             push 22  

0042D7BB     68 88774600       push x.00467788                      ; ASCII "453242fdljpewrpwtew345767i8u)\*\*^#%"  

0042D7C0     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D7C4     E8 9764FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D7C9     90                nop  

0042D7CA     90                nop  

0042D7CB     90                nop  

0042D7CC     90                nop  

0042D7CD     90                nop  

0042D7CE     90                nop  

0042D7CF     6A 1E             push 1E  

0042D7D1     68 68774600       push x.00467768                      ; ASCII "!#@\_+#$%#@#%#RWEQSADdsfg$#$DS2"  

0042D7D6     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D7DA     E8 8164FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D7DF     90                nop  

0042D7E0     90                nop  

0042D7E1     90                nop  

0042D7E2     6A 1F             push 1F  

0042D7E4     68 48774600       push x.00467748                      ; ASCII "fdsg#^DSDF@fhlpoyui90932sag87ds"  

0042D7E9     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D7ED     E8 6E64FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D7F2     90                nop  

0042D7F3     90                nop  

0042D7F4     90                nop  

0042D7F5     6A 21             push 21  

0042D7F7     68 24774600       push x.00467724                      ; ASCII "slkl34vo3245dsfa#^#%$#@#@!dg$#%54"  

0042D7FC     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D800     E8 5B64FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D805     90                nop  

0042D806     90                nop  

0042D807     90                nop  

0042D808     6A 20             push 20  

0042D80A     68 00774600       push x.00467700                      ; ASCII "43535234|~~@!#\`\`degfsdfVCNM<KJFH"  

0042D80F     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D813     E8 4864FDFF       call x.00403C60                      ; 与上面字符串相连  

0042D818     90                nop  

0042D819     90                nop  

0042D81A     90                nop  

0042D81B     90                nop  

0042D81C     6A 20             push 20  

0042D81E     68 DC764600       push x.004676DC                      ; ASCII "DYMGNsr354375uhyfgdsbnhgnfwe9257"  

0042D823     8D4C24 54         lea ecx,dword ptr ss:\[esp+54\]  

0042D827     E8 3464FDFF       call x.00403C60                      ; 与上面字符串相连  

......  

0042D841     8B7424 48         mov esi,dword ptr ss:\[esp+48\]  

0042D845     90                nop  

0042D846     90                nop  

0042D847     90                nop  

0042D848     85ED             test ebp,ebp  

0042D84A     0F8C 1D040000    jl x.0042DC6D  

0042D850     3B6E F4          cmp ebp,dword ptr ds:\[esi-C\]  

0042D853     0F8F 14040000    jg x.0042DC6D  

0042D859     0FBE042E         movsx eax,byte ptr ds:\[esi+ebp\]       ; 取用户 ID 字符的 ASCII 值  

0042D85D     03C5             add eax,ebp                             ; ebp 为计数器  

0042D85F     99               cdq  

0042D860     B9 1A000000      mov ecx,1A  

0042D865     F7F9             idiv ecx  

0042D867     80C2 41          add dl,41  

0042D86A     885424 24        mov byte ptr ss:\[esp+24\],dl           ; 运算结果  

0042D86E     90                nop  

0042D86F     90                nop  

0042D870     90                nop  

0042D871     8B5424 24        mov edx,dword ptr ss:\[esp+24\]  

0042D875     6A 01             push 1  

0042D877     52               push edx  

0042D878     8D4C24 20        lea ecx,dword ptr ss:\[esp+20\]  

0042D87C     E8 EFC9FFFF      call x.0042A270                         ; 存储结果  

0042D881     C64424 3C 03     mov byte ptr ss:\[esp+3C\],3  

0042D886     90                nop  

0042D887     90                nop  

0042D888     90                nop  

0042D889     55               push ebp  

0042D88A     8D4424 2C        lea eax,dword ptr ss:\[esp+2C\]  

0042D88E     50               push eax  

0042D88F     8D4C24 50        lea ecx,dword ptr ss:\[esp+50\]  

0042D893     E8 2887FDFF      call x.00405FC0  

0042D898     8B00             mov eax,dword ptr ds:\[eax\]  

0042D89A     8B48 F4          mov ecx,dword ptr ds:\[eax-C\]  

0042D89D     51               push ecx  

0042D89E     50               push eax  

0042D89F     8D4C24 20        lea ecx,dword ptr ss:\[esp+20\]  

0042D8A3     C64424 44 04     mov byte ptr ss:\[esp+44\],4  

0042D8A8     E8 B363FDFF      call x.00403C60                         ; 取的字符前的子字符串与运算结果连成新串  

0042D8AD     8B4424 28        mov eax,dword ptr ss:\[esp+28\]  

0042D8B1     83C0 F0          add eax,-10  

0042D8B4     C64424 3C 03     mov byte ptr ss:\[esp+3C\],3  

0042D8B9     8D48 0C          lea ecx,dword ptr ds:\[eax+C\]  

0042D8BC     83CA FF          or edx,FFFFFFFF  

0042D8BF     F0:0FC111        lock xadd dword ptr ds:\[ecx\],edx  

0042D8C3     4A               dec edx  

0042D8C4     85D2             test edx,edx  

0042D8C6     7F 08            jg short x.0042D8D0  

0042D8C8     8B08             mov ecx,dword ptr ds:\[eax\]  

0042D8CA     8B11             mov edx,dword ptr ds:\[ecx\]  

0042D8CC     50               push eax  

0042D8CD     FF52 04          call dword ptr ds:\[edx+4\]  

0042D8D0     90                nop  

0042D8D1     90                nop  

0042D8D2     90                nop  

0042D8D3     8BC5             mov eax,ebp  

0042D8D5     99               cdq  

0042D8D6     B9 03000000      mov ecx,3  

0042D8DB     F7F9             idiv ecx  

0042D8DD     85D2             test edx,edx  

0042D8DF     75 79            jnz short x.0042D95A                  ; 取字符的位数是否为 3 的倍数  

0042D8E1     90                nop  

0042D8E2     90                nop  
```



```text
0042D8E3     90                     nop  

0042D8E4     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D8E8     8B72 F4             mov esi,dword ptr ds:[edx-C]  

0042D8EB     90                     nop  

0042D8EC     90                     nop  

0042D8ED     90                     nop  

0042D8EE     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D8F2     8B42 FC             mov eax,dword ptr ds:[edx-4]  

0042D8F5     B9 01000000          mov ecx,1  

0042D8FA     2BC8                 sub ecx,eax  

0042D8FC     8B42 F8             mov eax,dword ptr ds:[edx-8]  

0042D8FF     2BC6                 sub eax,esi  

0042D901     0BC1                 or eax,ecx  

0042D903     7D 0E                jge short x.0042D913  

0042D905     56                     push esi  

0042D906     8D4C24 1C            lea ecx,dword ptr ss:[esp+1C]  

0042D90A     E8 9157FDFF          call x.004030A0  

0042D90F     8B5424 18            mov edx,dword ptr ss:[esp+18]  

0042D913     56                     push esi  

0042D914     52                     push edx  

0042D915     E8 8624FFFF          call x.0041FDA0                   ; 若是，生成的新字串做进一步运算  

0042D91A     8B5424 20            mov edx,dword ptr ss:[esp+20]  

0042D91E     83C4 08              add esp,8  

0042D921     83FE FF              cmp esi,-1  

0042D924     8BC6                 mov eax,esi  

0042D926     75 16                jnz short x.0042D93E  

0042D928     85D2                 test edx,edx  

0042D92A     75 04                jnz short x.0042D930  

0042D92C     33C0                 xor eax,eax  

0042D92E     EB 16                jmp short x.0042D946  

0042D930     8BC2                 mov eax,edx  

0042D932     8D70 01              lea esi,dword ptr ds:[eax+1]  

0042D935     8A08                 mov cl,byte ptr ds:[eax]  

0042D937     40                     inc eax  

0042D938     84C9                 test cl,cl  

0042D93A     75 F9                jnz short x.0042D935  

0042D93C     2BC6                 sub eax,esi  

0042D93E     85C0                 test eax,eax  

0042D940     0F8C 27030000        jl x.0042DC6D  

0042D946     3B42 F8              cmp eax,dword ptr ds:[edx-8]  

0042D949     0F8F 1E030000        jg x.0042DC6D  

0042D94F     8942 F4              mov dword ptr ds:[edx-C],eax  

0042D952     8B4C24 18            mov ecx,dword ptr ss:[esp+18]  

0042D956     C60408 00            mov byte ptr ds:[eax+ecx],0  

0042D95A     90                     nop                                ; 不是，跳到此处  

0042D95B     90                     nop  

0042D95C     90                     nop  

0042D95D     8B7424 4C            mov esi,dword ptr ss:[esp+4C]  

0042D961     51                     push ecx  

0042D962     8B4E F0              mov ecx,dword ptr ds:[esi-10]  

0042D965     8B11                 mov edx,dword ptr ds:[ecx]  

0042D967     83C6 F0              add esi,-10  

0042D96A     896424 34            mov dword ptr ss:[esp+34],esp  

0042D96E     8BDC                 mov ebx,esp  

0042D970     FF52 10              call dword ptr ds:[edx+10]  

0042D973     8B56 0C              mov edx,dword ptr ds:[esi+C]  

0042D976     85D2                 test edx,edx  

0042D978     8D4E 0C              lea ecx,dword ptr ds:[esi+C]  

0042D97B     7C 11                jl short x.0042D98E  

0042D97D     3B06                 cmp eax,dword ptr ds:[esi]  

0042D97F     75 0D                jnz short x.0042D98E  

0042D981     8BC6                 mov eax,esi  

0042D983     BA 01000000          mov edx,1  

0042D988     F0:0FC111            lock xadd dword ptr ds:[ecx],edx  

0042D98C     EB 36                jmp short x.0042D9C4  

0042D98E     8B4E 04              mov ecx,dword ptr ds:[esi+4]  

0042D991     8B10                 mov edx,dword ptr ds:[eax]  

0042D993     6A 01                push 1  

0042D995     51                     push ecx  

0042D996     8BC8                 mov ecx,eax  

0042D998     FF12                 call dword ptr ds:[edx]  

0042D99A     85C0                 test eax,eax  

0042D99C     0F84 27020000        je x.0042DBC9  

0042D9A2     8B56 04              mov edx,dword ptr ds:[esi+4]  

0042D9A5     8950 04              mov dword ptr ds:[eax+4],edx  

0042D9A8     8B4E 04              mov ecx,dword ptr ds:[esi+4]  

0042D9AB     41                     inc ecx  

0042D9AC     8BD1                 mov edx,ecx  

0042D9AE     C1E9 02              shr ecx,2  

0042D9B1     83C6 10              add esi,10  

0042D9B4     8D78 10              lea edi,dword ptr ds:[eax+10]  

0042D9B7     F3:A5                rep movs dword ptr es:[edi],dword ptr ds:>  

0042D9B9     8BCA                 mov ecx,edx  

0042D9BB     83E1 03              and ecx,3  

0042D9BE     F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[e>  

0042D9C0     8B7C24 20            mov edi,dword ptr ss:[esp+20]  

0042D9C4     83C0 10              add eax,10  

0042D9C7     8903                 mov dword ptr ds:[ebx],eax  

0042D9C9     8D4424 1C            lea eax,dword ptr ss:[esp+1C]  

0042D9CD     50                     push eax  

0042D9CE     E8 ED58FDFF          call x.004032C0                   ; 算法 call，跟进  

0042D9D3     83C4 08              add esp,8  

0042D9D6     90                     nop  

0042D9D7     90                     nop  

0042D9D8     90                     nop  

0042D9D9     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042D9DD     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042D9E0     85C9                 test ecx,ecx  

0042D9E2     0F8C 85020000        jl x.0042DC6D  

0042D9E8     8038 00              cmp byte ptr ds:[eax],0            ; 判断新字串首字节是否小于 0  

0042D9EB     7E 4F                jle short x.0042DA3C  

0042D9ED     90                     nop  

0042D9EE     90                     nop  

0042D9EF     90                     nop  

0042D9F0     90                     nop  

0042D9F1     90                     nop  

0042D9F2     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042D9F6     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042D9F9     85C9                 test ecx,ecx  

0042D9FB     0F8C 6C020000        jl x.0042DC6D  

0042DA01     0FBE00               movsx eax,byte ptr ds:[eax]        ; 是，取新字符串的首字符运算生成注册码  

0042DA04     99                     cdq  

0042DA05     B9 1A000000          mov ecx,1A  

0042DA0A     F7F9                 idiv ecx  

0042DA0C     8D4424 10            lea eax,dword ptr ss:[esp+10]  

0042DA10     C64424 11 00         mov byte ptr ss:[esp+11],0  

0042DA15     80C2 41              add dl,41  

0042DA18     885424 10            mov byte ptr ss:[esp+10],dl  

0042DA1C     8D50 01              lea edx,dword ptr ds:[eax+1]  

0042DA1F     90                     nop  

0042DA20     8A08                 mov cl,byte ptr ds:[eax]  

0042DA22     40                     inc eax  

0042DA23     84C9                 test cl,cl  

0042DA25     75 F9                jnz short x.0042DA20  

0042DA27     2BC2                 sub eax,edx  

0042DA29     50                     push eax  

0042DA2A     8D5424 14            lea edx,dword ptr ss:[esp+14]  

0042DA2E     52                     push edx  

0042DA2F     8D4C24 20            lea ecx,dword ptr ss:[esp+20]  

0042DA33     E8 6857FDFF          call x.004031A0  

0042DA38     90                     nop  

0042DA39     90                     nop  

0042DA3A     EB 54                jmp short x.0042DA90  

0042DA3C     90                     nop  

0042DA3D     90                     nop  

0042DA3E     90                     nop  

0042DA3F     90                     nop  

0042DA40     8B4424 18            mov eax,dword ptr ss:[esp+18]  

0042DA44     8B48 F4              mov ecx,dword ptr ds:[eax-C]  

0042DA47     85C9                 test ecx,ecx  

0042DA49     0F8C 1E020000        jl x.0042DC6D  

0042DA4F     0FBE00               movsx eax,byte ptr ds:[eax]        ; 不是，取新字符串的首字符运算生成注册码  

0042DA52     99                     cdq  

0042DA53     33C2                 xor eax,edx  

0042DA55     2BC2                 sub eax,edx  

0042DA57     99                     cdq  

0042DA58     B9 0A000000          mov ecx,0A  

0042DA5D     F7F9                 idiv ecx  

0042DA5F     8D4424 14            lea eax,dword ptr ss:[esp+14]  
```



# 汇编代码分析

```
0042DA63     C64424 15 00       mov byte ptr ss:[esp+15],0
0042DA68     80C2 30            add dl,30
0042DA6B     885424 14          mov byte ptr ss:[esp+14],dl

......

0042DAF4     0FBE042E           movsx eax,byte ptr ds:[esi+ebp]
0042DAF8     99                 cdq
0042DAF9     33C2               xor eax,edx
0042DAFB     2BC2               sub eax,edx
0042DAFD     8B5424 20          mov edx,dword ptr ss:[esp+20]
0042DB01     03D0               add edx,eax                         ; 计算用户 ID 所有字符的 ASCII 码和
0042DB03     8B4424 18          mov eax,dword ptr ss:[esp+18]
0042DB07     83C0 F0            add eax,-10
0042DB0A     895424 20          mov dword ptr ss:[esp+20],edx
0042DB0E     C64424 3C 02       mov byte ptr ss:[esp+3C],2
0042DB13     8D48 0C            lea ecx,dword ptr ds:[eax+C]

......

0042DB22     8B08               mov ecx,dword ptr ds:[eax]
0042DB24     8B11               mov edx,dword ptr ds:[ecx]
0042DB26     50                 push eax
0042DB27     FF52 04            call dword ptr ds:[edx+4]
0042DB2A     8B46 F4            mov eax,dword ptr ds:[esi-C]
0042DB2D     45                 inc ebp
0042DB2E     3BE8               cmp ebp,eax
0042DB30   ^ 0F8C 0BFDFFFF      jl x.0042D841                       ; 循环，直到取完全部字符

......

0042DB3B     8B4424 20          mov eax,dword ptr ss:[esp+20]
0042DB3F     99                 cdq
0042DB40     B9 0A000000        mov ecx,0A
0042DB45     F7F9               idiv ecx
0042DB47     8BDA               mov ebx,edx
0042DB49     80C3 30            add bl,30                           ; 生成注册码最后一位
```

================= 跟进 `0042D9CE` call `x.004032C0` =================

```
00403323     56                 push esi
00403324     8D4C24 34          lea ecx,dword ptr ss:[esp+34]
00403328     E8 73FDFFFF        call x.004030A0
0040332D     8B5C24 30          mov ebx,dword ptr ss:[esp+30]
00403331     8D4424 0C          lea eax,dword ptr ss:[esp+C]
00403335     50                 push eax
00403336     56                 push esi
00403337     53                 push ebx
00403338     E8 13FCFFFF        call x.00402F50                     ; 计算与用户 ID 相连接字符串的 MD5 值
0040333D     8B43 F4            mov eax,dword ptr ds:[ebx-C]        ; 结果存储在[1287B0]处
00403340     83C4 0C            add esp,0C
00403343     83F8 FF            cmp eax,-1
00403346     75 11              jnz short x.00403359
00403348     8BC3               mov eax,ebx
0040334A     8D50 01            lea edx,dword ptr ds:[eax+1]
0040334D     8D49 00            lea ecx,dword ptr ds:[ecx]
00403350     8A08               mov cl,byte ptr ds:[eax]
00403352     40                 inc eax
00403353     84C9               test cl,cl
00403355   ^ 75 F9              jnz short x.00403350
00403357     2BC2               sub eax,edx
00403359     85C0               test eax,eax
0040335B     0F8C F3000000      jl x.00403454
00403361     3B43 F8            cmp eax,dword ptr ds:[ebx-8]
00403364     0F8F EA000000      jg x.00403454
0040336A     8943 F4            mov dword ptr ds:[ebx-C],eax
0040336D     C60418 00          mov byte ptr ds:[eax+ebx],0
00403371     57                 push edi
00403372     8B7C24 30          mov edi,dword ptr ss:[esp+30]
00403376     8B07               mov eax,dword ptr ds:[edi]
00403378     8B48 F4            mov ecx,dword ptr ds:[eax-C]
0040337B     8B50 FC            mov edx,dword ptr ds:[eax-4]
0040337E     8B40 F8            mov eax,dword ptr ds:[eax-8]
00403381     8D69 08            lea ebp,dword ptr ds:[ecx+8]
00403384     B9 01000000        mov ecx,1
00403389     2BCA               sub ecx,edx
0040338B     2BC5               sub eax,ebp
0040338D     0BC1               or eax,ecx
0040338F     7D 08              jge short x.00403399
00403391     55                 push ebp
00403392     8BCF               mov ecx,edi
00403394     E8 07FDFFFF        call x.004030A0
00403399     8B17               mov edx,dword ptr ds:[edi]
0040339B     8B42 F4            mov eax,dword ptr ds:[edx-C]
0040339E     03C2               add eax,edx
004033A0     33C9               xor ecx,ecx
004033A2     8908               mov dword ptr ds:[eax],ecx
004033A4     8948 04            mov dword ptr ds:[eax+4],ecx
004033A7     8B07               mov eax,dword ptr ds:[edi]
004033A9     8B40 F4            mov eax,dword ptr ds:[eax-C]
004033AC     8BF0               mov esi,eax
004033AE     83E6 07            and esi,7
004033B1     8BC8               mov ecx,eax
004033B3     74 05              je short x.004033BA
004033B5     2BCE               sub ecx,esi
004033B7     83C1 08            add ecx,8
004033BA     2AC8               sub cl,al
004033BC     75 02              jnz short x.004033C0
004033BE     B1 08              mov cl,8
004033C0     0FBEF1             movsx esi,cl
004033C3     03F0               add esi,eax
004033C5     884C16 FF          mov byte ptr ds:[esi+edx-1],cl
004033C9     8D4C24 10          lea ecx,dword ptr ss:[esp+10]
004033CD     51                 push ecx
004033CE     56                 push esi
004033CF     52                 push edx
004033D0     E8 ABEFFFFF        call x.00402380                     ; 算法 call，跟进
004033D5     83C4 0C            add esp,0C
004033D8     83FD FF            cmp ebp,-1
004033DB     8BC5               mov eax,ebp

......
```

================= 跟进 `004033D0` call `x.00402380` =================

```
00402380     8B4C24 0C          mov ecx,dword ptr ss:[esp+C]
00402384     83EC 68            sub esp,68
00402387     8D0424             lea eax,dword ptr ss:[esp]
0040238A     50                 push eax
0040238B     51                 push ecx
0040238C     E8 2FFBFFFF        call x.00401EC0                     ; 根据 MD5 值生成一组数据表
00402391     8B4424 78          mov eax,dword ptr ss:[esp+78]
00402395     83C4 08            add esp,8
00402398     85C0               test eax,eax
0040239A     76 2B              jbe short x.004023C7
0040239C     56                 push esi
0040239D     8B7424 70          mov esi,dword ptr ss:[esp+70]
004023A1     57                 push edi
004023A2     8D78 FF            lea edi,dword ptr ds:[eax-1]
004023A5     C1EF 03            shr edi,3
004023A8     47                 inc edi
004023A9     8DA424 00000000    lea esp,dword ptr ss:[esp]
004023B0     8D5424 08          lea edx,dword ptr ss:[esp+8]
004023B4     52                 push edx
004023B5     56                 push esi
004023B6     56                 push esi
004023B7     E8 44FDFFFF        call x.00402100                     ; 字符串与数据表进行运算生成新的字符串
004023BC     83C4 0C            add esp,0C
004023BF     83C6 08            add esi,8
004023C2     4F                 dec edi
004023C3   ^ 75 EB              jnz short x.004023B0
004023C5     5F                 pop edi
004023C6     5E                 pop esi
004023C7     83C4 68            add esp,68
004023CA     C3                 retn
```

# 汇编注册机算法部分源码

无技术含量，纯体力劳动-\_-!

```
    .data
szStr1    db  'HyperCalendar 2',0
szStr2    db  'dDFsewrEeRWfdsg#@$SFF^%J&^\*hgj\*&(JIUL_fWWFFGG4534o-=iqew.,nva;gf'
    db  '453242134saf29050327%$&$%&@$#%#!'
    db  '%!#@\_+#$%#@#%#RWEQFSADdsfg$#$DS2'
    db  'AF#$&^%$$##@@#GF><MM<<:fdgdsewf::'
    db  ':@dd#$^DS@#$$%&DSGA\[\]\]\[\]{}reT{{'
    db  'ess###sf|~~@!#\`\`degfsdfVCNM<KJFHDYMGNsr354375uhyfgdsbnhgnfwe9257'
    db  '453242fdljpewrpwtew345767i8u)\*\*^#%'
    db  '!#@\_+#$%#@#%#RWEQSADdsfg$#$DS2'
    db  'fdsg#^DSDF@fhlpoyui90932sag87ds'
    db  'slkl34vo3245dsfa#^#%$#@#@!dg$#%54'
    db  '43535234|~~@!#\`\`degfsdfVCNM<KJFH'
    db  'DYMGNsr354375uhyfgdsbnhgnfwe9257',0

szName    db  512  dup (0)
szRegName db  64   dup (0)
```



```asm
szRegNum   db   64   dup (0)  
szRlt    db   64   dup (0)  
szNum    db   64   dup (0)  
szTmp    db   16   dup (0)  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
    .code  

InitData   proc  

    invoke   lstrcat,addr szName,addr szStr1  
    invoke   lstrcat,addr szName,addr szRegName  
    invoke   lstrcat,addr szName,addr szStr2  
    invoke   lstrlen,addr szName  
    invoke   MD5Encrypt,addr szName,eax,addr szRegNum  
    lea   ecx,szRegNum  
    lea   eax,szName  
    movsx   dx,BYTE ptr \[ecx\]  
    movsx   si,BYTE ptr \[ecx+1\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax\],dx  
    movsx   si,BYTE ptr \[ecx+3\]  
    movsx   dx,BYTE ptr \[ecx+2\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+2\],dx  
    movsx   si,BYTE ptr \[ecx+5\]  
    movsx   dx,BYTE ptr \[ecx+4\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+4\],dx  
    movsx   si,BYTE ptr \[ecx+7\]  
    movsx   dx,BYTE ptr \[ecx+6\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+6\],dx  
    movsx   si,BYTE ptr \[ecx+9\]  
    movsx   dx,BYTE ptr \[ecx+8\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+8\],dx  
    movsx   si,BYTE ptr \[ecx+0Bh\]  
    movsx   dx,BYTE ptr \[ecx+0Ah\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+0Ah\],dx  
    movsx   dx,BYTE ptr \[ecx+0Ch\]  
    movsx   si,BYTE ptr \[ecx+0Dh\]  
    shl   edx,8  
    add   edx,esi  
    mov   WORD ptr \[eax+0Ch\],dx  
    movsx   dx,BYTE ptr \[ecx+0Eh\]  
    movsx   cx,BYTE ptr \[ecx+0Fh\]  
    mov   esi,2Ch  
    shl   edx,8  
    add   edx,ecx  
    mov   WORD ptr \[eax+0Eh\],dx  
    xor   ecx,ecx  

 @@:  
    inc   ecx  
    lea   edi,DWORD ptr \[ecx+1\]  
    and   edi,7  
    movzx   edi,WORD ptr \[eax+edi\*2\]  
    mov   edx,ecx  
    and   edx,7  
    xor   ebx,ebx  
    mov   bx,WORD ptr \[eax+edx\*2\]  
    shr   di,7  
    shl   bx,9  
    or   edi,ebx  
    mov   WORD ptr \[eax+ecx\*2+0Eh\],di  
    and   ecx,8  
    dec   esi  
    lea   eax,\[eax+ecx\*2\]  
    mov   ecx,edx  
    jnz   @b  
    ret  

InitData   endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

Calculate   proc  
    pushad  
    lea   eax,szRlt  
    mov   cx,WORD ptr \[eax\]  
    xor   edx,edx  
    mov   dx,WORD ptr \[eax+2\]  
    add   eax,2  
    movzx   bx,ch  
    add   eax,2  
    mov   DWORD ptr \[szTmp\],edx  
    xor   edx,edx  
    mov   dx,WORD ptr \[eax\]  
    mov   ax,WORD ptr \[eax+2\]  
    mov   bh,cl  
    movzx   cx,BYTE ptr \[szTmp+1\]  
    mov   ch,BYTE ptr \[szTmp\]  
    mov   DWORD ptr \[szTmp+8\],edx  
    movzx   dx,BYTE ptr \[szTmp+9\]  
    mov   dh,BYTE ptr \[szTmp+8\]  
    mov   DWORD ptr \[szTmp\],ecx  
    movzx   cx,ah  
    mov   ch,al  
    mov   DWORD ptr \[szTmp+8\],edx      

    lea   edx,szName  
    mov   DWORD ptr \[szTmp+4\],8     
    mov   ebp,ecx  

 @@:  
    mov   ax,WORD ptr \[edx\]  
    add   edx,2  
    test   ax,ax  
    je   @1  
    test   bx,bx  
    je   @2  
    movzx   ecx,bx  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   ecx,eax  
    shr   ecx,10h  
    cmp   ax,cx  
    sbb   ebx,ebx  
    neg   ebx  
    sub   ebx,ecx  
    add   ebx,eax  
    jmp   @3  

 @2:  
    mov   ebx,1  
    sub   ebx,eax  
    jmp   @3  

 @1:  
    mov   eax,1  
    sub   eax,ebx  
    mov   ebx,eax  

 @3:  
    mov   cx,WORD ptr \[edx\]  
    mov   ax,WORD ptr \[edx+2\]  
    add   WORD ptr \[szTmp+8\],ax  
    add   WORD ptr \[szTmp\],cx  
    add   edx,2  
    mov   ax,WORD ptr \[edx+2\]  
    add   edx,4  
    test   ax,ax  
    je   @4  
    test   bp,bp  
    je   @5  
    movzx   ecx,bp  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   ecx,eax  
    shr   ecx,10h  
    cmp   ax,cx  
    sbb   ebp,ebp  
    neg   ebp  
    sub   ebp,ecx  
    add   ebp,eax  
    jmp   @6  

 @5:  
    mov   ebp,1  
    sub   ebp,eax  
    jmp   @6  

 @4:  
    mov   eax,1  
    sub   eax,ebp  
    mov   ebp,eax  

 @6:  
    mov   esi,DWORD ptr \[szTmp+8\]  
    mov   cx,WORD ptr \[edx\]  
    mov   DWORD ptr \[szTmp+0ch\],esi  
    xor   esi,ebx  
    add   edx,2  
    test   cx,cx  
    mov   DWORD ptr \[szTmp+8\],esi  
    je   @7  
    mov   ax,si  
    test   ax,ax  
    je   @8  
    movzx   eax,ax  
    movzx   ecx,cx  
    imul   ecx,eax  
    mov   esi,ecx  
    shr   esi,10h  
    cmp   cx,si  
    sbb   eax,eax  
    neg   eax  
    sub   eax,esi  
    add   eax,ecx  
    jmp   @9  

 @8:  
    mov   eax,1  
    sub   eax,ecx  
    jmp   @9  

 @7:  
    mov   eax,1  
    sub   eax,esi  

 @9:  
    mov   ecx,DWORD ptr \[szTmp\]  
    mov   di,WORD ptr \[edx\]  
    mov   esi,ebp  
    xor   esi,ecx  
    add   esi,eax  
    add   edx,2  
    test   di,di  
    mov   DWORD ptr \[szTmp+8\],ecx  
    mov   DWORD ptr \[szTmp\],esi  
    je   @10  
    mov   cx,si  
    test   cx,cx  
    je   @11  
    movzx   ecx,cx  
    movzx   esi,di  
    imul   esi,ecx  
    mov   edi,esi  
    shr   edi,10h  
    cmp   si,di  
    sbb   ecx,ecx  
    neg   ecx  
    sub   ecx,edi  
    add   ecx,esi  
    jmp   @12  

 @11:  
    mov   ecx,1  
    sub   ecx,edi  
    jmp   @12  

 @10:  
    mov   ecx,1  
    sub   ecx,esi  

 @12:  
    mov   esi,DWORD ptr \[szTmp+0ch\]  
    add   eax,ecx  
    xor   ebx,ecx  
    xor   ecx,esi  
    xor   ebp,eax  
    mov   DWORD ptr \[szTmp\],ecx  
    xor   eax,DWORD ptr \[szTmp+8\]  
    mov   DWORD ptr \[szTmp+8\],eax  
    dec   DWORD ptr \[szTmp+4\]  
    jnz   @b  
    mov   ax,WORD ptr \[edx\]  
    add   edx,2  
    test   ax,ax  
    je   @13  
    test   bx,bx  
    je   @14  
    movzx   ecx,bx  
    movzx   eax,ax  
    imul   eax,ecx  
    mov   esi,eax  
    shr   esi,10h  
    cmp   ax,si  
    sbb   ecx,ecx  
    neg   ecx  
    sub   ecx,esi  
    add   ecx,eax  
    jmp   @15  

 @14:  
    mov   ecx,1  
    sub   ecx,eax  
    jmp   @15  

 @13:  
    mov   ecx,1  
    sub   ecx,ebx  

 @15:  
    mov   ax,WORD ptr \[edx\]  
    add   WORD ptr \[szTmp+8\],ax  
    mov   ax,WORD ptr \[edx+2\]  
    add   WORD ptr \[szTmp\],ax  
    mov   dx,WORD ptr \[edx+4\]  
    test   dx,dx  
    je   @16  
    test   bp,bp  
    je   @17  
    movzx   eax,bp  
    movzx   edx,dx  
    imul   edx,eax  
    mov   esi,edx  
    shr   esi,10h  
    cmp   dx,si  
    sbb   eax,eax  
    neg   eax  
    sub   eax,esi  
    add   eax,edx  
    jmp   @18  

 @17:  
    mov   eax,1  
    sub   eax,edx  
    jmp   @18  

 @16:  
    mov   eax,1  
    sub   eax,ebp  

 @18:  
    movzx   dx,ch  
    mov   dh,cl  
    lea   ecx,szRlt  
    mov   WORD ptr \[ecx\],dx  
    movzx   dx,BYTE ptr \[szTmp+9\]  
    mov   dh,BYTE ptr \[szTmp+8\]  
    mov   WORD ptr \[ecx+2\],dx  
    movzx   dx,BYTE ptr \[szTmp+1\]  
    mov   dh,BYTE ptr \[szTmp\]  
    mov   WORD ptr \[ecx+4\],dx  
    mov   dh,al  
    mov   WORD ptr \[ecx+6\],dx  
    popad  
    ret  

Calculate   endp  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

GetRegKey   proc   hDlg:DWORD  

 local   con:DWORD,sum:DWORD  

 pushad  

 invoke   GetDlgItemText,hDlg,IDC\_NAME,addr szRegName,sizeof szRegName  

 .if   eax  
    invoke   RtlZeroMemory,addr szName,sizeof szName  
    invoke   RtlZeroMemory,addr szNum,sizeof szNum  
    call   InitData  
    invoke   lstrlen,addr szRegName  
    mov   con,eax  
    xor   esi,esi  
    mov   sum,esi  

 @@:  
    invoke   RtlZeroMemory,addr szRlt,sizeof szRlt  
    movsx   eax,BYTE ptr \[szRegName+esi\]  
    add   sum,eax  
    add   eax,esi  
    cdq  
    mov   ecx,1ah  
    idiv   ecx  
    add   dl,41h  
    mov   BYTE ptr \[szRlt\],dl  
    mov   dl,BYTE ptr \[szRegName+esi-1\]  
    mov   BYTE ptr \[szNum+esi-1\],dl  
    invoke   lstrcat,addr szRlt,addr szNum  
    invoke   lstrlen,addr szRlt  
    mov   ebx,eax  
    mov   edi,eax  
    and   edi,7  
    mov   ecx,eax  
    je   @5  
    sub   ecx,edi  
```



```asm
    add   ecx,8  

  @5:  sub   cl,al  

    jnz   @6  

    mov   cl,8  

  @6:  

    movsx edi,cl  

    add   edi,eax  

    mov   BYTE ptr [szRlt+edi-1],cl  

    mov   eax,esi  

    cdq  

    mov   ecx,3  

    idiv  ecx  

    test  edx,edx  

    jnz   @1  

    xor   edx,edx  

  @2:  

    mov   al,BYTE ptr [szRlt+edx]  

    xor   al,9  

    rol   al,1  

    mov   BYTE ptr [szRlt+edx],al  

    inc   edx  

    dec   ebx  

    jnz   @2  

  @1:  

    call  Calculate  

    cmp   BYTE ptr [szRlt],0  

    jle   @3  

    movsx eax,BYTE ptr [szRlt]  

    cdq  

    mov   ecx,1ah  

    idiv  ecx  

    add   dl,41h  

    mov   BYTE ptr [szRegNum+esi],dl  

    jmp   @4  

  @3:  

    movsx eax,BYTE ptr [szRlt]  

    cdq  

    xor   eax,edx  

    sub   eax,edx  

    cdq  

    mov   ecx,0ah  

    idiv  ecx  

    add   dl,30h  

    mov   BYTE ptr [szRegNum+esi],dl  

  @4:  

    inc   esi  

    dec   con  

    jnz   @b  

    mov   eax,sum  

    cdq  

    mov   ecx,0ah  

    idiv  ecx  

    add   dl,30h  

    mov   BYTE ptr [szRegNum+esi],dl  

    invoke SetDlgItemText,hDlg,IDC_REG,addr szRegNum  

  .else  

    invoke SetDlgItemText,hDlg,IDC_REG,CTXT("请输入用户 ID！")  

  .endif  

  popad  

  ret  

GetRegKey endp  
```

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
