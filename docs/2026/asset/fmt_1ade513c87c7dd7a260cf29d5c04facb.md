
## 寻找出错原因

修复IAT必须关闭`OD`，单独开加壳程序修复，不然`ImportREC`停止响应。最后用`ImportREC`的`Oep`填`B7220`，跟踪等级三轻松修复，无法运行！？

``` 
004F01F8   /78 01           js short dumped\_.004F01FB
004F01FA   |F8              clc
004F01FB   BF 58D5CC8F     mov edi, 8FCCD558
004F0200    8B45 00         mov eax, dword ptr ss:[ebp]
004F0203    7C 03           jl short dumped\_.004F0208
004F0205    7D 01           jge short dumped\_.004F0208
004F0207    E9 E9040000     jmp dumped\_.004F06F5
004F020C    004E 66         add byte ptr ds:[esi+66], cl
004F020F    1BD9            sbb ebx, ecx
004F0211    33C2            xor eax, edx
004F0213    50              push eax
004F0214    E8 01000000     call dumped\_.004F021A //发现这个Call可能是新版Acprotect的解码技术，和完美卸载Xp里面的一样的，这一段修复一直没搞定，比原先的Jmp xxxxx  那种语句难多了。
004F0219    EA 58587B05 668>jmp far 8166:057B5858
```

不好意思，文章没写完，卡住了。

今天看到Fly大侠的文章，呵呵，明白是什么一会事。

其实本版本Acprotect嵌有`Oep`处代码检验程序，发现`Oep`处代码被替换，立即拒绝解码，你失去了关键的代码，当然程序无法运行，还要承认`N`次错误确定按钮。

``` 
004254C9    55              push ebp       //Stolen Code
004254CA    8BEC            mov ebp, esp  //Stolen Code   //一切尽在掌握
004254CC    83EC 44         ADD ESP,-10  //Stolen Code
```

我们已经掌握。

``` 
004B7226    B8 A86F4B00     mov eax, Pey.004B6FA8   //临时Oep也知道。
```

现在做什么,让程序认为你没有脱壳,乖乖将`perplex`段里面有用的内容全部解码。

## 方法

``` 
push ebp          //Stolen Code
mov ebp, esp     //Stolen Code   //一切尽在掌握
ADD ESP,-10     //Stolen Code
jmp  004B7226  //直接跳向临时Oep
```

用`Od`载入脱壳修复后的程序，`Ctrl+G` `004EF000`。

``` 
004EF000 >/$  68 420E715B   PUSH 5B710E42  //4EF000是原来壳的入口点，狸猫换太子，将Oep入口代码复制到这里。
004EF005  |.  813424 A27989>XOR DWORD PTR SS:[ESP],2C8979A2
004EF00C  .  C3            RETN
004EF00D   .  68 59266136   PUSH 36612659
004EF012   .  813424 248C99>XOR DWORD PTR SS:[ESP],41998C24
004EF019   .  C3            RETN
004EF01A   .  68 24C959F5   PUSH F559C924
004EF01F   .  813424 6863A1>XOR DWORD PTR SS:[ESP],82A16368
004EF026   .  C3            RETN
004EF027   .  68 CF904873   PUSH 734890CF
004EF02C   .  813424 C01DAE>XOR DWORD PTR SS:[ESP],4AE1DC0
004EF033   .  C3            RETN
```

---

``` 
004EF000 > .  55            PUSH EBP
004EF001   .  8BEC          MOV EBP,ESP
004EF003   .  83C4 F0       ADD ESP,-10
004EF006   .- E9 1B82FCFF   JMP dumpedw4.004B7226
004EF00B      90            NOP   //注意保持代码完整，填入一个Nop
004EF00C   .  C3            RETN
004EF00D   .  68 59266136   PUSH 36612659
004EF012   .  813424 248C99>XOR DWORD PTR SS:[ESP],41998C24
004EF019   .  C3            RETN
004EF01A   .  68 24C959F5   PUSH F559C924
004EF01F   .  813424 6863A1>XOR DWORD PTR SS:[ESP],82A16368
004EF026   .  C3            RETN
004EF027   .  68 CF904873   PUSH 734890CF
004EF02C   .  813424 C01DAE>XOR DWORD PTR SS:[ESP],4AE1DC0
004EF033   .  C3            RETN
```

将替换的代码复制到程序中，另存为一个文件，用`PEedit`修正入口为`EF000`，正常运行。

## 破解总结

发现Dephi语言的Acprotect的壳采用了Stolen Code，动态解码，用Acprotect保护的它人没加上`RSAKEY`就有强劲的保护，加`RSAKEY`保护壳继续研究中，感谢你耐心看完。
