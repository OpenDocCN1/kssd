
# Ghost Demo (3.1d)

> 原文：[`www.pediy.com/kssd/pediy04/kanxue093.htm`](https://www.pediy.com/kssd/pediy04/kanxue093.htm)

原作者：不明  
翻译者：PeterChen  
破解对像：Ghost Demo (3.1d)  
神兵利器：TR\[Dos 版\]  
软件介绍：系统硬件初始化，现在就是用来 Back\[备份\]系统！

---------------------

## 1. 第一步脱壳:

`TR ghost.exe`    ;TR 加载 `ghost.exe`

`getknl`    ;到达 kernel

;等待一会儿

`mkexe ghost.exe`    ;生成 mem.exe 文件

你可以测试一下，脱壳文件 `Mem.exe`。

---------------------

## 2. 第二步破解 15 秒限制：

`TR mem.exe`             ;再一次加载 mem.exe

`g`    ;退出 TR 界面

当“等待 15 分钟”之后，压 Ctrl+D。然后再出现 TR 的界面，
再压 `<F10>` 去看什么样循环。\[也就是找出循环内容加以分析\]

很容易就找到延迟 `Proc_199f`:

```
汇编：                               偏移：代码：
PUSHF                             ;3304:199F  9C             
CLI                               ;3304:19A0  FA             
MOV      AL,00                    ;3304:19A1  B000            
OUT      43,AL                    ;3304:19A3  E643 <---           
CALL     199E                     ;3304:19A5  E8F6FF           
IN       AL,40                    ;3304:19A8  E440            
MOV      BL,AL                    ;3304:19AA  8AD8            
CALL     199E                     ;3304:19AC  E8EFFF           
IN       AL,40                    ;3304:19AF  E440            
MOV      BH,AL                    ;3304:19B1  8AF8            
NOT      BX                       ;3304:19B3  F7D3            
POPF                              ;3304:19B5  9D             
MOV      AX,BX                    ;3304:19B6  8BC3            
RET                               ;3304:19B8  C3             
```

修改：

`a 199f`     ;汇编内容

```
MOV      AX,SI            ;3304:199F  8BC6
DEC      AX               ;3304:19A1  48             
RET                       ;3304:19A2  C3
```

用十六进制工具修改 `Mem.exe`:

`搜索` :    `9c fa b0 00 46` <-- `43`

`修改` :    `8b c6 48 c3`

本文有误：`e6` 原作者写成了 `46`!

---------------------

## 3. 破解评估限制：

`TR MEM.exe`

`G`    ;退出

当"Connection type"(可能是当退出 TR,不久快快)界面出来之时，
压 `<Ctrl+D>`，
和按 `<Enter>`，又返回 TR 领域。

`Logpro 1`    ;log it \[log 是写日志\]

`g`

当 Ghost 执行完，就会生成'logpro.dat'文件

`Lopro`     ;在 Dos 会提示

它会读取'logpro.dat'而且生成'logpro.txt'。

`Ren logpro.txt log1.txt`    ;重名保存

`date`

`10-22-98`    ;产生最近时间

`TR mem.exe`

`g`

当"Connection type"(可能是当退出 TR,不久快快)界面出来之时，
压 `<Ctrl+D>`，
和按 `<Enter>`，又返回 TR 领域。

`Logpro 1`    ;log it\[log 是写日志\]

`g`

`logpro`

`ren logpro.txt log2.txt`

如今天有两个日志. `log1.txt` 是执行时间,`log2.txt` 过期时间。

比较结果：

`log1.txt`

......
```
CALL 0fdb            ;40f5:04dd e8fb0a                      
CALL FAR 32c4:0638  ;40f5:0ff5 9a3806c432                   
RETF                ;32c4:064a cb                        
CALL FAR 32c4:401d  ;40f5:10ac 9a1d40c432                   
```

`log2.txt`

......
```
CALL 0fdb            ;40f5:04dd e8fb0a                      
CALL FAR 32c4:0638  ;40f5:0ff5 9a3806c432                   
RETF                ;32c4:064a cb                        
CALL FAR 32c4:27e7  ;40f5:1054 9ae727c432                   
```

我首先发现不同之处 `proc_fdb`. 我们推理:

`CALL 0fdb`            ;call check_if_timeout \[测试过期时间\]

`CALL FAR 32c4:0638`  ;call get_time \[当前时间\]

`RETF`               

...       ;compare \[进行比较\]

`j?? @@`

`CALL FAR 32c4:27e7`  ;call error \[错误提示\]

`@@:`   

让我们严密一些:

`TR mem.exe`

`g fdb`    ;direct go to proc_fdb \[直接跳到 proc_fdb\]

`....`    ; some <enter>        \[压 enter\]

TR 界面:

`U $,105d>tem.txt` \[反汇编内容另存为 Tem.txt\]

'tem.txt'内容:

```
ENTER    001C,00                  ;40F5:0FDB  C81C0000        
CMP      [WORD 00B2],00           ;40F5:0FDF  833EB20000     
JNE      0FF0                    ;40F5:0FE4  750A            
CMP      [WORD 00AC],00           ;40F5:0FE6  833EAC0000     
JNE      0FF0                    ;40F5:0FEB  7503            
JMP      10D8                    ;40F5:0FED  E9E800            
PUSH     SS                       ;40F5:0FF0  16             
LEA      AX,[BP-04]               ;40F5:0FF1  8D46FC            
PUSH     AX                       ;40F5:0FF4  50             
CALL FAR  32C4:0638                ;40F5:0FF5  9A3806C432     
ADD      SP,04                    ;40F5:0FFA  83C404            
MOVSX    EAX,[WORD BP-04]         ;40F5:0FFD  660FBF46FC     
MOV      [BP-10],EAX              ;40F5:1002  668946F0        
MOVZX    EAX,[BYTE BP-01]         ;40F5:1006  660FB646FF     
MOV      [BP-14],EAX              ;40F5:100B  668946EC        
MOVZX    EAX,[BYTE BP-02]         ;40F5:100F  660FB646FE     
MOV      [BP-18],EAX              ;40F5:1014  668946E8        
MOV      EAX,[BP-10]              ;40F5:1018  668B46F0        
ADD      EAX,FFFFF894            ;40F5:101C  660594F8FFFF    
IMUL     EAX,EAX,00002710         ;40F5:1022  6669C010270000   
MOV      EDX,[BP-14]              ;40F5:1029  668B56EC        
IMUL     EDX,EDX,64              ;40F5:102D  666BD264        
ADD      EAX,EDX                  ;40F5:1031  6603C2            
ADD      EAX,[BP-18]              ;40F5:1034  660346E8        
MOV      [BP-0C],EAX              ;40F5:1038  668946F4        
MOV      AX,7A18                  ;40F5:103C  B8187A            
MOV      ES,AX                    ;40F5:103F  8EC0            
MOV      EAX,[ES:0A18]            ;40F5:1041  6626A1180A     
CMP      EAX,[BP-0C]              ;40F5:1046  663B46F4        
                                  <<<-----
JGE      105C                    ;40F5:104A  7D10            
PUSH     01                       ;40F5:104C  6A01            
PUSH     7A18                     ;40F5:104E  68187A            
PUSH     0FFA                     ;40F5:1051  68FA0F            
CALL FAR  32C4:27E7                ;40F5:1054  9AE727C432     
ADD      SP,06                    ;40F5:1059  83C406            
MOV      AX,7A18                  ;40F5:105C  B8187A            
```

到此我们要如何做了？

`a 1046`

`jmp 105c`  ;eb 14

`G`

它能运行!我们开始用十六进制工具修改 `Mem.exe`:

`搜索` :    `66 3b 46 f4 7d 10`

`修改` :    `eb 14`

至此，脱壳，破解，都完成了！

---------------------

## 4. 把 `Mem.exe` 拷贝到 `Ghost.exe` 的同目录下运行。

I think we can make the MEM.EXE run without ghost.exe:

我想应该制造一个脱离 ghost.exe 的 Mem:

a. make MEM.EXE as follow:

制造 Mem.exe 过程:

`mkexe`    ;不开参数

b. If MEM.exe large than ghost.exe's main part,

compress it.

如果 Mem.exe 比 ghost.exe 主要部分时可以压缩:

`pklite mem.exe`

`copy /b mem.exe+dummy`    ;制造合适长度

c. 覆盖 ghost.exe

d. 追加 mem.exe

`copy /b mem.exe+overlay`

If you have done it, Please put it to

`ftp://ftp.nease.net/incoming/tr`

and tell me.

Thank You!

--------------------------------------------------------------

一个朋友告诉我:

这里，我想告诉你关我破解 Ghost31d 的方法。因为你的方法有一些不全

'Load Disk/Partition to Image'.\[这个是 ghost 的功能，是加载或保存映象文件\]

After MKEXE Ghost.exe

之后 Mkexe ghost.exe

重名 mem.exe 为 GGGGG.exe

`TR GGGGG.exe -#e=g.env`

`bp ip=5497`

`g`

`a cs:ip`

`cs:5497 jmp 550E`

`bp ip=1386`

`g`

`a cs:ip`

`cs:1386 jmp 13A1`

`g`

`q`

运行 GGGGG.exe 时已经是注册版了!

PeterChen：

不是别人不全而是别人方法比反锁而已，破解就好像数学一样条条大路通罗马! 4dd466d0e1e0cc123afbe504da2462ca
