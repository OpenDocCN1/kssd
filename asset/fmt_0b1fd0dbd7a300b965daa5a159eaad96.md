
# 代码段 1

```
00EF6EF3 E8 54D70000 call <jmp.&HLK\_dll.#15\_hlk\_ReadDong> ;再读
00EF6EF8 85C0 test eax,eax ;判断返回值，如是 0
00EF6EFA 0F84 87000000 je user\_dll.00EF6F87 ;OVER
00EF6F00 B9 2848F100 mov ecx,user\_dll.00F14828 ;判断 ECX 是否为 0，跟踪发现该值是狗的编号
00EF6F05 85C9 test ecx,ecx ;是 0 与否，不影响程序运行
00EF6F07 74 2B je short user\_dll.00EF6F34
00EF6F09 8D55 98 lea edx,ss:\[ebp-68\] ;该地址存放狗的编号
00EF6F0C 85D2 test edx,edx
00EF6F0E 74 16 je short user\_dll.00EF6F26
00EF6F10 6A 06 push 6
00EF6F12 8D45 98 lea eax,ss:\[ebp-68\]
00EF6F15 50 push eax
00EF6F16 68 2848F100 push user\_dll.00F14828
00EF6F1B FF15 E875F000 call ds:\[<&MSVCR71.strncpy>\] ; MSVCR71.strncpy
```

## 11. 绕过狗检查

聪明的你肯定知道该做什么了，对，只要把 `00EF6ECF`、`00EF6EE6`、`00EF6EFA` 三处的 `je` 改为 `jne` 即可跳过狗的检查。我毫不犹豫地就改了，运行，然而并没有出现预想的兴奋的事情。出现一个错误提示“没有发现可选文件”。

## 12. 可选文件分析

“可选文件”是什么意思？到软件安装目录查看，有个 `OPTION` 文件夹，里面有很多文件。用 `UltraEdit` 打开，看不懂。

## 13. 狗数据与解密

将改文件夹中的所有文件删除，带狗运行，出现和 `11` 步同样的提示；恢复文件，运行正常。由此可见，解密那些文件需要狗中的数据。大家知道 `HASP` 狗用的是 `AES`，难道狗中存放着 `AES` 密码？？带着这样的疑问跟踪程序，来到下面代码：

```
00EF723F 8B8D 68F6FFFF mov ecx,ss:\[ebp-998\] ;压入的三个参数意义就不说明了，因为没必要
00EF7245 51 push ecx
00EF7246 8D95 64F8FFFF lea edx,ss:\[ebp-79C\]
00EF724C 52 push edx
00EF724D 8D85 BCFAFFFF lea eax,ss:\[ebp-544\]
00EF7253 50 push eax
00EF7254 E8 47180000 call user\_dll.00EF8AA0 ;该函数执行完后，EDX 会出现一些明码，明码信息
```

大概为 `APPLICAITON=XXX;OPTION=XXX.....` 是不是很像文件启动需要检查的选项？所以要弄懂解密算法，必须跟进。

## 14. 解密过程追踪

跟进后，部分代码如下，这部分代码很关键，所以贴的比较多。但是有很多是内核函数，很容易读懂。

```
00EF8B30 8D8D 68FDFFFF lea ecx,ss:\[ebp-298\]
00EF8B36 51 push ecx
00EF8B37 8B55 0C mov edx,ss:\[ebp+C\]
00EF8B3A 83C2 2C add edx,2C
00EF8B3D 52 push edx
00EF8B3E 68 5038F100 push user\_dll.00F13850
00EF8B43 E8 14BD0000 call <jmp.&XXX\_DLL.#100\_ExtendPathName> ; 获取文件完整路径
00EF8B48 6A 00 push 0
00EF8B4A 8D85 70FFFFFF lea eax,ss:\[ebp-90\]
00EF8B50 50 push eax
00EF8B51 8D8D 68FDFFFF lea ecx,ss:\[ebp-298\]
00EF8B57 51 push ecx
00EF8B58 E8 63BC0000 call <jmp.&XXX\_DLL.#47\_MakeSString>
00EF8B5D 50 push eax
00EF8B5E FF15 0872F000 call ds:\[<&KERNEL32.OpenFile>\] ; 打开文件
00EF8B64 8985 54FDFFFF mov ss:\[ebp-2AC\],eax
00EF8B6A 83BD 54FDFFFF FF cmp dword ptr ss:\[ebp-2AC\],-1
00EF8B71 74 55 je short user\_dll.00EF8BC8
00EF8B73 8B95 50FDFFFF mov edx,ss:\[ebp-2B0\]
00EF8B79 52 push edx
00EF8B7A 8B85 60FDFFFF mov eax,ss:\[ebp-2A0\] ;存放读入数据的缓冲区，关键（后面破解用上）
00EF8B80 50 push eax
00EF8B81 8B8D 54FDFFFF mov ecx,ss:\[ebp-2AC\]
00EF8B87 51 push ecx
00EF8B88 FF15 F071F000 call ds:\[<&KERNEL32.\_hread>\] ; 读文件，ebp-2A0 为存放读入数据的缓冲区
00EF8B8E 3B85 50FDFFFF cmp eax,ss:\[ebp-2B0\]
00EF8B94 75 19 jnz short user\_dll.00EF8BAF
00EF8B96 8B55 10 mov edx,ss:\[ebp+10\]
00EF8B99 52 push edx
00EF8B9A 8B85 64FDFFFF mov eax,ss:\[ebp-29C\]
00EF8BA0 50 push eax
00EF8BA1 8B8D 60FDFFFF mov ecx,ss:\[ebp-2A0\] ;存放解密后的数据，和读入缓冲区地址相同
00EF8BA7 51 push ecx
00EF8BA8 E8 B1BA0000 call <jmp.&HLK\_dll.#12\_hlk\_Crypt> ;对读入的数据进行解密，须弄懂算法
00EF8BAD EB 0A jmp short user\_dll.00EF8BB9
00EF8BAF C785 64FDFFFF 00000000 mov dword ptr ss:\[ebp-29C\],0
00EF8BB9 8B95 54FDFFFF mov edx,ss:\[ebp-2AC\]
00EF8BBF 52 push edx
00EF8BC0 FF15 F471F000 call ds:\[<&KERNEL32.\_lclose>\] ; 关闭文件
00EF8BC6 EB 0A jmp short user\_dll.00EF8BD2
00EF8BC8 C785 64FDFFFF 00000000 mov dword ptr ss:\[ebp-29C\],0 ;后面的函数不做解释了，只需知道解密后的
00EF8BD2 8B85 60FDFFFF mov eax,ss:\[ebp-2A0\] ;数据存放在 ebp-2A0
00EF8BD8 0385 64FDFFFF add eax,ss:\[ebp-29C\]
00EF8BDE C600 00 mov byte ptr ds:\[eax\],0
00EF8BE1 8B4D 08 mov ecx,ss:\[ebp+8\]
00EF8BE4 8B95 60FDFFFF mov edx,ss:\[ebp-2A0\]
00EF8BEA 8911 mov ds:\[ecx\],edx
00EF8BEC 8B85 64FDFFFF mov eax,ss:\[ebp-29C\]
00EF8BF2 8B4D FC mov ecx,ss:\[ebp-4\]
00EF8BF5 E8 E8C30000 call user\_dll.00F04FE2
00EF8BFA 8BE5 mov esp,ebp
00EF8BFC 5D pop ebp
00EF8BFD C3 retn
```

## 15. 利用自身程序解密

为了得到解密后的数据，跟进 `00EF8BA8` 处的 `call <jmp.&HLK\_dll.#12\_hlk\_Crypt>`，看看到底是什么算法。可是乐观的情绪又受打击，该函数有大量的花指令，菜鸟阶段，不懂如何去除花指令，研究肯定不是一天两天的事。所以不能在一棵树上吊死，换个思维，为什么不能用程序本身的代码进行解密呢！！！

对，就让程序自身帮我们干活！说干就干，在步骤 `13` 的 `00EF723F` 处设断，反复运行程序，每执行一次查看 `ebp-2A0` 处的值，并 dump 出来，另存为 16 进制文件，这些文件就是解密过的文件，将其覆盖原来的加密文件。剩下要做的就是跳过上面代码中的解密模块（`\_hlk\_Crypt`）。

这个很简单，来到 `00EF8B8E` 处，即程序刚进行完 `\_hread` 之后，将该处的代码 `cmp eax,ss:\[ebp-2B0\]` 改为 `jmp 00EF8BB9` 然后汇编。由于 `\_hread` 函数和 `\_hlk\_Crypt` 函数用的是同一个缓冲区，所以 `\_hread` 函数执行完毕后，`ebp-2A0` 处的已解密数据就可顺利为后续程序使用了。

## 16. 启动成功

经过上面修改后，按 `F9`，程序顺利启动！！！！

【继续完善】

## 17. 消除定时检查

程序运行一段时间后，弹出对话框，通知确定狗的存在！！看来程序启动中还是要定时检查的。

*   **第一个念头**：查找 `SetTimer` 函数，果然找到，且只有一次调用，心想就是这个函数在作怪了，该函数的 `uElapsed` 值为 `1` 秒，我将其改为 `9A7EC800`，即 `30` 天。运行，可是仍然定时检查狗的存在。
*   **第二个念头**：查找其他有关时间的函数，但感觉有点费劲，进而转第三个念头。
*   **第三个念头**：以其他的函数设断点，如大家常用的 `CreateFile`、`DeviceIoControl` 等，但没用过，感觉肯定不是跟踪一两句代码就可搞定的事情，进而转第四个念头。
*   **第四个念头**：一劳永逸的想法，从源头解决问题。为什么不直接修改读狗函数的返回值？这样程序无论何时检查狗都无所谓。

重新跟进 `hlk\_LogIn`、`hlk\_Avail`、`hlk\_ReadDong` 三个函数，发现这三个函数返回前都有共同的代码：

```
00B5126C 837D FC 00 cmp dword ptr ss:\[ebp-4\],0 ;看 ebp-4 是否为 0
00B51270 0F94C0 sete al ;是，则 eax＝1，有狗
00B51273 8B4D F4 mov ecx,ss:\[ebp-C\]
00B51276 E8 C6640400 call HLK\_dll.00B97741 ;该 CALL 并不是真正函数，经验证不影响上面的 EAX 值
00B5127B 8BE5 mov esp,ebp
00B5127D 5D pop ebp
00B5127E C2 0C00 retn 0C
```

不插狗的情况下 `\[ebp-4\]` ＝ `9`，因此 `eax` ＝ `0`，无狗，因此只需将 `sete al` 改为 `setne al`，汇编，即可。

至此，打狗结束，程序非常正常。将修改后的文件另存到可执行文件，作为补丁，覆盖原文件，狗解除。

## 后记

由于这是本人正式调试的第一个大型程序，也是第一次写破文，中间走了不少弯路，将调试心得写出来，作为后记，总结经验。

1.  动态调试前要充分发挥 `Win32Dasm` 等静态反汇编工具的作用，挖掘蛛丝马迹。如字符串提示、对话框提示等。
2.  对于用狗加密的商业软件，上面的提示可能比较难于找到，但可以先浏览安装目录，看一看都有那些 `dll`，能从名字猜出含义更好，尤其关注 `HLK` 等与狗相关的字样（偶接触狗不多，所以也总结不出多少狗的名字呵呵，只是要有个意识）。
3.  有了第二步的信息之后，运用 `Win32Dasm` 等工具先查看可疑 `dll` 的输入输出表，看看之间的相互调用关系，尤其关注读狗 `dll` 是哪个模块调用的，这会大大节省搜索读狗函数的时间。
