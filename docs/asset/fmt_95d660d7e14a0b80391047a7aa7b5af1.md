
# R！SC 的 CrackMe1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue055.htm`](https://www.pediy.com/kssd/pediy04/kanxue055.htm)

破解 `R！SC` 的 `CrackMe1` 并写一个 `LOADER`

娃娃(NYDoll)

最近没有什么好玩的软件，不是太难的破解不了；就是太简单不值得写个过程，于是来到 DDXia 前辈的主页上下载了一个 `r!sc` 所写的 `CrackMe1`。执行一看 —— 是一个去除 NAG 的 CrackMe，^\_^ 最近正好对这个东西感兴趣 正和我意 赶快动手吧。

先用 `FI` 检查一番，发现是 `PETILE2.1` 的壳，用专用的脱壳器可以很容易的脱壳，但是还是先不要脱壳 ^\_^ 跟跟看～～～

一运行，程序马上蹦出个窗口，提示：“This program has expired :( please purchase the full version”，再点一下确定，程序就退出了。

立刻祭出 `TRW`（我还是喜欢用我修改的 `TRW1.22` 增强版本 ^\_^ 尽快别的大哥说不好看），载入程序，出现提示窗口，然后下断点：`BPX LOCKMYTASK`，点击确定按钮，你就会发现又被 `TRW` 拦下了，下命令 `PMODULE`，你会发现 `TRW` 正好停在 API 函数 `MESSAGEBOXA` 的下面 ^\_^ 如果你也到了这里，那你就正确了一半了。

```
XXXX：00406282    PUSH            DWORD 004062E0
XXXX：00406287    PUSH            DWORD 00406360
XXXX：0040628C    PUSH            BYTE +00
XXXX：0040628E    CALL  USER！MessageBoxA            <——CrackMe 程序出现对话框信息所调用的 API
XXXX：0040628C    POPA                               <——TRW 应该这里
XXXX：0040628C    PUSH            BYTE +00
```

现在我们已经找到程序出现第一个对话框时所在的地址了，接下来我们在 `MessageboxA` 函数前下断点，`BPX 406282`，然后重新运行程序，当程序执行到 `406282` 的时候被 `TRW` 拦到，看看这个时候程序还没有跳出对话框吧 ^\_^ 接下来我们要跳过这个对话框，

用 `TRW` 向上翻，发现：

```
XXXX：0040622F    ADD             [EAX－41]，AH
XXXX：00406232    MOV             AL，[57004062]
XXXX：00406237    CALL  KERNEL32！GetLocalTime
XXXX：0040623D    CMP             WORD [EDI]，07CF
XXXX：00406242    JG             00406280
XXXX：00406244    JZ             0040624D
XXXX：00406246    CMP             WORD[EDI＋02]，BYTE ＋06
XXXX：0040624B    JNL             00406280
XXXX：0040624D    PUSH            BYTE ＋30
XXXX：0040624F    PUSH            DWORD 004062E0
XXXX：00406254    PUSH            DWORD 004062F0
XXXX：00406259    PUSH            BYTE ＋00
XXXX：0040625B    CALL  USER！MessageBoxA
XXXX：00406261    POPA
XXXX：00406262    JMP             00401000
```

发现这个地方比较可疑，无缘无故的你查什么系统时间阿（`KERNEL32！GetLocalTime`），于是设断点 `BPX 40622F`，重新运行程序，但是发现并没有断到 `40622F` 这里，这是什么原因呢？

猜测一：程序在这之前有一处可以跳过这里直接显示 NAG 对话框

猜测二：程序根本就无法执行到这个检测系统时间的地方

根据我的判断，我选择的是二，就是程序根本就不判断时间，只是一个样式，但是我们为了跳过这个 NAG 只好要让他判断时间试试啦（死马当成活马医吗～～～） ^\_^

在 `406282` 处重新下断点，程序被拦下，下 `A` 指令改变代码，将 `PUSH DWORD 004062E0` 这一句改成 `JMP 406237`，强制使程序检测系统时间，这样程序会自动跳转到 `406237` 处继续执行，取得系统的年份，然后和 1999 对比，大于则出现：This program has expired 的对话框，如果不大于则出现：“This program wont run after 32st may 1999,please register and recive a NAG! free version”，`07CF` 就是 1999 的十六进制值！但是由于我们改变了程序的运行方式，使程序运行到

```
XXXX：0040623D    CMP             WORD [EDI]，07CF
```

这一句时检测的系统年份必定将大于 1999，所以一定会跳到 `00406280` 这里执行，但是我们刚才改变了跳转，程序执行到 `00406282` 的时候还是会重新检查时间，这样就造成了程序的死循环，运行 2 分钟后机器一定会死机的，因此我们修正一下改动方式，

下断点 `BPX 4062E0`，程序被拦下的时候改变语句：`JMP 40624D`，让他直接弹出提示的对话框对话框，然后下断点

`BPX LOCKMYTASK` 点击确定后再次被 `TRW` 拦下，这次应该停在第二个 `MessageBoxA` 函数的后面，就是停在 `406261` 的地方，继续单步跟踪，发现程序只要执行到 `406262` 然后跳到 `00401000` 后就好像回到一个新的 PE 结构的头部一样，马上就是一个 `GetCommandLineA` 函数，这是一般的 PE 格式文件第一个 API 函数，所以如果这个时候你使用 `MakePE` 指令然后在手动修复输入表的话，生成的就完全是一个脱壳的而且没有任何期限的纯净的文件了 ^\_^ 可惜我不会修复输入表 ：（ ◎￥＃％◎＃￥％……￥％……

^\_^ 不过懒人有懒方法，既然不脱壳，那么我们就在程序执行到 `406282` 的时候直接改指令是程序一起把所有的 NAG 都跳过去不久完了吗 ^\_^ 重复上面的步骤（下断点 `BPX 406282`，重新启动程序，被拦下后用 `A` 修改指令，改成 `JMP 401000`）清除断点，返回 Windows 哈哈 程序可以正常的运行了。

知道改的方法后 一切就简单的多了 下面我们来用 `MASM` 编写一个载入器，使程序按照我们的方法运行：

```
.386
.model flat,stdcall
option casemap:none

include C:\masm32\INCLUDE\windows.inc  声明！
include C:\masm32\INCLUDE\kernel32.inc
include C:\masm32\INCLUDE\user32.inc
includelib C:\masm32\LIB\kernel32.lib
includelib C:\masm32\LIB\user32.lib

DlgProc PROTO :DWORD,:DWORD,:DWORD,:DWORD

.data
AppName db 'R'SC CrackMe1 Loader',0       程序标题名称
msg_run db 'Faild to launch program',0Dh,0Ah,'...',0  程序载入错误时候的提示
msg_read db 'ReadProcessMemory error ...',0  防止内存溢出
msg_ver db 'Wrong version ...',0     如果找不到相同的机器码则认为是无效的版本
msg_cap db 'Memory Patcher error ...',0    出错信息对话框标题
msg_write db 'WriteProcessMemory error ...',0

m_progname db 'r!sc.1.exe',0  需要载入的程序名

; 以下为关键代码段
m_addr_to_patch1 dd 00406282h  欲修改的虚拟地址
m_original_bytes1 db 068h, 0E0h, 062h, 040h, 000h    原始机器码 68E0624000
m_new_bytes1 db 0E9h, 079h, 0ADh, 0FFh, 0FFh         修改后的机器码 E979ADFFFF
m_num_of_bytes1 dd 5    一共 5 个字节

DlgName db "CCG",0  在补丁前显示一个对话框，标题为 CCG
TimerID dd 0
TimeOutValue dd 3000       显示 3000 毫秒
m_loop_me dd 10  尝试修改内存次数为 10 次

.data?
hInstance HINSTANCE ?
read_buffer db 512 dup (?)
startinfo STARTUPINFO <>
pi PROCESS_INFORMATION <>

.code
start:
invoke GetModuleHandle,NULL
mov    hInstance,eax
invoke DialogBoxParam,hInstance,addr DlgName,NULL,addr DlgProc,NULL
invoke GetStartupInfo,addr startinfo  载入程序
invoke CreateProcess,addr m_progname,NULL,NULL,NULL,FALSE,20h,NULL,NULL,addr startinfo,addr pi
test eax,eax
jz _launch_error
mov ebx,m_loop_me

_the_loop:  循环修改
invoke ReadProcessMemory,dword ptr [pi],m_addr_to_patch1,addr read_buffer,m_num_of_bytes1,0
test eax,eax
jz _read_proc_error
lea esi,read_buffer  读值并修改
lea edi,m_original_bytes1
mov ecx,m_num_of_bytes1
repz cmpsb
jnz _down
jmp _patch_the_mother

_down:
dec ebx
jnz _the_loop
jmp _wrong_ver

_patch_the_mother:  修改，若出错则跳到提示版本错误的对话框
invoke WriteProcessMemory,dword ptr [pi],m_addr_to_patch1,addr m_new_bytes1,m_num_of_bytes1,0
test eax,eax
jz _write_proc_error
jmp _exit

_launch_error:  无法载入程序时候的提示
invoke MessageBox,NULL,addr msg_run,addr msg_cap,MB_ICONSTOP+MB_OK
jmp _exit

_read_proc_error:  无法修改程序的提示
invoke MessageBox,NULL,addr msg_read,addr msg_cap,MB_ICONSTOP+MB_OK
jmp _exit

_wrong_ver:     没有找到相同的字符串则提示版本错！
invoke MessageBox,NULL,addr msg_ver,addr msg_cap,MB_ICONSTOP+MB_OK
jmp _exit

_write_proc_error:  无法写入进程的提示
invoke MessageBox,NULL,addr msg_write,addr msg_cap,MB_ICONSTOP+MB_OK

_exit:  退出程序，将存放句柄的寄存器清空
invoke CloseHandle,dword ptr [pi]
invoke CloseHandle,dword ptr [startinfo]
invoke ExitProcess,0  调用 ExitProcess 函数退出程序

DlgProc proc uses esi edi ebx,hWnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
    .IF uMsg==WM_INITDIALOG
        invoke SetTimer,hWnd,1,TimeOutValue,NULL
        mov TimerID,eax
    .ELSEIF uMsg==WM_TIMER
            invoke KillTimer,hWnd,TimerID
            invoke SendMessage,hWnd,WM_CLOSE,NULL,NULL
    .ELSEIF uMsg==WM_CLOSE
        invoke EndDialog,hWnd,NULL
    .ELSE
        mov eax,FALSE
        ret
    .ENDIF
    mov eax,TRUE
    ret
DlgProc endp
end start
```

编译后 5K，执行速度超快 ^\_^ `WinMe`，`MASM5` 下测试通过

^\_^ 感谢大家看完 有什么意见欢迎交流！谨以此文献给我们可爱的 `CCG`，希望他能蒸蒸日上（JOJO 老大的话 ^\_^）

娃娃(NYDoll)

属于中国破解组织 CCG(CHiNA CrACKiNG GrOUp) 4dd466d0e1e0cc123afbe504da2462ca
