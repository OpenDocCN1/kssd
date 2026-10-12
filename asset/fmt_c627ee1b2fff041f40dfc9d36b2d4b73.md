
# 《QQ 2005 贺岁版登录口令加密算法》完结篇

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-743.htm`](https://www.pediy.com/kssd/pediy07/pediy7-743.htm)

QQ 2005 贺岁版登录口令加密算法及其源代码  

Binny（Binny@vip.163.com）  

拿到 QQ 2005 贺岁版后，发现其加密原理并没有新的改变，经过跟踪和分析，编制出暴力破解本地 QQ 密码的程序。

需要说明的是，我的 QQ 标本其版本号为 `12.87.0.8059`（按鼠标右键看 `QQ.exe` 的属性），如果你手头的版本与我的有差异，请理解我的意图后再动手做实验。

QQ 密码在正确登陆后，会将加密的结果保存在用户目录的 `ewh.db` 文件中，加密采用公开的 `MD5` 算法，通过 `N` 次循环以及异或后求反，最终计算出加密的结果，与用户的 `ewh.db` 文件中的密文比较后，发出“输入密码与上次成功登录的密码不一致,$0A 是否到服务器验证?”（这条信息在 `BasicCtrlDll.dll` 的资源中，`$0A` 在 C 的格式化中为回车）。根据这个提示，完成本地 QQ 密码的暴力破解。

在 QQ 系统中，“QD”标志代表 QQ Data，例如，我们可以在文件 `User.db` 或 `ewh.db` 中找到这个以 QD 开头的数据结构。

## 一、 `ewh.db` 原始数据

```
51 44 01 01 03 00 04 03 00 BD AF A8 04 00 00 00
00 2E 06 00 07 03 00 B9 AB B4 10 00 00 00 07 22
AA 96 56 19 A3 9E 82 19 B7 2B BD 2D 34 4A 04 03
00 A9 B5 B2 04 00 00 00 3C A8 93 06
```

其中，红色为 `AST` 循环次数，蓝色为 `EWH` 加密字符串，绿色为 `UIN` QQ 号（`110340156=0x0693A83C`，Intel 体系内存中排列顺序为：`3CA89306`）。

## 二、 `ewh.db` 数据结构

| HEX 偏移 | DEC 偏移 | 数据 | 注释 | 变量标志 |
|---|---|---|---|---|
| `0000` | `1` | `51 44` | `QD`，QQ Data 数据标志 | `Flag` |
| `0002` | `3` | `01 01` | 保留的数据结构 | `Reserve` |
| `0004` | `5` | `03 00` | 总数据段（Data Sections）的个数 | `Sections` |
| `0006` | `7` | `04` | 第一个数据段（简称 1S，下同）的类型，可以从 `0x01` 到 `0x0F`，`04` 代表本数据没经过加密处理。 | `Type1S` |
| `0007` | `8` | `03 00` | 1S 标志的长度。 | `LenFlag1S` |
| `0009` | `10` | `BD AF A8` | 1S 标志（例如 `AST`、`UIN`、`EWH` 等），是经过简单的异或并求反计算处理的，此处是 `AST`，可能是 `Algorithm Shift Times` 或 `Axxx Switch Time`，管他的呢！ | `Flag1S` |
| `000C` | `13` | `04 00 00 00` | 1S 数据的长度 | `LenData1S` |
| `0010` | `17` | `00 2E 06 00\= (404992)` | 1S 数据，这里是进行 `MD5` 转换的次数。这个数据是同计算机的性能有关的，性能越高的计算机，在 QQ 注册成功后产生的这个循环控制变量就越大。 | `Data1S` |
| `0014` | `21` | `07` | 2S 数据的类型，`07` 代表使用 `MD5` 进行加密 | `Type2S` |
| `0015` | `22` | `03 00` | 2S 标志的长度 | `LenFlag2S` |
| `0017` | `24` | `B9 AB B4` | 2S 标志，此处是 `EWH`，代表本数据段是 `EWH` 密码数据，可能是 `Encrypt With Hash` 的缩写 | `Flag2S` |
| `001A` | `27` | `10 00 00 00` | 2S 数据的长度 | `LenData2S` |
| `001E` | `31` | `07 22 AA 9656 19 A3 9E82 19 B7 2BBD 2D 34 4A` | 2S 数据，是经过 `MD5` 加密计算后产生的数据，当然还要经过异或并求反的计算处理，参考下面程序中的 `1000B858` 行代码。 | `Data2S` |
| `002E` | `47` | `04` | 3S 数据的类型 | `Type3S` |
| `002F` | `48` | `03 00` | 3S 标志的长度 | `LenFlag3S` |
| `0031` | `50` | `A9 B5 B2` | 3S 标志，此处是 `UIN`，代表本数据段是 QQ 号码，可能是：`User Identifier Number` 的缩写 | `Flag3S` |
| `0034` | `53` | `04 00 00 00` | 3S 数据的长度 | `LenData3S` |
| `0038` | `57` | `3C A8 93 06` | 3S 数据，`3C A8 93 06 = 110340156` | `Data3S` |

## 三、 加密原理

下面 VB 伪代码的部分符号引自以上第二点《结构说明》中的变量标志，请注意理解：

```vb
Pwd = MD5(Pwd, Len(Pwd)) 

' Pwd 为用户输入的密码，第一轮 MD5 后，Pwd 成为 16 位字节长度的 MD5 串。

XorKey As Long = 0  'XorKey 为用于解密的字节

For k = 1 To Data1S – 1  '因为前面已经做过一轮，所以此处要减一
 Pwd = MD5(Pwd, 16)
Next k

XorKey = XorKey And &HFFFF

XorKey = (LenData2S And &HFF) Xor (LenData2S \ 256)

XorKey = &HFF - XorKey '求反

For k = 1 To 16
 Pwd(k) = Pwd(k) Xor XorKey
Next k

If Pwd <> Data2S Then
 MsgBox "输入密码与上次成功登录的密码不一致," & vbcrlf & "是否到服务器验证?"
End If
```

通过以上的流程，我真的佩服 QQ 的设计者，如此巨大的循环量，加上循环次数的随机性，如果希望产生一个 QQ `MD5` 词典简直不可能。虽然理论上，可以产生一个 `MD5` 字典，但是，这个字典将有 `1.15E+77*16` 个字节之巨，因此，只好根据 `ewh.db` 文件提供的数据暴力破解了，不知是不是有更好的方法呢？

不过我的感觉是，循环次数加多了，应该会产生更多的 `MD5` 碰撞，不见得是个好事。

还有一种破解思路，也许更加直接，将在后面的文章中详细探讨。但是我只有在有时间做完实验后才有资格评述，不在本文章的讨论范围内。

## 四、 破解算法

重复进行数十万次 `MD5` 加密，会消耗计算机很多时间，如果使用传统的 VB 或 VC，对于一个密码的等待时间也是很可观的（例如使用 VB 代码，消耗的时间可能是汇编的 400 倍），因此，我使用汇编语言来编制低层加密解密算法，通过 `MASM32` 编译连接，最后用高级语言调用。通过提供算法动态库的方式，方便其他有兴趣的读者自己增加丰富的功能。例如增加多线程等，这也将在以后的探讨中实现。在此不做深入讨论。

附带的例子为 VB 和 VC 调用汇编语言动态库的例子，VB 代码简单实现了通过密码字典进行单线程破解的功能，读者可以丰富其内容。增加更多的功能。

## 五、 QQ 数据结构分析

下面为动态库 `BasicCtrlDll.dll` 中反汇编的代码以及代码分析，主要用于分析 `EWH.DB` 中数据结构以及 QQ 数据解调算法的问题。另外，这个算法也可以将 `User.db` 中的数据提取出来。深入研究下去，做一个聊天记录查看器之类的软件也非难事。

```asm
1000B71D  mov eax,BasicCtr.100116AC
1000B722  call BasicCtr.1000FDB0
1000B727  sub esp,3C
1000B72A  mov eax,dword ptr ss:[ebp+8]    ; 将数据的开始地址赋给 EAX，实际数据为 Data，EAX=*Data
1000B72D  push ebx
1000B72E  push esi
1000B72F  push edi
1000B730  mov esi,dword ptr ds:[eax]     ; 需要转换的字符串，EAX 指示一个结构，第一个成员为实际的数据指针
1000B732  mov dword ptr ss:[ebp-28],ecx  ; 局部变量[ebp-28]保存全局的标志结构，ECX 为全局参数地址，在调用本函数时跟入
1000B735  mov eax,dword ptr ds:[esi-8]   ; 为 CString 结构中长度的成员，表示总共多少个字节
1000B738  cmp eax,6                      ; 如果长度小于 6，则为无效的数据
1000B73B  jb BasicCtr.1000B9C2           ; 如果比 6 小则跳转退出，说明数据量不够解调的
1000B741  cmp byte ptr ds:[esi],51       ; 是否为 QQ Data 的数据，QD 为 QQ 数据标志
1000B744  jnz BasicCtr.1000B9C2
1000B74A  cmp byte ptr ds:[esi+1],44
1000B74E  jnz BasicCtr.1000B9C2
1000B754  mov di,word ptr ds:[esi+4]     ; 对于 EWH 来说，为第 4+1 个字节，为 0003
1000B758  add esi,4                      ; 指向数据段(Sections)的个数
1000B75B  inc esi
1000B75C  add eax,-6                     ; EAX 去掉 6 个字节，对于 EWH 来说，剩下 36H 个字节
1000B75F  inc esi
1000B760  mov dword ptr ss:[ebp+8],eax   ; 指向第一个数据段
1000B763  call BasicCtr.1000BD36         ; 在内存（ECX+9C）处开辟一个(100H)字节的空间，空间地址返回到 EAX
1000B768  and dword ptr ss:[ebp-20],0    ; 局部变量[ebp-20]清零
1000B76C  movzx eax,di                  ; 转换到 EAX，对于 EWH，di=3。表示有 3 段数据
1000B76F  test eax,eax
1000B771  mov dword ptr ss:[ebp-48],eax  ; 局部变量[ebp-48]保存数据的段数
1000B774  jle BasicCtr.1000B99B
1000B77A  cmp dword ptr ss:[ebp+8],7     ; 如果整个长度小于 7，则剩下的应该是 QQ 号了。第一次进入时=36H
1000B77E  jb BasicCtr.1000B9C2            ; 如果剩余的数据长度小于 7 则退出
1000B784  mov al,byte ptr ds:[esi]       ; 第一次进入时，ESI 指向第 7 个数据即数据段的类型，例如 04
1000B786  mov cx,word ptr ds:[esi+1]     ; CX=后一个数据，及数据段标志的长度，例如 0003
1000B78A  inc esi
1000B78B  mov edx,dword ptr ss:[ebp+8]   ; 剩余的数据长度，如 36H
1000B78E  sub dword ptr ss:[ebp+8],3     ; 去掉 3 个，例如成为 33H=51
1000B792  mov dword ptr ss:[ebp-38],ecx  ; 局部变量[ebp-38]保存数据段中标志长度
1000B795  movzx edi,cx                  ; EDI 为标志长度了
1000B798  inc esi
1000B799  mov byte ptr ss:[ebp-1C],al    ; 局部变量[ebp-1C]保存本段的类型
1000B79C  lea ecx,dword ptr ds:[edi+4]   ; ECX 为取得的标志长度再加上 4，例如=7
1000B79F  inc esi                         ; 第一次时，ESI 指向第一个数据段的标志字段了，例如指向 BDAFA8，第 9 个数据开始
1000B7A0  cmp dword ptr ss:[ebp+8],ecx    ; 如果剩余的数据比“标志长度+4”还少，则没有数据，因此不进行处理
1000B7A3  jb BasicCtr.1000B9C2
1000B7A9  mov ecx,dword ptr ds:[edi+esi]  ; 跳过数据段的标志部分，将数据段的长度赋值给 ECX。例如[edi+esi]=4
1000B7AC  lea ebx,dword ptr ds:[edi+esi]  ; EBX 保存新的指针，指向数据段中数据部分的长度部分，例如，[EBX]=4
1000B7AF  mov dword ptr ss:[ebp-3C],ebx   ; 局部变量[ebp-3C]保存数据段中数据部分的长度指针
1000B7B2  lea ecx,dword ptr ds:[edi+ecx+7]; 从标志（自身长 3，加上 4 个长度位=7）开始，加上数据长度，为完整数据长度。例如 E
1000B7B6  cmp ecx,edx                     ; 如果剩下的数据长度（如 36H）不够，则退出
1000B7B8  mov dword ptr ss:[ebp-34],ecx   ; 局部变量[ebp-34]保存本数据段的总长度
1000B7BB  ja BasicCtr.1000B9C2            ; 如果小于则退出程序
1000B7C1  and dword ptr ss:[ebp-10],0     ; 局部变量[ebp-10]清零
1000B7C5  cmp al,1                        ; al 保存本段的类型，有效的类型是 4 或 7
1000B7C7  je short BasicCtr.1000B7E1
1000B7C9  cmp al,2
1000B7CB  je short BasicCtr.1000B7E1
1000B7CD  cmp al,3
1000B7CF  je short BasicCtr.1000B7E1
1000B7D1  cmp al,4
1000B7D3  je short BasicCtr.1000B7E1
1000B7D5  cmp al,5
1000B7D7  je short BasicCtr.1000B7E1
1000B7D9  cmp al,7
1000B7DB  je short BasicCtr.1000B7E1
1000B7DD  cmp al,6
1000B7DF  jnz short BasicCtr.1000B7FA
1000B7E1  push ecx                          ; 为本段的总长度，包括（段类型+标志长度+标志+数据长度+数据），例如，对于密码段=1AH
1000B7E2  call <MFC42.operator new>
1000B7E7  push dword ptr ss:[ebp-34]        ; n=[ebp-34]，局部变量[ebp-34]保存本数据段的总长度
1000B7EA  lea ecx,dword ptr ds:[esi-3]      ; [esi-3]指向本段的开始（从段类型算起）
1000B7ED  mov dword ptr ss:[ebp-10],eax     ; 局部变量[ebp-10]保存拷贝后的数据
1000B7F0  push ecx                          ; src
1000B7F1  push eax                          ; dest
1000B7F2  call <MSVCRT.memcpy>              ; memcpy，将本段的整段数据拷贝到新的地方，数据指针保存在局部变量[ebp-10]中
1000B7F7  add esp,10
```
