
# 汇编代码分析

### 连接字符串 "open" 与 "G:"

```
0040118C  |.  52                      push    edx                       ; /StringToAdd  
0040118D  |.  50                      push    eax                       ; |ConcatString  
0040118E  |.  FFD6                    call    esi                       ; \lstrcatA
```

//连接"open"+"G:"

### 连接 "open G:" 与 "type cdaudio alias x"

```
00401190  |.  8D4C24 08               lea     ecx, dword ptr [esp+8]  
00401194  |.  68 38904000             push    00409038                  ; / type cdaudio alias x  
00401199  |.  51                      push    ecx                       ; |ConcatString  
0040119A  |.  FFD6                    call    esi                       ; \lstrcatA
```

//连接"open G:"与"type cdaudio alias x"

### 调用 `mciSendStringA`

```
0040119C  |.  8B35 CC634000           mov     esi, dword ptr [<&WINMM.mciSendS>;  WINMM.mciSendStringA  
004011A2  |.  6A 00                   push    0  
004011A4  |.  8D5424 70               lea     edx, dword ptr [esp+70]  
004011A8  |.  6A 64                   push    64  
004011AA  |.  8D4424 10               lea     eax, dword ptr [esp+10]                   ; [ESP+10]的内容是"open G: type cdaudio alias x"  
004011AE  |.  52                      push    edx  
004011AF  |.  50                      push    eax  
004011B0  |.  FFD6                    call    esi                       ;  <&WINMM.mciSendStringA>
```

//`mciSendStringA("open G: type cdaudio alias x",[esp+70],64,0)`

### 关闭光驱门

```
004011B2  |.  6A 00                   push    0  
004011B4  |.  8D4C24 70               lea     ecx, dword ptr [esp+70]  
004011B8  |.  6A 64                   push    64  
004011BA  |.  51                      push    ecx  
004011BB  |.  68 58904000             push    00409058                  ;  set x door closed  
004011C0  |.  FFD6                    call    esi
```

//`mciSendStringA("set x door closed",[esp+70],64,0)`，到这里我的光驱就关闭了

### 关闭光驱别名

```
004011C2  |.  6A 00                   push    0  
004011C4  |.  8D5424 70               lea     edx, dword ptr [esp+70]  
004011C8  |.  6A 64                   push    64  
004011CA  |.  52                      push    edx  
004011CB  |.  68 20904000             push    00409020                  ;  close x  
004011D0  |.  FFD6                    call    esi
```

//`mciSendStringA("close x",[esp+70],64,0)`

### 恢复栈并返回

```
004011D2  |.  5E                      pop     esi  
004011D3  |.  81C4 CC000000           add     esp, 0CC  
004011D9  \.  C2 0400                 retn    4
```

# 检测光驱

判断是不是光驱的我们可以用 `GetDriveTypeA`，用类似下面的代码检测就可以了，作者也是这样的，代码在 `0040105B`---`004010A0` 之间，就不具体分析了，主要是获得思路。

```c
for(drive='c';drive<'z';drive++)
   {
      if GetDriveTypeA(drive)==5   
          s=drive;            //最好用一个数组来接受，有些用户可能有多个光驱
          return;
   }
```

# 作者感想

简单的分析了一下代码，这其实是我第 4 次这样分析别人的程序了，为什么呢！因为当初学习 `C` 的时候感觉只能用来写算术题，写不出什么好的应用程序来，所以后来就不学了！现在认识到要做高手必须得掌握一门编程语言(我的好友觉悟更高：高手也要吃饭！~_~)，我的目的就是想看人家是怎样写出自己的程序的！

这个程序我还是留到等我学会编程了再写!^\_^

# 版权声明

"破"文一篇，随意转载，但请保持文章的完整性

`4dd466d0e1e0cc123afbe504da2462ca`
