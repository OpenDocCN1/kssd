
# 对抗 OD 内存断点

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-287.htm`](https://www.pediy.com/kssd/pediy09/pediy09-287.htm)
>
> 10 月 11 号提前版，纪念“堕落天才”这个 ID 注册一周年。
>
> （标题看起来可能很诱人，内容却很简单，希望你不会太失望）

## 1. OD 内存断点原理

A, 内存访问断点，OD 将目标内存所在的页面（范围圆整为 `1000h` 的倍数）设置为 `PAGE_NOACCESS`，当被调试程序对这个内存进行任何“读、写或运行”操作时，都会触发异常。

B, 内存写入断点，OD 将目标内存所在的页面（范围圆整为 `1000h` 的倍数）设置为 `PAGE_EXECUTE_READ`，当被调试程序对这个内存进行“写”操作时触发异常。

（原来都不是传说中的 `PAGE_GUARD`? 太惊讶了。）

## 2. 反 OD 内存断点原理

要知道 OD 可以通过 `VirtualProtectEx` 改变内存页面属性，我们当然也可以修改，这样我们就可以发现内存断点或使其失效。

## 3. 代码 1

代码:

```asm
.386
.model flat,stdcall
option casemap:none

include windows.inc
include kernel32.inc
include user32.inc

includelib kernel32.lib
includelib user32.lib


.data
szAppName  db "Anti memory break point - By 堕落天才",0
szText     db "Hi,try to set a memory break point here!",0
szFound    db "A memory break point was found!",0

.data?
dwOldProtect dd ?

.code     
       
\_start:                   
      
    invoke lstrlen,addr szText
    inc eax   
             
    invoke VirtualProtect,addr szText,eax,PAGE_READWRITE,addr dwOldProtect    
  
    .if eax == FALSE
        
          invoke GetLastError 
          .if eax == ERROR_NOACCESS
             
             invoke MessageBox,NULL,addr szFound,addr szAppName,MB_OK
             jmp @ExitMain
                            
          .endif
          
    .endif   
    
    invoke MessageBox,NULL,addr szText,addr szAppName,MB_OK     
  
  @ExitMain:     
    invoke ExitProcess,0        
  
  

end \_start   
```

用 OD 载入附件中 `AntiMBP_ASM.exe`，对 `00403026`-`00403057`(`szText`)内存范围下内存访问断点，`F9` 运行，很快就中断了，是在 `lstrlen` 的内部。一直 `F9`，如无意外程序将会显示“A memory break point was found!”提示框(在我的电脑里面是这样)。因为 `lstrlen` 后面的 `VirtualProtect` 试图改变 `szText` 所在内存范围的属性，失败后，`LastError` == `ERROR_NOACCESS`，这样我们就知道 `szText` 被设置内存断点了。

## 4. 代码 2

代码:

```cpp
#include<windows.h>

char szAppName [] = "Anti memory break point - By 堕落天才";
char szFound[] = "A memory break point was found!";
char szText[] = "Hi! try to set a memory break point here! Success?";

int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nShowCmd )
{
    DWORD dwOldProtect;
  int nLen = lstrlen( szText );
  nLen++;
  if(! VirtualProtect( &szText,nLen,PAGE_READWRITE,&dwOldProtect ) ){
          
    if(GetLastError() == ERROR_NOACCESS ){
      MessageBox( NULL,szFound,szAppName,MB_OK );
    }

  }else{
              MessageBox(NULL,szText,szAppName,MB_OK);
  }
      
  return 0;
} 
```

用 OD 载入附件中的 `AntiMBP_CPP.exe`，对 `00405078`-`004050A9`(`szText`)内存范围下内存访问断点，`F9` 运行，一样很快就中断在 `lstrlen` 内部。一直 `F9`，如无意外程序将会显示“Hi! try to set a memory break point here! Success?”提示框(在我的电脑上是这样)。因为 `lstrlen` 后面的 `VirtualProtect` 试图改变 `szText` 所在内存范围的属性，成功后，下面的 `MessageBox` 函数对 `szText` 读取，不会再中断，这样 `szText` 上面设置的内存断点就失效了。

## 5. 最后

上面两段代码几乎一摸一样，我不知为什么会有两个结果，不过无论是发现还是使其失效，我们反 OD 内存断点的目标已经达到。

## 6. 反-反 OD 内存断点

对 `VirtualQuery`，`VirtualQueryEx`，`VirtualProtect`，`VirtualProtectEx` 下断点，应该很快就能到达反内存断点的地方。

***

“堕落天才”这个 ID 注册快一年了，认识看雪论坛也有一年了。一年前，不会汇编、不会驱动连 OD 都不知为何物，后来有少少发现就到论坛上灌，真是“不知自己无知”。半年前，终于“知道自己无知”，不敢再乱灌，就沉默了下来。一年来，在看雪上学到了很多，借烂文一篇感谢一下。

`4dd466d0e1e0cc123afbe504da2462ca`
