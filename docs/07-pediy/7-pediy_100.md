
# Mass Downloader v2.2.223 SR1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue304.htm`](https://www.pediy.com/kssd/pediy04/kanxue304.htm)

```
/////////////////////////////////////////////////////////////////////    
//   
//  目标软件：Mass Downloader   
//   
//  软件版本：v2.2.223 SR1   
//   
//  官方网站：http://www.metaproducts.com/   
//   
//  软件授权：共享软件   
//   
//  操作系统：Win95/98/ME、WinNT/2000   
//   
//  软件简介：   
//   
//  十分不错的下载工具软件，功能强大。支持多线程下载、断点续传，也具有   
//   
//  一定的下载文件管理能力。   
//   
//  软件保护：一般序列号保护、ASPACK 加壳   
//   
/////////////////////////////////////////////////////////////////////   

//   
//  使用工具：TRW2000 v1.22 娃娃修改版   
//               (主要用于调试分析)   
//   
//               HIEW v6.70 Registered - CORE   
//               (16 进制编辑、制作 SMC 版本)   
//   
//               Our Brain...:-)   
//   
/////////////////////////////////////////////////////////////////////   

//   
//  关于本文：本文主要目的在于教学，让初学者掌握一些基本的脱壳方法及   
//   
//               软件分析手段...请勿将此教程用于商业目的。   
//   
//                       Always Your Best Friend: FiNALSErAPH   
//   
//  水平有限，难免疏漏...   
//   
//  Any Question?   
//  Mail To: FiNALSErAPH@yahoo.com.cn   
//   
//                       2001-12-21   
//   
/////////////////////////////////////////////////////////////////////   

//   
//  第一部分：注册码校验分析 (以 Single User License 为例)   
//   
//               为了便于说明，使用 FiNALSErAPH 作用户名，7878787878787878    
//   
//               为输入的注册码   
//   
/////////////////////////////////////////////////////////////////////   

//   
//  为了更方便地找到注册码比对点，这里使用 HMEMCPY 作为断点，我假设读者   
//   
//  已经熟悉这一过程...如果你仍感到困惑...:-)继续努力吧   
//   
/////////////////////////////////////////////////////////////////////   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004EDDEA(C)   

|   

:004EDE0A 8D55FC               lea edx, dword ptr [ebp-04]   

:004EDE0D 8B83EC020000         mov eax, dword ptr [ebx+000002EC]   

:004EDE13 E8ACAAF4FF           call 004388C4   

                              //我们以 HMEMCPY 作断点，F12 多次，会   

                              //来到此处...   

:004EDE18 8D55F8               lea edx, dword ptr [ebp-08]   

:004EDE1B 8B83F4020000         mov eax, dword ptr [ebx+000002F4]   

:004EDE21 E89EAAF4FF           call 004388C4   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004EDE08(U)   

|   

* Possible StringData Ref from Code Obj ->"U = "   

                              |   

:004EDE26 68CCDF4E00           push 004EDFCC   

:004EDE2B FF75FC               push [ebp-04]   

* Possible StringData Ref from Code Obj ->", K = "   

                              |   

:004EDE2E 68DCDF4E00           push 004EDFDC   

:004EDE33 FF75F8               push [ebp-08]   

:004EDE36 8D45F4               lea eax, dword ptr [ebp-0C]   

:004EDE39 BA04000000           mov edx, 00000004   

:004EDE3E E8C962F1FF           call 0040410C   

:004EDE43 8B55F4               mov edx, dword ptr [ebp-0C]   

:004EDE46 8B07                 mov eax, dword ptr [edi]   

:004EDE48 E837DC0000           call 004FBA84   

:004EDE4D 8D55FC               lea edx, dword ptr [ebp-04]   

:004EDE50 8B07                 mov eax, dword ptr [edi]   

:004EDE52 8B4DF8               mov ecx, dword ptr [ebp-08]   

:004EDE55 E8FAE70100           call 0050C654   

:004EDE5A 84C0                 test al, al   

:004EDE5C 0F848C000000         je 004EDEEE   

                              //很明显，上面的调用就是校验的核心   

                              //F8 进入 004EDE55 处的调用   

/////////////////////////////////////////////////////////////////////   

* Referenced by a CALL at Addresses:   

|:004EDE55  , :004F9885   

|   

:0050C654 55                   push ebp   

:0050C655 8BEC                 mov ebp, esp   

:0050C657 6A00                 push 00000000   

:0050C659 6A00                 push 00000000   

:0050C65B 6A00                 push 00000000   

:0050C65D 6A00                 push 00000000   

:0050C65F 6A00                 push 00000000   

:0050C661 53                   push ebx   

:0050C662 894DFC               mov dword ptr [ebp-04], ecx   

:0050C665 8BDA                 mov ebx, edx   

:0050C667 8B45FC               mov eax, dword ptr [ebp-04]   

:0050C66A E8917BEFFF           call 00404200   

:0050C66F 33C0                 xor eax, eax   

:0050C671 55                   push ebp   

:0050C672 6857CB5000           push 0050CB57   

:0050C677 64FF30               push dword ptr fs:[eax]   

:0050C67A 648920               mov dword ptr fs:[eax], esp   

:0050C67D C645FB00             mov [ebp-05], 00   

:0050C681 8B55FC               mov edx, dword ptr [ebp-04]   

* Possible StringData Ref from Code Obj ->"dqma"   

                              |   

:0050C684 B870CB5000           mov eax, 0050CB70   

:0050C689 E8AA7CEFFF           call 00404338   

:0050C68E 85C0                 test eax, eax   

:0050C690 7E12                 jle 0050C6A4   

:0050C692 8D55FC               lea edx, dword ptr [ebp-04]   

:0050C695 8BC3                 mov eax, ebx   

:0050C697 E804F6FFFF           call 0050BCA0   

:0050C69C 8845FB               mov byte ptr [ebp-05], al   

:0050C69F E990040000           jmp 0050CB34   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0050C690(C)   

|   

:0050C6A4 8B03                 mov eax, dword ptr [ebx]   

* Possible StringData Ref from Code Obj ->"angus rod"   

                              |   

:0050C6A6 BA80CB5000           mov edx, 0050CB80   

:0050C6AB E8AC7AEFFF           call 0040415C   

:0050C6B0 0F847E040000         je 0050CB34   

:0050C6B6 8B03                 mov eax, dword ptr [ebx]   

* Possible StringData Ref from Code Obj ->"Henry Hooker"   

                              |   

:0050C6B8 BA94CB5000           mov edx, 0050CB94   

:0050C6BD E89A7AEFFF           call 0040415C   

:0050C6C2 0F846C040000         je 0050CB34   

:0050C6C8 8B03                 mov eax, dword ptr [ebx]   

* Possible StringData Ref from Code Obj ->"David Arthurs"   

                              |   

:0050C6CA BAACCB5000           mov edx, 0050CBAC   

:0050C6CF E8887AEFFF           call 0040415C   

:0050C6D4 0F845A040000         je 0050CB34   

:0050C6DA 8B03                 mov eax, dword ptr [ebx]   

 .   

 .   

 .   

//这是很长的一串程序，且重复性很强。说白了，作者写的 BlackList   

//(哈哈，我倒是希望作者将我的 Nick 加入...:-))   

//我们直接略过，来到下面地址   

* Possible StringData Ref from Code Obj ->"Mass Downloader 1"   

                              |   

:0050CA33 6894CF5000           push 0050CF94   

:0050CA38 FF33                 push dword ptr [ebx]   

* Possible StringData Ref from Code Obj ->"Single"   

                              |   

:0050CA3A 68B0CF5000           push 0050CFB0   

:0050CA3F 8D45F4               lea eax, dword ptr [ebp-0C]   

:0050CA42 BA03000000           mov edx, 00000003   

:0050CA47 E8C076EFFF           call 0040410C   

:0050CA4C BB01000000           mov ebx, 00000001   

:0050CA51 EB2B                 jmp 0050CA7E   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0050CA88(C)   

|   

:0050CA53 8B45FC               mov eax, dword ptr [ebp-04]   

:0050CA56 8A4418FF             mov al, byte ptr [eax+ebx-01]   

:0050CA5A 04D0                 add al, D0   

:0050CA5C 2C0A                 sub al, 0A   

:0050CA5E 721D                 jb 0050CA7D   

:0050CA60 04F9                 add al, F9   

:0050CA62 2C1A                 sub al, 1A   

:0050CA64 7217                 jb 0050CA7D   

:0050CA66 04FA                 add al, FA   
```



:0050CA68 `2C1A`              `sub al, 1A`

:0050CA6A `7211`              `jb 0050CA7D`

:0050CA6C `8D45FC`            `lea eax, dword ptr [ebp-04]`

:0050CA6F `B901000000`        `mov ecx, 00000001`

:0050CA74 `8BD3`              `mov edx, ebx`

:0050CA76 `E81978EFFF`        `call 00404294`

:0050CA7B `EB01`              `jmp 0050CA7E`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :0050CA5E(C), :0050CA64(C), :0050CA6A(C)

|

:0050CA7D `43`                `inc ebx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| :0050CA51(U), :0050CA7B(U)

|

:0050CA7E `8B45FC`            `mov eax, dword ptr [ebp-04]`

//在这里，`D EAX` 就可以见到我们输入的

//错误注册码，下 `BPM EAX` 的断点，`F5` 继

//续(按 2 次，就会发现如下的经典比对)。

///经典比对程序片段

:00404185 `8B0E`              `mov ecx, dword ptr [esi]`

:00404187 `8B1F`              `mov ebx, dword ptr [edi]`

:00404189 `39D9`              `cmp ecx, ebx`

:0040418B `7558`              `jne 004041E5`

//在这里分别 `D ESI`、`D EDI` 就可以看到

//参与比对的字符串，一个是我们输入

//的错误注册码，另一个...:-)赶紧用

//笔记录下来。

//在此处，我的用户名对应的值为：

//`8856852571979FL`

//`88568-52571979-FL(v2.2.212)`

//这个版本似乎没有对格式作严格要求

:0050CA81 `E8C675EFFF`        `call 0040404C`

:0050CA86 `3BD8`              `cmp ebx, eax`

:0050CA88 `7EC9`              `jle 0050CA53`

.

.

.

//找到正确的注册码了? 恐怕高兴得太早了...:-)不信试试看...

//此处还有如下的关键比对点

\* Possible StringData Ref from Data Obj ->`")?"`

|

:0050CB1F `B818455400`        `mov eax, 00544518`

//此处 `D EAX`，可以看到...我想是作者计

//算出的正版用户信息吧

//(这只是猜测，没有作具体分析)

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0050CB32(C)

|

:0050CB24 `3B10`              `cmp edx, dword ptr [eax]`

//注意此处的 `EDX` 值，这是由我的用户名

//算出的待校验值。我这里是 `03222F4B`

:0050CB26 `7506`              `jne 0050CB2E`

:0050CB28 `C645FB01`          `mov [ebp-05], 01`

//呵呵，是否注册成功的标志。我可不是

//正版用户，所以按照正常流程，我不可

//能到达这里，不过如果我将此时 `EDX` 的

//值预先置于 `00544518` 指向的地址空间，

//我就可以成功注册

:0050CB2C `EB06`              `jmp 0050CB34`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0050CB26(C)

|

:0050CB2E `83C004`            `add eax, 00000004`

:0050CB31 `4B`                `dec ebx`

:0050CB32 `75F0`              `jne 0050CB24`

.

.

.

//哦，错误注册的出口

///

//

// 第二部分：`SMC` 补丁制作

///

找到 `ASPACK` 解压缩完成，即将进入原始程序执行的地址。为什么? 这时候最好

打补丁嘛

具体方法：

我们就利用注册码错误的提示信息，下 `BPX MESSAGEBOXA` 的断点，中断并回到

主程序的领空后，记住当前的地址(在我这里是 `00456F56`)。清除当前的所有断

点，并下新断点 `BPM 00456F56`。重新运行程序...中断后再按一次 `F5`，你会看

见当前的指令 `REP MOVSD`，恭喜...你已经离原始程序入口不远了。向下搜索代

码，直到看到如下指令：

```
0167:0059F3BA 61              POPA
0167:0059F3BB 7508            JNZ     0059F3C5
0167:0059F3BD B801000000      MOV     EAX, 01
0167:0059F3C2 C20C00          RET     0C
0167:0059F3C5 6800000000      PUSH    DWORD 00                ***
0167:0059F3CA C3              RET
```

如果你曾研究过 `ASPACK` 的脱壳，就会很熟悉这一段程序了。现在我们注意到位于 `0059F3C5` 处的指令还没有变化，所以下断点 `BPX 0059F3BA`，见到变化如下：

```
0167:0059F3BA 61              POPA
0167:0059F3BB 7508            JNZ     0059F3C5
0167:0059F3BD B801000000      MOV     EAX, 01
0167:0059F3C2 C20C00          RET     0C
0167:0059F3C5 6858205400      PUSH    DWORD 00542058      ***OEP
0167:0059F3CA C3              RET
```

现在就可以打补丁了吗? 好象还不行哦。因为这是高版本的 `ASPACK` 压缩的，所以无法找到 `6800000000C3` 这一特征串，也就无法直接修改了。

先暂时不管这个问题，直接修改内存指令，作出补丁：

```
0167:0059F3BA 61              POPA
0167:0059F3BB E960030000      JMP     0059F720
                              //这是我找到的一处空闲段，直接在
                              //`TRW2000` 中用 `A` 命令修改
0167:0059F3C0 90              NOP
.
.
.
0167:0059F3CA 90              NOP
0167:0059F720 C7051884540003222F4B MOV    D, [00544518], 4B2F2203
0167:0059F72A 6858205400      PUSH    00542058
0167:0059F3CA C3              RET
                              //这是用 `HIEW` 修改的
```

好了，现在只剩下一个问题：如何正确地修改位于 `0059F3C5` 处的指令?

下 `BPM 0059F3BA` 的断点，断下后可以发现如下指令：

```
0167:0059F4FB 80043EE1        ADD     BYTE [ESI+EDI], E1
                              //其实就是减 `1F` 啦
0167:0059F4FF B14D            MOV     CL, 4D
                              //我们停在这里，可是当我们查看上
                              //面的指令会看到一个错误指令，这
                              //应该是 `ASPACK` 作者耍的花招了，一
                              //种花指令。你可以通过 `F10` 多单步执
                              //行一段，重新回到 `0059F4FB` 时，就
                              //可以看到正确的指令了
0167:0059F501 B26A            MOV     DL, 6A
```

现在，我们可以计算出 `0059F3BB` 处的数据了。

`E9 - 08`  `60 - 7F`  `03 - 22`  `00 - 1F`  `00 - 1F`  `90 - AF` ...  `90 - AF`

///

//

// 第三部分：补充说明

///

以上的过程对 `v2.2.212` 来说，已经完成了整个解密过程。但是对这个版本还有

遗留问题：并没有解决过期问题，即时间到了还是会提示注册。为了解决这个

问题，我以是否注册的标志作为断点，找到了最后的置标志的指令：

```
0167:004FC03B C6831C08000000  MOV     BYTE [EBX+081C], 00
```

所以只要将补丁程序改为如下：

```
0167:0059F720 C7051884540003222F4B MOV    D, [00544518], 4B2F2203
0167:0059F72A                       MOV    B, [004FC041], 1      ***
0167:0059F72A 6858205400          PUSH    00542058
0167:0059F3CA C3                  RET
                              //这是用 `HIEW` 修改的
```

(这一修改显得底气不足，毫无完美破解的美感)

///

//

// 第四部分：后记

///

现在，这个修改版本看起来就和正版注册用户一样了。但是，这个结论并不完

美，究竟作者在程序启动时多做了哪些校验? 我还不得而知...希望有兴趣的

自己进行研究。我也没有对注册码的形成算法作进一步的分析...如果读者对算

法有浓厚的兴趣，并有所收获，请不忘告诉我。热情期待有进一步的文章...

曾在“看雪”的论坛上看到“炎之川”的回帖：只改动 `3` 个字节。不知是如何做

到的? 有知道的请告诉我

谨将此文送给 `Turkey99`，没有他的热情回复，就没有这篇拙文...:-) `4dd466d0e1e0cc123afbe504da2462ca`
