

# 功能限制的程序

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-5.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-5.htm)

**功能限制的程序 **　**【习题】**

　　这种程序一般是 DEMO 版或菜单中部分选项是灰色。有些 DEMO 版本的部分功能里面根本就没有。而有些程序功能全有，只要注册后就正常了。

　　你使用这些 DEMO 程序部分被禁止的功能时，会跳出提示框，说这是 DEMO 版等话，它们一般都是调用**MessageBox\[A\]** 或 **DialogBox\[A\]**等函数。你可在 W32DASM 反汇编它，一般能找到如下字符串："Function Not Avaible in Demo" 或 "Command Not Avaible" 或 "Can't save in Shareware/Demo"等，这些 CALL 会被相应的调用，可作为你破解的一指示器。

　　另外，就是菜单中部分选项是灰色的不能用，一般它们是通过如下两种函数实现的： 

（1）EnableMenuItem

| 允许、禁止或变灰指定的菜单条目  |
| --- |
|  BOOL EnableMenuItem(  
HMENU hMenu, // 菜单句柄  
UINT uIDEnableItem, // 菜单 ID，形式为：充许,禁止,或灰  
UINT uEnable //菜单项目旗帜  
);  
Returns  |
| 在 ASM 代码形式如下:  
PUSH uEnable 　　　//uEnable=0 则菜单选项允许  
PUSH uIDEnableItem  
PUSH hWnd  
CALL \[KERNEL32!EnableMenuItem\] |

  
（2）EnableWindow  

| 允许或禁止鼠标和键盘控制指定窗口和条目（禁止时菜单变灰）  |
| --- |
| BOOL EnableWindow(   
HWND hWnd, // 窗口句柄  
BOOL bEnable // 允许/禁止输入  
);  
**Returns**  
如窗口以前被禁止则返回一 TRUE，否则返回 FALSE。 |

 

 4dd466d0e1e0cc123afbe504da2462ca