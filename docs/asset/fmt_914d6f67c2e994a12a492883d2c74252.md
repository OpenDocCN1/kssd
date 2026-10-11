
# Image to PDF V2.2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6523.htm`](https://www.pediy.com/kssd/pediy06/pediy6523.htm)

软件名称：`Image to PDF V2.2.0`

下载地址：[`www.adultpdf.com/products/image2pdf/index.htm`](http://www.adultpdf.com/products/image2pdf/index.htm)

软件大小：1.94M

软件简介：可将各种图形文件转换成一个 PDF 文件

注册限制：未注册版生成的 PDF 文件里有被打上了未注册的信息

破解者： `fengxu`

作者申明：对 `CRACK` 感兴趣，以学习为目的，别无他意。

破解工具：`PEiD0.9`, `AspackDie1.41`, `Dede3.5`, `Ollydbg1.10c`

-----------------------------------------------------------

## 破解过程

首先用 `PEiD` 检测发现是 `ASPack2.12` 的壳，用 `AspackDie` 脱壳成功。运行脱壳后程序，可以正常运行。

再次用 `PEiD` 检测脱壳后程序，发现是用 `BCB` 编写的程序。于是用 `Dede` 调入程序，反编译后可以很容易找到注册对话框 `OK` 按钮所对应的事件 `Button2Click` 程序的入口地址：`0040BAA0`。

`OK`, 现在需要动态调试了，启动 `OD`,载入并运行程序。出现注册窗口，输入：

`email:` `fengxu@mymail.com`
`serial No.:` `78787878`

切换到 `OD`, 下断点：`bp 0040BAA0`

返回程序，点击 `ok`,程序被 `OD` 中断再 `0040BAA0`.

```
0040BAA0  /.  55             PUSH EBP
0040BAA1  |.  8BEC           MOV EBP,ESP
0040BAA3  |.  83C4 B8        ADD ESP,-48
0040BAA6  |.  53             PUSH EBX
0040BAA7  |.  56             PUSH ESI
0040BAA8  |.  57             PUSH EDI
0040BAA9  |.  8BD8           MOV EBX,EAX
0040BAAB  |.  BE B7BE4D00    MOV ESI,unpacked.004DBEB7
0040BAB0  |.  8D7D CC        LEA EDI,DWORD PTR SS:[EBP-34]
0040BAB3  |.  B8 9CC14D00    MOV EAX,unpacked.004DC19C
0040BAB8  |.  E8 A7E00B00    CALL unpacked.004C9B64
0040BABD  |.  66:C747 10 14>MOV WORD PTR DS:[EDI+10],14
0040BAC3  |.  33D2           XOR EDX,EDX
0040BAC5  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
0040BAC8  |.  8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
0040BACB  |.  FF47 1C        INC DWORD PTR DS:[EDI+1C]
0040BACE  |.  8B83 28050000  MOV EAX,DWORD PTR DS:[EBX+528]
0040BAD4  |.  E8 F34D0600    CALL unpacked.004708CC         ; 取 EMAIL 地址
0040BAD9  |.  66:C747 10 08>MOV WORD PTR DS:[EDI+10],8
0040BADF  |.  66:C747 10 20>MOV WORD PTR DS:[EDI+10],20
0040BAE5  |.  33C9           XOR ECX,ECX
0040BAE7  |.  894D F8        MOV DWORD PTR SS:[EBP-8],ECX
0040BAEA  |.  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
0040BAED  |.  FF47 1C        INC DWORD PTR DS:[EDI+1C]
0040BAF0  |.  8B83 08050000  MOV EAX,DWORD PTR DS:[EBX+508]
0040BAF6  |.  E8 D14D0600    CALL unpacked.004708CC         ; 取输入的序列号
0040BAFB  |.  66:C747 10 08>MOV WORD PTR DS:[EDI+10],8
0040BB01  |.  837D FC 00     CMP DWORD PTR SS:[EBP-4],0     ; 地址为 NULL？
0040BB05  |.  8D4D B8        LEA ECX,DWORD PTR SS:[EBP-48]
0040BB08  |.  74 05          JE SHORT unpacked.0040BB0F     ; 是则跳转
0040BB0A  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
0040BB0D  |.  EB 03          JMP SHORT unpacked.0040BB12
0040BB0F  |>  8D56 1C        LEA EDX,DWORD PTR DS:[ESI+1C]
0040BB12  |>  8BC3           MOV EAX,EBX
0040BB14  |.  E8 A3050000    CALL unpacked.0040C0BC         ; 关键 call
0040BB19  |.  837D F8 00     CMP DWORD PTR SS:[EBP-8],0     ; 序列号为 NULL？
0040BB1D  |.  74 05          JE SHORT unpacked.0040BB24     ; 是则跳
0040BB1F  |.  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
0040BB22  |.  EB 03          JMP SHORT unpacked.0040BB27
0040BB24  |>  8D56 1D        LEA EDX,DWORD PTR DS:[ESI+1D]
0040BB27  |>  8BC3           MOV EAX,EBX
0040BB29  |.  E8 6E080000    CALL unpacked.0040C39C
0040BB2E  |.  84C0           TEST AL,AL
0040BB30  |.  0F85 97000000 JNZ unpacked.0040BBCD
0040BB36  |.  6A 10          PUSH 10
0040BB38  |.  837D F8 00     CMP DWORD PTR SS:[EBP-8],0
0040BB3C  |.  74 05          JE SHORT unpacked.0040BB43
0040BB3E  |.  8B4D F8        MOV ECX,DWORD PTR SS:[EBP-8]
0040BB41  |.  EB 03          JMP SHORT unpacked.0040BB46
0040BB43  |>  8D4E 1E        LEA ECX,DWORD PTR DS:[ESI+1E]
0040BB46  |>  51             PUSH ECX
0040BB47  |.  8D45 B8        LEA EAX,DWORD PTR SS:[EBP-48]
0040BB4A  |.  50             PUSH EAX
```

到这里我们可以看到 `ECX` 对应的是我们输入的字符串`"78787878"`,而 `EAX` 则对应着一串让人敢兴趣的字符串`"ECVHHWK8P5MQHV3D"`,是不是明码比较呢？

```
0040BB4B  |.  E8 C8DE0B00    CALL unpacked.004C9A18
0040BB50  |.  83C4 0C        ADD ESP,0C
0040BB53  |.  85C0           TEST EAX,EAX
0040BB55  |.  74 76          JE SHORT unpacked.0040BBCD     ; 若跳则显示注册通过，否则提示出错
```

看来前面的两个字符串应该是进行序列号比较了。`DISABLE` 所有断点，进入注册画面,在序列号中输入 `ECVHHWK8P5MQHV3D`，点击 `OK`，提示注册成功。

因此得到一组正确的序列号：

`email:` `fengxu@mymail.com`
`serial No.:` `ECVHHWK8P5MQHV3D`

下面我们来看看其注册码是怎样生成的：`f7` 进入关键 `call 0040C0BC`, `f8` 一路来到这里：

```
0040C0EE  |.  BE 93BE4D00    MOV ESI,unpacked.004DBE93      ; ASCII
                                                 ; "PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32"
0040C0F3  |.  8D7C24 08      LEA EDI,DWORD PTR SS:[ESP+8]
0040C0F7  |.  B9 09000000    MOV ECX,9
0040C0FC  |.  F3:A5          REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
0040C0FE  |.  8D4424 2C      LEA EAX,DWORD PTR SS:[ESP+2C]
0040C102  |.  50             PUSH EAX                     ; /Arg1
0040C103  |.  E8 C80D0000    CALL unpacked.0040CED0       ; \unpacked.0040CED0
```

`ESI` 对应的字符串是一张数据表，下面会用到，用来生成序列号。`f7` 进入 `40CED0`:

```
0040CED0  /$  55             PUSH EBP
0040CED1  |.  8BEC           MOV EBP,ESP
0040CED3  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
0040CED6  |.  33D2           XOR EDX,EDX
0040CED8  |.  8950 14        MOV DWORD PTR DS:[EAX+14],EDX
0040CEDB  |.  8950 10        MOV DWORD PTR DS:[EAX+10],EDX
0040CEDE  |.  C700 01234567  MOV DWORD PTR DS:[EAX],67452301
0040CEE4  |.  C740 04 89ABC>MOV DWORD PTR DS:[EAX+4],EFCDAB89
0040CEEB  |.  C740 08 FEDCB>MOV DWORD PTR DS:[EAX+8],98BADCFE
0040CEF2  |.  C740 0C 76543>MOV DWORD PTR DS:[EAX+C],10325476
0040CEF9  |.  5D             POP EBP
0040CEFA  \.  C3             RETN
```

上面的赋值语句显示这里用了 `MD5` 算法。

继续：

```
0040C138  |.  8D9C24 840000>LEA EBX,DWORD PTR SS:[ESP+84]
0040C13F  |>  33C0           /XOR EAX,EAX
0040C141  |.  8A03           |MOV AL,BYTE PTR DS:[EBX]
```

查看 `EBX` 处的数据为：`B518037B5843870723F1307112498AC6`

将 `EMAIL` 地址输入 `MD5` 运算器算一下，两者完全一样。

```
0040C143  |.  B9 23000000    |MOV ECX,23
0040C148  |.  99             |CDQ
0040C149  |.  F7F9           |IDIV ECX    ; 除以 0x23，即前面提到的数据表长度
0040C14B  |.  33C0           |XOR EAX,EAX
0040C14D  |.  8A4414 08      |MOV AL,BYTE PTR SS:[ESP+EDX+8] ; 取余数，查表
0040C151  |.  50             |PUSH EAX                     ; /<%c>
0040C152  |.  68 C7C04D00    |PUSH unpacked.004DC0C7       ; |Format = "%c"
0040C157  |.  8D5424 08      |LEA EDX,DWORD PTR SS:[ESP+8] ; |
0040C15B  |.  52             |PUSH EDX                     ; |s
0040C15C  |.  E8 F99F0C00    |CALL <JMP.&USER32.wsprintfA> ; \wsprintfA
0040C161  |.  83C4 0C        |ADD ESP,0C
0040C164  |.  8A0C24         |MOV CL,BYTE PTR SS:[ESP]
0040C167  |.  880E           |MOV BYTE PTR DS:[ESI],CL  ; 保存结果
0040C169  |.  47             |INC EDI
0040C16A  |.  46             |INC ESI
0040C16B  |.  43             |INC EBX
0040C16C  |.  83FF 10        |CMP EDI,10      ; 循环次数
0040C16F  |.^ 7C CE          \JL SHORT unpacked.0040C13F
0040C171  |.  C645 11 00     MOV BYTE PTR SS:[EBP+11],0
```

## 算法总结

使用数据表：`PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32`

将输入的 `EMAIL` 地址进行 `MD5` 加密，将加密结果的每个字节除以数据表长度 `0X23`,取余数，将余数作为指针查数据表得到序列号。序列号长度为 16 位字符。

`4dd466d0e1e0cc123afbe504da2462ca`
