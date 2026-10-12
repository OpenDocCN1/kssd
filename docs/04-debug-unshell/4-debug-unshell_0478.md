
# IconEdit2 v2.4

> 原文：[`www.pediy.com/kssd/pediy05/pediy50444.htm`](https://www.pediy.com/kssd/pediy05/pediy50444.htm)

IconEdit2 脱壳，Asprotect 的脱壳高手们已经写过很过了，我菜鸟也凑个热闹，多少给和我一样菜的同志们借鉴一下，也希望高手们多多指教。

IconEdit2(v2.3)的壳很有意思，用 FI245 查出是 Asprotect1.3 的，用 FI249 查出是 Asprotect1.21 的，估计 FI249 可能会准确一点吧（可是我觉得 FI249 没有 FI245 好用，所以很少用，只有 245 查不到的时候才用，^\_^）。IconEdit2(v2.4)的壳是 Asprotect1.22 的，大家都差不多，就那 v2.3 开刀吧。

因为以前跟过 1.2 的壳，这次碰到个 1.3 的(也不知道真的假的)，也跟一跟，我是从头到尾一个劲的傻跟了一遍，没设过 API 断点(其实也不知具体该设什么)，最后终于到达 OEP，感觉和 1.2 版的壳差别不大。具体跟踪细节我这里就不说了，高手们有很多文章了。而且这个程序是 Delphi 写的，可以用快速找 OEP 的方法快速脱壳。我来说说 dump 出来以后的事情。

从 OEP 处 dump 出 `iEditu.exe`，由于 Asprotect 对 import table 做了破坏，所以程序不能运行，必须修复 IAT，ImportREC142 是个好东东，可以很快很方便的修复 IAT，就用它来做。在 `Process` 下拉栏选择运行的进程(运行脱壳前的程序)，然后在 OEP 栏输入我们得到的 OEP 值(0x00491D80)，注意不要加 base，即 491D80。接着就 IAT Autosearch，提示说找到，跟着 Get Imports，窗口里看到了很多找到的引入函数，用 `Show Invalid` 看到一些未解析到的，在其上面点鼠标右键选 `Trace Level1`，哇，又解析出一大堆来，还剩 10 个左右，再用右键的 `Plugin Tracer`，嘿嘿，Congratulation! 全部解析成功。暗暗窃喜... 最后就是 Fix Dump，当然最好选中 `Add new section`。生成修复过的文件 `iEditu\_.exe`

满怀热情的双击运行，咦，什么反应都没有？考，是不是鼠标坏了，咚咚咚，帮帮帮......还好没真把我的鼠标敲坏了。冷静下来开始分析，好像记得哪位大侠说过“一运行就退出，一看就是 CRC”。唉，看来这回是遇到棘手的了，没办法，跟一跟了。入口如下：

```
018F:00491D80  PUSH     EBP          <-- OEP
018F:00491D81  MOV      EBP,ESP
018F:00491D83  ADD      ESP,BYTE -0C
018F:00491D86  PUSH     EBX
018F:00491D87  PUSH     ESI
018F:00491D88  PUSH     EDI
018F:00491D89  MOV      EAX,00491B30
018F:00491D8E  CALL     00406AD4
018F:00491D93  XOR      EAX,EAX
018F:00491D95  PUSH     EBP
018F:00491D96  PUSH     DWORD 00491E14
018F:00491D9B  PUSH     DWORD [FS:EAX]
018F:00491D9E  MOV      [FS:EAX],ESP
018F:00491DA1  CALL     0048A390
018F:00491DA6  MOV      EAX,[004938C4]
018F:00491DAB  MOV      EAX,[EAX]
018F:00491DAD  CALL     00448FC8
018F:00491DB2  CALL     0048A410
018F:00491DB7  MOV      EAX,[004938C4]
018F:00491DBC  MOV      EAX,[EAX]
018F:00491DBE  MOV      EDX,00491E30
018F:00491DC3  CALL     00448BCC
018F:00491DC8  CALL     0048A438       <-- F10 带过就没了(1)
018F:00491DCD  MOV      EAX,[004938C4]
018F:00491DD2  MOV      EAX,[EAX]
018F:00491DD4  ADD      EAX,BYTE +40
018F:00491DD7  MOV      EDX,00491E44
018F:00491DDC  CALL     00403B8C
018F:00491DE1  CALL     0048A480       <-- F10 带过也没了(2)
018F:00491DE6  MOV      ECX,[00493714]
018F:00491DEC  MOV      EAX,[004938C4]
018F:00491DF1  MOV      EAX,[EAX]
018F:00491DF3  MOV      EDX,[0048BA8C]
018F:00491DF9  CALL     00448FE0       <-- F10 带过还是没了(3)
018F:00491DFE  MOV      EAX,[004938C4]
018F:00491E03  MOV      EAX,[EAX]
018F:00491E05  CALL     00449060       <-- 这里主程序开始运行
018F:00491E0A  XOR      EAX,EAX
```

F8 进到第(1)个 CALL 看看，如下：

```
018F:0048A438  PUSH     EBP
018F:0048A439  MOV      EBP,ESP
018F:0048A43B  PUSH     ECX
018F:0048A43C  MOV      BYTE [EBP-01],00
018F:0048A440  MOV      EAX,[00493624]
018F:0048A445  CMP      BYTE [EAX],00
018F:0048A448  JNZ      0048A463
018F:0048A44A  PUSH     ESI
018F:0048A44B  MOV      ESI,[004944D8]      <--这里取基址 400000
018F:0048A451  MOV      EAX,[ESI+3C]
018F:0048A454  MOV      EAX,[ESI+EAX+28]    <--EAX 为 OEP
018F:0048A458  CMP      AX,1000              <--OEP 不为 1000 就退出
018F:0048A45C  JZ       0048A462             <--改这里强行跳走。
018F:0048A45E  OR       BYTE [EBP-01],01
018F:0048A462  POP      ESI
018F:0048A463  CMP      BYTE [EBP-01],00
018F:0048A467  JZ       0048A47C
018F:0048A469  XOR      ECX,ECX
018F:0048A46B  MOV      DL,01
018F:0048A46D  MOV      EAX,[00407B60]
018F:0048A472  CALL     0040BA4C
018F:0048A477  CALL     00403608
018F:0048A47C  POP      ECX
018F:0048A47D  POP      EBP
018F:0048A47E  RET
```

再进到第(2)个 CALL 看看，如下：

```
018F:0048A480  PUSH     EBP
018F:0048A481  MOV      EBP,ESP
018F:0048A483  PUSH     ECX
018F:0048A484  PUSH     EBX
018F:0048A485  MOV      EAX,[00406C4A]
018F:0048A48B  MOV      EBX,[EAX]
018F:0048A48D  PUSH     DWORD [EBX]      \ 这个 push 和 pop 没什么意义，
018F:0048A48F  MOV      [EBP-04],EBX     | 我估计也是用来检查脱壳的，
018F:0048A492  POP      DWORD [EBX]      / 这时 EBX＝BFFxxxxx，异常。
018F:0048A494  MOV      EAX,[EBP-04]       把 Push 和 Pop 都 NOP 掉。
018F:0048A497  POP      EBX
018F:0048A498  POP      ECX
018F:0048A499  POP      EBP
018F:0048A49A  RET
```

改了这两个地方后，觉得该行了吧，没想到到(3)的 CALL 又死悄悄了。真让人上火，它到底要设多少陷阱呀！这个 CALL 可是个很大的呀，用原版带过也要等一阵才过来的，进去以后简直像进了一个无底洞，半天出不来，而且也看不到什么可疑的类似 CRC 的校验地方。这可真是大海捞针呀！用脱壳后的程序跟进去，一不留神就退出了，发现都是产生异常而退出的，基本排除有 CRC 校验的保护。

一筹莫展之际，dede 反编译，发现这个 CALL 似乎是来建立资源的，难道是资源在脱壳中有什么不测？然后用一些资源编译器试着编辑一下，果然有很多异常现象。初步认为是 ASPR 对资源做了特殊压缩，这样可麻烦了。用 procdump 查知 `.rsrc` 段 RVA 是 4A4000，size 是 7D000，载入原版在 OEP 中断，看看资源段，一切正常，干脆把它都 dump 出来，随即 dump 出个 `rsrc.bin` 准备替换脱壳版的 `.rsrc` 段。可是一看原来一模一样，那还换个什么劲呀。再载入脱壳后的程序停在入口，看资源段，咦？发现只有 4A4000 到 4A6000 有东西，后面的都是????，难道是段的 size 有问题，翻来覆去的查，可是什么问题也看不出来。现在我还不知道那些????????是怎么回事，有知道的请教教我。

穷途末路了，还是用 TRW 跟吧，硬着头皮一层层的跟，设了无数断点，记录了我无数辛苦的脚步，最后总算把错误定位在了一条语句上。对比原版，发现好像就是和上面的那些????????有关，可是那些????????我又搞不定，难道就这么宣告失败了？

最后最后，我又想到是不是我当初 dump 出来的程序就有问题呀(是用 Procdump)，我再用 TRW 直接 pedump 一次，然后一样用 ImportREC 修复 IAT，咦，这次居然有 10 个函数没有解析出来，真是见了鬼了。没办法，手动来吧，还好没有太难的，全部找出来，发现有两个是空函数，想在 import list 中删掉，可是一不小心竟然连整个 Trunk 都给删掉了，晕倒(是不是 ImportREC 没有删函数的功能呀？)只好重新再来抓，哇，这回居然又全部解析成功了，真不知怎么搞的。成功就成功吧，刚才也不能白忙活呀，对比对比吧，发现和我手动找的基本一致，只有一个函数不一样，还有就是那两个空函数都被 `FreeResource` 代替了。

“FreeResource”，看到这个字眼，我的眼睛突然一亮，我上面定位的出错点反推回去好像就是在一个 `CALL FreeResource` 附近呀，还好断点还在，马上来到这里：

```
018F:00413305  PUSH     EAX
018F:00413306  PUSH     ESI
018F:00413307  CALL     `KERNEL32!FindResourceA`
018F:0041330C  MOV      EDI,EAX
018F:0041330E  MOV      [EBX+10],EDI
018F:00413311  TEST     EDI,EDI
018F:00413313  JNZ      0041331C
018F:00413315  PUSH     EBP
018F:00413316  CALL     00413280
018F:0041331B  POP      ECX
018F:0041331C  MOV      EAX,[EBX+10]
018F:0041331F  PUSH     EAX
018F:00413320  PUSH     ESI
018F:00413321  CALL     `KERNEL32!LoadResource`
018F:00413326  MOV      EDI,EAX
018F:00413328  MOV      [EBX+14],EDI
018F:0041332B  TEST     EDI,EDI
018F:0041332D  JNZ      00413336
018F:0041332F  PUSH     EBP
018F:00413330  CALL     00413280
018F:00413335  POP      ECX
018F:00413336  MOV      EAX,[EBX+10]
018F:00413339  PUSH     EAX
018F:0041333A  PUSH     ESI
018F:0041333B  CALL     `KERNEL32!SizeofResource`
018F:00413340  PUSH     EAX
018F:00413341  MOV      EAX,[EBX+14]        <-- EAX=0x4AF088(就是????)
018F:00413344  PUSH     EAX                 这个值很关键，后面有用
018F:00413345  CALL     `KERNEL32!FreeResource`
018F:0041334A  MOV      EDX,EAX              <-- 到这 EAX＝0，导致后面的
018F:0041334C  MOV      EAX,EBX               异常错误产生。
018F:0041334E  POP      ECX
018F:0041334F  CALL     00413058
018F:00413354  POP      EDI
018F:00413355  POP      ESI
018F:00413356  POP      EBX
018F:00413357  POP      ECX
018F:00413358  POP      EBP
018F:00413359  RET      04
```

看来这里来一个 `FreeResource` 很不合适，既然原来是空函数，那就 NOP 掉它吧，这样保持 `EAX` 的值不变，至少后面的那个出错点不会再有问题了，改过后 F5 跑，有好多次都会来到这里，全部跳过去后，程序画面总算出来了，啊，老天有眼可算是搞定了。既然有两个空函数，找到另一个，也 NOP 掉，另外一个不一样的函数好像没什么影响，不管了吧。改完以后终于可以正常运行了，^\_^

总结一下，这次脱壳成功，运气成分不可少，要不是不留神将 Trunk 删了，可能还发现不了 ImportREC 自动解析的错误，那样还不知道什么时候才能搞定这个壳呢。由此看出，ImportREC 自动解析的引入函数也不能保证都正确，最好在使用 TraceLavel1 后还剩余的函数上留意一下，如有什么问题可以在这里找找原因，嘿嘿，这是我得出的一点经验，与大家分享。有不对的地方请帮我指出来。

最后说一下，IconEdit2(v2.4)脱壳后，只需要修正 IAT，并改一下(1)(2)两个陷阱，即可正常运行，没那么麻烦了。

上次脱了 IconEdit2 v2.3 的壳，发现有个重大问题，就是无法存盘。

v2.3 的破解倒不难，很容易抓出注册码来，用
```
Name: chilly200
Code: 8kHXkiJ8qGfk9ig7hwy7bhMT
```
注册成功后，存盘时不会弹出什么对话框，好像一切正常似的，可实际上却没有



### 存盘 BUG 排查

在硬盘上找不到存盘的文件，而原版则可以。

继续查找 BUG：载入原版，在存盘时下断点 `bpx createfilea`，拦住以后 `F12` 几次返回。

### 程序空间分析

代码如下：

```
018F:0048D780  CALL       00403B8C  
018F:0048D785  CALL       00491098  
018F:0048D78A  TEST       AL,AL  
018F:0048D78C  JNZ        NEAR 0048D9A2  
018F:0048D792  PUSH       EBP  
018F:0048D793  CALL       0048D1D0       <-- 这个 CALL 是关键。  
018F:0048D798  POP        ECX            <-- 停在这  
018F:0048D799  JMP        0048D9A2
```

第二次，用 `F8` 进入上面的 `CALL`，一直 `F10` 跟到这里：

```
018F:0048D1F5  PUSH       DWORD 0048D5DB  
018F:0048D1FA  PUSH       DWORD [FS:EDX]  
018F:0048D1FD  MOV        [FS:EDX],ESP  
018F:0048D200  CALL       0048A2EC        <-- 这个 CALL 关键，要进去  
018F:0048D205  JMP        0048D5CC  
018F:0048D20A  SUB        [EDI+9C849801],EBP  
018F:0048D210  MOV        CH,E9  
018F:0048D212  JS         0048D1F0  
018F:0048D214  STOSB     
018F:0048D215  DB         82  
018F:0048D216  PUSH       DWORD 1D8D0E8D  
018F:0048D21B  MOV        ESP,57A1446B  
018F:0048D220  SHL        DWORD [ESI+9B724265],CL  
018F:0048D226  MOV        DH,1D  
018F:0048D228  OUTSB      
```

### SMC 代码变异现象

上面的 `CALL` 里面肯定有 SMC（Self-Modifying Code），因为执行过后代码变了：

```
018F:0048D1F5  PUSH       DWORD 0048D5DB  
018F:0048D1FA  PUSH       DWORD [FS:EDX]  
018F:0048D1FD  MOV        [FS:EDX],ESP  
018F:0048D200  CALL       0048A2EC  
018F:0048D205  JMP        0048D20B    <-- 变了！上面的 CALL 好厉害呀  
018F:0048D20A  NOP                    <-- 这里原本是花指令，呵呵  
018F:0048D20B  MOV        EAX,[EBP+08]  
018F:0048D20E  MOV        EAX,[EAX-04]  
018F:0048D211  MOV        EAX,[EAX+02E0]  
018F:0048D217  MOV        AX,[EAX+0246]  
018F:0048D21E  SUB        AX,BYTE +10  
018F:0048D222  JZ         NEAR 0048D2C1  
018F:0048D228  SUB        AX,BYTE +10  
018F:0048D22C  JNZ        NEAR 0048D35C  
018F:0048D232  LEA        EAX,[EBP+FFFFF342]  
018F:0048D238  MOV        ECX,FF  
018F:0048D23D  MOV        EDX,0CBE  
018F:0048D242  CALL       00402B70
```

### Dump 数据段

跟进这个关键 `CALL` 进行追踪，一步步耐心地跟，最终发现，这段代码用到了原来壳中的一些代码和数据。一共有三段（当然是一段一段来的，不是一下子全找到的，呵呵），全部 dump 出来，分别是：

```
file       start       end       lenth  
add1.bin:  0530000     0535000   5000  
add2.bin:  11B1000     11C8000   17000  
add3.bin:  11D0000     11D9000   9000  
```

注意，这三段不是在 OEP 处 dump 的，而要在 `cs: 00491DF9 CALL 00448FE0` 的 `CALL` 过后再 dump（因为这里面好像也有些 SMC）。

不巧的是 `importREC` 加的段 `.mackt` 的 RAV 和 `add1` 的有冲突，将 `.mackt` 的 RAV 改为 `140000`，当然也要改文件中相关的 IT 和 IAT 的 RAV 值（用查找替换就行了），最后再改一下 `180H` 处的 `00001300` 为 `00001400` 即可。

然后把上面 dump 出的三个文件内容添加到原脱壳后的文件中，用 `PEditor1.7` 分别添加三个段 `.add1`、`.add2`、`.add3`，并改好相应的数值。

添加第二个段的时候，`PEditor` 说 PE 头没有足够的空间添加段了，不知道是怎么回事，诚请高手指点一下！我看到有几个段的 size 是 0，估计删掉应该可以吧，可是 `PEditor` 中却不能删，用 `Procdump` 的 PE editor 删掉了。然后就可以继续添加段了。最后用 `rebuilder` 只选中 `make PE Header win nt/2000 compatible` 进行 PE 头文件重建，就 OK 了。

### 修复数据被修改问题

不幸，还是不能成功存盘。继续跟踪得知，dump 出来的数据在使用的时候已经改变了，是有什么程序又用了 SMC 把我们千辛万苦抓出来的数据和代码给改了，真是可恶呀！我也尝试了把这几个段改成只读，没成功。只好继续耐心地追踪，终于找到了这段 SMC 的代码在这里：

```
018F:0048A1EC  MOV        BYTE [004935DC],01 <-- 这里必须置 1，存盘判断  
018F:0048A1F3  PUSH       EDX  
018F:0048A1F4  PUSH       EAX  
018F:0048A1F5  CALL       NEAR [004935E0]    <-- CALL 壳里的一段 SMC  
018F:0048A1FB  RET        
```

就是这里改变了我们 dump 出来的东西，把从 `push` 开始的全 `NOP` 掉吧。

存盘试一下，大功告成了！

### IconEdit2 v2.4 脱壳与注册问题

`IconEdit2 v2.4` 是 `Asprotec1.22` 加密的，脱壳请看我写的“IconEdit2 脱壳实录”。

与 `v2.3` 版不同的是，这次的注册码不是在源程序里进行比对的，而是利用了 ASPR 壳的 Key 保护，在壳里面进行计算，并且根据 Key 使用 SMC 技术动态改变程序代码。

脱了壳的程序自然就没有了 Key 的计算部分，所以即使有正确的注册码也不可能注册成功了。

当然可以跟踪壳里的计算代码，找到它的算法和 SMC，可是这对我是太难了，我跟了老半天，越过无数陷阱，可是还是连注册码该有多少位都没搞清楚，只好放弃了。不知道有没有高人研究过，给咱菜鸟们传授传授吧！

对付这种程序真让人头疼，我原来汉化的一个 `HTMLZip1.04` 也是类似的，害得我把个程序改的面目全非的（最后吕大哥说我汉化不完全没发表，^\_^）。

我想，对这种程序最好的办法是先用正确的注册码先注册成功，然后再脱壳修改，这样应该会比较顺利吧。可是咱们穷哥们是掏不起那注册费的呀，就来用暴暴暴暴力的吧！

首先来看看脱壳后的程序，有个 `About`，上面写着 `Unregister Version`，还有是存盘的时候会蹦出注册对话框，不注册不能存盘！天哪，不能存盘这个软件简直成了个废物了。

修改 `About` 和弹出的注册框并不是很难，用 `TRW` 跟踪吧，不过最好能把程序反编译出来，就可以看得更清晰了。顺便说一下，原来脱过壳的程序我用 `W32Dasm` 和 `DeDe` 都不能反编译，不知道为什么。不在 OEP 处 dump，而在后面的 `00493137` 处的 `CALL` 后再 dump，则可以用 `W32Dasm` 反编译，但 `DeDe` 还是不能反编。后来改了 `DeDe` 一个字节，使其可以反编译了（没有 Process 了，只能 RAV 反编译，呵呵，不过也够了）。

`About` 和注册框先搞定它，下面是显示 `About` 的一段代码：

```
018F:0048D12A  LEA        EAX,[EBP-48]  
018F:0048D12D  CALL       0048AFD4        <-- 这个 CALL 不要进去了，只要 018F:0048D132  CMP  DWORD [EBP-48],BYTE +00  使[EBP-48]不为零就行  
018F:0048D136  JNZ        0048D16D        <-- 这里一定要跳  
        :  
        :  
        :  
018F:0048D16D  MOV        EAX,[EBP-04]  
018F:0048D170  MOV        EAX,[EAX+02F4]  
018F:0048D176  MOV        DL,01  
018F:0048D178  CALL       0042BFAC  
018F:0048D17D  MOV        EAX,[EBP-04]  
018F:0048D180  MOV        EAX,[EAX+02D8]  
018F:0048D186  XOR        EDX,EDX  
018F:0048D188  CALL       0042BFAC  
018F:0048D18D  LEA        EAX,[EBP-4C]  
018F:0048D190  CALL       0048AFD4        <-- 这个 CALL 也不进了，  
018F:0048D195  MOV        EDX,[EBP-4C]    <-- EDX 指向注册用户名  
018F:0048D198  MOV        EAX,[EBP-04]  
018F:0048D19B  MOV        EAX,[EAX+02F8]  
018F:0048D1A1  CALL       0042C0C4
```

跳过存盘时的注册对话框和上面第一段的代码基本一样，同样改一下就行了。

### 实现存盘功能

这时程序看上去已经是一个注册版了，但最困难的是实现存盘功能，总不能虚有其表吧。这个真的是很难，`v2.3` 版中也有同样的问题，可以看我的“脱壳实录”中的解决方法。但是 `v2.3` 是在注册成功的基础上而实现的，`v2.4` 版却不行了，即使将那三段都找出来加上可能也不行。我只加了一段看了看，发现根本不行，程序和壳中 SMC 的地方太多，不注册成功不可能恢复出正确的代码来的。

为这个我可是苦恼了好几个小时，最后终于灵光一现（可能你早就想到了），`v2.3` 的我不是搞定了吗，估计 `v2.4` 版的存盘这部分的代码应该差不多吧，能不能用 `v2.3` 版的正确的存盘代码来替换 `v2.4` 版的呢？想到这，马上行动，还记得我“脱壳实录”中的这段吗：

```
018F:0048D1F5  PUSH       DWORD 0048D5DB  
018F:0048D1FA  PUSH       DWORD [FS:EDX]  
018F:0048D1FD  MOV        [FS:EDX],ESP  
018F:0048D200  CALL       0048A2EC  
018F:0048D205  JMP        0048D20B    <-- 变了！上面的 CALL 好厉害呀  
018F:0048D20A  NOP                    <-- 这里原本是花指令，呵呵  
018F:0048D20B  MOV        EAX,[EBP+08]  
018F:0048D20E  MOV        EAX,[EAX-04]  
018F:0048D211  MOV        EAX,[EAX+02E0]  
018F:0048D217  MOV        AX,[EAX+0246]  
018F:0048D21E  SUB        AX,BYTE +10  
018F:0048D222  JZ         NEAR 0048D2C1  
018F:0048D228  SUB        AX,BYTE +10  
018F:0048D22C  JNZ        NEAR 0048D35C  
018F:0048D232  LEA        EAX,[EBP+FFFFF342]  
018F:0048D238  MOV        ECX,FF  
018F:0048D23D  MOV        EDX,0CBE  
018F:0048D242  CALL       00402B70
```

我们现在不能用上面的 `CALL` 恢复出下面正确的代码，可是 `v2.3` 的不是已经出来了嘛，跟了一下，哈哈，存盘果然在这段代码中完成。好，把这段全 dump 出来，换到 `v2.4` 版中去，下面就是替换后的代码（够长的呀）：

```
018F:0048ECF0 E883C3FFFF       CALL       0048B078   <-- 这个 CALL 没用了  
018F:0048ECF5 E901000000       JMP        0048ECFB   <-- 下面是正确代码  
018F:0048ECFA 90              NOP               
018F:0048ECFB 8B4508          MOV        EAX,[EBP+08]  
018F:0048ECFE 8B40FC          MOV        EAX,[EAX-04]  
018F:0048ED01 8B80E0020000    MOV        EAX,[EAX+02E0]  
018F:0048ED07 668B8046020000  MOV        AX,[EAX+0246]  
018F:0048ED0E 6683E810        SUB        AX,BYTE +10  
018F:0048ED12 0F8499000000    JZ         NEAR 0048EDB1  
018F:0048ED18 6683E810        SUB        AX,BYTE +10  
018F:0048ED1C 0F852A010000    JNZ        NEAR 0048EE4C  
018F:0048ED22 8D8542F3FFFF    LEA        EAX,[EBP+FFFFF342]  
018F:0048ED28 B9FF000000      MOV        ECX,FF  
018F:0048ED2D BABE0C0000      MOV        EDX,0CBE  
018F:0048ED32 E82959F7FF      CALL       00404660   改>>>00402B70  
018F:0048ED37 BB3E000000      MOV        EBX,3E  
018F:0048ED3C 8D8542F3FFFF    LEA        EAX,[EBP+FFFFF342]  
018F:0048ED42 C60000          MOV        BYTE [EAX],00  
018F:0048ED45 40              INC        EAX  
018F:0048ED46 4B              DEC        EBX  
018F:0048ED47 75F9            JNZ        0048ED42  
018F:0048ED49 C68544F3FFFF01 MOV        BYTE [EBP+FFFFF344],01  
018F:0048ED50 C68546F3FFFF01 MOV        BYTE [EBP+FFFFF346],01  
018F:0048ED57 C68548F3FFFF20 MOV        BYTE [EBP+FFFFF348],20  
018F:0048ED5E C68549F3FFFF20 MOV        BYTE [EBP+FFFFF349],20  
018F:0048ED65 C68550F3FFFFA8 MOV        BYTE [EBP+FFFFF350],A8  
018F:0048ED6C C68551F3FFFF0C MOV        BYTE [EBP+FFFFF351],0C  
018F:0048ED73 C68554F3FFFF16 MOV        BYTE [EBP+FFFFF354],16  
018F:0048ED7A C68558F3FFFF28 MOV        BYTE [EBP+FFFFF358],28  
018F:0048ED81 C6855CF3FFFF20 MOV        BYTE [EBP+FFFFF35C],20  
018F:0048ED88 C68560F3FFFF40 MOV        BYTE [EBP+FFFFF360],40  
```



```
018F:0048ED8F C68564F3FFFF01   MOV      BYTE [EBP+FFFFF364],01

018F:0048ED96 C68566F3FFFF18   MOV      BYTE [EBP+FFFFF366],18

018F:0048ED9D BB80000000       MOV      EBX,80

018F:0048EDA2 8D4580           LEA      EAX,[EBP-80]

018F:0048EDA5 C60000           MOV      BYTE [EAX],00

018F:0048EDA8 40               INC      EAX

018F:0048EDA9 4B               DEC      EBX

018F:0048EDAA 75F9             JNZ      0048EDA5

018F:0048EDAC E99B000000       JMP      0048EE4C

018F:0048EDB1 8D8542F3FFFF     LEA      EAX,[EBP+FFFFF342]

018F:0048EDB7 B9FF000000       MOV      ECX,FF

018F:0048EDBC BABE0C0000       MOV      EDX,0CBE

018F:0048EDC1 E89A58F7FF       CALL     00404660   改>>>00402B70

018F:0048EDC6 BB3E000000       MOV      EBX,3E

018F:0048EDCB 8D8542F3FFFF     LEA      EAX,[EBP+FFFFF342]

018F:0048EDD1 C60000           MOV      BYTE [EAX],00

018F:0048EDD4 40               INC      EAX

018F:0048EDD5 4B               DEC      EBX

018F:0048EDD6 75F9             JNZ      0048EDD1

018F:0048EDD8 C68544F3FFFF01   MOV      BYTE [EBP+FFFFF344],01

018F:0048EDDF C68546F3FFFF01   MOV      BYTE [EBP+FFFFF346],01

018F:0048EDE6 C68548F3FFFF10   MOV      BYTE [EBP+FFFFF348],10

018F:0048EDED C68549F3FFFF10   MOV      BYTE [EBP+FFFFF349],10

018F:0048EDF4 C68550F3FFFF68   MOV      BYTE [EBP+FFFFF350],68

018F:0048EDFB C68551F3FFFF03   MOV      BYTE [EBP+FFFFF351],03

018F:0048EE02 C68554F3FFFF16   MOV      BYTE [EBP+FFFFF354],16

018F:0048EE09 C68558F3FFFF28   MOV      BYTE [EBP+FFFFF358],28

018F:0048EE10 C6855CF3FFFF10   MOV      BYTE [EBP+FFFFF35C],10

018F:0048EE17 C68560F3FFFF20   MOV      BYTE [EBP+FFFFF360],20

018F:0048EE1E C68564F3FFFF01   MOV      BYTE [EBP+FFFFF364],01

018F:0048EE25 C68566F3FFFF18   MOV      BYTE [EBP+FFFFF366],18

018F:0048EE2C BB3E030000       MOV      EBX,033E

018F:0048EE31 C6841D42F3FFFF00 MOV      BYTE [EBP+EBX+FFFFF342],00

018F:0048EE39 C6841D43F3FFFF00 MOV      BYTE [EBP+EBX+FFFFF343],00

018F:0048EE41 83C304           ADD      EBX,BYTE +04

018F:0048EE44 81FB7A030000     CMP      EBX,037A

018F:0048EE4A 7EE5             JNG      0048EE31

018F:0048EE4C B201             MOV      DL,01

018F:0048EE4E A198914100       MOV      EAX,[00419198] 改>>>[00419130]

018F:0048EE53 E82422F9FF       CALL     0042107C    改>>>0041F524

018F:0048EE58 898530F3FFFF     MOV      [EBP+FFFFF330],EAX

018F:0048EE5E 8B4508           MOV      EAX,[EBP+08]

018F:0048EE61 8B40FC           MOV      EAX,[EAX-04]

018F:0048EE64 8B80E0020000     MOV      EAX,[EAX+02E0]

018F:0048EE6A E8E9A9FDFF       CALL     00469858    改>>>00468AF0

018F:0048EE6F 8BD0             MOV      EDX,EAX

018F:0048EE71 8B8530F3FFFF     MOV      EAX,[EBP+FFFFF330]

018F:0048EE77 8B08             MOV      ECX,[EAX]

018F:0048EE79 FF5108           CALL     NEAR [ECX+08]

018F:0048EE7C 33D2             XOR      EDX,EDX

018F:0048EE7E 55               PUSH     EBP

018F:0048EE7F 681FD54800       PUSH     DWORD 0048D51F

018F:0048EE84 64FF32           PUSH     DWORD [FS:EDX]

018F:0048EE87 648922           MOV      [FS:EDX],ESP

018F:0048EE8A C6853BF3FFFF00   MOV      BYTE [EBP+FFFFF33B],00

018F:0048EE91 8B4508           MOV      EAX,[EBP+08]

018F:0048EE94 8B40FC           MOV      EAX,[EAX-04]

018F:0048EE97 8B80E0020000     MOV      EAX,[EAX+02E0]

018F:0048EE9D 668B8046020000   MOV      AX,[EAX+0246]

018F:0048EEA4 6683E810         SUB      AX,BYTE +10

018F:0048EEA8 741C             JZ       0048EEC6

018F:0048EEAA 6683E810         SUB      AX,BYTE +10

018F:0048EEAE 752A             JNZ      0048EEDA

018F:0048EEB0 C7853CF3FFFFDE0B+MOV      DWORD [EBP+FFFFF33C],0BDE

018F:0048EEBA C78534F3FFFFBA0C+MOV      DWORD [EBP+FFFFF334],0CBA

018F:0048EEC4 EB14             JMP      SHORT 0048EEDA

018F:0048EEC6 C7853CF3FFFF0E03+MOV      DWORD [EBP+FFFFF33C],030E

018F:0048EED0 C78534F3FFFF7A03+MOV      DWORD [EBP+FFFFF334],037A

018F:0048EEDA 8B4508           MOV      EAX,[EBP+08]

018F:0048EEDD 8B40FC           MOV      EAX,[EAX-04]

018F:0048EEE0 8B80E0020000     MOV      EAX,[EAX+02E0]

018F:0048EEE6 0FB78048020000   MOVZX    EAX,WORD [EAX+0248]

018F:0048EEED 48               DEC      EAX

018F:0048EEEE 85C0             TEST     EAX,EAX

018F:0048EEF0 0F8C00010000     JL       NEAR 0048EFF6

018F:0048EEF6 40               INC      EAX

018F:0048EEF7 89852CF3FFFF     MOV      [EBP+FFFFF32C],EAX

018F:0048EEFD 33FF             XOR      EDI,EDI

018F:0048EEFF 8B4508           MOV      EAX,[EBP+08]

018F:0048EF02 8B40FC           MOV      EAX,[EAX-04]

018F:0048EF05 8B80E0020000     MOV      EAX,[EAX+02E0]

018F:0048EF0B 0FB7B046020000   MOVZX    ESI,WORD [EAX+0246]

018F:0048EF12 4E               DEC      ESI

018F:0048EF13 85F6             TEST     ESI,ESI

018F:0048EF15 0F8CCE000000     JL       NEAR 0048EFE9

018F:0048EF1B 46               INC      ESI

018F:0048EF1C 33DB             XOR      EBX,EBX

018F:0048EF1E 8B8530F3FFFF     MOV      EAX,[EBP+FFFFF330]

018F:0048EF24 E8BF26F9FF       CALL     004215E8    改>>>0041FA90

018F:0048EF29 8BCF             MOV      ECX,EDI

018F:0048EF2B 8BD3             MOV      EDX,EBX

018F:0048EF2D E852DDF8FF       CALL     0041CC84    改>>>0041B12C

018F:0048EF32 8B5508           MOV      EDX,[EBP+08]

018F:0048EF35 8B52FC           MOV      EDX,[EDX-04]

018F:0048EF38 8B92E0020000     MOV      EDX,[EDX+02E0]

018F:0048EF3E 3B8228000000     CMP      EAX,[EDX+0228]

018F:0048EF44 7512             JNZ      0048EF58

018F:0048EF46 55               PUSH     EBP

018F:0048EF47 33C0             XOR      EAX,EAX

018F:0048EF49 E8F2FCFFFF       CALL     0048EC40    >>>不用改

018F:0048EF4E 59               POP      ECX

018F:0048EF4F 55               PUSH     EBP

018F:0048EF50 E837FDFFFF       CALL     0048EC8C    >>>不用改

018F:0048EF55 59               POP      ECX

018F:0048EF56 EB1B             JMP      SHORT 0048EF73

018F:0048EF58 55               PUSH     EBP

018F:0048EF59 8B8530F3FFFF     MOV      EAX,[EBP+FFFFF330]

018F:0048EF5F E88426F9FF       CALL     004215E8    改>>>0041FA90

018F:0048EF64 8BCF             MOV      ECX,EDI

018F:0048EF66 8BD3             MOV      EDX,EBX

018F:0048EF68 E817DDF8FF       CALL     0041CC84    改>>>0041B12C

018F:0048EF6D E8CEFCFFFF       CALL     0048EC40    >>>不用改

018F:0048EF72 59               POP      ECX

018F:0048EF73 FE853BF3FFFF     INC      BYTE [EBP+FFFFF33B]

018F:0048EF79 80BD3BF3FFFF07   CMP      BYTE [EBP+FFFFF33B],07

018F:0048EF80 760D             JNA      0048EF8F

018F:0048EF82 C6853BF3FFFF00   MOV      BYTE [EBP+FFFFF33B],00

018F:0048EF89 FF8534F3FFFF     INC      DWORD [EBP+FFFFF334]

018F:0048EF8F 83853CF3FFFF03   ADD      DWORD [EBP+FFFFF33C],BYTE +03

018F:0048EF96 8B4508           MOV      EAX,[EBP+08]

018F:0048EF99 8B40FC           MOV      EAX,[EAX-04]

018F:0048EF9C 8B80E0020000     MOV      EAX,[EAX+02E0]

018F:0048EFA2 668B8046020000   MOV      AX,[EAX+0246]

018F:0048EFA9 0FB7D0           MOVZX    EDX,AX

018F:0048EFAC 4A               DEC      EDX

018F:0048EFAD 3BDA             CMP      EBX,EDX

018F:0048EFAF 7530             JNZ      0048EFE1

018F:0048EFB1 8B5508           MOV      EDX,[EBP+08]

018F:0048EFB4 6683E810         SUB      AX,BYTE +10

018F:0048EFB8 7419             JZ       0048EFD3

018F:0048EFBA 6683E810         SUB      AX,BYTE +10

018F:0048EFBE 7521             JNZ      0048EFE1

018F:0048EFC0 81AD3CF3FFFFC000+SUB      DWORD [EBP+FFFFF33C],C0

018F:0048EFCA 83AD34F3FFFF08   SUB      DWORD [EBP+FFFFF334],BYTE +08

018F:0048EFD1 EB0E             JMP      SHORT 0048EFE1

018F:0048EFD3 83AD3CF3FFFF60   SUB      DWORD [EBP+FFFFF33C],BYTE +60
```



```
018F:0048EFDA 83AD34F3FFFF06   SUB      DWORD [EBP+FFFFF334],BYTE +06  
018F:0048EFE1 43               INC      EBX  
018F:0048EFE2 4E               DEC      ESI  
018F:0048EFE3 0F8535FFFFFF     JNZ      NEAR 0048EF1E  
018F:0048EFE9 47               INC      EDI  
018F:0048EFEA FF8D2CF3FFFF     DEC      DWORD [EBP+FFFFF32C]  
018F:0048EFF0 0F8509FFFFFF     JNZ      NEAR 0048EEFF  
018F:0048EFF6 33C0             XOR      EAX,EAX  
018F:0048EFF8 5A               POP      EDX  
018F:0048EFF9 59               POP      ECX  
018F:0048EFFA 59               POP      ECX  
018F:0048EFFB 648910           MOV      [FS:EAX],EDX  
018F:0048EFFE 6826D54800       PUSH     DWORD 0048D526  改>>>0048F016  
018F:0048F003 8B8530F3FFFF     MOV      EAX,[EBP+FFFFF330]  
018F:0048F009 E85259F7FF       CALL     00404960    改>>>00402E70  
018F:0048F00E C3               RET      
018F:0048F00F E9AC60F7FF       JMP      004050C0  
018F:0048F014 EBED             JMP      SHORT 0048F003  
018F:0048F016 8D95DCF1FFFF     LEA      EDX,[EBP+FFFFF1DC]  
018F:0048F01C 8B4508           MOV      EAX,[EBP+08]  
018F:0048F01F 8B40FC           MOV      EAX,[EAX-04]  
018F:0048F022 8B80DC020000     MOV      EAX,[EAX+02DC]  
018F:0048F028 E84F0BFCFF       CALL     0044FB7C    改>>>0044F3B4  
018F:0048F02D 8B95DCF1FFFF     MOV      EDX,[EBP+FFFFF1DC]  
018F:0048F033 8D85E0F1FFFF     LEA      EAX,[EBP+FFFFF1E0]  
018F:0048F039 E84089F7FF       CALL     0040797E    改>>>00405E8E  
018F:0048F03E 8B4508           MOV      EAX,[EBP+08]  
018F:0048F041 8B40FC           MOV      EAX,[EAX-04]  
018F:0048F044 8B80E0020000     MOV      EAX,[EAX+02E0]  
018F:0048F04A 668B8046020000   MOV      AX,[EAX+0246]  
018F:0048F051 6683E810         SUB      AX,BYTE +10  
018F:0048F055 741D             JZ       0048F074  
018F:0048F057 6683E810         SUB      AX,BYTE +10  
018F:0048F05B 752C             JNZ      0048F089  
018F:0048F05D BABE0C0000       MOV      EDX,0CBE  
018F:0048F062 8D85E0F1FFFF     LEA      EAX,[EBP+FFFFF1E0]  
018F:0048F068 E8FB8BF7FF       CALL     00407C68    改>>>00406178  
018F:0048F06D E86A52F7FF       CALL     004042DC    改>>>004027EC  
018F:0048F072 EB15             JMP      SHORT 0048F089  
018F:0048F074 BA7E030000       MOV      EDX,037E  
018F:0048F079 8D85E0F1FFFF     LEA      EAX,[EBP+FFFFF1E0]  
018F:0048F07F E8E48BF7FF       CALL     00407C68    改>>>00406178  
018F:0048F084 E85352F7FF       CALL     004042DC    改>>>004027EC  
018F:0048F089 6A00             PUSH     BYTE +00  
018F:0048F08B 8D9542F3FFFF     LEA      EDX,[EBP+FFFFF342]  
018F:0048F091 B901000000       MOV      ECX,01  
018F:0048F096 8D85E0F1FFFF     LEA      EAX,[EBP+FFFFF1E0]  
018F:0048F09C E87F89F7FF       CALL     00407A20    改>>>00405F30  
018F:0048F0A1 E83652F7FF       CALL     004042DC    改>>>004027EC  
018F:0048F0A6 8D85E0F1FFFF     LEA      EAX,[EBP+FFFFF1E0]  
018F:0048F0AC E8D389F7FF       CALL     00407A84    改>>>00405F94  
018F:0048F0B1 E82652F7FF       CALL     004042DC    改>>>004027EC  
018F:0048F0B6 EB04             JMP      SHORT 0048F0BC  
018F:0048F0B8 886199           MOV      [ECX-67],AH  
018F:0048F0BB 49               DEC      ECX  
018F:0048F0BC E843CDFFFF       CALL     0048BE04    改>>>0048B0A0  
018F:0048F0C1 33C0             XOR      EAX,EAX  
018F:0048F0C3 5A               POP      EDX  
018F:0048F0C4 59               POP      ECX  
018F:0048F0C5 59               POP      ECX  
018F:0048F0C6 648910           MOV      [FS:EAX],EDX  
018F:0048F0C9 EB0A             JMP      SHORT 0048F0D5  
018F:0048F0CB E93C5DF7FF       JMP      00404E0C  
018F:0048F0D0 E89360F7FF       CALL     00405168    改>>>00403678  
018F:0048F0D5 33C0             XOR      EAX,EAX  
018F:0048F0D7 5A               POP      EDX  
018F:0048F0D8 59               POP      ECX  
018F:0048F0D9 59               POP      ECX  
018F:0048F0DA 648910           MOV      [FS:EAX],EDX  
018F:0048F0DD 6805D64800       PUSH     DWORD 0048D605  改>>>0048F0F5  
018F:0048F0E2 8D85DCF1FFFF     LEA      EAX,[EBP+FFFFF1DC]  
018F:0048F0E8 E83B65F7FF       CALL     00405628    改>>>00403B38  
018F:0048F0ED C3               RET      
018F:0048F0EE E9CD5FF7FF       JMP      004050C0  
018F:0048F0F3 EBED             JMP      SHORT 0048F0E2  
018F:0048F0F5 5F               POP      EDI  
018F:0048F0F6 5E               POP      ESI  
018F:0048F0F7 5B               POP      EBX  
018F:0048F0F8 8BE5             MOV      ESP,EBP  
018F:0048F0FA 5D               POP      EBP  
018F:0048F0FB C3               RET      
```

由于是从`v2.3`版中dump出来的，许多偏移量都不一样，上面标着"改"的均需要改为后面的值。这是我用`DeDe`＋`HexWorkshop`＋偏移量转换器，慢慢的一个个找出来的，有几十个之多，修改的过程是痛苦的。载入调试的时候可是心惊胆战，万一没有效果，我可要吐血了，上天保佑，居然顺利的跑到`RET`了，看一下文件夹，哇存好了！

大功告成，亲个嘴吧，呵呵呵^\_^ `4dd466d0e1e0cc123afbe504da2462ca`
