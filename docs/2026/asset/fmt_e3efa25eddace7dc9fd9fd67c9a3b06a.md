
在调试记录一下调用前后的`vESP`，分别为`0013F95C`和`0013F96C`，分析结果是4，调试得到的结果是10，所以应该填写`10-4=0C`，就是`/*esp(0C)*/`。实际上确实是这样，`GetDlgItem`有两个参数，返回时弹出两个参数和一个返回地址。由于插件不知道`ESP`改变，所以没有自动添加参数提示，这里也加上`arg(1,hWnd);arg(2,ControlID)`，再加个标签。`GetDlgItem`也有对其他寄存器的改变，比如`ECX`等，应该填`regModify(ecx)`的，但是普通的函数返回后一般不会使用除`EAX`（返回值）以外的寄存器，所以这些不填也没关系。后面所有API都这样处理，就不多做说明了，加密输入表后虚拟机中调用的API比较多的话还是有些麻烦，其实分析提示只填`ESP`改变就可以，加其他内容只是为了代码更好看。参数提示最好也填上，因为这个提示会告诉插件对应的数据时有效的，不应该被作为垃圾指令，否则函数返回后把参数弹出而且这之前没有对参数的引用的话，插件可能会认为参数对应的数据是没有用的。

这里开始准备取输入的用户名，调用前那一堆带`EXIT`的都是真实寄存器，只有退出虚拟机的调用才会标出寄存器名，插件为了调试方便把调用相关的代码比如恢复寄存器等都保留了，如果不需要的话不管他就行了，这里贴出的代码为了节省空间直接去掉了。

```
0049DD33  |.  34  vReadMemSs4     DWORD s0 = Stack(vESP + 38, ESP + 4, 4)

004C7830  |.  32  vWriteMemSs4    ControlID EXIT DWORD v32 = 3E8	//用户名输入框ID

00486CDA  |.  BF  vWriteMemSs4    hWnd EXIT DWORD v33 = s0		//主窗口句柄，ESP + 4说明是第一个参数传进来的

0046540A  |.  D9  vJmp_00411C30   callVM <VMPCrack.GetDlgItem>		//取用户名输入框句柄

004B5E69  |.  32  vWriteMemSs4    hWnd EXIT DWORD v42 = "用户名hwnd"

0048B7F5  |.  09  vJmp_00411C30   callVM <VMPCrack.GetWindowTextLengthA>//取输入的用户名长度

0048324C  |.  34  vReadMemSs4     DWORD m5 = DWORD SS:[Je(AndFlag("用户名长度", "用户名长度")) + 0FFFFFF44]

00483229  |.  FC  vPushReg4 vR4   DWORD v51 = "用户名长度"

00483227  |.  1C  vPushReg4 vR10  DWORD v52 = "用户名长度"

0048321A  |.  F1  vJmp_00412430   if ("用户名长度" != 0) goto VMPCrack.0044E3A0//检查输入的用户名长度是否为0

004565AC  |.  DC  vPushReg4 vR6   Style EXIT DWORD v53 = v52

0043E13E  |.  32  vWriteMemSs4    Title EXIT DWORD v54 = 4074EC		//标题"VMPCrackMe"

00456CC4  |.  E4  vWriteMemSs4    Text EXIT DWORD v55 = 4074DC		//内容"请输入用户名"

00483041  |.  E4  vWriteMemSs4    hOwner EXIT DWORD v56 = v38

004B6338  |.  D9  vJmp_00411C30   callVM <VMPCrack.MessageBoxA>		//弹出消息框

0042F277  |.  34  vReadMemSs4     DWORD s1 = Stack(vESP + 34, ESP + 0, 4)

...

0042EF25  |.  44  vRet            return v67				//函数结束
```

上面是检查用户名是否为空，为空的话弹出消息"请输入用户名"并结束。

```
004482FA  |.  50  vAdd4          DWORD v76 = SubFlag(v51, 20)

00479AB2  |.  34  vReadMemSs4    DWORD m10 = DWORD SS:[Ja(v76) + 0FFFFFF44]

00479A81  |.  05  vJmp_00412430  if (v51 <= 20) goto VMPCrack.004D0510		//v51 = "用户名长度"

0043EDAB  |.  34  vReadMemSs4    DWORD m11 = DWORD SS:[Jpo(v76) + 0FFFFFF44]

0043ED7A  |.  05  vJmp_00412430  if (Jpe(v76)) goto VMPCrack.004D4ABA
```

这里有一个比较`if (v51 <= 20) goto VMPCrack.004D0510`，在信息窗口看看，`v51`的值是"用户名长度"，说明用户名长度不能超过`0x20`（32）个字符。后面的那个`if (Jpe(v76))`不知道是什么意思，`v76`等于`SubFlag(v51, 20)`，正常代码中不太可能有这种东西，除非是转移类型分析错了。如果加了变形的话可能是垃圾分支，这种情况下两个分支的代码是相同的，等于把代码复制了一份。往下看看确实是这样，执行的代码都是弹出消息框然后退出，肯定是垃圾分支了，加个分析提示清除掉。添加提示`/*jmp(4D4ABA)*/`，告诉插件作为只有一个目标的转移处理，因为转移在反汇编阶段处理，必须重新反汇编才能生效，可以在分析点窗口中选择重新分析或直接按空格键。重新反汇编前一定要注意删除或禁用所有断点，因为普通断点会在代码中插入`vRet`，这样反出来的代码是错误的，到插入的`vRet`时就会停止。

重新反汇编后代码变成了

```
0043ED7A  |. /05  vJmp_00412430   goto m11 ^ 6F67D83E

00440CB7  |.  50  vAdd4           Style EXIT DWORD v83 = 0

004DA716  |.  BF  vWriteMemSs4    Title EXIT DWORD v84 = 4074EC		//标题"VMPCrackMe"

004346AA  |.  32  vWriteMemSs4    Text EXIT DWORD v85 = 4074C4		//内容"用户名不能超过32个字符"

00439EB1  |.  32  vWriteMemSs4    hOwner EXIT DWORD v86 = v38

00439DA2  |.  09  vJmp_00411C30   callVM <VMPCrack.MessageBoxA>		//弹出消息框

004D90C8  |.  94  vReadMemSs4     DWORD s2 = Stack(vESP + 34, ESP + 0, 4)

...

004D8D23  |.  16  vRet            return v96				//函数结束
```

再往下看有一个调用，也是只有一个指令块，像是调用API，但最后不是调用，而是返回了

```
00450450  /$  7E  vPopReg4 vR2

004502F0  |.  0C  vPopfd          EFL = 0FFFFF700 & unknownInit6

004502ED  |.  4C  vReadMemSs4     DWORD s0 = Stack(vESP + 34, ESP + 0, 4)

004502EC  |.  20  vReadMemDs1     BYTE m0 = BYTE DS:[s0]

0045018F  |.  10  vMul2           WORD v0 = GetBytes(9 * (0 : (2595 ^ GetBytes(Rdtsc(), 0, 2)) % 1B5), 0, 2)

0045013C  |.  8C  vAdd4           DWORD v1 = 428F55 + v0

0045010A  |.  5A  vReadMemDs1     BYTE m1 = BYTE DS:[4 + v1]

00450108  |.  24  vReadMemDs4     DWORD m2 = DWORD DS:[v1]

00450005  |.  2D  vReadMemDs4     DWORD m3 = DWORD DS:[5 + v1]

00450002  |.  0B  vAdd4           DWORD v2 = m3 + Check(0 + (400000 + (862FE2A3 + ((708F098F ^ m2) + 1 - 1))), 0 : (0 : ByteToWord(m1))) + (((40 & AddFlag(34, m0)) >> 3) + Stack(34, 4))

00450000  |.  F6  vPushReg4 vR10  EBP DWORD v3 = unknownInit8

0044FFFF  |.  FB  vPushReg4 vR5   EDX DWORD v4 = unknownInit9

0044FFFE  |.  F7  vPushReg4 vR9   EAX DWORD v5 = unknownInit5

0044FFFD  |.  FF  vPushReg4 vR1   EBX DWORD v6 = 77D1216B

0044FFFB  |.  00  vPushReg4 vR0   ESI DWORD v7 = unknownInit4

0044FFFA  |.  F9  vPushReg4 vR7   EFL DWORD v8 = unknownInit6

0044FFF9  |.  F5  vPushReg4 vR11  ECX DWORD v9 = unknownInit7

0044FFF8  |.  F3  vPushReg4 vR13  EDI DWORD v10 = unknownInit10

0044FFF5  \.  86  vRet            return v2
```

这里除`EBX`以外的寄存器都没有改变，`EBX`变成了`77D1216B`，看看地址，是`GetWindowTextA`，应该也是解密输入表，对应的汇编代码类似`MOV EBX,[<GetWindowTextA>]`，再往后可能会有像`CALL EBX`这样的代码，往下看看果然是这样，有一个`0048EA90 |. B5 vRet call v120`，就是调用`GetWindowTextA`，但是插件不知道，可以加一个提示`/*call(77D1216B,4D7656)*/`，表示调用目标为`77D1216B`（`GetWindowTextA`），返回到`4D7656`，返回到的地址是被调用函数返回时的地址，不是虚拟机中的地址，注意不同的系统API地址可能不同。

```
004337E4  |.  E4  vWriteMemSs4    EXIT DWORD v105 = 21		//参数Count

00493ACB  |.  E4  vWriteMemSs4    EXIT DWORD v106 = 0FFFFFFE8	//参数Buffer

00493A68  |.  32  vWriteMemSs4    EXIT DWORD v107 = v51		//参数hWnd，v51 = "用户名hwnd"

0049398D  |.  EB  vJmp_00411C30   callVM VMPCrack.00450450	//解密GetWindowTextA的地址，保存到EBX

...

0048EA90  |.  B5  vRet            call v120; user32.GetWindowTextA
```

由于加了最大保护，调用前的那些`vNand`是加密寄存器，先返回到解密寄存器的代码，解密后再调用，还有检查虚拟机完整性，会用`vCheck`随机检查一段内存和返回地址计算，所以内容有点长，这里没贴出来，不用管这些。由于参数压栈后有一个虚拟机内调用，然后才是API，所以没有自动标出，参数`Buffer`因为是堆栈地址，在分析时以`vESP`偏移的形式保存，`FFFFFFE8`表示`vESP+18`。

往下又是3个调用

```
0043365C  |.  BF  vWriteMemSs4   "MD5对象" EXIT DWORD v129 = 0CC75CF87

...

004886EF  |.  16  vRet           call v135; <VMPCrack.MD5初始化>

0043D766  |.  32  vWriteMemSs4   "长度" EXIT DWORD v144 = v141

004AD7AB  |.  BF  vWriteMemSs4   "内容" EXIT DWORD v145 = 235A22B8

004B18A6  |.  BF  vWriteMemSs4   "MD5对象" EXIT DWORD v146 = 235A2234

...

004BCE2E  |.  B2  vRet           call v152; <VMPCrack.MD5计算>

004B2E32  |.  32  vWriteMemSs4   "MD5对象" EXIT DWORD v161 = 15CC7877

00496BA8  |.  BF  vWriteMemSs4   "保存MD5值" EXIT DWORD v162 = 15CC7933

...

004963CC  |.  B5  vRet           call v168; <VMPCrack.MD5取结果>
```

这些代码中都把检查虚拟机完整性和加密寄存器的部分去掉了，太占地方，而且一眼就能看出来，不用管它就行了。先跟进第一个调用看看，那些常量很熟悉吧，就是MD5初始化。由于这几个函数都是调用方弹出参数，被调用函数只弹出返回地址，所以没有自动加参数提示，自己添加一下。但是分析出来的参数好像不对，比如要初始化的MD5对象是`0CC75CF87`，这显然不像是一个有效的地址，往前跟踪数据来源看看怎么回事。在信息窗口中一直跟随引用数据，最后跟到了调用`GetWindowTextA`时的`EBP`，由于`EBP`这时是常量，而且最大保护下调用函数前要加密寄存器，就是异或一个常量，结果还是常量，分析时自动计算了。但是调用函数时要先解密，插件没有处理，认为调用后还是加密的值，所以错了。找到原因就好，可以在返回后添加一个常量提示，返回后弹出`EBP`是在

```
0045B289  |.  08  vPopReg4 vR11  DWORD \_t22128 = 0CC75D053
```

添加提示`/*const(1,20)*/`，表示第一个操作数的值是常量20，也就是加密前的`EBP`，后面的调用也做相同处理，其实不填写也可以，只要自己能看明白就行。后面的两个函数猜都能猜出来是什么了，初始化后肯定是计算、取MD5值，跟进去看看确实是这样，顺便加上标签和参数提示。

再往下看

```
00430681  |.  C0  vNand4          ControlID EXIT DWORD v175 = 3E9	//注册码输入框ID

004305A8  |.  32  vWriteMemSs4    hWnd EXIT DWORD v176 = v173		//主窗口句柄

004A9744  |.  09  vJmp_00411C30   callVM <VMPCrack.GetDlgItem>		//取注册码输入框句柄

004462EC  |.  32  vWriteMemSs4    hWnd EXIT DWORD v185 = "注册码hwnd"

00498016  |.  09  vJmp_00411C30   callVM <VMPCrack.GetWindowTextLengthA>//取输入的注册码长度

00455B49  |.  55  vReadMemSs4     DWORD m41 = DWORD SS:[Jnz(AndFlag("注册码长度", "注册码长度")) + 0FFFFFF50]

00455B26  |.  CC  vPushReg4 vR7   DWORD v193 = "注册码长度"

00455B17  |.  F1  vJmp_00412430   if ("注册码长度" == 0) goto VMPCrack.00466E05//检查是否输入了注册码，转到弹出提示"请输入注册码"

004DF80B  |.  10  vAdd4           DWORD v194 = SubFlag(v193, 28)

00467664  |.  34  vReadMemSs4     DWORD m42 = DWORD SS:[Je(v194) + 0FFFFFF50]

00467632  |.  F1  vJmp_00412430   if (v193 != 28) goto VMPCrack.00441B20//v193 = "注册码长度"，检查注册码是否为28位，转到弹出提示"注册码错误。"

0048A6D2  |.  32  vWriteMemSs4    Count EXIT DWORD v195 = 29		//长度

0048A665  |.  32  vWriteMemSs4    Buffer EXIT DWORD v196 = 0FFFFFFAC	//保存注册码

004812BF  |.  32  vWriteMemSs4    hWnd EXIT DWORD v197 = v190		//v190 = "注册码hwnd"

...

004B2243  |.  16  vRet            call v203; user32.GetWindowTextA
```
