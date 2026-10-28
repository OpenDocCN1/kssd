
# 利用特洛依动态库注入的学习总结

- 【文章标题】: 利用特洛依动态库注入的学习总结
- 【文章作者】: rockhard
- 【作者邮箱】: wnh1[at]sohu.com
- 【软件名称】: 某象棋软件
- 【下载地址】: d31K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4U0K9r3g2K6M7#2)9J5k6h3q4S2i4K6u0W2N6r3!0H3P5X3A6Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4N6r3S2J5k6h3q4V1i4K6u0W2M7r3S2H3i4K6y4r3N6r3W2V1i4K6y4p5x3e0M7%4x3e0p5#2i4K6t1$3k6Y4m8S2k6$3g2Q4x3@1b7I4
- 【加壳方式】: ASPR
- 【保护方式】: 注册码保护
- 【编写语言】: VC
- 【操作平台】: XP+SP2
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 【详细过程】

首先说明的是这个动态库是看雪论坛中一位大牛作品,不是我的.在此表示致敬!让我们也品尝下野猪的味道 :)

可以从163K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4U0K9r3g2K6M7#2)9J5k6h3q4S2i4K6u0W2N6r3!0H3P5X3A6Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4N6r3S2J5k6h3q4V1i4K6u0W2M7r3S2H3i4K6y4r3N6r3W2V1i4K6y4p5x3e0M7%4x3e0p5#2i4K6t1$3k6Y4m8S2k6$3g2Q4x3@1b7I4i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1#2i4@1q4q4i4K6R3&6i4@1f1^5i4@1p5K6i4K6R3#2i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1#2i4@1u0m8i4K6S2r3i4@1f1#2i4K6V1J5i4K6S2o6i4@1f1%4i4@1p5H3i4@1t1@1i4@1f1^5i4@1p5%4i4@1p5K6i4@1f1%4i4K6V1@1i4@1p5^5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6S2m8i4@1p5^5i4@1f1$3i4K6R3H3i4K6R3I4i4@1f1#2i4@1u0m8i4K6V1K6i4K6t1^5i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1^5i4K6R3K6i4@1u0p5i4@1f1^5i4@1p5$3i4K6R3I4i4@1f1$3i4@1t1K6i4@1p5^5i4@1f1#2i4K6R3$3i4K6S2o6i4@1f1^5i4@1q4q4i4@1u0m8i4@1f1#2i4K6W2p5i4K6W2n7i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1$3i4K6R3&6i4K6S2p5i4@1f1^5i4K6R3K6i4@1u0p5i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4K6t1&6i4K6u0W2

软件用ASPR加壳了.破解手法让人赞叹.将`ws2_32.dll`拷贝到程序的安装目录就完成了破解工作.不用脱壳,不用打补丁.下面是个人的分析与总结.

### 一.软件注册算法分析.

这个版本的注册算法不复杂,与先前的2.3版本一样.仅仅用到2048的RSA,曾修改过RSA的`N`,`D`,`E`写过其注册机.

关键的验证算法在`40AA10`处.前面有点小检测,就是判断有没有`key.dat`文件,文件大小是不是介于`0xc8`-`0x12c`之间(`0x40b0fe`处代码)符合条件再进行注册算法的验证，此处略去.

下面这段代码就是取硬盘物理系统号并生成32位ASICC值.也就是后面提到的程序生成的硬件信息.由于硬盘物理系列号的唯一性,这硬件信息每个电脑也就唯一.

```
0040AA1F    E8 BC040100     call    0041AEE0
0040AA24    33FF            xor     edi, edi
0040AA26    8D4424 40       lea     eax, [esp+40]
0040AA2A    57              push    edi
0040AA2B    50              push    eax
0040AA2C    E8 6F360200     call    0042E0A0
0040AA31    68 00040000     push    400
0040AA36    8D4C24 4C       lea     ecx, [esp+4C]
0040AA3A    68 F0605A00     push    005A60F0                         ; ASCII "           Z5FM1251TTOSHIBA MK4026GAX"
0040AA3F    51              push    ecx
0040AA40    E8 AB360200     call    0042E0F0
0040AA45    8D5424 54       lea     edx, [esp+54]
0040AA49    52              push    edx
0040AA4A    E8 A1370200     call    0042E1F0
0040AA4F    33C0            xor     eax, eax
0040AA51    894424 34       mov     [esp+34], eax
0040AA55    894424 38       mov     [esp+38], eax
0040AA59    894424 3C       mov     [esp+3C], eax
0040AA5D    894424 40       mov     [esp+40], eax
0040AA61    894424 44       mov     [esp+44], eax
0040AA65    894424 48       mov     [esp+48], eax
0040AA69    894424 4C       mov     [esp+4C], eax
0040AA6D    894424 50       mov     [esp+50], eax
0040AA71    83C4 18         add     esp, 18
0040AA74    884424 3C       mov     [esp+3C], al
0040AA78    33F6            xor     esi, esi
0040AA7A    8D5C24 1C       lea     ebx, [esp+1C]
0040AA7E    8BFF            mov     edi, edi
0040AA80    0FB68C34 980000>movzx   ecx, byte ptr [esp+esi+98]
0040AA88    51              push    ecx
0040AA89    68 D4A65500     push    0055A6D4                         ; ASCII "%02X"
0040AA8E    53              push    ebx
0040AA8F    E8 479A0400     call    004544DB
0040AA94    83C4 0C         add     esp, 0C
0040AA97    46              inc     esi
0040AA98    83C3 02         add     ebx, 2
0040AA9B    83FE 10         cmp     esi, 10
0040AA9E  ^ 7C E0           jl      short 0040AA80
```

往下继续:

调用`mirsys`函数.初始化大整数运算库`miracl`

```
0040AAA0    57              push    edi
0040AAA1    6A 64           push    64
0040AAA3    893D 80D15900   mov     [59D180], edi
0040AAA9    893D 84D15900   mov     [59D184], edi
0040AAAF    E8 3C880300     call    004432F0                         ;mirsys函数
0040AAB4    57              push    edi
```

下面的`call 00443120` ,是调用`mirvar`初始化变量 多处调用此函数，初始化变量

```
0040AAB4    57               push    edi
0040AAB5    C780 34020000 10>mov     dword ptr [eax+234], 10
0040AABF    E8 5C860300      call    00443120                       ;mirvar
0040AAC4    57               push    edi
0040AAC5    8BD8             mov     ebx, eax
0040AAC7    E8 54860300      call    00443120                       ;mirvar
0040AACC    57               push    edi
0040AACD    894424 2C        mov     [esp+2C], eax
0040AAD1    E8 4A860300      call    00443120                       ;mirvar
0040AAD6    57               push    edi
0040AAD7    894424 28        mov     [esp+28], eax
0040AADB    E8 40860300      call    00443120                       ;mirvar
```

RSA的`N`出现在:

```
0040AB22    68 D0A55500      push    0055A5D0       ; ASCII "B80BCBA9EFD...."...
0040AB27    56               push    esi
0040AB28    E8 B3A60300      call    004451E0       ;此处是库函数cinstr将串转为大整数 下面还有此函数调用.相同
```

RSA的`E`出现在:

```
0040AB31    68 C8A55500      push    0055A5C8                         ; ASCII "1C883"
0040AB57    57               push    edi
0040AB58    56               push    esi
0040AB59    51               push    ecx
0040AB5A    53               push    ebx
0040AB5B    E8 00A40300      call    00444F60         ;RSA计算,调用 powmod:(文件key.dat中的值) ^ E % N ,
                                                        ;key.dat内容为0-9,A-F组成的256个字符串
0040AB60    6A 00            push    0
0040AB62    8D9424 BC000000  lea     edx, [esp+BC]
0040AB69    52               push    edx              ;!!!!这个地址要注意，上面计算的结果就通过big\_to\_bytes转化成串了。该地址记为string1。
0040AB6A    57               push    edi
0040AB6B    68 00010000      push    100
0040AB70    E8 EB9D0300      call    00444960         ; big\_to\_bytes
```

这里有必要说一下正确注册时`string1`应该是什么样的一个值。它是由你想要注册的名称（比如无敌棋手）加上下划线`_`再加上上面所提到的硬件信息

如果此处的硬件信息与上面生成的不一样，说明你把别人的注册文件放到自己的电脑上来了，也不能正正确注册。

只所以要提到这地方，跟后面的特洛依DLL有关。如果此时path一个串到该地址，符合上面的条件(任意的用户名+'_' + 当前电脑的硬件信息)，就可以正确注册了。

```
0040AB96    8D4C24 40       lea     ecx, [esp+40]
0040AB9A    51              push    ecx
0040AB9B    8D9424 DC000000 lea     edx, [esp+DC]
0040ABA2    52              push    edx
0040ABA3    66:C74424 48 5F>mov     word ptr [esp+48], 5F  ;5F，下划线 '\_'的ascii值
0040ABAA    E8 77980400     call    00454426           ;从解密的明文中取出用户名（以下划线'\_'做为标记）（这个CALL中调用了RtlGetLastWin32Error,patch工作就是用到这个函数的）

                                                         ;特洛依DLL修改了RtlGetLastWin32Error地址为其一子函数地址,当调用RtlGetLastWin32Error时执行了我们的代码,patch了string1.从而让程序认为正确注册了,后话
0040ABAF    83C4 38         add     esp, 38
0040ABB2    85C0            test    eax, eax
```

下面将分离出来的用户名转存到固定地址`005a69f0`处

```
0040ABC7    2BCE            sub     ecx, esi                                    ; ecx,用户名长度
0040ABC9    8BF0            mov     esi, eax
0040ABCB    8BC1            mov     eax, ecx
0040ABCD    C1E9 02         shr     ecx, 2                                      ;4的倍数，先4个字节4个字节的移，
0040ABD0    BF F0695A00     mov     edi, 005A69F0                               ; ASCII "wwww "
0040ABD5    F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
0040ABD7    8BC8            mov     ecx, eax
0040ABD9    83E1 03         and     ecx, 3                                      ;移4的余数
0040ABDC    F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
0040ABDE    8D4C24 10       lea     ecx, [esp+10]
0040ABE2    51              push    ecx
0040ABE3    6A 00           push    0
0040ABE5    E8 3C980400     call    00454426
0040ABEA    83C4 08         add     esp, 8
0040ABED    85C0            test    eax, eax                                     ;取得硬件信息
0040ABEF    74 35           je      short 0040AC26
```

下面类似保存从注册文件中生成的硬件信息并比较与程序生成的是不是一致。如果从注册文件中通过RSA运算的硬件信息与先前生成的一致，认为注册成功
