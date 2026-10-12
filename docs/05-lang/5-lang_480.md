
# SoftICE4.05 for win9x

> 原文：[`www.pediy.com/kssd/pediy03/forum999.htm`](https://www.pediy.com/kssd/pediy03/forum999.htm)

## SoftICE4.05 for win9x 的安装序列号破解（1）

**Passion**

破解 `InstallShield` 安装序列号应该说有一套稍微“专门”的一点的方案，准备写出来和大家交流交流。一般对于 5.5 版本以下的 `InstallShield` 可以用专门的`*.INS`文件反编译器来分析其安装流程，这比起分析 `setup.exe` 来说似乎大多数情况下都能起到事半功倍的效果。（例外？当然有，这个就是。^\_^）

对于 `Cracker` 来说，`SoftICE4.05` 的序列号怕不少朋友都记住了。这里拿这个破解工具开刀有好几个原因：第一就是这个要跳过序列号检验很简单，但要寻出可用序列号来便有些难度，第二就是这个 `INSTALLSHIELD` 比较典型全面，既有普通的安装过程，又有大量的外部程序调用，能读懂它再读其他的应该也就没什么问题，第三便是由于皮大客的点将了。废话少说，先用 `ISDCC2` 反编译 `setup.ins` 再说：

```
isdcc2 setup.ins > sice405.txt
```

反编译出来的文档还不小，其中含有大量的外部 DLL 调用，如检测鼠标设置显卡等，要找出序列号判断的地方还真不容易。

### 寻找序列号判断逻辑

打开 `sice405.txt`，第一条路是查出错信息“The serial …… invalid ……”（不好意思，忘了，反正只查一两个单词），结果呢？没有！（事实上出错提示是在嵌入的外部动态链接库 `nminst32.dll` 中。）

第二条路，查字符串“MESSAGEBOX”，找倒是能找到一大片，可惜大多数明文的出错信息都不是我们要找的，有两三个类似于 `MessageBox(string2, -65534);`等字样的，也弄不拎清这个 `string2` 到底在哪儿被赋值了，想来想去，也不容易成。

第三条路，从头细细看，看哪儿有突破口。这号反编译出来的东西，前边一点点是主程序，后边一大部分是各种各样的 `function` 实现部分，因此先看看程序体：（虽然相对来讲不多，可贴出来不少哇！）

```
// ------------- MAIN PROGRAM CODE --------------

program

start:

00136E:0002:     Disable(12);
001375:0021:     number60 = 0;
00137F:0021:     number56 = 0;
001389:0021:     number59 = 0;
001393:0021:     number51 = 1;
00139D:0021:     number70 = 0;
0013A7:0021:     number49 = 0;
0013B1:00B5:     function109();
0013B9:00B5:     function107();
0013C1:00B5:     function108();                  //估计是检验操作系统。
0013C9:0022:     if (number63 = 0) then
                    goto label2;
                endif;
      ……
//省略 NT 部分的一些处理过程。
//这里下面要装载一些 DLL 来进行安装，如果装载失败则出错。

label1:
001690:0125:     string22 = SUPPORTDIR ^ "NMINST32.dll";
0016A7:00B2:     UseDLL(string22);
0016AC:0021:     number71 = LAST_RESULT;
0016B4:0128:     number71 = number71 != 0;
0016C6:0022:     if (number71 = 0) then
                    goto label3;
                endif;
0016D4:002A:     MessageBox("Setup is unable to load the support DLLs needed to perform this SoftICE installation.", -65533);
001733:002B:     exit;

label2:
001739:0125:     string23 = SUPPORTDIR ^ "SINSETUP.dll";
001750:00B2:     UseDLL(string23);
      ……………… //省略部分代码
001826:002A:     MessageBox("Setup is unable to load the support DLLs needed to perform this SoftICE installation.", -65533);
001885:002B:     exit;
//以上是一些 DLL 的装载过程。下面正式开始安装过程。
//由于安装程序运行时，顺序是封面、协议、序列号，因此序列号检验怎么也不会太靠后。

label4:
00188B:0013:     string30 = "";
001893:0013:     string31 = "";
00189B:0021:     number64 = 0;
0018A5:0021:     number65 = 0;
0018AF:0021:     number66 = 0;
0018B9:0021:     number61 = 0;
0018C3:0021:     number68 = 4;
0018CD:0021:     number69 = 0;
0018D7:0021:     number67 = 4;
0018E1:00B5:     function140();                    //注意这个 Function
0018E9:0021:     number71 = LAST_RESULT;
0018F1:0128:     number71 = number71 = 1;
001903:0022:     if (number71 = 0) then
                    goto label6;
                endif;
001911:00B5:     function141();

label5:
00191D:0022:     if (0 = 0) then
                    goto label8;
                endif;
      ………… //省略部分代码

label7:
001990:0128:     number71 = number62 = 1;
0019A2:0022:     if (number71 = 0) then
                    goto label9;
                endif;
0019B0:0023:     StrCompare(string13, "");
0019B8:0128:     number71 = LAST_RESULT = 0;
0019CA:0022:     if (number71 = 0) then
                    goto label9;
                endif;
0019D8:002A:     MessageBox("Unable to verify Serial Number. \n This program will now exit!", -65533);
001A1F:0159:     abort;
//这里大概是要装入 DLL 文件来检验序列号。
//这下面的大概就没用了，省之。

label8:
001A25:00B5:     function99();
001A2D:0021:     number71 = LAST_RESULT;
001A35:0128:     number71 = number71 < 0;
001A47:0022:     if (number71 = 0) then
                    goto label10;
                endif;
001A55:002C:     goto label21;
      ………… //省略其他大部分安装代码。
001D43:00B3:     UnUseDLL(string22);
001D48:00B3:     UnUseDLL(string24);
001D4D:00B3:     UnUseDLL(string23);
001D52:0159:     abort;

label22:
001D58:002B:     exit;

endprogram
```

需要看看那个 `function140();`的调用，下面是 `function140()`的代码：

它完成的功能是检验注册表里是否有 `SICE` 的注册信息，如果有则读出来检验检验是否正确，如果正确则填到序列号输入框中免得再辛苦，这种体贴用户的做法可真不忍心来破哇！——但既然输入界面的检验过程难找，有这个检验过程也不错嘛！

```
// ------------- FUNCTION function140 --------------

function function140()
    number lNumber0;
    number lNumber1;
    number lNumber2;
    number lNumber3;
    number lNumber4;
    number lNumber5;
    number lNumber6;
    number lNumber7;
    string lString0;
    string lString1;
    string lString2;
    string lString3;
    string lString4;

begin
label158:

005CB5:0110:        RegDBSetDefaultRoot(-2147483646);            //就是 H_L_M
005CBC:0088:        RegDBKeyExist("\\Software\\NuMega\\SoftICE");
005CD9:0021:        lNumber6 = LAST_RESULT;
005CE1:0128:        lNumber6 = lNumber6 = 1;
005CF3:0022:        if (lNumber6 = 0) then
                        goto label160;
                    endif;
005D01:0013:        lString2 = "\\Software\\NuMega\\SoftICE";
005D21:002C:        goto label161;
005D2A:0013:        lString2 = "\\Software\\Nu-Mega\\SoftICE";

label160:
005D4F:0152:        RegDBGetKeyValueEx(lString2, "InstallDir", lNumber1, lString0, lNumber0);
005D6A:0021:        number50 = LAST_RESULT;
005D72:0128:        lNumber6 = number50 = 0;
005D84:0022:        if (lNumber6 = 0) then
                        goto label162;
                    endif;
005D92:0013:        string10 = lString0;

label161:
005D9E:0152:        RegDBGetKeyValueEx(lString2, "User", lNumber1, lString0, lNumber0);
005DB3:0021:        number50 = LAST_RESULT;
005DBB:0128:        lNumber6 = number50 = 0;
005DCD:0022:        if (lNumber6 = 0) then
                        goto label163;
                    endif;
005DDB:0013:        string11 = lString0;

label162:
005DE7:0152:        RegDBGetKeyValueEx(lString2, "Company", lNumber1, lString0, lNumber0);
005DFF:0021:        number50 = LAST_RESULT;
005E07:0128:        lNumber6 = number50 = 0;
005E19:0022:        if (lNumber6 = 0) then
                        goto label164;
                    endif;
005E27:0013:        string12 = lString0;

label163:
005E33:0128:        lNumber6 = number62 = 1;
005E45:0022:        if (lNumber6 = 0) then
                        goto label166;
                    endif;
005E53:0021:        lNumber5 = 0;
005E5D:0013:        TARGETDIR = string10;
005E65:0125:        lString4 = TARGETDIR ^ "\\nmtrans.dll";
005E7C:007C:        GetFileInfo(lString4, 4, lNumber3, lString3);
005E8C:0125:        lString4 = TARGETDIR ^ "\\loader32.exe";
005EA4:007C:        GetFileInfo(lString4, 4, lNumber4, lString3);
005EB4:0128:        lNumber6 = lNumber3 = 0x8bc00;
005EC6:0128:        lNumber7 = lNumber4 = 0x1c4fec;
005ED8:0127:        lNumber6 = lNumber6 && lNumber7;
005EE3:0022:        if (lNumber6 = 0) then
                        goto label165;
                    endif;
005EF1:0021:        lNumber5 = 1;

label164:
005EFF:0128:        lNumber6 = lNumber5 = 0;
005F11:0022:        if (lNumber6 = 0) then
                        goto label166;
                    endif;
005F1F:002A:        MessageBox("Setup could not locate the correct version of \n SoftIce to update and thus will exit!", -65533);
005F7E:0159:        abort;

//上面从注册表里读安装路径、用户名、公司信息、版本号等。

label165:
005F84:0152:        RegDBGetKeyValueEx(lString2, "Serial", lNumber1, lString0, lNumber0);
```



005F9B:0021:       `number50 = LAST_RESULT;`   

005FA3:00B5:       `function143(lString0);`    //读出序列号后的这个 Function 重要！    

005FAE:0021:       `lNumber6 = LAST_RESULT;`   

005FB6:0128:       `lNumber6 = lNumber6 = 1;`   

005FC8:0022:       `if (lNumber6 = 0) then`   

                      `goto label167;`   

                  `endif;`   

005FD6:0013:       `string13 = lString0;`   

005FDE:012F:       `return(1);`   

005FE7:002C:       `goto label168;`   

`label166:`   

005FF0:012F:       `return(0);`   

`label167:`   

005FFD:012F:       `return(0);`   

006006:00B8:       `return;`   

一层层跟下去，看 `function143` 的代码，很好，不太长：   

```
// ------------- FUNCTION function143 --------------

function function143(pString0)
    number lNumber0;
    number lNumber1;
    string lString0;
    string lString1;
    string lString2;
    string lString3;
    string lString4;
    string lString5;
    string lString6;
begin
```

`sice` 的序列号形如 `xxxx-xxxxxx-xx`，保存在注册表里的也是这个格式。

`label190:`

0067BE:002F:       `StrLength(pString0);`
0067C3:0021:       `lNumber1 = LAST_RESULT;`
0067CB:0128:       `lNumber1 = lNumber1 != 14;`
0067DD:0022:       `if (lNumber1 = 0) then`
                      `goto label192;`
                  `endif;`
0067EB:012F:       `return(0);`

//这里我怀疑有编译错误，应该是如果序列号长 14 则序次执行到这里，否则返回 0。

0067F8:0030:       `StrSub(lString1, pString0, 0, 4);`
00680A:0030:       `StrSub(lString2, pString0, 5, 6);`
00681C:0030:       `StrSub(lString3, pString0, 12, 2);`

//从注册表里的序列号中取除“-”号外的子串。

00682E:0124:       `lString6 = lString1 + lString2;`
006839:0124:       `lString0 = lString6 + lString3;`

//再拼起来，也就是相当于序列号去掉“-”号。

//下面仍旧怀疑有误。

006844:0031:       `StrFind(lString0, "-");`
00684D:0128:       `lNumber1 = LAST_RESULT >= 0;`
00685F:0022:       `if (lNumber1 = 0) then`
                      `goto label193;`
                  `endif;`
00686D:012F:       `return(0);`

`label192:`

00687A:0125:       `lString5 = SUPPORTDIR ^ "UTILITY.dll";`
006890:00B2:       `UseDLL(lString5);`

//装载 `utility.dll` 来准备进行进一步的序列号检验。

006895:0021:       `lNumber1 = LAST_RESULT;`
00689D:0128:       `lNumber1 = lNumber1 = 0;`
0068AF:0022:       `if (lNumber1 = 0) then`
                      `goto label194;`
                  `endif;`
0068BD:00B4:       `UTILITY.DigitCheck(lString0);`    //这一步是关键检验过程。
0068C5:0021:       `lNumber0 = LAST_RESULT;`
0068CD:00B3:       `UnUseDLL(lString5);`

`label193:`

0068D6:0128:       `lNumber1 = lNumber0 = 1;`
0068E8:0022:       `if (lNumber1 = 0) then`
                      `goto label195;`
                  `endif;`
0068F6:012F:       `return(1);`    //序列号正确则返回 1。
0068FF:002C:       `goto label196;`

`label194:`

006908:012F:       `return(0);`

`label195:`

006915:00B8:       `return;`
```
    end;
```

这段说明，`SOFTICE` 安装时会读取注册表中的序列号信息（如果有的话）并且进行判断，这一段的机制与效果和检验输入的序列号是否正确是一样的。

现在就 `UTILITY.DigitCheck(lString0);` 是关键了，参数是去掉了“-”号的序列号。

`SOFTICE` 安装程序运行时会在 `windows/temp` 目录下生成一些临时的目录和文件，其中就包括刚才提到的 `utility.dll`，下面是其 `DigitCheck` 的输出代码。

跟踪过程也不算太复杂，先填序列号，我填的是 `5100-0099BB-BB`，下断 `bpx loadlibrarya do "d esp->8"` 看载入的 `DLL` 文件名，等按“Next”按钮的时候便会装入 `UTILITY.DLL`，因此断在系统领空，返回后看看 `MOD UTILITY`，我机子上的 `BASE` 是 `1BC0000`，因此下面的地址都有一步重定位的变换，为了简明起见，我使用 `1000000` 做基址，这与 `W32DASM` 的反汇编代码相符合。

`digitcheck` 的入口是 `1110`，因此下断 `bpx 0167:1bc1110`，断后便开始了漫长的跟踪过程。（见下篇）

### `SoftICE4.05 for win9x` 的安装序列号破解（2）

`Passion`（接上半部分）

跟踪过程：

`Exported fn(): DigitCheck - Ord:0001h`

```
:10001110 56                    push esi
:10001111 57                    push edi
:10001112 8B7C240C              mov edi, dword ptr [esp+0C]     //当然是序列号的地址了。
:10001116 83C9FF                or ecx, FFFFFFFF
:10001119 33C0                  xor eax, eax
:1000111B F2                    repnz
:1000111C AE                    scasb      //先求长度。
:1000111D F7D1                  not ecx
:1000111F 2BF9                  sub edi, ecx
:10001121 8BC1                  mov eax, ecx
:10001123 8BF7                  mov esi, edi
:10001125 BF80910010            mov edi, 10009180
:1000112A C1E902                shr ecx, 02
:1000112D F3                    repz
:1000112E A5                    movsd      //再搬地方。
:1000112F 8BC8                  mov ecx, eax
:10001131 83E103                and ecx, 00000003
:10001134 F3                    repz
:10001135 A4                    movsb      //怕没搬完，再搬零头。
:10001136 E895000000            call 100011D0
:1000113B E840010000            call 10001280
```

//这两个 `CALL` 会根据序列号的前 8 位数字经过查表运算得来一系列值，也就内定了前八个字符必须是数字。详细内容见后面。

计算出来的八个值：`04 0C 09 03 0D 07 06 08`，放在 `100091B0`

```
:10001140 33C9                  xor ecx, ecx
:10001142 5F                    pop edi
:10001143 890DAC910010          mov dword ptr [100091AC], ecx
:10001149 5E                    pop esi
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
*:1000118E(C)*

```
:1000114A 8A81B4910010          mov al, byte ptr [ecx+100091B4]   //取第一个
:10001150 8A91B0910010          mov dl, byte ptr [ecx+100091B0]   //取第四个
:10001156 32C2                  xor al, dl     //先异或
```

//八个值中，第一个值和第四个值异或，再和最后一个值或，结果如不是数字或者字母则加 7 变成字母，比较后通过则继续计算第二个和第五个，并且仍和最后一个或，依此类推。

```
:10001158 8A15B7910010          mov dl, byte ptr [100091B7]    //取最后一个
:1000115E 0AC2                  or al, dl      //再或
:10001160 0C30                  or al, 30
:10001162 3C39                  cmp al, 39
:10001164 8881B8910010          mov byte ptr [ecx+100091B8], al  //存结果
:1000116A 7E08                  jle 10001174
:1000116C 0407                  add al, 07     //变字母
:1000116E 8881B8910010          mov byte ptr [ecx+100091B8], al  //也是存结果
```

//这里的 `AL` 是算出来的一个字符，算出一个后马上跟输入的序列号的最后四位比较。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
*:1000116A(C)*

```
:10001174 8A81B8910010          mov al, byte ptr [ecx+100091B8]
:1000117A 8A9188910010          mov dl, byte ptr [ecx+10009188]   
```

//这是输入序列号的后四位地址。

```
:10001180 3AC2                  cmp al, dl
```

//前八位数字计算出来的四个字符必须和序列号末尾的四个字母一样才行。

```
:10001182 7406                  je 1000118A   
```

//比较一个字符，这里得修改程序流程，让它继续跳回去计算正确的后几个字母。

```
:10001184 0C20                  or al, 20
:10001186 3AC2                  cmp al, dl
:10001188 7512                  jne 1000119C
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
*:10001182(C)*

```
:1000118A 41                    inc ecx
:1000118B 83F904                cmp ecx, 00000004   //比较四次也就是四个字符。
:1000118E 7CBA                  jl 1000114A
:10001190 890DAC910010          mov dword ptr [100091AC], ecx
:10001196 B801000000            mov eax, 00000001   //比较通过就返回 1。
:1000119B C3                    ret
```

程序算出来的四个字符是“`9BFB`”

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
*:10001188(C)*

```
:1000119C 890DAC910010          mov dword ptr [100091AC], ecx
:100011A2 33C0                  xor eax, eax   //否则返回 0。
:100011A4 C3                    ret
```

这里是第一个 `CALL` 的内容，它的功能也就是先根据输入的前 8 个字符查一张表，如果查出来的东西符合要求（也就是前 8 个字符都是数字）则把前 8 个字符的 `ASCII` 码都转换成实际的数字存放在 `100091B0` 处：

* Referenced by a `CALL` at Address:
*:10001136*

```
:100011D0 33D2                  xor edx, edx
:100011D2 56                    push esi
:100011D3 8915A8910010          mov dword ptr [100091A8], edx
:100011D9 33C9                  xor ecx, ecx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
*:1000123D(U)*

```
:100011DB A16C830010            mov eax, dword ptr [1000836C]
```



:100011E0 890DAC910010                     mov dword ptr [100091AC], ecx

:100011E6 83F801                         cmp eax, 00000001

:100011E9 7E20                           jle 1000120B

//也不知是什么判断，反正总是跳。

:100011EB 0FBE8180910010                 movsx eax, byte ptr [ecx+10009180]

:100011F2 6A04                           push 00000004

:100011F4 50                             push eax

:100011F5 E8C4070000                     call 100019BE

:100011FA 8B15A8910010                   mov edx, dword ptr [100091A8]

:10001200 8B0DAC910010                   mov ecx, dword ptr [100091AC]

:10001206 83C408                         add esp, 00000008

:10001209 EB13                           jmp 1000121E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100011E9(C)

|

:1000120B 0FBE8180910010                 movsx eax, byte ptr [ecx+10009180]    //表首址

//这里是第一张表

* Possible StringData Ref from Data Obj ->"                   (((((                    "

                                                                 ->"  H 剟剟剟剟剟亖亖亖"

                                                                 ->"倐倐倐"

                                                                 ->" "

                                                                 |

:10001212 8B3560810010                   mov esi, dword ptr [10008160]

:10001218 8A0446                         mov al, byte ptr [esi+2*eax]        //查表

:1000121B 83E004                         and eax, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001209(U)

|

:1000121E 85C0                           test eax, eax

:10001220 741D                           je 1000123F        //如果输入的不是数字，则这里会跳。

:10001222 83F908                         cmp ecx, 00000008  //满了 8 个则跳。

:10001225 7D18                           jge 1000123F

:10001227 8A8180910010                   mov al, byte ptr [ecx+10009180]

:1000122D 240F                           and al, 0F

:1000122F 8882B0910010                   mov byte ptr [edx+100091B0], al

:10001235 42                             inc edx

:10001236 8915A8910010                   mov dword ptr [100091A8], edx

:1000123C 41                             inc ecx

:1000123D EB9C                           jmp 100011DB

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001220(C), :10001225(C)

|

:1000123F 83FA08                         cmp edx, 00000008

:10001242 5E                             pop esi

:10001243 7D34                           jge 10001279

:10001245 4A                             dec edx

:10001246 B807000000                     mov eax, 00000007

:1000124B 8915A8910010                   mov dword ptr [100091A8], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000126C(C)

|

:10001251 85D2                           test edx, edx

:10001253 7D09                           jge 1000125E

:10001255 C680B091001000                 mov byte ptr [eax+100091B0], 00

:1000125C EB0D                           jmp 1000126B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001253(C)

|

:1000125E 8A8AB0910010                   mov cl, byte ptr [edx+100091B0]

:10001264 4A                             dec edx

:10001265 8888B0910010                   mov byte ptr [eax+100091B0], cl

//这里 100091b0 处执行完后就存放了前 8 位数字的实际值，也就是

05 01 00 00 00 00 09 09

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000125C(U)

|

:1000126B 48                             dec eax

:1000126C 79E3                           jns 10001251

:1000126E A3AC910010                     mov dword ptr [100091AC], eax

:10001273 8915A8910010                   mov dword ptr [100091A8], edx

//这里存放俩处理过的字节数，都是 8。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001243(C)

|

:10001279 C3                             ret

下面是第二个 CALL 的内容，这个 CALL 把第一个 CALL 计算出来的数字再查一次表，结果覆盖了 100091b0 处的 8 个字节：

* Referenced by a CALL at Address:

|:1000113B

|

:10001280 33C0                           xor eax, eax

:10001282 A3AC910010                     mov dword ptr [100091AC], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100012A7(C)

|

:10001287 0FBE88B0910010                 movsx ecx, byte ptr [eax+100091B0]

:1000128E 8B148590800010                 mov edx, dword ptr [4*eax+10008090]   //又查一次表

:10001295 40                             inc eax

:10001296 83F808                         cmp eax, 00000008        //处理 8 个字符

:10001299 8A0C0A                         mov cl, byte ptr [edx+ecx]

:1000129C A3AC910010                     mov dword ptr [100091AC], eax

:100012A1 8888AF910010                   mov byte ptr [eax+100091AF], cl      //存查表结果

:100012A7 7CDE                           jl 10001287

:100012A9 C3                             ret                          //返回

查出来的结果仍然放在 100091b0，这里变成了 04 0C 09 03 0D 07 06 08

最后得可以用的序列号： 5100-000099-9BFB

需要说明的是，这一大段是检验从注册表里读出来的序列号，不是检验输入的。——其实都大同小异嘛，不信，在反编译的 `sice405.txt` 中再查找一下“`function143`”看看，还有这一处被调用了：

`label185`:

00664E:0152:       `RegDBGetKeyValueEx(lString0, "Serial", lNumber1, lString1, lNumber3)`;

006665:0021:       `lNumber4 = LAST_RESULT`;

00666D:0128:       `lNumber5 = lNumber4 = 0`;

00667F:0022:       if (`lNumber5 = 0`) then

                            goto `label187`;

                      endif;

00668D:0013:       `lString3 = lString1`;

`label186`:

006699:0152:       `RegDBGetKeyValueEx(lString0, "Onomatopoeia", lNumber1, lString1, lNumber3)`;

0066B6:0021:       `lNumber4 = LAST_RESULT`;

0066BE:0128:       `lNumber5 = lNumber4 = 0`;

0066D0:0022:       if (`lNumber5 = 0`) then

                            goto `label188`;

                      endif;

0066DE:006D:       `StrToNum(lNumber0, lString1)`;

0066E6:0122:       `lNumber0 = lNumber0 >> 13`;

0066F3:006E:       `NumToStr(lString1, lNumber0)`;

//推测应该是序列号前 8 位转化成纯数字值再右移 13 位后成的数字必须能通过下面的这个函数检验。

`label187`:

0066FF:00B4:       `NMINST32.ValidCookie(2, 0, lString1)`;

//这是多出来的一重检验函数，但运行时 `nminst32.dll` 的这个输出函数无论如何都跟不进去（不断），也不知道是怎么回事。

006711:0021:       `lNumber5 = LAST_RESULT`;

006719:0128:       `lNumber5 = lNumber5 = 1`;

00672B:0022:       if (`lNumber5 = 0`) then

                            goto `label190`;

                      endif;

006739:00B5:       `function143(lString3)`;

//这里就是检验输入序列号是否合法的，检验形式基本一样，只不过前面多了一重束缚。

006744:0021:       `lNumber5 = LAST_RESULT`;

00674C:0128:       `lNumber5 = lNumber5 = 1`;

00675E:0022:       if (`lNumber5 = 0`) then

                            goto `label189`;

                      endif;

00676C:0013:       `string13 = lString3`;

006774:012F:       `return(1)`;

00677D:002C:       goto `label190`;

还有一点，前 8 位数字不是随便填什么都能在 `10001140` 处中断的，估计就是那个 `NMINST32.ValidCookie(2, 0, lString1);` 搞的鬼。我在跟踪过程中却不见程序执行到这个函数的入口，也不知道是什么原因。

另外 `SoftICE4.05` 是要找出能用的序列号才需要这么麻烦地跟，倘若只想用暴力跳过检验安装成功，只需来一个 `bpx messageboxA` 再看看上面第一个 `JNZ` 的指令就行，实在不难。

再写几个随便跟出来的、可以用的序列号（在已知某些可用的序列号的基础上，;P ）

`5107-0145BF-BB`

`5107-0045BB-BB`

`1107-0745FF-BF`

其实这个破得也不算完美，只是时间紧，写得不好，大伙儿包涵包涵。——这还是皮特陈大客“点将”的“恶果”，有气儿找他去，我躲了。哈！  `4dd466d0e1e0cc123afbe504da2462ca`
