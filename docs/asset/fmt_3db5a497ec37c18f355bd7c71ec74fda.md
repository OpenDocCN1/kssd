
# crackcode2000 注册机篇

## 第一种：

`[Options]`

`CommandLine=regeditor.exe`

`Mode=2`

`First_Break_Address=403EDF`

`First_Break_Address_Code=39`

`First_Break_Address_Code_Lenth=2`

`Save_Code_Address=edx`

## 第二种：

`[Options]`

`CommandLine=regeditor.exe`

`Mode=0`

`First_Break_Address=470219`

`First_Break_Address_Code=8B`

`First_Break_Address_Code_Lenth=2`

`Save_Code_Address=EAX`

## 第三种：

`[Options]`

`CommandLine=regeditor.exe`

`Mode=3`

`First_Break_Address=403EDF`

`First_Break_Address_Code=39`

`First_Break_Address_Code_Lenth=2`

`Save_Code_Address=edx`

(以上 It's very cool! Why? 注册码错误也成功注册! 不信你重启就成的注册版 不过就是用它注册所以文字都失踪不过也能注册!)

相信连我师傅也可能不知道有这么利害的功能？! 哈。。。哈！

`HKEY_LOCAL_MACHINE\`
  `Software\CLASSES\`
  `CLSID\`
  `{A4E78360-03CA-11D4-9395-981897B1F059}\`
  `Version\[(默认)FFFF]`

---->删除这个值又变回未注册版!

```
注册表暴力把它的变成 FFFF 就是注册版!
```

# 暴力破解窗口篇：

```
0167:0047AA98 E87F54FFFF       CALL    0046FF1C
0167:0047AA9D 84C0             TEST    AL,AL
0167:0047AA9F 7553             JNZ       0047AAF4//----jz
0167:0047AAA1 8B0D04C24700     MOV       ECX,[0047C204]
0167:0047AAA7 8B09             MOV       ECX,[ECX]
0167:0047AAA9 B201             MOV       DL,01
0167:0047AAAB A1BCA34700       MOV       EAX,[0047A3BC]
0167:0047AAB0 E8EFC6FCFF       CALL    004471A4
0167:0047AAB5 A358D94700       MOV       [0047D958],EAX
0167:0047AABA 33C0             XOR       EAX,EAX
0167:0047AABC 55               PUSH    EBP
0167:0047AABD 68EDAA4700       PUSH    DWORD 0047AAED
0167:0047AAC2 64FF30           PUSH    DWORD [FS:EAX]
0167:0047AAC5 648920           MOV       [FS:EAX],ESP
0167:0047AAC8 A158D94700       MOV       EAX,[0047D958]
0167:0047AACD 8B10             MOV       EDX,[EAX]
0167:0047AACF FF92D8000000     CALL    NEAR [EDX+D8]//---->注册提示窗!
0167:0047AAD5 33C0             XOR       EAX,EAX
```

但仍然在退出时出现没有未注册提示,这个提示非常难破解!

# 自动脱壳篇：

工具: `fi(三个版)` and `fs` `[测壳工具]`, `Procdump32162(脱壳工具)`

测试结果：

`UPX v0.94 (■PE) ObHu/Molnar UPX\`
`UPX v0.93 (■PE) Bl.Panther UPX fi`
`UPX v0.94 (■PE) ObHu/Molnar UPX/`
`pe/exe.packer UPX 0.82-1.01 (type 2)-----unpacker (fs)`

用"pr"选 `upx` 一会就脱壳成功！

# 手动脱壳篇：

参考：

## chm 帮助编辑器 v2.6 注册码破解详谈之一*脱壳篇*

其实没这个必要,是应 `peterchen` 要求所写!

就先说说软件的脱壳吧!它用 `upx` 加壳.

**第一种脱壳方法: `trw`+冲击波 `0.2`**

启动冲击波,点击 `Trace`,然后运行要脱壳的软件,此时冲击波会显示一个数值(这里应该是 `00488C78`),这就是程序的入口.

`trw` 载入文件, `Ctrl+N`,

```
g 00488C78
MakePe 或 pedump
```

脱壳成功!

**第二种脱壳方法:只用 `trw`.**

用 `trw` 载入程序,选择加载,这时 `TRW` 弹出,按一次 `F10`,程序来到这里:

```
:004BA000 60                   pushad
:004BA001 E800000000           call 004BA006
```

一直按 `F10` 运行到上面这一行,注意这个 `call` 一定要进去,因为你再按一次 `F10` 的话,程序就运行了!

于是按 `F8` 进入:

```
* Referenced by a CALL at Address:
|:004BA001
|
:004BA006 83CDFF               or ebp, FFFFFFFF
:004BA009 31DB                 xor ebx, ebx
:004BA00B 5E                   pop esi
:004BA00C 8DBEFA6FF4FF         lea edi, dword ptr [esi+FFF46FFA]
:004BA012 57                   push edi
:004BA013 668187F4830B000100   add word ptr [edi+000B83F4], 0001
:004BA01C 81C6B3010000         add esi, 000001B3
:004BA022 EB0A                 jmp 004BA02E
```

到了这里,再继续按 `F10` 跳转到 `004BA02E`,继续跟踪

```
:004BA02E 01DB                 add ebx, ebx
:004BA030 7507                 jne 004BA039
:004BA032 8B1E                 mov ebx, dword ptr [esi]
:004BA034 83EEFC               sub esi, FFFFFFFC
:004BA037 11DB                 adc ebx, ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004BA030(C)
|
:004BA039 72ED                 jb 004BA028
:004BA03B B801000000           mov eax, 00000001
```

在 `004BA039` 这一行程序将*向上*跳到 `004BA028`,进行循环解压,而我们没有必要跟着它走,所以将光标定位于 `004BA03B`,按 `F7`,这样就省去了许多步骤了,接着软件还会有很多类似于这里的地方,都可以用这种方法跳过去.若是程序向下跳跃的时候,就不要多此一举了,让它跳就是了,还要注意的是按 `F7` 前,最好先在跳转的地方设个断点,再 `bd` 掉,这样即使判断错误,也还有的救,`be` 就可以了继续跟踪了.继续向下走:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004BA04F(C), :004BA05A(C)
|
:004BA040 01DB                 add ebx, ebx
:004BA042 7507                 jne 004BA04B      \\这里会*向下*跳转,让它跳就是了.
:004BA044 8B1E                 mov ebx, dword ptr [esi]
:004BA046 83EEFC               sub esi, FFFFFFFC
:004BA049 11DB                 adc ebx, ebx
```

这两种情况已经明白了吧!那我就省去一些了.

来到这里

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004BA177(C)
|
:004BA18A 240F                 and al, 0F
:004BA18C C1E010               shl eax, 10
:004BA18F 668B07               mov ax, word ptr [edi]
:004BA192 83C702               add edi, 00000002
:004BA195 EBE2                jmp 004BA179

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004BA173(C)
|
:004BA197 61                   popad  运行到这里以后看看下面这个跳转
:004BA198 E9DBEAFCFF           jmp 00488C78     //这里程序便完成来内存中的解工作,
                                               将要跳到程序入口点 00488C78
```

然后就来到了这里:

```
00488C78  PUSH EBP    //在这里下 MAKEPE 命令(但我的 TRW 经常出错,Why)或是 Pedump
00488C79  MOV  EBP,ESP  但若你用了 Pedump 那脱壳后的文件将不能在其它操作系统和
                        其它机器上运行!
```

工具: `trw2000`, `Bw2k02(冲击波)`

惜助工具脱壳：

1. 把"bw" copy 到你要脱壳的软件目录中
2. 先启动它
3. 再启动要脱壳的软件

使用法：

`Track:` 开始进行跟踪，然后运行要跟踪的程序.
找到入口点后，会在 `Enter point` 显示.

`Stop :` 停止跟踪

`Exit :` 退出

我的 `epo` 是 `0047AA50`

用 `trw2000`, `ctrl+n`

```
g 47aa50
PEDUMP
```

为什么不用 `MAKEPE`? 我也与 `TAE!` 一样！

```
0167:004A87FB  ADD       EBX,BYTE +04--------->F9
0167:004A87FE  JMP       SHORT 004A87E1//----->注意这里是环循？
0167:004A8800  CALL    NEAR [ESI+000A96DC]--->来到这里就显示注册窗！
0167:004A8806  ADD       EDI,BYTE +04-------->光标放在这里按 F7!
```

为什么要这样做了？由于来到环循点不可能让跳回去要向下走才行，为了怕判定之误所以要中断不然又要再来！这里就什么搞鬼？可逼非回去不可也就是不给你脱壳！所以只要发现你跳下一行就让注册窗提早出来！这也是程序流程的定律！不过你只要一下跳多一行就行了！

```
0167:004A8832  ADD       EDI,BYTE +02
0167:004A8835  JMP       SHORT 004A8819
0167:004A8837  POPA//----------->非常明显的标志!
0167:004A8838  JMP       0047AA50
```

```
0167:0047AA50  PUSH    EBP//----------->真正的入口点!不过要注意不要把光标
0167:0047AA51  MOV      EBP,ESP          不然这个入口点变样!不是如此还有
0167:0047AA53  ADD      ESP,BYTE -0C      汇编我不知道你有没有看台湾火大客
0167:0047AA56  MOV      EAX,0047A7F8      加、脱壳教程，也许是加了壳所以有
0167:0047AA5B  CALL    004065E0          这样的变化！
0167:0047AA60  MOV      EAX,[0047C204]
0167:0047AA65  MOV      EAX,[EAX]
0167:0047AA67  CALL    0044E138
0167:0047AA6C  MOV      EAX,[0047C204]
0167:0047AA71  MOV      EAX,[EAX]
0167:0047AA73  ADD      EAX,BYTE +40
0167:0047AA76  MOV      EDX,0047AB10
```
