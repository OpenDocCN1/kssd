
# Araxis Merge 2001 Professional Evaluation v6.0 时间限制去除

(7千字)

by liangs

## (1)去除Nag和30天限制

根据Nag的标题"Welcome to Araxis Merge"我们在W32Dasm中可以查到那个窗口的ID是`DialogID_0FA8`，再查`DialogID_0FA8`在程序中出现的地方，可找到如下代码处：

```
:00469A79 3BF3 cmp esi, ebx
:00469A7B 8B55F0 mov edx, dword ptr [ebp-10]
:00469A7E 8B45EC mov eax, dword ptr [ebp-14]
:00469A81 8990000E0000 mov dword ptr [eax+00000E00], edx
:00469A87 7C05 jl 00469A8E //改为EB05
:00469A89 83FE1E cmp esi, 0000001E //判断时间是否大于30天
:00469A8C 7E05 jle 00469A93
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00469A77(U), :00469A87(C)
|
:00469A8E BE1E000000 mov esi, 0000001E
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00469A8C(C)
|
:00469A93 395D08 cmp dword ptr [ebp+08], ebx
:00469A96 747D je 00469B15 //改为EB7D
:00469A98 53 push ebx
* Possible Reference to Dialog: DialogID_0FA8 //Nag过期窗口提示
★★★ 一定要跳过这个Nag，上面的两个修改都是为了要Jump过它
:00469A99 68A80F0000 push 00000FA8
:00469A9E 8D8D04FFFFFF lea ecx, dword ptr [ebp+FFFFFF04]
:00469AA4 E81D270700 call 004DC1C6
:00469AA9 B91E000000 mov ecx, 0000001E
:00469AAE 2BCE sub ecx, esi
:00469AB0 898D60FFFFFF mov dword ptr [ebp+FFFFFF60], ecx
:00469AB6 8D8D64FFFFFF lea ecx, dword ptr [ebp+FFFFFF64]
:00469ABC C645FC03 mov [ebp-04], 03
:00469AC0 E8FB5F0000 call 0046FAC0
:00469AC5 A140E95300 mov eax, dword ptr [0053E940]
:00469ACA 8945A0 mov dword ptr [ebp-60], eax
:00469ACD 8945A4 mov dword ptr [ebp-5C], eax
:00469AD0 68AC355400 push 005435AC
:00469AD5 8D4DA0 lea ecx, dword ptr [ebp-60]
:00469AD8 C645FC06 mov [ebp-04], 06
:00469ADC C78504FFFFFF34AD5000 mov dword ptr [ebp+FFFFFF04], 0050AD34
:00469AE6 E80E1E0700 call 004DB8F9
:00469AEB 68AC355400 push 005435AC
:00469AF0 8D4DA4 lea ecx, dword ptr [ebp-5C]
:00469AF3 E8011E0700 call 004DB8F9
:00469AF8 8D8D04FFFFFF lea ecx, dword ptr [ebp+FFFFFF04]
:00469AFE C645FC07 mov [ebp-04], 07
:00469B02 E895270700 call 004DC29C
:00469B07 8D8D04FFFFFF lea ecx, dword ptr [ebp+FFFFFF04]
:00469B0D 885DFC mov byte ptr [ebp-04], bl
:00469B10 E89B29FBFF call 0041C4B0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00469A96(C)
|
:00469B15 83FE1E cmp esi, 0000001E
:00469B18 8D4DA8 lea ecx, dword ptr [ebp-58]
:00469B1B 0F9CC3 setl bl
:00469B1E C745FCFFFFFFFF mov [ebp-04], FFFFFFFF
:00469B25 E8E676FAFF call 00411210
:00469B2A 8B4DF4 mov ecx, dword ptr [ebp-0C]
:00469B2D 5F pop edi
:00469B2E 5E pop esi
:00469B2F 8AC3 mov al, bl
★★★ 这个很重要，一定要使al为1
下面我们将会看到这点
:00469B31 64890D00000000 mov dword ptr fs:[00000000], ecx
:00469B38 5B pop ebx
:00469B39 8BE5 mov esp, ebp
:00469B3B 5D pop ebp
:00469B3C C20400 ret 0004
这里将返回到00412ABC的调用
:00469B3F 90 nop
```

-----------------------------------------------------------

为使`00469B3C`处返回时，`al=1`我们可以修改上面代码如下:

```
:00469B2F 8AC3 mov al, bl //改为 xor eax,eax
          33c0
:00469B31 64890D00000000 mov dword ptr fs:[00000000], ecx
:00469B38 5B pop ebx
:00469B39 8BE5 mov esp, ebp
:00469B3B 5D pop ebp
插入语句 40 inc eax //这里使al=1
:00469B3C C20400 ret 0004
:00469B3F 90 nop //这个nop将被占用
```

------------------------------------------------------------

下面代码为上面子程序的返回后的处理，程序会继续处理时间判断：

```
* Reference To: KERNEL32.GetLocalTime, Ord:011Bh
            |
:00412A87 FF151C645000 Call dword ptr [0050641C]
:00412A8D 8B45E0 mov eax, dword ptr [ebp-20]
:00412A90 663DD007 cmp ax, 07D0
:00412A94 722E jb 00412AC4
:00412A96 663DD207 cmp ax, 07D2
:00412A9A 7728 ja 00412AC4
:00412A9C 7513 jne 00412AB1
:00412A9E 668B45E2 mov ax, word ptr [ebp-1E]
:00412AA2 663D0300 cmp ax, 0003
:00412AA6 771C ja 00412AC4
:00412AA8 7507 jne 00412AB1
:00412AAA 66837DE61E cmp word ptr [ebp-1A], 001E
:00412AAF 7713 ja 00412AC4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00412A9C(C), :00412AA8(C)
|
:00412AB1 53 push ebx
:00412AB2 B960365400 mov ecx, 00543660
:00412AB7 E8446E0500 call 00469900
:00412ABC 84C0 test al, al
★★★ 从上面的00469B3C处的RET返回到此，如果al=0，将跳往过期的处理，
几个F12后程序将退出，所以一定要在前面的call 00469900中将al置为1
:00412ABE 0F85A7000000 jne 00412B6B
上面al为1后，这里就会跳往正确的地方
```

## (2)去除2002年3月31号的限制

这个程序的作者有点变态，他在程序中调用`GetLocalTime`有几十次之多，而且每次调用后都作了判断，判断如果时间过期后都要跳往不同的过期处理，比如上面`00412A87`处开始的一段`GetLocalTime`的处理要按如下方法处理：

```
* Reference To: KERNEL32.GetLocalTime, Ord:011Bh
            |
:00412A87 FF151C645000 Call dword ptr [0050641C]
:00412A8D 8B45E0 mov eax, dword ptr [ebp-20]
:00412A90 663DD007 cmp ax, 07D0 //改为9090
            07D0也就是2000年
:00412A94 722E jb 00412AC4 //改为9090
            如果时间小于2000年则跳往00412AC4
:00412A96 663DD207 cmp ax, 07D2 //改为9090
            07D2也就是2002年
:00412A9A 7728 ja 00412AC4 //改为9090
            如果时间大于2002年则跳往00412AC4
:00412A9C 7513 jne 00412AB1 //改为EB13
            //这里一定要跳，下面就要跳往过期处理
:00412A9E 668B45E2 mov ax, word ptr [ebp-1E]
:00412AA2 663D0300 cmp ax, 0003
:00412AA6 771C ja 00412AC4 //这里跳往过期处理
:00412AA8 7507 jne 00412AB1
:00412AAA 66837DE61E cmp word ptr [ebp-1A], 001E
:00412AAF 7713 ja 00412AC4
```

如果不怕麻烦的话，就处理每个`GetLocalTime`后的代码，共有几十处，我用的一种投机的方法就简单了这个问题。

我们已经知道`07D2`也就是2002年，用UltraEdit打开`Merge.exe`，查找替换文件中所有的`07D2`为`0BB8`（也就是3000年），当然你也可以改为N千年也是可以的 :-)，重新运行程序Nag、30天限制、及2002年3月限制都没有了，这种方法并没有处理时间小于2000年的问题，当然现在已经2001年了，不会有人会把机器时间改为2000年以前吧，如果要解决掉这个小于2000年的问题，须按上面我写的修改每个`GetLocelTime`返回后的代码的方法就可搞定，这就需要耐心了。
