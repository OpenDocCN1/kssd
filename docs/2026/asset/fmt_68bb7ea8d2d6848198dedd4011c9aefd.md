
```
				.386

				.model	flat,stdcall

				option	casemap:none

include	windows.inc

include	user32.inc

include	kernel32.inc

includelib	user32.lib

includelib	kernel32.lib

include	hook.inc

includelib	hook.lib

IDD_DIALOG1        equ             101

IDC_EDIT1          equ             1000

IDC_EDIT2          equ             1001

IDC_EDIT3          equ             1002

IDC_EDIT4          equ             1003

IDC_BUTTON1        equ             1004

IDC_STATIC         equ             -1

IDT_TIME				 equ  				1

ELE_RAW		EQU	02E0A424h

ELE_HP		EQU	06E68270H

ELE_MP		EQU	025CH

ELE_MAXHP	EQU	06E68288H

ELE_MAXMP	EQU	06E68284H

				.data?

hInstance	dword	?

hProcess		dword	?;打开的进程

stMsg			MSG	<?>;消息结构

dwRaw			dword	?

dwHp			dword	?

dwMp			dword	?

dwMaxHp		dword	?

dwMaxMp		dword	?

 				.data

szErr			byte	'游戏并没有启动,请先启动游戏!',0

szErr2		byte	'权限不够，打开进程失败!',0

szTitle		byte	'QElementClient Window',0

				.code

_DiaProc		proc	uses	ebx esi edi hWnd,uMsg,wParam,lParam

				local		@ProcessId

				mov		eax,uMsg

				.if		eax==WM_COMMAND

							mov		eax,wParam

							.if		ax==IDC_BUTTON1

										invoke	_msg

							.endif

				.elseif	eax==WM_TIMER

							invoke	ReadProcessMemory,hProcess,ELE_RAW,addr dwRaw,4,NULL

							add		dwRaw,ELE_MP

 							invoke	ReadProcessMemory,hProcess,dwRaw,addr dwMp,4,NULL

 							invoke	SetDlgItemInt,hWnd,IDC_EDIT1,dwMp,FALSE

 							

				.elseif	eax==WM_CLOSE	

							invoke	UninstallHook

							invoke	DestroyWindow,hWnd

							invoke	PostQuitMessage,-1

				.elseif	eax==WM_INITDIALOG

							invoke	SetTimer,hWnd,IDT_TIME,1000,NULL

							invoke	FindWindow,addr szTitle,NULL

							mov		ebx,eax

							invoke	GetWindowThreadProcessId,ebx,addr @ProcessId

							invoke	OpenProcess,PROCESS_VM_OPERATION OR PROCESS_VM_READ OR PROCESS_VM_WRITE OR PROCESS_CREATE_THREAD,NULL,@ProcessId

							.if		eax

										mov		hProcess,eax

							.else

										invoke	MessageBox,NULL,addr szErr2,0,MB_OK

 							.endif

							invoke	InstallHook,hWnd

				.else	

							mov	eax,FALSE

							RET

				.endif

				mov		eax,TRUE

				RET					

_DiaProc		endp

start:

				invoke	GetModuleHandle,NULL

				mov		hInstance,eax

				invoke	FindWindow,addr szTitle,NULL

 				.if		eax

 							invoke	CreateDialogParam,hInstance,IDD_DIALOG1,eax,addr _DiaProc,NULL

							invoke	ShowWindow,eax,SW_SHOW

							.while	TRUE

										invoke	GetMessage,addr stMsg,NULL,0,0

										.break	.if	!eax

										invoke	TranslateMessage,addr stMsg

										invoke	DispatchMessage,addr stMsg

							.endw

				.else

							invoke	MessageBox,NULL,addr szErr,0,MB_OK

				.endif

				invoke	ExitProcess,-1

				end		start

```

第一次做这样的工作，如有任何不足之处请见凉.（本章只是学习，就不具体指出是哪个游戏）

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
