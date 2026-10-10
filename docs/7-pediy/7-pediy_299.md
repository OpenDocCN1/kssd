

# 给 ACDSee2.44 绿色版来个添油加醋 diy

> 原文：[`www.pediy.com/kssd/pediy12/134232.html`](https://www.pediy.com/kssd/pediy12/134232.html)

ACD 是个很好的东西，我一直保留着很经典的一个版本，版本号是 2.44 绿色版，文件小，速度也快。  
一直用着，发现有个缺点，设置的修改没法保存，打开一些比屏幕宽或屏幕高的文件时，需要手动按 Ctrl+-才能自适应大小。  
好吧，来 PEdiy 一下。用 OD 加载发现加壳了，手动脱壳程序无法正常运行，可能是没脱好。那就不脱壳，直接写汇编吧。  
  
思路：  
1.为 exe 添加新的可执行可写数据段，用来存放新的汇编代码与相关数据。  
2.修改 exe 的 AddressOfEntryPoint 到新添加的数据段。  
3.程序思路：先抓取需要的函数，然后创建一个枚举窗口的线程，等待程序的主窗口加载完成后向窗口发送自适应大小的命令。  
4.发送的 WM\_COMMAND 是 0x4F5E 是跟踪程序时找到的，过程就不写了。  
  
汇编代码在其它汇编工具里写好，编译成 EXE 文件，然后将主要的执行代码写到 exe 中新添加的数据段中。写汇编时，需要注意的就是相对地址的处理。  
为了汇编看上去好看点，可以适当的用宏处理一下。  
  
write by ohuangkeo @ 2011-05-19 23:18:36 \[[原地址](http://blog.ohko.cn/?p=208)\]  
  
\===============  
以下是汇编代码：  
\===============  

引用: .586  
.model flat, stdcall  
option casemap :none  
  
include windows.inc  
include user32.inc  
include kernel32.inc  
  
includelib user32.lib  
includelib kernel32.lib  
  
;计算基地址  
base MACRO  
  call @F  
@@:  
  pop ebp  
  sub ebp, @B  
  add ebp, MYDATA  
ENDM  
  
;push 变量  
myPush MACRO b, a  
  lea ecx, \[b+(a-MYDATA)\]  
  push ecx  
ENDM  
  
;call 变量  
callMy MACRO b, a  
  lea ecx, \[b+(a-MYDATA)\]  
  call DWORD ptr \[ecx\]  
ENDM  
;call 变量  
myCall MACRO b, a  
  mov ecx, \[b+(a-MYDATA)\]  
  call DWORD ptr \[ecx\]  
ENDM  
  
;保存变量  
mySave MACRO b, a, d  
  lea ecx, \[b+(a-MYDATA)\]  
  mov DWORD ptr \[ecx\], d  
ENDM  
    
.CODE  
START:  
  jmp @F  
  MYDATA:  
  \_GetModuleHandle           dd 56AF60h ;绝对调用地址  
  \_GetProcAddress            dd 56AF5Ch ;绝对调用地址  
  \_kernel                    db "Kernel32",00  
  \_kernel\_                   dd ?  
  \_CreateThread              db "CreateThread",00  
  \_CreateThread\_             dd ?  
  \_ThreadID\_                 dd ?  
  \_CloseHandle               db "CloseHandle",00  
  \_CloseHandle\_              dd ?  
  \_Sleep                     db "Sleep",00  
  \_Sleep\_                    dd ?  
  \_GetCurrentProcessId       db "GetCurrentProcessId",00  
  \_GetCurrentProcessId\_      dd ?  
    
  \_user32                    db "User32",00  
  \_user32\_                   dd ?  
  \_PostMessageA              db "PostMessageA",00  
  \_PostMessageA\_              dd ?  
  \_MessageBoxA               db "MessageBoxA",00  
  \_MessageBoxA\_              dd ?  
  \_EnumWindows               db "EnumWindows",00  
  \_EnumWindows\_              dd ?  
  \_GetWindowThreadProcessId  db "GetWindowThreadProcessId",00  
  \_GetWindowThreadProcessId\_ dd ?  
  \_getProcessId              dd ? ;用于保存枚举出的窗口进程 ID  
@@:  
  pushad  
  base  
  
  
  myPush ebp, \_kernel          ;push "Kernel32"  
  myCall ebp, \_GetModuleHandle ;call GetModuleHandle  
  mySave ebp, \_kernel\_, eax    ;mov \_kernel\_  
  mov ebx, eax  
  
  myPush ebp, \_CreateThread       ;push "CreateThread"  
  push ebx  
  myCall ebp, \_GetProcAddress     ;call GetProcAddress  
  mySave ebp, \_CreateThread\_, eax ;mov \_CreateThread\_  
  
  myPush ebp, \_CloseHandle       ;push "CloseHandle"  
  push ebx  
  myCall ebp, \_GetProcAddress    ;call GetProcAddress  
  mySave ebp, \_CloseHandle\_, eax ;mov \_CloseHandle\_  
  
  myPush ebp, \_Sleep          ;push "Sleep"  
  push ebx  
  myCall ebp, \_GetProcAddress ;call GetProcAddress  
  mySave ebp, \_Sleep\_, eax    ;mov \_Sleep\_  
  
  myPush ebp, \_GetCurrentProcessId       ;push "GetCurrentProcessId"  
  push ebx  
  myCall ebp, \_GetProcAddress            ;call GetProcAddress  
  mySave ebp, \_GetCurrentProcessId\_, eax ;mov \_GetCurrentProcessId\_  
  
  
  myPush ebp, \_user32          ;push "User32"  
  myCall ebp, \_GetModuleHandle ;call GetModuleHandle  
  mySave ebp, \_user32\_, eax    ;mov \_user32\_  
  mov ebx, eax  
  
  myPush ebp, \_PostMessageA       ;push "PostMessageA"  
  push ebx  
  myCall ebp, \_GetProcAddress     ;call GetProcAddress  
  mySave ebp, \_PostMessageA\_, eax ;mov \_PostMessageA\_  
  
  myPush ebp, \_MessageBoxA       ;push "MessageBoxA"  
  push ebx  
  myCall ebp, \_GetProcAddress    ;call GetProcAddress  
  mySave ebp, \_MessageBoxA\_, eax ;mov \_MessageBoxA\_  
  
  myPush ebp, \_EnumWindows       ;push "EnumWindows"  
  push ebx  
  myCall ebp, \_GetProcAddress    ;call GetProcAddress  
  mySave ebp, \_EnumWindows\_, eax ;mov \_EnumWindows\_  
  
  myPush ebp, \_GetWindowThreadProcessId       ;push "GetWindowThreadProcessId"  
  push ebx  
  myCall ebp, \_GetProcAddress                 ;call GetProcAddress  
  mySave ebp, \_GetWindowThreadProcessId\_, eax ;mov \_GetWindowThreadProcessId\_  
  
  
  myPush ebp, \_ThreadID\_  
  push 0  
  myPush ebp, thread  
  push 0  
  push 0  
  callMy ebp, \_CreateThread\_  
  
  
  popad  
  mov eax, 0056A001h  
  jmp eax  
  
thread: ;查找窗口的线程  
  pushad  
  
  base  
@@:  
  push 100  
  callMy ebp, \_Sleep\_  
  push 0  
  myPush ebp, enum  
  callMy ebp, \_EnumWindows\_  
  test eax, eax  
  jne @B  
  popad  
  ret  
  
enum: ;枚举查找窗口  
  pushad  
    
  base  
    
  push ebp  
  mov ebp, esp  
  mov ebx, DWORD ptr \[ebp+28h\]  
  pop ebp  
    
  myPush ebp, \_getProcessId  
  push ebx  
  callMy ebp, \_GetWindowThreadProcessId\_  
  lea ecx, \[ebp+(\_getProcessId-MYDATA)\]  
  mov ebx, DWORD ptr \[ecx\]  
    
  callMy ebp, \_GetCurrentProcessId\_  
  cmp eax, ebx  
  jne @F  
  push 0  
  push 4F5Eh  
  push WM\_COMMAND  
  push \[esp+30h\]  
  callMy ebp, \_PostMessageA\_  
  popad  
  xor eax,eax  
  ret  
@@:  
  popad  
  mov eax, 1  
  ret  
  
end START 

\===========================  
再写一段将汇编写到 exe 的 C 代码：  
\===========================  

引用: #include <stdio.h>  
  
int main()  
{  
  char list\[0x400\];  
  
  FILE \*f, \*t;  
  f  = fopen("e:/desktop/hkacd.exe", "rb");  
  t  = fopen("D:/Vsoft/ACDsee2.44.exe.hk", "rb+");  
  
  // 移位  
  fseek(f, 0x200, SEEK\_SET);  
  // 读取汇编代码  
  fread(list, sizeof(char), sizeof(list), f);  
  // 移位  
  fseek(t, 0xAE600, SEEK\_SET);  
  // 写入汇编代码  
  fwrite(list, sizeof(char), sizeof(list), t);  
  
  fclose(f);  
  fclose(t);  
  return 0;  
} 

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || ACDSee2.44-HK-2001.rar  
 |