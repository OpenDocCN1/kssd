

# winrar“秒破”尝试(感谢 107 楼小聪分析)

> 原文：[`www.pediy.com/kssd/pediy10/98684.html`](https://www.pediy.com/kssd/pediy10/98684.html)

第一步：用 winhex 将下面数据生成 rar 格式的文件。  
526172211A0700CF907300000D0000000000000056947424965E00600000004900000002E3B1696DEE413D3B1D33310020000000C3EBC6C6B2E2CAD44279CCECD2D76C6F76652E74787400796AD234784B6DD58B0A427929591366006C6F7665002E7478742E2E5B7A2D7B7D2E2E39423843569449C8691BEC768E16663C5F9ED737AE6CDDC6178C0837F6BB88DAA8356B02A700C776FC0F1091C1D16712FC075A011D5B5DEF7E46966E8B878B80DABCDF9683C49165FFB993A77CDE8600A1262200F3D3D5315DF0FC4E2B3ACAA3943F142EC43D7B00400700  
第二步：在 winhex 中取 65h-71h 一段数据 ctrl+c 后粘贴到密码输入框点确定完成秒破。（实例一） 

实例二  
526172211A0700CE997380000D00000000000000395B7A2D7B7D2E2ED417190FDC99688612D2B31773CD93FE082F30D3229C8F77F51B936AD003E65AADF605471FA4ED0E3655748BCC8F5FA5BFDE4651275A2AA306CB7999C579C249C5AF56A0D3744BD7A695586FEF9FB1AB146CA80DED886936DE3AB003EC44215BA8497E4C1A974D45810C8F1277726881548FBAB842BCF9E17B815C116260ADFEB9151DF97C6F93A673629691B767F883950018B7DE7C0D86FFF04A10905E32BD1DB015B709A298DB8D2C42DF23A131F2AD999AA3BB2316F031C5115c179a2f877a41393761683939  
选中 14h-1bh 一段数据 ctrl+c 后粘贴到密码输入框中后面再加上 dch-e3h 处的 ASCII 字符，确定后即秒破了！信不信由你！  
天易 love  2009-9-29 

都测试成功了，楼主厉害呀![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  
第一个密码是：

代码:

```
..\z-{}..9B8C

```

第二个前一段是：代码:

```
9\[z-{}..

```

后一段是：代码:

```
zA97ah99

```

合起来是：代码:

```
9\[z-{}..zA97ah99

```

  
不过不知道有没有通用性，楼主有原理说明吗，具体点的分析

上传的附件
| ![ || 测试 1.rar   || ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 测试 2.rar  
 |

看了楼主给的文件，再学习一下 RAR 文件头结构，算是明白楼主的意思：  
RAR 加密的原理，是将 UNICODE 格式的密码，与随机生成的一个 8 字节的 SALT 连在一起，根据它生成 AES 算法的密钥来进行加密。  
而加密后的 RAR 中，除了加密后的原压缩内容之外，文件头结构还保存了这个 SALT 值用于解密。  
原则上 SALT 的生成过程跟用户输入时的密码一点关系都没有，因此它本来不保存关于密码的显式信息，但是我们可以对 WINRAR 程序进行 patch，使 SALT 跟密码发生直接联系，从而使这个 SALT 成为潜在的“后门”。  
由于 SALT 只有 8 个字节，所以对于 8 个字节以内的密码，可以直接保存在 SALT 中（或者简单的 XOR 一下或其他，反正可以用很容易的可逆的算法处理）。  
那么密码超出 8 字节怎么办？这里楼主分为两种方式处理，分别是实例一和实例二。  
  
实例一应该是利用了把文件头的 HEAD\_SIZE 字段改大，这样文件头多出来的地方（WINRAR 读取文件头时是跳过这部分的）就可以保存密码 8 字节以后的内容。实例一中的 65h-6ch 内容就是 SALT，而 6dh-71h 就属于把 HEAD\_SIZE 改大之后多出来的部分了。  
  
但是实例一这种方法应用的前提是文件头没加密。如果文件头被加密（这时 MAIN\_HEAD 的 HEAD\_FLAGS 包含 0x0080），那么 MAIN\_HEAD 后面紧跟着的就是 SALT（实例二的 14h-1bh 内容），SALT 后面则是被加密的所有内容（包括文件头）。那么怎么把密码 8 字节以后部分搞进去？实例二的做法，我猜是因为加密过程是 16 字节为一组，加密后的内容也应该是 16 字节的倍数，所以解密时也以 16 字节为单位读取，当文件内容最后跟着一段小于 16 字节的数据时，WINRAR 解密时直接无视这段数据。于是就可以把密码的后半部分放在文件最后（但是不能超过 16 字节），这样用这种方法可以保存不超过 23 字节的密码。  
  
按照这个思路，我调试了一下我电脑上的 WINRAR 3.70。  
我只考虑密码不超过 8 字节，实际上问题就是找到 WINRAR 程序操作压缩的过程，在程序把 SALT 和密码连在一起之前，把 SALT 改为和密码一样，这样压缩出来的就是那样的文件了。超过 8 字节的代码只要按照上述思路自己修改生成后的 RAR 就可以了。  
  
具体步骤：  
  
创建一个新 RAR 文件，OD 载入 WINRAR 程序，F9 跑起来，打开这个空 RAR 文件，往里面添加一个文件，这时 WINRAR 程序会弹出添加文件到压缩文件的对话框。第一步要做的是在这个对话框保存我们输入的密码时断下。  
  
怎么定位这个并不难，因为当在添加文件的对话框中高级选项卡点击“设置密码”的时候，会弹出输入密码的框，就从拦截这个对话框入手，程序弹出这个子对话框采用的是 DialogBoxParamA，只要在这个 API 上下断就可以断下：  
  
77D3B144 >  8BFF            mov     edi, edi                         ; WinRAR.004BB5D0  
  
看堆栈：  
  
0012A404   00482A3B  /CALL 到 DialogBoxParamA 来自 WinRAR.00482A36  
0012A408   00400000  |hInst = 00400000  
0012A40C   004AC73C  |pTemplate = "GETPASSWORD2"  
0012A410   00040714  |hOwner = 00040714 ('高级',class='#32770',parent=001B06FA)  
0012A414   0047D78A  |DlgProc = WinRAR.0047D78A  
0012A418   0012A428  \\lParam = 0012A428  
  
进到调用来源 00482A36，看一下调用过程：  
  
00482A1E  |.  8D95 7CFFFFFF lea     edx, dword ptr \[ebp-84\]  
00482A24  |.  52            push    edx                              ; /lParam  
00482A25  |.  68 8AD74700   push    0047D78A                         ; |DlgProc = WinRAR.0047D78A  
00482A2A  |.  53            push    ebx                              ; |hOwner  
00482A2B  |.  68 3CC74A00   push    004AC73C                         ; |pTemplate = "GETPASSWORD2"  
00482A30  |.  FF35 48214B00 push    dword ptr \[4B2148\]               ; |hInst = 00400000  
00482A36  |.  E8 95F10100   call    <jmp.&USER32.DialogBoxParamA>    ; \\DialogBoxParamA  
00482A3B  |.  85C0          test    eax, eax  
00482A3D  |.  0F95C1        setne   cl  
00482A40  |.  83E1 01       and     ecx, 1  
00482A43  |.  8BD9          mov     ebx, ecx  
00482A45  |.  84DB          test    bl, bl  
00482A47  |.  74 15         je      short 00482A5E  
00482A49  |.  8D85 7CFFFFFF lea     eax, dword ptr \[ebp-84\]  
00482A4F  |.  50            push    eax  
00482A50  |.  57            push    edi  
00482A51  |.  E8 2A520100   call    00497C80                         ;  CopyString  
  
由于对话框调用完后紧接着是一个字符串拷贝，因此可以推断传送给密码对话框的这个 lParam 参数就是让密码对话框保存密码的位置，对话框关闭之后又将密码 Copy 到一个全局数据区保存起来（对对话框窗口过程的调试证明这个推测是正确的），而此时的 edi 就是保存密码的全局数据区地址，这个值是 004BB5D0。  
  
在 call DialogBoxParamA 的下一句下断，F9 让密码对话框出现，填写密码（不要超过 8 字节），按确定，断下了，然后 F8 观察 CopyString 的参数证实上面的推测是正确的。  
现在仍然在对话框的窗口过程中：  
  
00446AB4  |.  E8 27BF0300   call    004829E0  
00446AB9  |.  84C0          test    al, al  
00446ABB  |.  74 47         je      short 00446B04  
00446ABD  |.  B8 E6000000   mov     eax, 0E6  
00446AC2  |.  803D D0B54B00>cmp     byte ptr \[4BB5D0\], 0  
00446AC9  |.  75 05         jnz     short 00446AD0  
00446ACB  |.  05 F8020000   add     eax, 2F8  
00446AD0  |>  E8 FBBAFCFF   call    004125D0  
  
显然后续代码证实了 4BB5D0 开始的全局数据区的确是在存取密码的。  
于是下一步在 WINRAR 程序从这里取出密码进行加密操作的时候把它断下。  
  
对 4BB5D1 下硬件访问断点（为什么要下在 4BB5D1 而不直接在 4BB5D0，那是因为如果下在 4BB5D0，将会断到大量的 cmp     byte ptr \[4BB5D0\], 0，而只有开始读取 4BB5D1 处才能说明是真正的在读取其内容而不是单独地比较密码是否为空），F9 跑起来，完成添加文件对话框的设置点确定关闭这个对话框，OD 断下：  
  
00497C53  |.  2BC3          |sub     eax, ebx  
00497C55  |.  75 26         |jnz     short 00497C7D  
00497C57  |.  84DB          |test    bl, bl  
00497C59  |.  74 22         |je      short 00497C7D  
00497C5B  |.  8A41 02       |mov     al, byte ptr \[ecx+2\]  
00497C5E  |.  8A5A 02       |mov     bl, byte ptr \[edx+2\]  
  
这是一个 CompareString，Ctrl+F9  
  
0041139D  |> /FF75 FC       /push    dword ptr \[ebp-4\]  
004113A0  |. |69C3 AA000000 |imul    eax, ebx, 0AA  
004113A6  |. |81C0 303F4A00 |add     eax, 004A3F30                   ;  ASCII ".,\];/\[\]"  
004113AC  |. |50            |push    eax  
004113AD  |. |E8 82680800   |call    00497C34  
004113B2  |. |83C4 08       |add     esp, 8                          ;  到这里  
004113B5  |. |85C0          |test    eax, eax  
004113B7  |. |0F85 8F000000 |jnz     0041144C  
  
在一个循环里，在下面 F4 跳出循环：  
  
00411456  |> \\807D EB 00    cmp     byte ptr \[ebp-15\], 0  
0041145A  |.  0F85 0B020000 jnz     0041166B  
00411460  |.  8D95 60FEFFFF lea     edx, dword ptr \[ebp-1A0\]         ;  保存 UNICODE 形式的密码  
00411466  |.  B9 7F000000   mov     ecx, 7F  
0041146B  |.  8B45 FC       mov     eax, dword ptr \[ebp-4\]           ;  004BB5D0，密码字符串指针  
0041146E  |.  E8 41F0FFFF   call    004104B4                         ;  ANSI 转换成 UNICODE  
00411473  |.  66:C785 5EFFF>mov     word ptr \[ebp-A2\], 0  
0041147C  |.  8D95 58FDFFFF lea     edx, dword ptr \[ebp-2A8\]  
00411482  |.  8D85 60FEFFFF lea     eax, dword ptr \[ebp-1A0\]  
00411488  |.  B9 00000001   mov     ecx, 1000000  
0041148D  |.  E8 42F0FFFF   call    004104D4                         ;  string copy  
00411492  |.  8D85 60FEFFFF lea     eax, dword ptr \[ebp-1A0\]  
00411498  |.  E8 CBF2FFFF   call    00410768                         ;  取 UNICODE 字符串长度  
0041149D  |.  03C0          add     eax, eax  
0041149F  |.  85FF          test    edi, edi  
004114A1  |.  8945 E4       mov     dword ptr \[ebp-1C\], eax  
004114A4  |.  74 19         je      short 004114BF  
004114A6  |.  6A 08         push    8  
004114A8  |.  57            push    edi  
004114A9  |.  8D95 58FDFFFF lea     edx, dword ptr \[ebp-2A8\]  
004114AF  |.  0355 E4       add     edx, dword ptr \[ebp-1C\]  
004114B2  |.  52            push    edx  
004114B3  |.  E8 D8650800   call    00497A90                         ;  memcpy？  
004114B8  |.  83C4 0C       add     esp, 0C  
004114BB  |.  8345 E4 08    add     dword ptr \[ebp-1C\], 8  
  
上面这段代码就很清楚了，就是把密码变为 UNICODE 并把 SALT 加在后面。因此此时的 edi 就是保存 SALT 的指针，数据窗口中观察：  
02A32D6C  EC D4 D1 4B 47 60 0F 6E                          煸 G\`n?.  
  
如果此时不修改这个 SALT 值，让 WINRAR 完成整个过程，再在 RAR 文件中观察文件头结构会发现 SALT 的确是这个内容。  
  
现在在这里直接把这个位置用密码字符串代替（为了学习楼主，我把密码同样搞成几个标点符号）：  
  
02A32D6C  2E 2C 5D 3B 2F 5B 5D 00                          .,\];/\[\]  
  
解除断点让 WINRAR 完成操作。  
  
生成的 RAR 文件内容：  
526172211A0700CF907300000D000000000000003A917424843000200000000800000002FA9EF760CB28553B1D3308002000000073686F772E7478742E2C5D3B2F5B5D007FAA3C3EB323E42592EE85B9EC80F2BAA3389C0C5733224118F8A872B9086EF3C43D7B00400700  
  
WINHEX 打开文件，在文件 3ch-43h 处正是 SALT 值，也正是我设的密码（只有 7 个字节，所以最后是 00）：  
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F  
  
00000030                                        2E 2C 5D 3B               .,\];  
00000040   2F 5B 5D 00                                        /\[\].  
  
至此我成功在 WINRAR3.70 中通过 OD 调试 WINRAR 程序实践了楼主所说的内容，显然只剩下把这个过程转化成 patch 就可以实用化了。