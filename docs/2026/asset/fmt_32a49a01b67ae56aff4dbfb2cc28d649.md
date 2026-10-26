
C++

```
const char *psBlackCodes[] = { "mvg21951736", "mg374604342", "mg370534035", "mg373465241", "mg37NTi", "mg372503958", "mg379843149",

"mg370151347", "mg370353008", "mg372021424", "mg375953248", "mg379223953", "mg373473759", "mg378542544", "mg370473710", "mg37064348",

"mg378822469", "mg374394987", "mg371073478", "mg379773651", "mg371895266", "mg373223554", "mg377583454", "mg37644957", "mg370342692",

"mg376484039", "mg376871434", "mg370704788", "mg377643863", "mg377753931", "mg379342689", "mg374344777" };

const char* pszRandomCharTable = "0123456789abcdefghijklmnopqrstuvwxyz";

const int lengthOfCharTable = lstrlenA(pszRandomCharTable);

const char *pResourceTable = "HKRWQV2958DWNTQRGNSCFSXAZPYK";





inline char randomChar(void){

	return pszRandomCharTable[rand() % (lengthOfCharTable + 1 - 0) + 0];

}
```

如果dump代码，怎么搞都没搞成，不用ebp的。。

```
//see c8fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9q4)9J5k6r3y4F1i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3y4Z5K9o6y4X3j5U0m8C8i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6m8Q4x3U0W2Q4x3X3g2S2M7%4m8^5  f10K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9q4)9J5k6r3y4F1i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8U0u0C8P5s2R3#2N6o6u0U0i4K6t1^5g2W2y4Q4x3X3f1^5x3q4)9J5z5g2)9J5k6h3q4K6M7s2R3`.
//#pragma optimize( "y", off )  // 对下面的代码使用  /Oy 优化 Generate frame pointers on the program stack.

int keygen_dump(const char *arg_0, const char *arg_4){

	//框架指针寄存器“ebp”被内联程序集代码修改 

	_asm{
		mov     ebp, arg_4	//就乱了。。。。
		cmp     byte ptr[ebp + 0], 6Dh
		jnz     loc_4341EF
		cmp     byte ptr[ebp + 1], 67h
		jnz     loc_4341EF
		cmp     byte ptr[ebp + 2], 33h
		jnz     loc_4341EF
		cmp     byte ptr[ebp + 3], 37h
		jnz     loc_4341EF
		xor     ebx, ebx

	loc_434162 :
		mov     edi, psBlackCodes
		mov     eax, edi
		lea     edx, [eax + 1]
		lea     ecx, [ecx + 0]

	loc_434170 :
			   mov     cl, [eax]
			   inc     eax
			   test    cl, cl
			   jnz     short loc_434170
			   sub     eax, edx
			   mov     ecx, eax
			   mov     esi, ebp
			   xor     eax, eax
			   repe cmpsb
			   jz      short loc_4341E8
			   add     ebx, 4
			   cmp     ebx, 80h
			   jb      short loc_434162
			   cmp     byte ptr[ebp + 4], 73h
			   jnz     short loc_434195
			   inc     ebp

		   loc_434195 :
	lea     ecx, [ebp + 7]
		push    ecx
		push	ecx
		call    codekeygen   //----------------------------------
		pop ecx
		pop ecx
		push    ecx
		call    atol
		mov     ebx, arg_0
		mov     dl, [ebx]
		add     esp, 4
		xor     ecx, ecx
		test    dl, dl
		mov     edi, ebx
		mov     esi, 0BDFh
		jz      short loc_4341DA

	loc_4341B4 :
	movsx   edx, dl
		inc     ecx
		imul    edx, ecx
		add     esi, edx
		cmp     esi, 17BEh
		jle     short loc_4341CB
		sub     esi, 17BEh

	loc_4341CB:
	cmp     ecx, 0Ah
		jle     short loc_4341D2
		xor     ecx, ecx

	loc_4341D2 :
	mov     dl, [edi + 1]
		inc     edi
		test    dl, dl
		jnz     short loc_4341B4

	loc_4341DA :
	push esi

	cmp     esi, eax

		jnz     short loc_4341F3
		pop     ebp
		mov     eax, 1
		pop     ebx
		retn//移除pop esi和pop ebp

	loc_4341E8 :
	pop     edi

		xor     eax, eax
		pop     ebx
		retn //移除pop esi和pop ebp

	loc_4341EF :

	mov     ebx,  arg_0

	loc_4341F3 :
	push    ebp
		push    ebx
		call    codekeygen  //移除两参数
			   pop     ebp
			   pop     ebx
			   retn//移除pop esi和pop ebp
}
}

//#pragma optimize("", on)    // 恢复到编译器 /O 选项指定的优化
```

### 5.3. 汇编（跃然纸上）

注册时让注册码跃然纸上，这招貌似叫内嵌补丁。

个人觉得算是个费时的活儿，又写汇编，又要了解PE等，知识点散。而且只适用于程序把注册码计算出来和输入的Code直接比较，即`Code == Func(Name)`???型。避免了写注册机算法等等，如果很复杂这很适合，当然也可以远程调用。

为了便于好看，有大量截图，也是用心写的证明，就想发点有价值的东西，不然不好。

#### 5.3.1. 第一处改写。

主要获取Code窗口句柄`HWND`和保存，如果是局部变量就好了，但是我没看看它是不是，直接保存到代码区的。

修改前两处

修改前后

#### 5.3.2. 目标代码。

来到我们预定的空隙处，这里也是准备些代码的地方，这里`call api`暂时是拷贝的，后面修改。

设置访问方式

修改访问后，再执行此条指令，如下图我们看到地址`0x0047EA94`值变了，共4字节。注意这里是临时修改，要实际修改，还要保存。君子善假于物也。使用论坛工具`LordPE`。

继续执行到原来`PUSH EAX`后的指令`CALL EBX`时，即此处的寄存器状态和之前未修改的图是一致的。这里要说下，原来未修改的截图还未执行`PUSH EAX`，所以原`ESP`比这里的大4，截得不好。

#### 5.3.3. 第二处改写

来到注册码已经出来的地方，然后我们想在界面Code编辑框里打印显示出来。

说明下，由于`ESP`寻找局部变量，所以只要找到`ESP+84`（十六进制）处看看就可，右下角堆栈窗口就可以看；或者直接点击代码行右键-数据窗口中跟随-内存地址，即可左小角查看。

这条指令大于5字节，直接线程替换为我们的跳转`JMP 0047EA9E`，剩余的`90`填充然后`F7`或`F8`跟随进去。

当我们获取原来`LEA ESI, [LOCAL.24]`里的`[LOCAL.24]=[ESP + 84]`时，堆栈和原来不一样的差了8.*4 = 32. = 20h，从右下角的图也可以看出，当时记录的`ESP`和此时`ESP`相差`$-20`。所以取值时多加了这个差值。

我们走一步，通过赋值给`ESI`看下这时的值，`OllyDbg`会提示的。

再看看第一个入参，`[Alt+W]`打开当前可预见的窗口。

#### 5.3.4. 这里`call API`有一定讲究。

导入`API`

首先看它导入了没有，OK。

`OllyDbg`的查找所有模块中的名称（`Ctrl+N`）

`call`地址计算

目标进程右键-载入PE编辑器-目录-输入表一行左边省略号按钮上面ListView点选`USER32.dll`-下面输入`SetWindowTextA`搜索自动定位。

注意未勾选总是查看`FisrtThunk`时值时，`ThunkRVA`按照函数列表计算的值是不一样的。

未勾选时，第一个函数`SystemParametersInfoA`的`ThunkRVA`是`0008BCC8`和`USER32.dll`的`OrigianlFirstThunk`一样的，以后相差+4。

勾选后，地址为`0007F30C`，记下它，后面会眼熟的。

我们来拿个现成例子看看地址间关系。

回到`OllyDbg`，在程序模块`movgear.exe`，右键查找-所有模块间调用

搜索`System`看到

点击进入

我们看看`SystemParametersInfoA`的调用方式不是`CALL xx`，而是`CALL DWORD PTR DS:[xx]`，即`call`机器码不一样。

再看看地址关系，可以看出`CALL DWORD PTR DS:[xx]`的`xx`是`0047F30C`，后4位是不是很眼熟。是的，和前面看到的`ThunkRVA = 0047F30C`，是一致的，只是多了个`400000`，其实`40`万（十六进制）就是我们在前面一起看到的，基本PE头信息里的镜像基址（`BaseImage`）`400000`。

接下来看我们的`SetWindowTextA`，以此类推，调用写法，`400000 + 0007F448 = 0047F448` ，`48F44700`，`CALL DWORD PTR DS:[ 0047F448]`，`OllyDbg`自动解析为对应符号。

#### 5.3.5. 修改的代码

```
CPU Disasm

地址        十六进制数据            指令                                       注释

00434531     /E9 68A50400   JMP 0047EA9E
```
```
CPU Disasm

地址        十六进制数据            指令                                       注释

00433F60     /E9 39AB0400   JMP 0047EA9E
```
```
CPU Disasm

地址        十六进制数据            指令                                       注释

0047EA97      00            DB 00                                    ; 原来.text区是不可写，这也证明了足够
0047EA98      00            DB 00                                    ; 安全性，而且原是0映射后还是0，
0047EA99      00            DB 00                                    ; 不用搞检测看这段是否可能被修改。
0047EA9A      00            DB 00                                    ; 一般程序也不会调用API来改吧，又没壳。
0047EA9B      00            DB 00
0047EA9C      00            DB 00
0047EA9D      00            DB 00
0047EA9E  />  60            PUSHAD                                   ; /Arg1_4, >>保存8大寄存器状态，也可仅push使用的
0047EA9F  |.  E8 00000000   CALL 0047EAA4                            ; \movgear.0047EAA4, 随意搞吧，不管你怎么使用寄存器，只要栈操作平衡。
0047EAA4  |$  59            POP ECX                                  ; 获取EIP，即ECX = EIP = 0047EAA4
0047EAA5  |.  8B51 F0       MOV EDX,DWORD PTR DS:[ECX-10]            ; 这里把.text区地址0047EA94上DWORD大小给了EDX
0047EAA8  |.  85D2          TEST EDX,EDX
0047EAAA  |.  75 12         JNZ SHORT 0047EABE
0047EAAC  |.  8941 F0       MOV DWORD PTR DS:[ECX-10],EAX
0047EAAF  |.  61            POPAD                                    ; <<恢复8大寄存器状态
0047EAB0  |.  FFD3          CALL EBX                                 ; >>恢复原指令的调用流程
0047EAB2  |.  8D8CE4 C40000 LEA ECX,[LOCAL.24]                       ; <<恢复原指令的调用流程
0047EAB9  |.^ E9 7C5AFBFF   JMP 0043453A                             ; 回去并执行接下来的指令
0047EABE  |>  C741 F0 00000 MOV DWORD PTR DS:[ECX-10],0
0047EAC5  |.  8DB4E4 A40000 LEA ESI,[LOCAL.24]                       ; 这里使用ESP的局部变量要注意。
0047EACC  |.  56            PUSH ESI                                 ; /Text
0047EACD  |.  52            PUSH EDX                                 ; |hWnd
0047EACE  |.  FF15 48F44700 CALL DWORD PTR DS:[<&USER32.SetWindowTex ; \USER32.SetWindowTextA, call xx 需要计算xx = ImageBase + ThunkRVAOfAPI
0047EAD4  |.  61            POPAD                                    ; <<恢复8大寄存器状态
0047EAD5  |.  8DB4E4 840000 LEA ESI,[LOCAL.24]                       ; >><<恢复原指令的调用流程
0047EADC  \.^ E9 8654FBFF   JMP 00433F67                             ; 回去并执行接下来的指令
0047EAE1      E8 6B25EB74   CALL SetWindowTextA                        ;直接修改为CALL SetWindowTextA，错，仅临时偏移对
```

#### 5.3.6. 最后看下效果图：

### 6. 总结

`LoadPE`使用，熟练`OllyDbg`，`Call API`

### 7. 附件
