
# 给记事本添加启动窗口

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-435.htm`](https://www.pediy.com/kssd/pediy09/pediy09-435.htm)
> 【文章标题】: 给记事本添加启动窗口  
> 【文章作者】: Suyana  
> 【作者邮箱】: Suyasha@163.com  
> 【软件名称】: Notepad  
> 【下载地址】: WinXP 自带  
> 【作者声明】: 我只是一只小菜鸟，失误之处难免，敬望诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

为了不写很多代码，使用 `dll` 的方式添加启动窗口，在 `exe` 中修改也可以，但代码太多了，容易出错，不过 `dll` 也是用 `Win32ASM` 编写的。

一开始，先记住以下几个地址：

- `01006AE0`：OEP，记事本原始的入口点
- `01007D73`：New OEP，新的入口点，我们的代码就写在这里

### 操作步骤

1. **添加必要的字符串**：用 `WinHex` 打开，在 `00030560` 添加字符串 `Splash.dll`
2. **添加导入函数**：记事本里有 `LoadLibrary` 函数，但没有 `FreeLibrary` 函数，用 `PE` 编辑器手工添加。
   - `LoadLibrary` 地址：`010010CC`（可用 `win32dasm` 或用 `Ollydbg` 查看）
   - `FreeLibrary` 地址：`010130E4`（`PE` 编辑器添加函数时就有了）
3. **反汇编**：用 `OllyDbg` 打开，来到 `01007D73`，写入如下代码：

```asm
push    01008560      ;字符串"Splash.dll"的地址
call    [10010CC]     ;LoadLibraryA,载入该 dll
test    eax, eax      ;是否成功
je      01006AE0      ;失败的话不释放资源
push    eax
call    [10130E4]     ;FreeLibrary,释放资源
jmp     01006AE0      ;继续执行
```

4. **更改入口点**：为 `01007D73`。运行就可以看到效果了。![微笑](img/9e827a3de2ad55eed0f95082425861b2.jpg "微笑")

### Splash.dll

功能只有一个：建立一个对话框，安装一个定时器，5 秒后结束。对话框也是用汇编编写。该 `DLL` 可用任何语言编写。![恐怖!](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

### 代码

```asm
.386
.model flat, stdcall
option casemap:none
include windows.inc
include user32.inc
include kernel32.inc
includelib user32.lib
includelib kernel32.lib

.data
TimerID    dd 0      ; 定时器 ID
hInstance  dd 0      ; 实例句柄

.code

DllEntry proc hInst:DWORD, reason:DWORD, reserved1:DWORD
    .if reason==DLL_PROCESS_ATTACH    ; 当 DLL 载入时执行
        push hInst
        pop hInstance
        call Splash
    .endif
    mov eax,TRUE
    ret
DllEntry Endp

_ProcDlgMain proc uses ebx edi esi hWnd,wMsg,wParam,lParam
        mov   eax,wMsg
        .if   eax == WM_INITDIALOG
            invoke SetTimer,hWnd,1,5000,NULL
            mov TimerID,eax
        .elseif eax == WM_TIMER
            invoke SendMessage,hWnd,WM_LBUTTONDOWN,NULL,NULL
            invoke KillTimer,hWnd,TimerID
        .elseif eax==WM_LBUTTONDOWN          ; 单击左键关闭对话框
            invoke EndDialog,hWnd,NULL
        .else
            mov eax,FALSE
            ret
        .endif
        mov   eax,TRUE
        ret
_ProcDlgMain endp

Splash  proc
        invoke DialogBoxParam,hInstance,101,NULL,offset _ProcDlgMain,NULL
Splash  endp

End DllEntry
```

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
