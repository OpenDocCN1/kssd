
# AuctionSleuth 2.6.4

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-96.htm`](https://www.pediy.com/kssd/pediy08/pediy8-96.htm)

`AuctionSleuth` 2.6.4

作者利用 SHA 加密的思路值得学习。不注册想得到注册码就得穷举，而作者却只要根据机器码对他的密钥进行简单的 `XOR` 就算出注册码。

### 1. 脱壳

`yoda's cryptor` 1.2

可以根据 `RORDBG` 的提示直接下断，也可以用 `OD` step by step 进行，代码不是很长。

#### 1.1 入口点

```
00571060 a> 60              pushad
00571061    E8 00000000     call auctions.00571066
00571066    5D              pop ebp
00571067    81ED F31D4000   sub ebp,auctions.00401DF3
```

#### 1.2 INT3 异常

```
0057110C    CC              int3
0057110D    8BEF            mov ebp,edi             ; 断在此，F8 往下
0057110F    33DB            xor ebx,ebx
00571111    64:8F03         pop dword ptr fs:[ebx]
```

#### 1.3 检测调试器：`IsDebuggerPresent`

```
0057169D    0BC0            or eax,eax
0057169F    74 08           je short auctions.005716A9
005716A1    FFD0            call eax                ; KeRnEl32.IsDebuggerPresent
005716A3    0BC0            or eax,eax
005716A5    74 02           je short auctions.005716A9
005716A7    61              popad
005716A8    C3              retn
```

#### 1.4 INT68 异常

```
005716DD    CD 68           int 68                  ; 异常
005716DF    33DB            xor ebx,ebx
005716E1    64:8F03         pop dword ptr fs:[ebx]
005716E4    83C4 04         add esp,4
```

发生异常后来到 `ntdll.dll` 的 `call ecx`：

```
77F8EB6E    FFD1            call ecx                ; auctions.005717A6
```

注意堆栈第 3 项，`dd 12FCD0+B8`＝`DD 12FD88`，在数据窗口跟随，显示：

```
0012FD88  005716DD  auctions.005716DD
0012FD8C  0000001B
0012FD90  00010346  UNICODE ".VBS;.VBE;.JS;.JSE;.WSF;.WSH"
```

步进 `call ecx`，注意数据窗口 `12FD88` 的变化：

```
005717A6    55              push ebp
005717A7    8BEC            mov ebp,esp
005717A9    57              push edi
005717AA    8B45 10         mov eax,dword ptr ss:[ebp+10]
005717AD    8BB8 9C000000   mov edi,dword ptr ds:[eax+9C]
005717B3    FFB7 EC264000   push dword ptr ds:[edi+4026EC]
005717B9    8F80 B8000000   pop dword ptr ds:[eax+B8]     ; 到这里变化
005717BF    89B8 B4000000   mov dword ptr ds:[eax+B4],edi
005717C5    C780 9C000000 0>mov dword ptr ds:[eax+9C],0
005717CF    B8 00000000     mov eax,0
005717D4    5F              pop edi
005717D5    C9              leave
005717D6    C3              retn
```

运行到 `005717B9` 后数据窗口变化：

```
0012FD88  005716DF  auctions.005716DF
0012FD8C  0000001B
0012FD90  00010346  UNICODE ".VBS;.VBE;.JS;.JSE;.WSF;.WSH"
```

`G 005716DF`：

```
005716DD    CD 68           int 68
005716DF    33DB            xor ebx,ebx
```

好，在 `005716DF` 下断，运行到此。

#### 1.5 跳往 OEP 的最后异常

```
0057175C    61              popad
0057175D    50              push eax
0057175E    33C0            xor eax,eax
00571760    64:FF30         push dword ptr fs:[eax]
00571763    64:8920         mov dword ptr fs:[eax],esp
00571766   /EB 01          jmp short auctions.00571769    ; 跳往异常处
00571768   |8700           xchg dword ptr ds:[eax],eax
0057176A    0000           add byte ptr ds:[eax],al
0057176C    0000           add byte ptr ds:[eax],al
0057176E    0000           add byte ptr ds:[eax],al
```

再次来到 `ntdll.dll` 的 `call ecx`：

```
77F8EB6E    FFD1            call ecx        ; auctions.0057170C
```

堆栈：

```
0012FC18   0012FCD4
0012FC1C   0012FFBC
0012FC20   0012FCF0
```

`DD 12FCF0+B8`：

```
0012FDA8  00571769  auctions.00571769
```

进 `call ecx` 后：

```
0057170C    55              push ebp
0057170D    8BEC            mov ebp,esp
0057170F    57              push edi
00571710    8B45 10         mov eax,dword ptr ss:[ebp+10]
00571713    8BB8 C4000000   mov edi,dword ptr ds:[eax+C4]
00571719    FF37            push dword ptr ds:[edi]
0057171B    33FF            xor edi,edi
0057171D    64:8F07         pop dword ptr fs:[edi]
00571720    8380 C4000000 0>add dword ptr ds:[eax+C4],8
00571727    8BB8 A4000000   mov edi,dword ptr ds:[eax+A4]
0057172D    C1C7 07         rol edi,7
00571730    89B8 B8000000   mov dword ptr ds:[eax+B8],edi     ; 在此变化
00571736    B8 00000000     mov eax,0
0057173B    5F              pop edi
0057173C    C9              leave
0057173D    C3              retn
```

数据窗口变化为：

```
0012FDA8  0040FCDC  auctions.0040FCDC
0012FDAC  0000001B
0012FDB0  00010346  UNICODE "OM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH"
```

`G 0040FCDC`：

```
0040FCDC    68 A0ED4100     push auctions.0041EDA0
0040FCE1    E8 EEFFFFFF     call auctions.0040FCD4          ; jmp to MSVBVM60.ThunRTMain
0040FCE6    0000            add byte ptr ds:[eax],al
```

呵呵，异常处理后就是从 `OEP` 处开始，明显是 `VB`。

`OD` 的插件，`dump`。运行 `ok`。

`Microsoft Visual Basic` 5.0 / 6.0

### 2. 破解

#### 2.1 `VBExplorer` 定位注册按钮

`frmRegPage:[cmdButton1.Click][&Register]:004D8490`

#### 2.2 机器码的由来

```
004D8656    E8 65E30100     call dumped.004F69C0
```

取得机器的一些参数：

```
eax=03455BD4, (UNICODE "52273-005-0056827-09813")        ; Windows NT ProductId
eax=001D7D7C, (UNICODE "3D2A-1BDB")                     ; C 盘 Serial Volume Number
eax=0344BBC4, (UNICODE "234340352")
eax=0345D744, (UNICODE "G-JH")                          ; ComputerName
eax=0345D70C, (UNICODE "09/05/02")                      ; SystemBiosDate
```

对参数的 `SHA-160` 散列：`call dumped.004F4590`

```
eax=0022C654, (UNICODE "75188C77 0C31F809 2F76E498 EE85200D 09269F53") ？
eax=03464654, (UNICODE "9115A710 422DB992 05D0F4E9 854D87DE 56977D7E")
eax=001B5A8C, (UNICODE "DAFDB61B 8B205226 C1D2AF44 F926711D 69FF850F")
eax=03468D6C, (UNICODE "A739E5FB BB45D49C DC947165 F8E972E6 3310EF07")
eax=0344B9E4, (UNICODE "3D5238D8 90C7D7E6 3BACF4D7 53B119D8 64E8E52D")
```

第一个不符合 `SHA-160`，其他的都符合。

然后各按顺序取 8 位（第一个取 1－8 位，第二个取 9－16 位，...）得到 5×8 个字符：

`75188C77 422DB992 C1D2AF44 F8E972E6 64E8E52D`

`SHA-160` 散列根据这几个参数确定的：

```
004F48B5    BA 44F04400    mov edx,dumped.0044F044      ; UNICODE "67452301"
004F48FD    BA 5CF04400    mov edx,dumped.0044F05C      ; UNICODE "efcdab89"
004F4945    BA 74F04400    mov edx,dumped.0044F074      ; UNICODE "98badcfe"
004F498D    BA 8CF04400    mov edx,dumped.0044F08C       ; UNICODE "10325476"
004F49D5    BA A4F04400    mov edx,dumped.0044F0A4       ; UNICODE "c3d2e1f0"
```

#### 2.3 对注册码的检查，如果相等提示出错

`BP MSVBVM60.__vbaStrComp`：

```
0012ED14   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012ED18   00000000
0012ED1C   034B034C  UNICODE "7742C4F66D462"
0012ED20   001D733C  UNICODE "8765432112345"

0012EBBC   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012EBC0   00000000
0012EBC4   0044BB5C  UNICODE "A9876543-BCDEF012-CBA09876-DEF01234-FA189567"
0012EBC8   034B04AC  UNICODE "8765432112345678"
```

这个很有隐蔽性，刚开始还以为明码比较，输入后才发现提示 `xx` 错误。

#### 2.4 经过上面的比较后，比较注册码长度

```
004F5F2F    50              push eax
004F5F30    FF15 44104000   call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]
004F5F36    83F8 2C         cmp eax,2C
004F5F39    74 0A           je short dumped.004F5F45
```

如果长度不是 44 的话，程序取用内置的字符串进行计算比较，值是固定的与注册码无关：

`0044F1E8=dumped.0044F1E8 (UNICODE "12345678-90123456-78901234-56789012-34567890")`

`00439DF8=dumped.00439DF8 (UNICODE "471B374072CBFEC8BB06D21CA182CC11DA39C173")`

```
0012EBBC   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012EBC0   00000000
0012EBC4   00167AA4  UNICODE "471B3740"
0012EBC8   00160CE4  UNICODE "52149160"

0012EBBC   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012EBC0   00000000
0012EBC4   001D7404  UNICODE "72CBFEC8"
0012EBC8   00227F2C  UNICODE "C73ACF45"

0012EBBC   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012EBC0   00000000
0012EBC4   034AC064  UNICODE "BB06D21C"
0012EBC8   034AFC3C  UNICODE "9961FE5F"

0012EBBC   660E8A12  返回到 MSVBVM60.660E8A12 来自 MSVBVM60.__vbaStrComp
0012EBC0   00000000
0012EBC4   034AC3A4  UNICODE "A182CC11"
0012EBC8   034AC1DC  UNICODE "077DF54D"
```



`0012EBBC`   `660E8A12`  返回到 `MSVBVM60.660E8A12` 来自 `MSVBVM60.__vbaStrComp`  

`0012EBC0`   `00000000`  

`0012EBC4`   `034AF814`  UNICODE `"DA39C173"`  

`0012EBC8`   `034AFC3C`  UNICODE `"5F6A5D31"`  

然后返回到：  

```
004DA3AA     E8 31BA0100          call dumped.004F5DE0
004DA3AF     66:3D FFFFF          cmp ax,0FFFF
004DA3B3     0F85 AB000000        jnz dumped.004DA464
```

在这里永远都不等，所以注册码的位数要＝44 位。  

### 2.5 如果输入的注册码刚好 44 位（2C）：  

那么先隔位取 8 位：  

练码：`87654321-12345678-87654321-12345678-11223344`  

得到：`8765432112345678876543211234567811223344`  

然后转换为字节：  

`87 65 43 21 12 34 56 78 87 65 43 21 12 34 56 78 11 22 33 44`  

然后与机器码 XOR，4 个字节一组：  

```
004F8542     3245 D4   xor al,byte ptr ss:[ebp-2C]
```

```
75 18 8C 77 42 2D B9 92 C1 D2 AF 44 F8 E9 72 E6 64 E8 E5 2D
87 65 43 21 12 34 56 78 87 65 43 21 12 34 56 78 11 22 33 44
```

得到 5 组，再分别 SHA-160：  

`eax=001F6E24`, (UNICODE `"F27DCF56"`)  

`eax=037DEE24`, (UNICODE `"8D0FED85A754328606E51AE97AB133C607336D69"`)  

`eax=00200274`, (UNICODE `"5019EFEA"`)  

`eax=037DEE24`, (UNICODE `"CC01A787FDD138A090DEF94D85E490C852391558"`)  

`eax=037FCEDC`, (UNICODE `"46B7EC65"`)  

`eax=037DEE24`, (UNICODE `"A54F5DE99F0518297AE785153D56C364F805A507"`)  

`eax=001F6FAC`, (UNICODE `"EADD249E"`)  

`eax=037DEE24`, (UNICODE `"5FD2A17D030E18FCF71144AA5FD43EAD600FA25D"`)  

`eax=00200274`, (UNICODE `"75CAD669"`)  

`eax=037DEE24`, (UNICODE `"3BB61E3CFE4F0859D9B801CCA189BBCFC032D956"`)  

每次 SHA－160 的值取前 8 位，顺序与 `471B374072CBFEC8BB06D21CA182CC11DA39C173` 比较。  

爆破后写入注册表：  

`HKCU\Software\AuctionSleuth\RegInfo\RegNumber`  

因为是错误的注册码，所以启动程序的时候要把注册表的值删除，否则又提示出错。  

### 2.6 算法小结：  

根据系统的信息获得的值进行 SHA－160 加密，各取前 8 位，这样有 5×8 个字符，作为机器码的一部分：  

`75188C77 422DB992 C1D2AF44 F8E972E6 64E8E52D`  

我的机器码为：`DL264-75188C77-422DB992-C1D2AF44-F8E972E6-64E8E52D`  

然后对输入的注册码有要求，为 44 位，每取 8 位隔 1 位，比如：  

练码：`87654321-12345678-87654321-12345678-11223344`  

得到：`8765432112345678876543211234567811223344`  

然后注册码与机器码一部分进行 XOR 运算，分 5 组进行：  

注册码：`87654321 12345678 87654321 12345678 11223344`  

机器码：`75188C77 422DB992 C1D2AF44 F8E972E6 64E8E52D`  

得到值：`F27DCF56 5019EFEA 46B7EC65 EADD249E 75CAD669`  

得到的 5 组值分别 SHA-160 得到 5 组散列值，各取前 8 位：  

`8D0FED85 CC01A787 A54F5DE9 5FD2A17D 3BB61E3C`  

如果等于 `471B3740 72CBFEC8 BB06D21C A182CC11 DA39C173` 的话，注册成功！  

作者用不可逆算法，所以反推注册码难度相当大，需要 5 次穷举 SHA-160。猜测作者注册的办法：  

有 5 组 8 字符组成的密钥（暂且称之，长度 5×8），这 5 组的 SHA-160 值的前 8 位分别为：  

`471B3740 72CBFEC8 BB06D21C A182CC11 DA39C173`  

这样购买者提供机器码，作者只要根据机器码与密钥的 XOR 值＝注册码  

如果作者想加强难度，那么就搞长点密钥。我相信没人会去穷举 n 次 SHA-160 的。  

### 2.7 爆破：  

其实就是用：`8D0FED85 CC01A787 A54F5DE9 5FD2A17D 3BB61E3C`  

替换掉：`471B3740 72CBFEC8 BB06D21C A182CC11 DA39C173`  

因为原程序加了壳，所以只能对脱完壳的程序下手，注意是 VB 的 UNICODE。  

`Winhex` 搜索，替换，很快。  

机器码：`DL264-75188C77-422DB992-C1D2AF44-F8E972E6-64E8E52D`  

注册码：`87654321-12345678-87654321-12345678-11223344`  

试用时间是 15 天，系统时间调后 1 个月，爆破后可以继续使用，未爆破的提示 15 天试用到期，如果不注册就退出程序。呵呵，看来爆破成功。 

`4dd466d0e1e0cc123afbe504da2462ca`
