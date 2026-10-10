

# 几种典型程序 Button 处理代码的定位

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-381.htm`](https://www.pediy.com/kssd/pediy08/pediy8-381.htm)

首先  
1 od 下运行程序，F12 暂停；  
2 View 菜单中选击 Windows 项，在打开的窗口中可以从 Title 栏看到目标按钮，从而找到它的 Handle(xxxxxxxx) ;  
  
对不同平台生成的程序，分别处理：  
  
一、VB, Delphi, CBuilder 程序：  
3 在 CallWindowProcA 入口下条件断点: \[esp+8\]==xxxxxxxx && \[esp+0c\]==202；  
4 F9 继续程序，点击目标按钮，程序中断；  
5 Alt+F4，在代码段（.text）上下访问断点；  
6 F9 执行程序，程序中断,  
   1). VB 程序中断在下面代码  
     PUSH DWORD PTR DS:\[EAX+EBX\]              ; yyyyyyy  
     上面\[EAX+EBX\]的值（yyyyyy）就是我们要找的位置。  
  
   2). Delphi, CBuilder 的程序  
     程序直接断在我们要找的位置。  
  
借 moon 一句，这姑妄称作 CallWindowProcA 条件断点加上 code 段内存断点法吧。![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)   
  
二、VC 程序  
又分 MFC 和 Win32 两种情况，二者相同之处：  
3 在 IsDialogMessageW 入口下条件断点: \[\[esp+8\]\]==xxxxxxxx && \[\[esp+8\]+4\]==202  
4 F9 继续程序，点击目标按钮，程序中断；  
5 Alt+F4，在代码段上下访问断点；  
6 F9 执行程序，程序中断, 注意这里虽然中断在 code 段，但却不是处理 Button 点击事件的代码处  
  这时：  
  对 Win32 程序，只需要按几下 F7，当回到 User32.dll 领空后再重复一次第 5、6 步就可以了；  
  而对于 MFC 程序，我们不得不多次重复这样的操作：单步回到 MFC 领空，再第 5、6 步。好在已经看到大陆啦！  
  
类比，这就叫 IsDialogMessageW 条件断点加上 code 段内存断点法了。![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)   
  
敬请各位指正。 

bp IsDialogMessageW \[\[esp+8\]\]==00106AE && \[\[esp+8\]+4\]==202  
  
\====>  
  
bp IsDialogMessageW \[\[esp+8\]\]==00106AE && MSG==202  
  
:)