
### 反汇编代码片段

```
003847CB    8D89 A5494100       lea ecx,dword ptr ds:[ecx+4149A5]  

003847D1    FFD1                 call ecx  

003847D3    83F8 FF             cmp eax,-1  

003847D6    74 1D               je short 003847F5  

003847D8    F645 0D 80          test byte ptr ss:[ebp+D],80  

003847DC    74 1B               je short 003847F9  

003847DE    E8 00000000         call 003847E3  

003847E3    59                  pop ecx  

003847E4    81E9 36364100       sub ecx,413636  

003847EA    68 38270000         push 2738  

003847EF    FF91 F3434100       call dword ptr ds:[ecx+4143F3]  

003847F5    834D 10 FF          or dword ptr ss:[ebp+10],FFFFFFFF  

003847F9    8B45 10             mov eax,dword ptr ss:[ebp+10]  

003847FC    8BE5                mov esp,ebp  

003847FE    5D                  pop ebp  

003847FF    83C4 14             add esp,14  

00384802    FF6424 EC           jmp dword ptr ss:[esp-14]  

00384806    E8 04000000         call 0038480F  

0038480B    0FEB0CC9            por mm1,qword ptr ds:[ecx+ecx*8]  

0038480F    58                  pop eax  

00384810    EB 01               jmp short 00384813  

00384812    0F40EB              cmovo ebp,ebx  

00384815    01CD                add ebp,ecx  

00384817    FFE0                jmp eax  

00384819    C7                  ???                          ; 未知命令  

0038481A    6A 00               push 0  

0038481C    50                  push eax  

0038481D    8B85 654A4100       mov eax,dword ptr ss:[ebp+414A65]  
```

#### 入口处被抽取代码的还原。

```
0059F888    55                  push ebp  

0059F889    8BEC                mov ebp,esp  

0059F88B    83C4 F0             add esp,-10  

0059F88E    B8 38F35900         mov eax,main.0059F338  

0059F893    E8 3879E6FF         call main.004071D0  

0059F898    B8 585A5A00         mov eax,main.005A5A58  

0059F89D    8B00                mov eax,dword ptr ds:[eax]  

0059F89F    8B00                mov eax,dword ptr ds:[eax]  

0059F8A1    E8 C2C6ECFF         call main.0046BF68  

0059F8A6    8B0D E0535A00       mov ecx,dword ptr ds:[5A53E0]           ; main.007422A8  

0059F8AC    B8 585A5A00         mov eax,main.005A5A58  

0059F8B1    8B00                mov eax,dword ptr ds:[eax]  

0059F8B3    8B00                mov eax,dword ptr ds:[eax]  

0059F8B5    8B15 30CB5700       mov edx,dword ptr ds:[57CB30]           ; main.0057CB7C  

0059F8BB    E8 C0C6ECFF         call main.0046BF80  

0059F8C0    8B0D E8515A00       mov ecx,dword ptr ds:[5A51E8]           ; main.00741F44  

0059F8C6    B8 585A5A00         mov eax,main.005A5A58  

0059F8CB    8B00                mov eax,dword ptr ds:[eax]  

0059F8CD    8B00                mov eax,dword ptr ds:[eax]  

0059F8CF    8B15 80755600       mov edx,dword ptr ds:[567580]           ; main.005675CC  

0059F8D5    E8 A6C6ECFF         call main.0046BF80  

0059F8DA    B8 585A5A00         mov eax,main.005A5A58  

0059F8DF    8B00                mov eax,dword ptr ds:[eax]  

0059F8E1    8B00                mov eax,dword ptr ds:[eax]  

0059F8E3    E8 18C7ECFF         call main.0046C000  

0059F8E8    E8 3752E6FF         call main.00404B24  

0038B003    68 6DF85900         push 59F86D        -------伪 oep  

0038B008    C3                  retn  
```

```
55 8B EC 83 C4 F0 B8 38 F3 59 00 E8 38 79 E6 FF B8 58 5A 5A 00 8B 00 8B 00 E8 C2 C6 EC FF 8B 

0D

E0 53 5A 00 B8 58 5A 5A 00 8B 00 8B 00 8B 15 30 CB 57 00 E8 C0 C6 EC FF 8B 0D E8 51 5A 00 B8 

58

5A 5A 00 8B 00 8B 00 8B 15 80 75 56 00 E8 A6 C6 EC FF B8 58 5A 5A 00 8B 00 8B 00 E8 18 C7 EC 

FF

E8 37 52 E6 FF 
```

### API 函数列表

这是我以前根据那张表来排的，其实`hying`基本上是按字母顺序排下来的

```
DialogBoxIndirectParamA  

?????(第二个我也不知道)  

EqualRect  

ExitProcess  

FreeResource  

GetProcAddress  

GetVersion  

GetModuleHandleA  

GetCurrentProcess  

GetCurrentProcessId  

??????(个人觉得这里应当不是 SendMessageA，但那个程序没用到，所以也不知了)   

GetCommandLineA  

InflateRect  

IntersectRect  

IsRectEmpty  

LoadLibraryA  

LockResource  

MulDiv  

PtInRect  

OffsetRect   

SendMessageA  

SetRect  

send  

recv 
```

### 特殊函数处理

关于特殊函数处理，我是修改`lstrcmpA`，使之返回-1。 

![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

### 备注

`to forgot`：

你看看`xxx2.5xxx`版的，就知道你的`lstrcmpA`是没有用了

可以把它自己的`strcmp`也改了

`4dd466d0e1e0cc123afbe504da2462ca`
