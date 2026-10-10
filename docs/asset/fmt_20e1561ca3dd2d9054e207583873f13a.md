
# MASM 之 ShellCode 框架编写\[合并帖\]

> 原文：[`www.pediy.com/kssd/pediy12/135062.html`](https://www.pediy.com/kssd/pediy12/135062.html)

编写 ShellCode 前首先要做的是确定获取 `kernel32` 基地址的方法,在如今 WIN7 插手的年代,不得不抛弃以前在 XP 下通用的获取 `kernel32` 基地址的方法,而选择能同时在 WIN7 下运行的代码,参考文章:http://skypher.com/wiki/index.php/Hacking/Shellcode/kernel32,我们就拥有了兼容 WIN7 的代码了:(代码中寄存器改了下,所以和链接中的原文不一样了,不要奇怪哦)

代码:

```
assume fs:nothing
xor ecx,ecx
mov esi,fs:[030H] ; ESI = &(PEB) ([FS:0x30])
mov esi,[esi+0cH] ; ESI = PEB->Ldr
mov esi,[esi+01cH] ; ESI = PEB->Ldr.InInitOrder
next_module:
mov eax,[esi+08H] ; eax = InInitOrder[X].base_address
mov edi,[esi+020H] ; eax = InInitOrder[X].module_name (unicode)
mov esi,[esi] ; ESI = InInitOrder[X].flink (next module)
cmp [edi+12*2],cl ; modulename[12] == 0 ?
JNE next_module ; No: try next module.
```

该代码执行之后,`EAX` 会被设置成 `kernel32` 的基地址,之后就是动态获取 API 的方法了,可以自己实现 `GetProcAddress`,也可以直接调用 `GetProcAddress`,这里我偷下懒,直接调用原来的 `GetProcAddress`

代码:

```
mov esi,eax
mov edi,30
assume eax:ptr IMAGE_DOS_HEADER
mov ecx,DWORD ptr [eax+edi*2] ; 03cH .e_lfanew,用了点迷惑作用
lea eax,[eax+ecx+2]
assume eax:ptr IMAGE_NT_HEADERS
add edi,078H-2-30
mov ecx,DWORD ptr [eax+edi] ; 078H .OptionalHeader.DataDirectory.VirtualAddress,用了点迷惑作用
.if ecx
add ecx,esi
assume ecx:ptr IMAGE_EXPORT_DIRECTORY
mov ebx,cGetProcAddress
sub ebx,[ecx].nBase
shl ebx,2
add ebx,[ecx].AddressOfFunctions
add ebx,esi
mov ebx,[ebx]
;dec ebx ;API 地址故意减去 1,让 OD 无法显示 API 名称
add ebx,esi ;ebx = "GetProcAddress" address
mov dGetProcAddress,ebx
@fcall ebx,esi,cLoadLibraryA
;dec eax ;API 地址故意减去 1,让 OD 无法显示 API 名称
mov dLoadLibraryA,eax
@fcall ebx,esi,cGetModuleHandleA
;dec eax ;API 地址故意减去 1,让 OD 无法显示 API 名称
mov dGetModuleHandleA,eax
```

之后要考虑到是问题是,如果插入字符串,在 SHELLCODE 中字符串只能插入到 CODE 中了,但也有几种插入的方法,这里说下我常用的方法:

代码:

```
CALL @F
db "这里是字符串",0
pop ...
```

但如果代码中插入大量这样的代码,不仅操作麻烦,还影响阅读,所以直接用一个宏来解决,如下:

代码:

```
settext macro arg,text:VARARG
local @name
call @F
@name:
db text,0
@@:
ifnb <arg>
pop arg
endif
endm
```

有了这几样之后,编写 SHELLCODE 基本上不成问题了,下面就来编写一个弹出信息框的例子:

代码:

```
settext szMsgName,"MessageBoxA"
settext szMsg,"SHELLCODE 测试信息框"
settext szDllName,"user32"
;下面要用到 GetProcAddress & LoadLibraryA & GetModuleHandleA 组合来获取 API 函数 MessageBoxA,并弹出一个信息框
@fcall dGetModuleHandleA,szDllName
.if !eax
@fcall dLoadLibraryA,szDllName
.endif
.if eax
mov hDLL,eax
@fcall dGetProcAddress,hDLL,szMsgName
.if eax
@fcall eax,0,szMsg,0,0 ;弹出一个信息框
.endif
.endif
.endif
```

可能你会说,每次这样调用太麻烦了,的确,所以建议大家可以建一张表,把要载入的 API 事先都读取好,之后只要用索引或者常量就可以调用了!如果下次有机会的话,我再写一份代码

补上源代码:

`frame1.rar`

`bin.rar`

注意:本来以为 API 的序号导出换了系统还是一样的,现在才知道是不一样的,所以在其他系统上运行才会出错,额额,看来还是要改改了!!![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

与之前写的不同在于(http://bbs.pediy.com/showthread.php?t=135062),这次都用了宏来做框架,调用 API 也是非常方便了,不废话,直接看代码

代码:

```
.386
.model flat, stdcall
option casemap :none

include windows.inc
include myMacro.asm
.CODE
START proc
local APIArrayBuff[2]:DWORD ;设置一个 API 缓冲区,只要设置 API 总数就足够了

local chBuf[128]:BYTE ,dwRet
pushad
mov dwRet,128

;*********************************************************************************
; SHELLCODE 新构架 设置导入表,注意,这里都不用双引号
Import user32,MessageBoxA
Import Advapi32,GetUserNameA
; LdrImport 的功能是将所有 API 装载到目标地址 APIArrayBuff 则为 API 存放地址
;调用的时候 只要 APIArrayBuff[cMessageBoxA] 就可以访问到 API 地址
;也可以直接用 ImportApiCall cMessageBoxA 的方式直接调用
;cMessageBoxA 常量是自动生成的,是由"c" + API 名称组成
LdrImport APIArrayBuff
;*********************************************************************************


;下面是两种调用方式 ImportApiCall 和 InlineCall
;区别是:ImportApiCall 需要调用 Import 和 LdrImport 之后才能使用,他可以直接用 API 内部常量来调用 API
;而 InlineCall 可以直接使用,但不可以直接用 API 内部常量来调用 API

ImportApiCall cGetUserNameA,addr chBuf,addr dwRet ;获取当前用户名

ImportApiCall cMessageBoxA,0,addr chBuf,"ImportApiCall:",48

InlineCall APIArrayBuff[cMessageBoxA],0,addr chBuf,"InlineCall:",16

popad
ret
START endp
end START
```

编译后得出代码:

代码:

```
55 8B EC 81 C4 74 FF FF FF 60 C7 85 74 FF FF FF 80 00 00 00 60 83 EC 10 83 64 24 0C 00 33 C9 64
8B 35 30 00 00 00 8B 76 0C 8B 76 1C 8B 46 08 8B 7E 20 8B 36 38 4F 18 75 F3 8B F0 BF 1E 00 00 00
8B 0C 78 8D 44 01 02 83 C7 58 8B 0C 07 0B C9 74 34 03 CE BB 99 01 00 00 2B 59 10 C1 E3 02 03 59
1C 03 DE 8B 1B 4B 03 DE 89 1C 24 68 45 02 00 00 56 FF D3 48 89 44 24 04 68 77 01 00 00 56 FF D3
48 89 44 24 08 E8 12 00 00 00 75 73 65 72 33 32 00 01 41 64 76 61 70 69 33 32 00 01 5E E8 19 00
00 00 4D 65 73 73 61 67 65 42 6F 78 41 00 47 65 74 55 73 65 72 4E 61 6D 65 41 00 5F B9 02 00 00
00 EB 4E 51 56 FF 54 24 10 0B C0 75 05 56 FF 54 24 0C 0B C0 74 39 8B D8 56 E8 3D 00 00 00 8D 74
30 02 0F B6 4E FF EB 23 51 57 53 FF 54 24 10 0B C0 74 0C 8B 4C 24 14 89 44 8D F8 FF 44 24 14 57
E8 16 00 00 00 8D 7C 38 01 59 49 0B C9 75 D9 59 49 0B C9 75 AE 83 C4 10 61 EB 17 57 8B 7C 24 08
B9 FF FF FF FF 33 C0 F2 AE F7 D1 49 8B C1 5F C2 04 00 8D 85 74 FF FF FF 50 8D 85 78 FF FF FF 50
FF 55 FC 6A 30 E8 0F 00 00 00 49 6D 70 6F 72 74 41 70 69 43 61 6C 6C 3A 00 8D 85 78 FF FF FF 50
6A 00 FF 55 F8 6A 10 E8 0C 00 00 00 49 6E 6C 69 6E 65 43 61 6C 6C 3A 00 8D 85 78 FF FF FF 50 6A
00 FF 55 F8 61 C9 C3
```

易语言版:

代码:

```
.版本 2

置入代码 ({ 85, 139, 236, 129, 196, 116, 255, 255, 255, 96, 199, 133, 116, 255, 255, 255, 128, 0, 0, 0, 96, 131, 236, 16, 131, 100, 36, 12, 0, 51, 201, 100, 139, 53, 48, 0, 0, 0, 139, 118, 12, 139, 118, 28, 139, 70, 8, 139, 126, 32, 139, 54, 56, 79, 24, 117, 243, 139, 240, 191, 30, 0, 0, 0, 139, 12, 120, 141, 68, 1, 2, 131, 199, 88, 139, 12, 7, 11, 201, 116, 52, 3, 206, 187, 153, 1, 0, 0, 43, 89, 16, 193, 227, 2, 3, 89, 28, 3, 222, 139, 27, 75, 3, 222, 137, 28, 36, 104, 69, 2, 0, 0, 86, 255, 211, 72, 137, 68, 36, 4, 104, 119, 1, 0, 0, 86, 255, 211, 72, 137, 68, 36, 8, 232, 18, 0, 0, 0, 117, 115, 101, 114, 51, 50, 0, 1, 65, 100, 118, 97, 112, 105, 51, 50, 0, 1, 94, 232, 25, 0, 0, 0, 77, 101, 115, 115, 97, 103, 101, 66, 111, 120, 65, 0, 71, 101, 116, 85, 115, 101, 114, 78, 97, 109, 101, 65, 0, 95, 185, 2, 0, 0, 0, 235, 78, 81, 86, 255, 84, 36, 16, 11, 192, 117, 5, 86, 255, 84, 36, 12, 11, 192, 116, 57, 139, 216, 86, 232, 61, 0, 0, 0, 141, 116, 48, 2, 15, 182, 78, 255, 235, 35, 81, 87, 83, 255, 84, 36, 16, 11, 192, 116, 12, 139, 76, 36, 20, 137, 68, 141, 248, 255, 68, 36, 20, 87, 232, 22, 0, 0, 0, 141, 124, 56, 1, 89, 73, 11, 201, 117, 217, 89, 73, 11, 201, 117, 174, 131, 196, 16, 97, 235, 23, 87, 139, 124, 36, 8, 185, 255, 255, 255, 255, 51, 192, 242, 174, 247, 209, 73, 139, 193, 95, 194, 4, 0, 141, 133, 116, 255, 255, 255, 80, 141, 133, 120, 255, 255, 255, 80, 255, 85, 252, 106, 48, 232, 15, 0, 0, 0, 73, 109, 112, 111, 114, 116, 65, 112, 105, 67, 97, 108, 108, 58, 0, 141, 133, 120, 255, 255, 255, 80, 106, 0, 255, 85, 248, 106, 16, 232, 12, 0, 0, 0, 73, 110, 108, 105, 110, 101, 67, 97, 108, 108, 58, 0, 141, 133, 120, 255, 255, 255, 80, 106, 0, 255, 85, 248, 97, 201, 195 })
```

ps:感谢 `Mr.Crowley` 的帮忙~

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `ShellCode frame2.rar` |
