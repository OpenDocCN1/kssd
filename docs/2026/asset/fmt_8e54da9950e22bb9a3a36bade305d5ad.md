
注意修改的这些指令块都是反向的，内存里的代码指令顺序相反。还有压栈的转移目标是重定位前的，如果修改DLL的话要注意这一点。现在在运行一下结果出错了，因为修改范围太大被查到了。重启程序，不用插件的汇编功能，只写入修改过的代码，随便输入40位16进制数就注册成功了，长度限制就不改了，否则输入超过40位可能会溢出。现在的问题是要修改的部分是解压出来的，不能直接修改文件，只能在解压后再写入。这也有很多方法，关键是解压后转到我们的代码，最简单的方法就是拦截程序真正的入口，走到这里肯定是解压完成的。但是这可能有一个问题，VMP在解压后会把段的内存属性设置成和加壳前相同，一般程序的代码段都是不可写的，这时可以自己修改内存属性或在VMP修改属性前写入补丁。因为这次只修改虚拟机代码，这些代码在VMP自己的段中，这个段始终是可写的，可以不考虑这个问题，等讲完脱壳后你会发现更好的拦截点。下面是找入口，VMP加壳后程序的入口很好找，先重启程序停在模块入口点然后分析，完成后看看记录，会发现这样的提示

```
  开始反汇编虚拟程序...

    指令块44DB51没有初始化，可能已加壳还没有解压完成，请解压完成后重新分析

  反汇编完成，75个指令块，7892个指令
```

因为真正的入口代码是压缩的，分析的时候还没有解压，插件不能继续分析，所以会给出这个提示，这里的`44DB51`就是真正的入口。只有入口代码也被虚拟化后才这样，如果没有被虚拟化的话就是一个`vRet`退出虚拟机返回到入口。到`44DB51`看看

```
004E68CD  |.  BD  vJmp_005BED37  ; 连接 VMPCrack.0044DB51; VMPCrack.0044DB51

0044DB51  |>  00  DB 00          ; 指令块没有被初始化
```

等壳的入口执行完后就会通过这个转移转到真正的入口，可以在这里插入一段代码写入补丁。如果不想出虚拟机的话可以使用`vWriteMemDs`来写入，但是直接写虚拟指令的话有点麻烦，如果补丁较长要写大量代码，可以用`vCall`调用一段汇编。`vCall`指令好像不是所有版本都有的，如果遇到没有的版本就只能通过`vRet`来调用了，但是要自己处理真实寄存器的恢复、返回时的虚拟程序初始化等，比较麻烦，还不如直接写虚拟指令了。先在`.vmp1`段里找一块空位，这个段是不需要解压的，写入补丁代码

```
005C0900    C705 A2004400 64191472    MOV DWORD PTR DS:[4400A2],72141964

005C090A    66:C705 A6004400 7C8A     MOV WORD PTR DS:[4400A6],8A7C

005C0913    C705 5F964C00 23C8CD07    MOV DWORD PTR DS:[4C965F],7CDC823

005C091D    66:C705 63964C00 9047     MOV WORD PTR DS:[4C9663],4790

005C0926    C705 FA9A4C00 A295AD0C    MOV DWORD PTR DS:[4C9AFA],0CAD95A2

005C0930    66:C705 FE9A4C00 E922     MOV WORD PTR DS:[4C9AFE],22E9

005C0939    C705 FD7E4600 8214AEF0    MOV DWORD PTR DS:[467EFD],F0AE1482

005C0943    66:C705 017F4600 B421     MOV WORD PTR DS:[467F01],21B4

005C094C    C705 4D7E4300 B6E3CBEF    MOV DWORD PTR DS:[437E4D],EFCBE3B6

005C0956    66:C705 517E4300 7671     MOV WORD PTR DS:[437E51],7176

005C095F    C3                        RETN
```

注意保护寄存器，`vCall`指令不会保护任何寄存器，要自己来做，这里没有用到寄存器所以不用处理。然后在那个转到入口的`vJmp`前插入指令

```
005C0FFF      97           vPopReg4 vR12        ; 弹出重定位

005C0FFE      C6 49EC81D9  vPushImm4 5C0900	//补丁代码地址

005C0FF9      FC 4F        vCall 0		//调用补丁代码，0个参数

005C0FF7      EF           vPopReg4 vR10	//弹出返回值

005C0FF6      BE 802FF1B7  vPushImm4 0EF74CFD4  ; 被覆盖的指令

005C0FF1      C2           vAdd4                ; 被覆盖的指令

005C0FF0      D1           vPopReg4 vR10        ; 被覆盖的指令

005C0FEF      CC           vPushReg4 vR12       ; 被覆盖的指令

005C0FEE      D1           vPushReg4 vR12       ; 重定位

005C0FED      80 5893A8AA  vPushImm4 4E68CF     ; 目标地址

005C0FE8      51           vJmp_005BED37        ; 转到原来的位置
```

先不保存，直接运行试试，提示文件损坏，看来有内存检查。重启一下，先把那段汇编代码填进去，没有问题，可能VMP不检查空位，只检查有效代码。再把虚拟指令的修改加上，这回查到了，设个硬件访问断点试试，断不到，可能被清了，换内存访问断点试试，又提示发现调试器，没办法了，只能看代码了。这些都是壳的入口里检查，还是留到脱壳时再讲吧，先直接给一个地址

```
005B15C3  |. /06  ||vJmp_005BED37  if (GetBytes(v331, 0, 1) != 0) goto <VMPCrack.检测到文件改变>
```

让他不跳就可以了，这回用插入指令的方法修改吧，因为有很多转移转到这个目标。注意现在直接反汇编入口的话看不到这个指令，因为中间有异常处理，插件不能自动分析，以后再讲。把所有添加的部分（包括插入指令时修改的虚拟机代码）选择复制到可执行文件然后保存，启动保存的文件又提示文件损坏，内存检查不是已经去掉了吗？保存文件后才被查到，那就可能是文件检查，还是看代码吧，改下面的地址

```
005A9B1F  |. /6A  ||vJmp_005BED37  if (GetBytes(v313, 0, 1) != 0) goto <VMPCrack.检测到文件改变>
```

让他不跳，其他的以后再讲。这回再保存文件，可以正常运行了，但是输入注册码后点确定进程结束，退出代码是`DEADC0DE`，肯定又被查到了。检查注册码时不是调了SDK吗，把它也改掉，由于这也是压缩的代码，放到上面的补丁里吧。

爆破完成，下面看看能不能找出算法，跟进验证函数里看看，这里贴出的代码还是去掉调用相关代码和垃圾转移后的。

```
0045B998  /$  C8  vPopReg4 vR14

0045B8A4  |.  CA  vPopVEsp       vESP = 0FFFFFFFC

0045B722  |.  F5  vReadMemSs4    DWORD s0 = Stack(vESP + 38, ESP + 4, 4)

00485989  |.  98  vWriteMemSs4   EXIT DWORD v0 = unknownInit2

004858DE  |.  95  vWriteMemSs4   EXIT DWORD v1 = unknownInit6

004BBC36  |.  37  vWriteMemSs4   EXIT DWORD v2 = unknownInit5

00456E9A  |.  A1  vWriteMemSs4   "长度" EXIT DWORD v3 = 10

00484B5C  |.  5A  vWriteMemSs4   "结果" EXIT DWORD v4 = 0FFFFFFFC

004A64A2  |.  39  vWriteMemSs4   "内容" EXIT DWORD v5 = s0		//用户名MD5

004A63DD  |.  12  vWriteMemSs4   "密码" EXIT DWORD v6 = 407498	//QWERTYUI

004B865E  |.  63  vWriteMemSs4   "加解密" EXIT DWORD v7 = 1		//解密

...

00484180  |.  2A  vRet           call v13; <VMPCrack.DES>
```

开始就是一个调用，跟进去看看，好像内容很多，没准是通用的算法。用`peid`打开进程，Krypto ANALyzer插件显示有base64、crc32、des、md5，md5刚才看到了，crc32和base64又不像，那是des？我对算法不是很熟悉，看看能不能找到什么特征，des应该有好几个表。跟进第一个调用里就有

```
0040122C  |> /8A86 D0604000 /MOV AL,BYTE PTR DS:[ESI+4060D0]
```

`4060D0`的内容是

```
004060D0  01 01 02 02 02 02 02 02 01 02 02 02 02 02 02 01
```

找个des代码看看，确实有这个，就是其中的`bitDisplace`。肯定是des了，加个标签，再把参数提示填上，因为这是调用方弹出参数，插件不会自动添加。遇到这种情况最好添加提示，否则插件不知道函数引用这些参数，可能把他们当成没用的而不显示。也可以在调用的`vRet`添加`/*exitValid(1)*/`，这样会认为退出虚拟机后可以引用的数据都是有效的，但不会在代码中标出参数。函数还会把加解密的结果保存到堆栈里，看看参数，地址是`0FFFFFFFC`就是-4，在`vRet`添加一个提示`/*vStackModify(-4,10,MD5)*/`，表示函数把堆栈-4开始10字节内容变成了MD5，以后用到时会自动标出来。参数加解密为1是解密，要解密的内容是从`ESP+4`读出来的，应该是第一个参数，就是用户名的MD5，密码是`QWERTYUI`。

```
0042AAF1  |.  D0  vReadMemSs4    DWORD m6 = DWORD SS:[Jnz(~v17 | AddFlag(LoWord(v21), 0FFFF)) + 0FFFFFFD4]

0042AABF  |.  55  vJmp_00411670  if (Jnz(~v17 | AddFlag(LoWord(v21), 0FFFF))) goto VMPCrack.004DD64A

0047F53A  |.  68  vReadMemSs4    DWORD s1 = Stack(vESP + 3C, ESP + 8, 4)

0048656B  |.  1E  vWriteMemSs4   EXIT DWORD v22 = 10

00486460  |.  93  vPopVEsp       vESP = 0FFFFFFD8

004862BF  |.  30  vReadMemSs4    DWORD m7 = DWORD SS:[Jle(v17) + 0FFFFFFD0]

00486298  |.  92  vPushReg4 vR0  DWORD v23 = s1

0048628D  |.  1D  vJmp_00411670  if (Jg(SubFlag(30, 34) ^ 8C4)) goto VMPCrack.004DBBDF
```

又有两个奇怪的比较，看看分支的代码都很像，可能还是垃圾分支，加提示清除掉。由于第二个垃圾分支在第一个后面，可以直接让第一个跳过去，添加`/*jmp(4DD64A)*/`后重新反汇编。这里再次提醒重新反汇编前要禁用或删除所有断点，否则断点插入到程序中的`vRet`会对反汇编造成影响，我就经常忘记，结果反出来很奇怪的代码。现在变成了

```
0042AABF  |. /55  vJmp_00411670    goto 0ACA0F192 ^ m6

004DD5E1  |.  88  vReadMemSs4      DWORD s1 = Stack(vESP + 3C, ESP + 8, 4)

004D7808  |.  2D  vWriteMemSs4     EXIT DWORD v22 = 10		//参数"长度"

004778EA  |.  9E  vWriteMemSs4     EXIT DWORD v23 = 20		//参数"结果"

0046732B  |.  54  vWriteMemSs4     EXIT DWORD v24 = 20		//参数"内容"，输入的注册码

00467233  |.  DC  vWriteMemSs4     EXIT DWORD v25 = 40748C	//参数"密码"，ASDFGHJK

00438232  |.  CA  vWriteMemSs4     EXIT DWORD v26 = 1		//参数"加解密"，解密

004380AF  |.  79  vReadMemSs4      DWORD m7 = DWORD SS:[Je(AddFlag(5, 0)) + 0FFFFFF78]

0043808B  |.  97  vPushReg4 vR2    DWORD v27 = 20

00438089  |.  9F  vPushReg4 vR9    DWORD v28 = 5

00438088  |.  B6  vPushReg4 vR15   DWORD v29 = s1

0043807D  |.  43  vJmp_00411670    if (5 + 0 == 0) goto VMPCrack.0046964E

0043807C  |>  10  /vPopReg4 vR13

0043804F  |.  F1  |vReadMemDs4     DWORD m8 = DWORD DS:[v29]

0043804D  |.  FA  |vWriteMemEs4    DWORD ES:[v27] = v30; DWORD v30 = m8

00437FB4  |.  BA  |vAdd4           DWORD v31 = 0FFFFFFFC + ((0FFFFFBFF ~& v17) >> 7)

00437EA2  |.  78  |vReadMemSs4     DWORD m9 = DWORD SS:[Jnz(AddFlag(v28, 0FFFFFFFF)) + 0FFFFFF78]

00437E7E  |.  F8  |vPushReg4 vR7   DWORD v27 = v27 + v31

00437E7C  |.  04  |vPushReg4 vR0   DWORD v28 = v28 + 0FFFFFFFF

00437E7B  |.  1D  |vPushReg4 vR11  DWORD v29 = v29 + v31

00437E70  |.^ B4  \vJmp_00411670   if (v28 != 0) goto VMPCrack.0043807C

0046964E  |>  26  vPopReg4 vR12

...

004E0003  |.  43  vRet             call v37; <VMPCrack.DES>
```

有一个循环，前面还有一个判断`if (5 + 0 == 0)`，常量和常量比较？看看是干什么的。循环里有`vWriteMemEs4`，一般很少用到`ES`段，可能是`MOVS`之类的指令。地址每次循环增加`0FFFFFFFC + ((0FFFFFBFF ~& v17) >> 7)`，很奇怪的计算，往前看看`v17`的值是`SubFlag(30, 34) ^ 8C4`，用标志来计算地址？既然是`MOVS`之类的指令前面很可能有`REP`，`0FFFFFBFF`二进制是`11111111111111111111101111111111`，第10位为0，对应`DF`（方向标志），和标志与非再右移7等于取出`DF`取反并设置到第3位，结果是0或8，再加上`0FFFFFFFC`（-4）就是-4或4。这就明白了，这个循环就是`REP MOVS`，根据方向标志每次对地址加4或减4，因为长度已知（5个`DWORD`），所以前面的判断显示为常量。

因为循环中写入的地址不是常量，插件不会把对应的数据添加到已知数据中，应该在写入的位置添加一个提示`/*vStackModify(20,14,"注册码")*/`，表示这个指令把堆栈20开始的14字节设为注册码。如果有写入地址不是常量的内存而你又知道地址的话，最好填上`vStackModify`或`vMemModify`提示，否则后面分析到这个地址的时候可能不知道数据已经改变。如果改变前数据没有初始化还好，直接生成一个读堆栈或读内存操作，否则分析时会认为数据还是初始化的内容，得到错误的结果，尤其是有循环的时候。比如

```
int a[4] = {0};//a初始化为0

int b;



for (int i = 0; i < 4; i++)

{

	a[i] = 1;//把a中所有的值设为1，由于这里i不是常量，不会把这个赋值添加到已知数据中

}

b = a[0];//这里会认为a中的值还是0
```

遇到这种情况只要在对a赋值的地方填上`vStackModify`提示就可以了，因为循环4次，每次写4字节，应该填写`vStackModify(a的地址,10,写入内容（可不填）)`。

后面又是des，因为参数压栈和调用隔着一个循环，没有自动标出，插件只能标出和调用在同一个指令块中的参数。地址是20，就是刚才那个循环复制的内容，`ESP+8`读出来的，应该是第二个参数，看看内容是`1234567812345678123456781234567812345678`，是我们输入的注册码，这里是要把输入的注册码解密。
