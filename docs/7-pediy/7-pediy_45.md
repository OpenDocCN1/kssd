

# 动态修改别的进程运行流程【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-238.htm`](https://www.pediy.com/kssd/pediy08/pediy8-238.htm)

动态修改别的进程运行流程（原创）  
  
【研究作者】 盖重阳  
【使用工具】 OllyDbg1.10,VS2003  
【研究平台】 Win2k3  
【软件名称】 自己写的 T.exe  
【研究日期】 2006-5-20  
简介  
  
因为工作需要 我要完成这个功能 所以就研究研究了 昨日研究了一天 已经成功  
现贴出来大家一起分享   
\------------------------------------------------------------------------------------  
原理  
  
和静态修改程序运行流程类似  
动态的就是修改了映射在内存中的代码~  
  
当双击 exe 程序以后 windows 会把 在磁盘上的文件映射到内存中然后开始运行  
只要修改该进程的内存中的运行的代码 就可以实现动态修改  
  
  
问：修改程序流程有什么用  
答：加入自己的代码实现某些功能等  
  
问：动态修改比静态修改有什么好处。  
答：不用考虑壳的检测等....  
\------------------------------------------------------------------------------------  
  
具体实现  
  
以下详细说明 是程序自己修改自己的运行流程   // 注意 是自己进程修改自己的 并不是修改其他进程的  
  
为了简单起见  
  
我用 MFC 创建了一个对话框  
写了一个 Test 按钮 该功能是弹出 MessageBoxA   
  
就从修改 MessageBoxA 开始  
原始的按钮  
  
![](img/eddbafce9888e86aaaf688de04a3ff4a.jpg)   
  
第一步   
要修改内存中的数据 首先要查询要修改的段的权限,然后设置读写权限.  
  
  
od 载入 找到 Test 按钮的 地址  
  
  
![](img/d48bdd032f02acf4439482eb666a8b64.jpg)   
  
  
00401150 t.> .  6A 00             PUSH    0                                                   ; /Style = MB\_OK|MB\_APPLMODAL  
00401152     .  68 58264000       PUSH    t.00402658                                          ; |Title = "test"  
00401157     .  68 58264000       PUSH    t.00402658                                          ; |Text = "test"  
0040115C     .  6A 00             PUSH    0                                                   ; |hOwner = NULL  
0040115E     .  FF15 E4224000     CALL    DWORD PTR DS:\[<&USER32.MessageBoxA>\]                ; \\MessageBoxA  
00401164     .  C3                RETN  
  
  
好 先修改 Title 和 Text 的 数据 看看效果  
  
由于 该数据存在 00402658  所以就先把该段权限设置为 读写  
  
MEMORY\_BASIC\_INFORMATION mbi;  
//PAGE\_EXECUTE\_READ  
VirtualQuery((LPCVOID)0x00402000,&mbi,sizeof(mbi));    //查询.rdata 段   
DWORD dwOLD;  
VirtualProtect((LPVOID)0x00402000,0x1000,PAGE\_EXECUTE\_READWRITE,&dwOLD);  //修改.rdata 段  
    
  
一会这里还有修改运行时的代码 这里也把代码段权限设置了  
  
VirtualQuery((LPCVOID)0x00401000,&mbi,sizeof(mbi));    //查询 .text 段  
    
VirtualProtect((LPVOID)0x00401000,0x1000,PAGE\_EXECUTE\_READWRITE,&dwOLD);  //修改 .text 段  
  
  
\-----------------------------------------------------------------------------  
第二步  
  
先修改 MessageBoxA 内容和标题 练练手  
  
新建一个按钮   
代码如下  
  
修改程序中运行的数据,看看效果  
  \_\_asm  
  {  
    mov eax,0x00402658    //MessageBoxA 的 字符串地址(Test) 上边找到的  
    mov dword ptr \[eax\], 0x30303030    //动态修改数据 修改为 0000  
  }  
好 到这里 已经动态修改内容和标题了  
运行 Test 按钮 发现变成 0000 了 ^\_^   
![](img/3877b564e9086cd0732799943fce6399.jpg)   
  
  
好继续  
  
\-----------------------------------------------------------------------------  
第三步  
  
功能实现：  
  
这次我要让 Test 按钮 点了以后 出现 2 个 MessageBoxA  
多加入一个 MessageBoxA  
  
准备工作：  
  
找到 Test 的按钮地址  
  
![](img/d48bdd032f02acf4439482eb666a8b64.jpg)   
00401150 t.> .  6A 00             PUSH    0                                                   ; /Style = MB\_OK|MB\_APPLMODAL  
00401152     .  68 58264000       PUSH    t.00402658                                          ; |Title = "test"  
00401157     .  68 58264000       PUSH    t.00402658                                          ; |Text = "test"  
0040115C     .  6A 00             PUSH    0                                                   ; |hOwner = NULL  
0040115E     .  FF15 E4224000     CALL    DWORD PTR DS:\[<&USER32.MessageBoxA>\]                ; \\MessageBoxA  
00401164     .  C3                RETN  
  
  
  
我这里要 让程序本身的 MessageBoxA 运行完以后再运行我加入的代码  
  
所以 我就先找一段空地址  我用 od 拉到最下边找到 0x00401fd0 地址   
这里没数据  
我写上代码 调用 MessageBoxA 的代码  
  
push 0  
push 00402658  
push 00402658  
push 0  
  
CALL DWORD PTR DS:\[<&USER32.MessageBoxA>\]   //这里可以 cpy 上边 MessageBoxA 的机器码 到这里粘贴出来就可以了  
  
好 把这一段代码的机器码 cpy 出来   
  
然后把   
00401164 这里修改为 jmp 0x00401fd0 //跳转到我的地址  
复制机器码 留备修改  
  
  
正式修改运行流程：  
  
BYTE codeJmp\[\]={0xE9,0x67,0x0e,0x00,0x00};  // jmp 0x00401fd0   
BYTE codejz\[\]={0x6A,0x00,0x68,0x58,0x26,0x40,0x00,0x68,0x58,0x26,0x40,0x00,0x6A,0x00,0xFF,0x15,0xE4,0x22,0x40,0x00,0xc3,0x90}; //自己的 MessageBoxA  
  
::WriteProcessMemory(GetCurrentProcess(),(LPVOID)0x00401164,(LPCVOID)codeJmp,5,NULL); //改变 00401164 的 ret 为 jmp 0x00401fd0  
  
![](img/6daf8e711c78fa6307a5d25ebdfc9c2d.jpg)   
  
  
::WriteProcessMemory(GetCurrentProcess(),(LPVOID)0x00401fd0,(LPCVOID)codejz,22,NULL); //在 0x00401fd0 写入自己的代码  
  
![](img/5491c10f54554e40912595caa9d3ff4f.jpg)   
  
  
我这里用了 ::WriteProcessMemory 写入 也可以 用 asm 写入 都一样  
  
  
我的程序中还有一个还原的按钮 自己可以看源代码 比较简单  
  
  
\---------------------------------------------------------------------  
第四步  
修改别的进程的运行代码  
  
呵呵 注入了修改~   
  
  
完成   
  
  
\------------------------------------------------------------------  
win2k3  
vs2003 编译成功 修改运行成功   
  
  
题外话：大家别看这我写的省事，当自己修改自己的时候，每次编译时只要添加了代码 数据地址就会跟着变得 自己修改的代码的位置也要更新 另外代码段 也会改变的 注意。 我就费了很多事，还真不如修改别的程序省事 呵呵   
  
  
此方法 比较有局限性 配合 Hook Api 一起使用 较好~  
可以实现 很多功能！~！  

引用:  *最初由** njzzzzzz **发布*  
设置该段 属性如何操作  

  
  
  
  
MEMORY\_BASIC\_INFORMATION mbi;  
//PAGE\_EXECUTE\_READ  
VirtualQuery((LPCVOID)0x00402000,&mbi,sizeof(mbi));    //查询.rdata 段   
DWORD dwOLD;  
VirtualProtect((LPVOID)0x00402000,0x1000,PAGE\_EXECUTE\_READWRITE,&dwOLD);  //修改.rdata 段的属性  
  
  
一会这里还有修改运行时的代码 这里也把代码段权限设置了  
  
VirtualQuery((LPCVOID)0x00401000,&mbi,sizeof(mbi));    //查询 .text 段  
  
VirtualProtect((LPVOID)0x00401000,0x1000,PAGE\_EXECUTE\_READWRITE,&dwOLD);  //修改 .text 段的属性  
  
  
VirtualProtect 设置属性的 api 

 4dd466d0e1e0cc123afbe504da2462ca