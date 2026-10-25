
```asm
.code
MyHook proc
cmp r8,1ED39h
jnz Exit3
cmp r14,1ED39h
jnz Exit3 ; r8 == 1ED39h && r14 == 1ED39h 确定目标 如不是直接跳走
; 保存环境 太长删了
mov byte ptr [rbp-40h],40h ;进入正确流程后 先修改对象属性
mov byte ptr [rbp-38h],41h
jmp Signature ;重定位获得特征码地址
RelocationCode:
mov r10,rdx ;r10 保存目标代码地址
mov rcx,rdx
pop rdx ;strstr的参数
cld
Strstr: ;strstr功能 找到特征码的位置
cmp byte ptr [rdx], 0
mov r9, rcx
jnz lable1
mov rax, rcx
jmp Exit3
lable1:
cmp byte ptr [rcx], 0
jz lable5
mov r8, rcx
sub r8, rdx
lable2:
mov rax, rdx
lable3:
movzx ecx, byte ptr [rax]
test cl, cl
jz lable6
cmp [r8+rax], cl
jnz lable4
inc rax
cmp byte ptr [r8+rax], 0
jnz lable3
lable4:
cmp byte ptr [rax], 0
jz lable6
inc r9
inc r8
cmp byte ptr [r9], 0
jnz lable2
lable5:
xor eax, eax
lable6:
mov rax, r9
cmp rax,0
jz Exit3 ;没找到特征码就退出 当然不太可能
CalcSize:
mov rsi,rax
lea rsi,[rsi + 1Ah]
mov r15,rsi ;r15保存写入地址
mov r11,rsi
sub r11,r10
mov r12,1ED39h
sub r12,r11
mov rax,r12
cqo ;符号扩展到rdx
mov r13,08h
div r13d
mov rcx,rax ;循环移动次数，每次8字节 给patch 7个字节的空间
MoveCode:
lodsq ;读8个字节到rax rsi + 8
lea r14,[rsi - 1] ;得到第一次写的位置
mov rdx,rax ;保存第一次要写的数据
Next:
lodsq ;得到下8字节的数据到rax rsi+8 就这样保持读正常
mov qword ptr [r14],rdx ;第一次写入数据 此时下一次要写的数据在rax 下一次要写的位置应该为 r14+8
add r14,8 ;r14 + 8
mov rdx,rax ;第二次要写的数据保存到rdx 就这样保持写正常
dec ecx
jnz Next ;循环直到ecx为0
mov word ptr [r14],ax ;最后数据不足8个 则把读到的数据直接写入即可
WritePatch:
mov dword ptr [r15],72743D73h ;这里是patch数据
mov word ptr [r15+4],6575h
mov byte ptr [r15+6],3Bh
jmp UnHook
Exit:
;恢复环境 太长删了
mov r8,qword ptr [rbp-40h]
jmp Exit3
Exit2:
dq 1234567812345678h ;这里需要原函数地址
Exit3:
jmp qword ptr [Exit2]
Signature:
call RelocationCode
db 63h,6fh,6eh,73h,74h,20h,51h,3dh
dq 0h
UnHook:
mov rax,1234567812345678h ;这里需要导出表地址，恢复hook位置
mov ebx,12345678h ;这里需要原导出函数RVA
mov dword ptr [rax],ebx
jmp Exit
dq 0
MyHook endp
end
```

### 调试与完善

好吧，经过一段时间的代码编写，我们的破解程序已经出炉了，调试的话，有兴趣的网友可以自行编写调试，Shellcode 中有两个简单实现的算法，`strstr` 与 挪动数据

然后我还为程序添加了图标与 隐藏控制台，算是比较不错的一个程序了，并且我写的代码具有一定的通用性，应该可以实现全版本通吃，但是我没有测试，可能还有一些细节需要完善，例如特征码的查找与关键调用的判断

我测试了最新的 `1.25`版本，没有问题，下面给出适合新手的PE注入实现

# 信手拈来 ：PE注入

### 前期分析

众所周知，PE中的`text`段就是代码段，我们可以在`text`段中找到一个空闲的位置，来把`s`hellcode放在其中

那么需要解决几个问题

1.空闲位置的RVA填入导出函数地址表
2.`shellcode`代码的十六进制形式
3.空闲位置需要有足够的位置存放`s`hellcode
4.随机基址，如果PE注入，那么在跳回原函数的时候，需要它的位置是固定的（当然，如果有足够的大小，可以让Shellcode自定位）

### 开始行动

1.根据分析的4点，首先解决第4点，关闭PE的随机基址属性位 ， `010editor`打开 `typora.exe` （把文件属性的只读去掉）

![](upload/attach/202205/930159_5FVV6XZTNRRTMXE.png)

`010editor`有PE格式解析脚本，找到选项头中的属性，把这一项改为0即可

2.找到空闲的`text`段位置，这里我找到了`71B4A50`的位置，大小刚好够存放我的`s`hellcode

![](upload/attach/202205/930159_DDNRUSBSW4WH8G7.png)

3.注入代码的位置的FOA转RVA

可以使用PE工具来计算，也可以自己编写程序计算 ，我使用看雪资源下载中的PE编辑器，可以直接下载，这里得到了RVA ，`71B5650`

![](upload/attach/202205/930159_VMRMVKAP7QASTEC.png)

4.在导出函数地址中修改为这个RVA

我在上面的程序中已经找到了这个位置，FOA为`0x8602fc5` ，来到文件中的这个位置，写入`s`hellcode 的RVA，小端序写入 `071B5650`

![](upload/attach/202205/930159_48DDRB85V7WEXA2.png)

5.然后需要修改一下`s`hellcode，并转换为机器码

把原函数地址写入`s`hellcode，通过计算得出 `1426FB7A0` 为原函数地址（原函数RVA转VA）

调试查看一下，RVA修改成功，并且来到该地址，接下来把`s`hellcode的十六进制复制到PE文件中即可

![](upload/attach/202205/930159_F4J8XEBWXQSVZAT.png)

把Shellcode机器码从调试器中拷贝出来，放到自己破解的`010editor`中（新版有反汇编功能）

![](upload/attach/202205/930159_5CVKFJP3MJGFVCV.png)

有一说一 ，还挺好用的，然后把原函数地址 填写到跳转的位置 , 这是我修改后的`s`hellcode

```
49 81 F8 39 ED 01 00 0F 85 05 01 00 00 49 81 FE
39 ED 01 00 0F 85 F8 00 00 00 50 53 51 52 55 54
56 57 41 50 41 51 41 52 41 53 41 54 41 55 41 56
41 57 C6 45 C0 40 C6 45 C8 41 E9 D9 00 00 00 4C
8B D2 48 8B CA 5A FC 80 3A 00 4C 8B C9 75 08 48
8B C1 E9 BB 00 00 00 80 39 00 74 31 4C 8B C1 4C
2B C2 48 8B C2 0F B6 08 84 C9 74 23 41 38 0C 00
75 0A 48 FF C0 41 80 3C 00 00 75 E9 80 38 00 74
0E 49 FF C1 49 FF C0 41 80 39 00 75 D5 33 C0 49
8B C1 48 83 F8 00 74 7A 48 8B F0 48 8D 76 1A 4C
8B FE 4C 8B DE 4D 2B DA 49 C7 C4 39 ED 01 00 4D
2B E3 49 8B C4 48 99 49 C7 C5 08 00 00 00 41 F7
F5 48 8B C8 48 AD 4C 8D 76 FF 48 8B D0 48 AD 49
89 16 49 83 C6 08 48 8B D0 FF C9 75 F0 66 41 89
06 41 C7 07 73 3D 74 72 66 41 C7 47 04 75 65 41
C6 47 06 3B 41 5F 41 5E 41 5D 41 5C 41 5B 41 5A
41 59 41 58 5F 5E 5C 5D 5A 59 5B 58 4C 8B 45 C0
EB 00 FF 25 15 00 00 00 E8 22 FF FF FF 63 6F 6E
73 74 20 51 3D 00 00 00 00 00 00 00 00 A0 B7 6F
42 01 00 00 00
```

![](upload/attach/202205/930159_VPHKUFAAVHUGZX2.png)

好的，保存即可

# 使用方法

1.内存破解程序为 `Crack_Typora.exe` ，把它放在Typora的目录，双击打开即可
2.PE注入破解程序为`Typora_Patch.exe` ，把它放在Typora的目录，双击打开即可

**注：你还可以把它两发送到桌面快捷方式 更为方便快捷**

![](upload/attach/202205/930159_XEGDTP54KPXRRZ2.png)

# 完结撒花

![](upload/attach/202205/930159_5AKBSR2MDA4YDTC.png)

# 结语

每个人都有很多想法，但是真正去用行动实现的却只有少数人，而我，就是那少数人。

**注：经过大哥的提醒，防止我的程序被恶意传播，所以已经取消了附件下载链接，望谅解**

# 前言

我是逆向练习生，羽墨

目前最为流行的`md`文件编辑器，当属Typora，免费，简洁，让人爱不释手

接上篇 [Typora破解之逆向分析](https://bbs.pediy.com/thread-272538.htm) ， 实现二进制层面的内存破解（证明内存破解的可行性，了却心愿）

经过几天断断续续的研究，我去逆向了Node API ，逆向了V8引擎，逆向了Electron框架，走入了无数的深坑。。。终于完美实现了内存破解。

在各路大神破解Typora的文章中，我看到的都是千篇一律的破解办法，所有我也本着论证我在上篇文章中提出的破解思路，而实现了我所希望的目标。

**本文中会给出两种办法，一种是基于病毒木马技术，一种是基于PE注入技术（适合新手复现）**

**注：文章重在讲述逆向过程中的思路与方法 不然发文章除了炫耀毫无意义**

---

# 愚公移山 ：内存patch

## 前期分析

1.根据上篇文章分析结果，继续进行深入分析，终于在我努力下，成功找到了关键函数，这个函数调用是`napi_create_string_utf8`，它会把`ascii`形式的JS代码当作参数传递，`rbp - 50` 的位置为这个字符对象的指针，里面存储的数据，经过我观察如下(只说重点的)

`0x0`偏移为JS代码的缓冲区，`0x10`偏移为JS代码的大小，`0x18`偏移为JS代码的关键验证点，必须比`0x10`位置的数据大1，但是据我观察他正常的时候不是这样的，可是在引擎中还是框架中（记不清了，只知道是海一样的汇编代码）会这样进行判断，好家伙太坑了，我差点晕倒

顺便提一下逆向引擎过程中发现的东西，V8引擎读取JS代码后，经过一些函数，之后申请一个缓冲区，代码流程跳到缓冲区执行，所以说引擎类似于一个动态的编译器，最后也是在内存中执行二进制代码

**具体的逆向引擎与框架的过程就不说了。。。**

![](https://bbs.kanxue.com/upload/attach/202205/930159_TE4BSAZY2EQCVSR.png)

2.根据上篇总结的思路，笔者准备采用病毒木马常用的技术 ， 进程替换（傀儡进程）来实现`hook`关键函数，达到破解的目的

为什么使用这个技术，主要是因为，`main.node`是动态加载的模块，我必须在它加载之前进行`hook`，不然它加载起来会直接进入JS代码执行的步骤，所以需要这个技术

那么关键函数怎么`hook`，`hook`什么位置呢，上篇中有提到，这是一个导入函数，导入了`electron`的导出函数，所以目标就明确了，我们可以通过遍历导出表，来找到这个函数的RVA，之后获得目标进程`BaseAddr`，加上RVA，就可以定位到这个函数了

那么还有一个问题，既然`main.node`使用的是导入函数，熟悉PE格式的同学可能就会知道了，加载导入表是通过`GetProcAddress`来加载的，那么是不是只需要修改`electron`框架的导出表的导出地址就可以了？对，就是这样，当然，直接`hook`函数也是可以的，但是毕竟要修改人家的原始函数，我感觉多多少少有点不靠谱。。。

3.要实现这个技术，要点是在`CreateProcess`上， 查看微软文档可知， 创建进程时， 以**`CREATE_SUSPENDED`**标志创建，即可在进程创建后，把主线程挂起，挂起以后，程序会处在PE映射完毕，进入`Startup`入口处的状态

![](https://bbs.kanxue.com/upload/attach/202205/930159_UMBS5QR3B5GBFU8.png)

4.挂起以后，需要定位导出函数在导出函数地址表中的RVA

5.进程肯定是会开随机基址的，所以我们不能使用固定的地址

6.定位导出函数地址以后，通过代码注入技术，把`hook`代码写入到 `electron`中，**`hook`代码应为`s`hellcode形式**

7.修改导出函数的导出函数地址表的地址，指向注入的代码，在代码中根据我们的逆向分析结果，判断是否为关键调用

8.如果判断为JS代码，则向内存中写入`Patch`，如不是，则调用原函数

9.还有很多细节，在下面解释，大体方向搞清楚了，可以开工了，`VS2019` 启动！！！

## Patch程序的编写

好的，说起来简单做起来难

因为我们使用的是`x64`版本的Typora，所以也需要编写同样`x64`的程序来实现破解，这`64`位程序就有很多坑等着你

### 创建傀儡进程

这部分比较简单，一行代码搞定

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `//创建进程并挂起`  `if` `(!CreateProcess(NULL, cmdline, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi))`  `{`  `printf("CreateProcess failed (%d)\n", GetLastError());`  `return` `0;`  `}` |

接下来按照前面的分析，应该准备好导出表相关的数据了

### 64位程序的导出表遍历

虽然我曾写过遍历的函数，我想大家肯定也有不少人写过这种东西，但是32位的不能直接拿来用

这里给出的实现代码，简单说一下坑的地方

**第一个坑** 遍历目标进程 与 遍历文件做选择，我最后选择了遍历文件

遍历目标进程映射后的PE格式是比较简单，但是我得写多少`ReadProcessMem`。。。。

那遍历文件也有坑啊，我全踩完了，首先读文件，读进来必定是64位地址，所以相关地方全换成64位值

**第二个坑** 遍历到的数据是FOA ，还得把FOA转RVA ， RVA转FOA ，xxx转VA ，一顿乱七八糟的转换（好吧，还不如多写点ReadProcessMem）

**第三个坑** 遍历时一般用到循环，循环通过下标遍历就OK ，但PE格式很多数据都为32位值，程序又用到了64位的地址，且64位指针步进都为8 ，然后开始调开始改吧。。。

通过修改，调试，最后得到了需要的数据 ：

目标函数的导出地址表的内容 RVA

目标函数的导出地址表的地址 FOA转RVA

目标函数的真实地址 是个FOA转RVA 之后再转VA

代码注释写起来比代码都复杂，就没写了，理清了关键内容就可以自己写出来

|  |  |

| --- | --- |
