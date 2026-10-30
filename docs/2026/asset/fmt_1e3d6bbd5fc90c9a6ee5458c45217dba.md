
整个看起来有点长，尾部的`security_check_cookie`函数附近可以看到：

```
.text:1000B929 8B 8C 24 94 00 00 00                    mov     ecx, [esp+98h+var_4]
```

此处`[esp+98h+var_4]`保存的就是当前的返回地址数值了（或许被改变）。

通过阅读代码我们发现：

```
.text:1000B919 8D 54 24 10                             lea     edx, [esp+98h+var_88]

.text:1000B91D 52                                    push    edx //将局部变量地址压栈，大概是string类型变量

.text:1000B91E 8D 8E F8 00 00 00                       lea     ecx, [esi+0F8h]

.text:1000B924 E8 DB 87 00 00                         call    sub_10014104 
```

所以看一下`sub_10014104`吧：

```
.text:10014104 56                                      push    esi

.text:10014105 6A 15                                   push    15h       //传递的参数15h，此时有没有想到些什么

.text:10014107 FF 74 24 0C                             push    [esp+8+arg_0] //传递的局部变量地址

.text:1001410B 8B F1                                   mov     esi, ecx

.text:1001410D E8 A3 9B FF FF                          call    sub_1000DCB5

.text:10014112 59                                      pop     ecx

.text:10014113 59                                      pop     ecx

.text:10014114 50                                      push    eax             ; Str

.text:10014115 8D 8E B0 00 00 00                       lea     ecx, [esi+0B0h]

.text:1001411B E8 10 64 FF FF                          call    sub_1000A530

.text:10014120 5E                                      pop     esi

.text:10014121 C2 04 00                                retn    4
```

继续查看`sub_1000DCB5`：

```
.text:1000DCB5 8B 44 24 04                             mov     eax, [esp+arg_0]  //传递的局部变量地址

.text:1000DCB9 66 8B 54 24 08                          mov     dx, [esp+arg_4]   //传递的参数15h

.text:1000DCBE 66 31 10                                xor     [eax], dx

.text:1000DCC1 56                                      push    esi

.text:1000DCC2 66 8B 30                                mov     si, [eax]

.text:1000DCC5 8B C8                                   mov     ecx, eax

.text:1000DCC7 5E                                      pop     esi

.text:1000DCC8 74 0B                                   jz      short locret_1000DCD5

.text:1000DCCA

.text:1000DCCA                         loc_1000DCCA:                           ; CODE XREF: sub_1000DCB5+1Ej

.text:1000DCCA 41                                      inc     ecx         //堆栈增长

.text:1000DCCB 41                                      inc     ecx

.text:1000DCCC 66 31 11                                xor     [ecx], dx //堆栈中的WORD数值与0x15H异或

.text:1000DCCF 66 83 39 00                             cmp     word ptr [ecx], 0

.text:1000DCD3 75 F5                                   jnz     short loc_1000DCCA

.text:1000DCD5

.text:1000DCD5                         locret_1000DCD5:                        ; CODE XREF: sub_1000DCB5+13j

.text:1000DCD5 C3                                      retn    
```

于是看到这里可以猜测在当前函数栈空间没有`0x15`数值来与`0x15H`异或结束循环，所以便溢出到上层栈空间了。

通过windbg的栈回溯我们看到大量的返回地址中含有`0015`，可以断定是这个函数的问题了。

当然可以用OD验证一下，事实上验证的结果与此一致！函数之前通过异或`15`解密了两个网址，最后估计是打算重新加密结果却忽略了结束标识，不知有意还是无意（如果是有意的却真是没有技术含量了，算不上合格的病毒），这里就不说了，题外话。

到了现在我的桌面还没出来呢，怎么办？搜索注册表看看吧，先搜索一下模块名，找到：

```
HKEY_CLASSES_ROOT\CLSID\{12CE2CCC-53FD-4855-80D9-B2F4F96C5DAF}\InprocServer32

C:\Program Files\Common Files\Microsoft Shared\bg\BGCloudSH.1.0.0.1.(750).dll
```

接着搜索对应的CLSID找到：

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers\bgcontrol

{12CE2CCC-53FD-4855-80D9-B2F4F96C5DAF}
```

基本上就这两个关键处了吧，当然还要百度一下模块看看具体情况，结果发现已经有人遇到过并且分析了，呵呵！

删除文件和注册表项重新运行`explorer`，我的桌面又回来了！

本想上传样本，但是发现该样本具有有效的数字签名：

![](https://bbs.kanxue.com/upload/attach/201310/546812_x22c9bj401js1ys.png)

只好先发邮件确认一下！

以上就是一次简单的应用程序错误分析，因为发生在自己的机器上就顺便做个笔记。

附件为溢出时`EXPLORER`转储文件，样本等一天后若没有回复便上传！

收到邮件回复，证书所有者称并非他们的模块，因此上传样本。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [image001.png](javascript:void(0);)（10.53kb，5次下载）
- [image002.png](javascript:void(0);)（4.35kb，7次下载）
- [image003.png](javascript:void(0);)（4.76kb，3次下载）
- [image004.png](javascript:void(0);)（6.10kb，3次下载）
- [PID-672__EXPLORER.EXE__full.part01.rar](javascript:void(0);)（2.50MB，38次下载）
- [PID-672__EXPLORER.EXE__full.part02.rar](javascript:void(0);)（2.50MB，59次下载）
- [PID-672__EXPLORER.EXE__full.part03.rar](javascript:void(0);)（2.50MB，49次下载）
- [PID-672__EXPLORER.EXE__full.part04.rar](javascript:void(0);)（2.50MB，27次下载）
- [PID-672__EXPLORER.EXE__full.part05.rar](javascript:void(0);)（2.50MB，18次下载）
- [PID-672__EXPLORER.EXE__full.part06.rar](javascript:void(0);)（1.38MB，50次下载）
- [样本BGCloudSH.1.0.0.1.(750).rar](javascript:void(0);)（163.30kb，21次下载）
