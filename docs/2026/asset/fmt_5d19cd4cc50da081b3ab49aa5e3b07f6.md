
# [原创]masm32学习之环境配置以及Win32汇编库初步封装之一

以前买了好大一堆书，都没几本看过。真是所谓的书非借而不能读也哎！看到满书柜上一堆一堆的书，满心的惭愧，所以打算把这些买回来的书都好好的看一番。翻来翻去，翻到了罗云彬的Window32汇编语言程序开发。汇编也基本上忘记的7788了，就从这本书开始吧。那么首先要整的无疑就是学习的开发环境了。masm32这个东西，比较轻量级，有库，有汇编器，有连接器，外加一个记事本，基本上就能写代码编译成`exe`了。书上没指定固定的开发环境和编辑器，编译汇编代码的编译模式都是用的make文件，然后用VS的`nmake`进行编译。先说编辑器，编辑器，记事本就行了，不过系统自带的记事本比较简陋，所以就选择了Notepad++来作为代码编辑的工具，选他最主要的就是功能足够（支持多语法高亮，支持各种插件）。代码写好了，要编译生成可执行文件。这个可以用写批处理或者自己在命令行下根据`ml`,`link`等命令来生成对应的文件。这里就有我选择notepad++的一个最主要的原因了，因为NotePad++有一个插件叫做NppExec，使用这个插件写上相关的脚本，就可以设置快捷键，然后用快捷键自动来编译生成需要的东西了，相当方便。书上的建议是制作`makefile`的文件，然后用`namek`进行编译连接。由于我是个人学习，多数情况下都是单个单个的汇编文件，所以，个人就不采用make文件的方式了，因为独立文件多，每测试都要写`make`比较麻烦，所以主要就写两个执行脚本就行了，一个脚本用来连接生成`lib`，另一个脚本用来连接生成`exe`文件，这样就基本上能写大部分的程序了，然后再写一个通过扩展名来调用不同的脚本来编译连接文件的通用脚本。之后每次，就只用调用这个通用脚本就行了（设置一个快捷键，代码好了之后保存一下，一键就行了）。这几个脚本文件如下

先是`.asm`后缀的，用来生成`exe`的脚本

```
cd $(CURRENT_DIRECTORY)

ENV_SET PATH = D:\Develop_Tools\Masm32\bin; D:\Develop_Tools\Masm32\include; D:\Develop_Tools\Masm32\lib

ml /c /coff /I D:\Develop_Tools\Masm32\include  $(NAME_PART).asm

link /LIBPATH![](https://bbs.kanxue.com/view/img/face/27.gif)\Develop_Tools\Masm32\lib  /subsystem:windows  $(NAME_PART).obj  /out:$(NAME_PART).exe

NPP_RUN $(NAME_PART).exe
```

这个脚本保存为`Compile@.asm`，其中的

`ENV_SET PATH = D:\Develop_Tools\Masm32\bin; D:\Develop_Tools\Masm32\include; D:\Develop_Tools\Masm32\lib`

这个是设置环境变量路径的，这里设置成你自己电脑上的路径则可

然后在写一个`.inc`文件用来生成`lib`库的脚本

```
cd $(CURRENT_DIRECTORY)

ENV_SET PATH = D:\Develop_Tools\Masm32\bin; D:\Develop_Tools\Masm32\include; D:\Develop_Tools\Masm32\lib

ml /c /coff /I D:\Develop_Tools\Masm32\include  $(NAME_PART).asm

link -lib *.obj  /out:$(NAME_PART).lib
```

保存为`Compile@.inc`

最后一个通用脚本

```
SET Compiler = Compile@$(EXT_PART)

NPP_EXEC "$(Compiler)"
```

随便取一个名字，写代码保存之后，我们直接调用这个脚本就行，注意，如果要编译生成`lib`，要打开对应的`.inc`文件，要生成`exe`就打开`.asm`文件，设置一个快捷键，于是此汇编环境配置完成

环境配置完成了之后，基本上就可以开始着手Win32的程序开发了，刚学习初步，就发现，有点麻烦，好些个代码基本上都是一种固定模式，虽然可以指定模版，但是每次拷贝来拷贝去，也比较麻烦，于是想着是否可以按照高级语言的特性对这些个比较常用的东西进行一个封装。就像Delphi有VCL,VC有MFC，于是就想着，何妨边学masm32边对Win32的汇编程序进行一个简单的封装，以达到引用`lib`就能简单的实现一个Windows程序的开发（就像Delphi和VC那种方式一样）。

我们都知道VC和Delphi中都有对象的概念，对象空间中，默认`eax`是指向对象本身。汇编中没有对象的概念，不过好在masm32中还有`struct`这个东西，不妨用他来作为一个简单对象的模拟。比如，我们可以定义一个对象结构和窗体结构如下

```
DxForm	struct

		Handle			dd		?

		Caption			dd		?

		WindowState		byte	?		

		WndProc			dd		?

DxForm	ends



DxApp	struct

		hInstance		dd		?;程序实例句柄

		Run  			dd		?;运行过程

		CreateForm		dd	? 				

		MainForm		dd	?;地址

		Forms			dd	?;Forms数组结构

		DefWndProc 		dd	?;默认的DefWindowProc

DxApp	ends 
```

这样，我们就可以通过`DxApp`这个来作为一个对象操作，因为是模拟对象操作，所以和Delphi,VC等高级语言一样，可以将`DxApp`地址放入`Eax`中，然后传递参数，之后可以用`Call`进行函数调用。

可以看到，在`DxApp`中定义的东西，有`Run`，运行本实例程序，有`CreateForm`用来创建窗体，`MainFrom`保存的是`DxForm`的数据地址，`Forms`保存的是程序中的创建的窗体列表，呵呵，熟悉Delphi的应该很明确，我这个是抄袭Delphi的模式来的。不多说，且看初步封装

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;Application头文件
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

DxForm	struct

		Handle			dd		?

		Caption			dd		?

		WindowState		byte	?		

		WndProc			dd		?

DxForm	ends



DxApp	struct

		hInstance		dd		?;程序实力

		Run  			dd		?;运行过程

		CreateForm		dd	? 				

		MainForm		dd	?;地址

		Forms			dd	?;Forms数组结构

		DefWndProc 		dd	?;默认的DefWindowProc

DxApp	ends 



;创建一个应用程序对象，App由外部传入DxApp地址

CreateApplication	proto	App:ptr DWORD





RGB	macro	red,green,blue

	xor	eax,eax

	mov	ah,blue

	shl	eax,8

	mov	ah,green

	mov	al,red

endm
```

上面的是`.inc`文件

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;DxApplication对象库
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	.386

	.model	flat,stdcall

	option	casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;包含库信息
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>	

include	windows.inc

include	user32.inc

include	kernel32.inc

include	gdi32.inc

includelib	user32.lib

includelib	kernel32.lib

includelib gdi32.lib 

include		DxApplication.inc



;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;数据定义区
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>
	.data

Application		dd	?

className	db	'DxForm',0

	.code





;>>>>>>>>>>>>>>>>>>Application的函数

ApplicationRun		proc	uses esi

	local	msg:MSG

	mov			esi, Application	

	mov			esi,[esi].DxApp.MainForm

	.if			[esi].DxForm.Handle==0

				invoke	ExitProcess,0

				ret

	.endif	

	invoke	ShowWindow,[esi].DxForm.Handle,SW_SHOWNORMAL	

	invoke	UpdateWindow,[esi].DxForm.Handle

	.while	TRUE

		invoke	GetMessage,addr msg,NULL,0,0

		.break	.if(!eax)

		invoke	TranslateMessage,addr msg

		invoke	DispatchMessage,addr msg

	.endw

	invoke		ExitProcess,0

	ret

ApplicationRun		endp





DefWndProc	proc	hWnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM

	local		ps:PAINTSTRUCT

	local		hdc:DWORD	

	local		rec:RECT	

   .if		uMsg==WM_DESTROY

   			invoke		PostQuitMessage,NULL

   			xor			eax,eax

   			ret

   .elseif	uMsg==WM_PAINT

   			;invoke		BeginPaint,hWnd,addr ps

   			;mov			hdc,eax

   			;invoke		GetClientRect,hWnd,addr rec

   			;RGB			255,0,0

   			;invoke		SetTextColor,hdc,eax

   			;invoke		DrawText,hdc,,-1,addr rec,DT_CENTER or DT_VCENTER or DT_SINGLELINE   			

   			;invoke		EndPaint,hWnd,addr ps

   			invoke		DefWindowProc,hWnd,uMsg,wParam,lParam

   			ret

   .else

   			invoke		DefWindowProc,hWnd,uMsg,wParam,lParam

   			ret

   .endif

   xor		eax,eax

   ret

DefWndProc	endp





CreateForm			proc	uses esi edi FormInfo:ptr DWORD

	local	wc:WNDCLASSEX	

	mov		wc.cbSize,SIZEOF	WNDCLASSEX

	mov		wc.style,CS_HREDRAW or CS_VREDRAW	

	mov		wc.cbClsExtra,NULL

	mov		wc.cbWndExtra,NULL

	mov		edi,Application

	push	        [edi].DxApp.hInstance

	pop		wc.hInstance

	mov		esi,FormInfo

	.if		(!esi)

			mov	esi,[edi].DxApp.MainForm ;esi为0，就表示为主窗体		

			.if([esi].DxForm.Handle);主窗体已经创建过了，创建失败！

				mov	eax,1

				ret

			.endif

	.endif

	.if([esi].DxForm.WndProc)				

			push	[esi].DxForm.WndProc

			pop		wc.lpfnWndProc

	.else

			mov	wc.lpfnWndProc,offset DefWndProc

	.endif

	mov		wc.hbrBackground,COLOR_WINDOW+1

	mov		wc.lpszMenuName,NULL

	mov		wc.lpszClassName,offset className

	invoke	LoadIcon,NULL,IDI_APPLICATION

	mov		wc.hIcon,eax

	mov		wc.hIconSm,eax

	invoke	LoadCursor,NULL,IDC_ARROW

	mov		wc.hCursor,eax

	invoke	RegisterClassEx,ADDR wc	

	invoke	CreateWindowEx,NULL,wc.lpszClassName,NULL,WS_OVERLAPPEDWINDOW,\

					CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,NULL,NULL,[edi].DxApp.hInstance,NULL

	.if		[esi].DxForm.Handle==0

			mov	[esi].DxForm.Handle,eax

	.endif				

	xor		eax,eax

	ret

CreateForm			endp



CreateApplication	proc	USES esi App:ptr DWORD

        mov			esi,App

	invoke		GetModuleHandle,NULL

	mov			[esi].DxApp.hInstance,eax	

	mov			[esi].DxApp.Run, offset ApplicationRun

	mov			[esi].DxApp.CreateForm,	offset CreateForm			;

	mov			[esi].DxApp.DefWndProc,offset DefWndProc ;指定系统默认的窗口过程

	mov			Application,esi

	ret

CreateApplication	endp
;>>>>>>>>>>>>>>>>>>Application的函数



end
```

这样，这个最基本的`Application`的汇编程序库就完成了，于是写一个简单的Win32的窗体就可以

```
	.386

	.model	flat,stdcall

	option	casemap:none

include	windows.inc

include	DxApplication.inc

includelib	DxApplication.lib



	.data

Application		DxApp	<>

	.code



winmain		proc

	local		mform:DxForm

	mov		mform.Handle,0

	mov		mform.WndProc,0

	mov		Application.hInstance,0

	lea		esi,mform

	mov		Application.MainForm,esi

	invoke	CreateApplication,offset Application	

	push		NULL;传递空，表示创建主窗体

	call		Application.CreateForm

	mov		eax,esi

	call		Application.Run  

winmain		endp



start:

	call	winmain

end		start	
```

初步封装完成，下一步想做的是封装一个类是Delphi的`TList`的链表对象的汇编库，然后就是在`Application`中整合窗体的保存以及消息流通，然后实现类似Delphi的`TControl`，以及`TGraphicControl`的初步封装，之后是控件的封装。。。。呵呵

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
