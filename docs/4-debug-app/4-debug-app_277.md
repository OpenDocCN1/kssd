
# 打造扫雷终极外挂

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-212.htm`](https://www.pediy.com/kssd/pediy09/pediy09-212.htm)

> 【文章标题】: 打造扫雷终极外挂

> 【文章作者】: CARY

> 【作者邮箱】: caryzyu@hotmail.com

> 【软件名称】: 扫雷 5.1.2600.0

> 【下载地址】: XP 自带

> 【编写语言】: VC++ 6.0

> 【使用工具】: OD

> 【操作平台】: Windows XP

> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

> 分隔线

> 【详细过程】

一日,向往常那样用 OD 随便加载一些东东小研究一翻,便把 WINDOWS XP 自带扫雷扔到了 OD 窗口里,随便调之,不料小有深入,拿出来与大家分享..嘻...

在这里我要感谢 Backer 老师在调试分析过程中给我的帮助！

(下面先是分析部分然后是代码的实践部分)

首先先要了解我们要做的外挂,在功能上我为大家准备了 2 个,一是让扫雷程序自动把所有有雷的地方自动换成用小红旗表示,二是自动帮我们完成扫雷任务,也就是代替我们完成游戏.哈哈 这个可是秒杀哦.

我们先分析这个游戏,要想完成上面的两个功能首先应该要猜测这个游戏的内部实践,只有知道了游戏的运行原理才能找到针对这个游戏的关键部分进行 XXXX,才能实现功能嘛~~ 呵呵

OK,进入正题,先用 `PEID` 查壳为 `Microsoft Visual C++ 7.0 Method2 [调试]` 没有加壳(太棒了!)在运行扫雷,如果是第一次运行则会默认为初级状态,也就是 9\*9 小格的布局,如果你是老玩家了,那么也要把扫雷调到初级状态,因为关联数据少好分析嘛`

下面是我的猜测

猜测一:方格布局应该是随机生成

猜测二:扫雷的方格布局应该是数组或巨阵

猜测三:在我们点击方块时肯定有变量记录我们点击的是哪个方块 比如:`x = 1 y = 3` 既 第 1 列 第 3 排

我们先从第一个猜测开始,扫雷程序是用 VC 开发的,随机函数应该用到库函数 `rand` 我们用 OD 下断 `rand`(命令:`bp rand`)跟到主程序领空可以看到:

这是扫雷里计算随机数用到的函数

```
01003940  /$  FF15 B0110001  CALL DWORD PTR DS:[<&msvcrt.rand>]     ; [rand
01003946  |.  99             CDQ
01003947  |.  F77C24 04      IDIV DWORD PTR SS:[ESP+4]             ;相当于 EDX = rand % arg1
0100394B  |.  8BC2           MOV EAX,EDX                            ;EDX 给 EAX 做为返回值
0100394D  \.  C2 0400        RETN 4
```

我们来看看谁来调用上面的函数(被 `010036CD` 地址代码调用)

这个函数应该是随机布雷函数

```
0100367A  /$  A1 AC560001   MOV EAX,DWORD PTR DS:[10056AC]         ;横向方格数
0100367F  |.  8B0D A8560001 MOV ECX,DWORD PTR DS:[10056A8]         ;纵向方格数
01003685  |.  53            PUSH EBX
01003686  |.  56            PUSH ESI
01003687  |.  57            PUSH EDI
01003688  |.  33FF          XOR EDI,EDI
0100368A  |.  3B05 34530001 CMP EAX,DWORD PTR DS:[1005334]         ;检测方格数是否改变
01003690  |.  893D 64510001 MOV DWORD PTR DS:[1005164],EDI
01003696  |.  75 0C         JNZ SHORT winmine.010036A4             ;改变则跳
01003698  |.  3B0D 38530001 CMP ECX,DWORD PTR DS:[1005338]         ;检测方格数是否改变
0100369E  |.  75 04         JNZ SHORT winmine.010036A4             ;改变则跳
010036A0  |.  6A 04         PUSH 4
010036A2  |.  EB 02         JMP SHORT winmine.010036A6
010036A4  |>  6A 06         PUSH 6
010036A6  |>  5B            POP EBX
010036A7  |.  A3 34530001   MOV DWORD PTR DS:[1005334],EAX
010036AC  |.  890D 38530001 MOV DWORD PTR DS:[1005338],ECX
010036B2  |.  E8 1EF8FFFF   CALL winmine.01002ED5
010036B7  |.  A1 A4560001   MOV EAX,DWORD PTR DS:[10056A4]
010036BC  |.  893D 60510001 MOV DWORD PTR DS:[1005160],EDI
010036C2  |.  A3 30530001   MOV DWORD PTR DS:[1005330],EAX
****关键的随机布雷循环****
010036C7  |>  FF35 34530001 PUSH DWORD PTR DS:[1005334]            ;横向方格数  布雷循环开始处
010036CD  |.  E8 6E020000   CALL winmine.01003940                  ;根据横向方格数产生随机数
010036D2  |.  FF35 38530001 PUSH DWORD PTR DS:[1005338]            ;纵向方格数
010036D8  |.  8BF0          MOV ESI,EAX                            ;横向坐标给 ESI
010036DA  |.  46            INC ESI
010036DB  |.  E8 60020000   CALL winmine.01003940                  ;根据纵向方格数产生随机数
010036E0  |.  40            INC EAX
010036E1  |.  8BC8          MOV ECX,EAX                            ;纵向坐标给 ECX
010036E3  |.  C1E1 05       SHL ECX,5
010036E6  |.  F68431 405300>TEST BYTE PTR DS:[ECX+ESI+1005340],80
010036EE  |.^ 75 D7         JNZ SHORT winmine.010036C7
010036F0  |.  C1E0 05       SHL EAX,5                              ;算出内存布局的纵坐标 相当于 Y * 20
010036F3  |.  8D8430 405300>LEA EAX,DWORD PTR DS:[EAX+ESI+1005340] ;方格内存布局的地址给 EAX
010036FA  |.  8008 80       OR BYTE PTR DS:[EAX],80                ;把雷写入方格内存布局
010036FD  |.  FF0D 30530001 DEC DWORD PTR DS:[1005330]             ;布雷的个数（也是循环次数）每次-1
01003703  |.^ 75 C2         JNZ SHORT winmine.010036C7             ;未布完则循环
****
01003705  |.  8B0D 38530001 MOV ECX,DWORD PTR DS:[1005338]
0100370B  |.  0FAF0D 345300>IMUL ECX,DWORD PTR DS:[1005334]
01003712  |.  A1 A4560001   MOV EAX,DWORD PTR DS:[10056A4]
01003717  |.  2BC8          SUB ECX,EAX
01003719  |.  57            PUSH EDI
0100371A  |.  893D 9C570001 MOV DWORD PTR DS:[100579C],EDI
01003720  |.  A3 30530001   MOV DWORD PTR DS:[1005330],EAX
01003725  |.  A3 94510001   MOV DWORD PTR DS:[1005194],EAX
0100372A  |.  893D A4570001 MOV DWORD PTR DS:[10057A4],EDI
01003730  |.  890D A0570001 MOV DWORD PTR DS:[10057A0],ECX
01003736  |.  C705 00500001>MOV DWORD PTR DS:[1005000],1
01003740  |.  E8 25FDFFFF   CALL winmine.0100346A
01003745  |.  53            PUSH EBX                                 ; /Arg1
01003746  |.  E8 05E2FFFF   CALL winmine.01001950                   ; \winmine.01001950
0100374B  |.  5F            POP EDI
0100374C  |.  5E            POP ESI
0100374D  |.  5B            POP EBX
0100374E  \.  C3            RETN
```

`[ECX+ESI+1005340]` 这个代码实际上是二维数组寻址 可以理解为 `1005340[X + Y * 20]`

`010036FD |. FF0D 30530001 DEC DWORD PTR DS:[1005330]` 这段代码的内存地址`[1005330]`存的是要布的雷数,你高兴也可以把它写为 1 个

我们知道了它的内存结构很容易就能算出每个方格在内存中的位置了,那我们先看看第一个方格在内存中是什么样的

注意: `y * 20` 是因为这条指令 `010036E3 |. C1E1 05 SHL ECX,5` 一个数左移 5 位等于这个数乘以 20

第 1 个方格 `1005340[1 + 1 * 20] = 1005361`

最后 1 个方格 `1005340[9 + 9 * 20] = 1005469`

哈哈,是不是计算出来了呢.我们可以在 OD 内存窗口里看看(命令:`d 1005361`)

```
01005361  0F 0F 8F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F  ?
01005371  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
01005381  8F 0F 0F 8F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F  ??
01005391  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
010053A1  0F 0F 0F 0F 0F 0F 8F 0F 0F 10 0F 0F 0F 0F 0F 0F  ?
010053B1  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
010053C1  0F 0F 0F 0F 0F 0F 8F 0F 0F 10 0F 0F 0F 0F 0F 0F  ?
010053D1  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
010053E1  0F 0F 0F 0F 0F 0F 0F 0F 8F 10 0F 0F 0F 0F 0F 0F  ?
010053F1  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
01005401  0F 0F 0F 0F 0F 8F 0F 8F 0F 10 0F 0F 0F 0F 0F 0F  ??
01005411  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
01005421  0F 0F 0F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F
01005431  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
01005441  0F 0F 0F 0F 0F 8F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F  ?
01005451  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10
01005461  0F 0F 0F 0F 8F 0F 0F 0F 0F                 ?
```

哇这么多一片！那么我们怎么知道哪块是雷呢..... 看看下面代码

`010036FA |. 8008 80 OR BYTE PTR DS:[EAX],80`

这段代码每次都让 `0F` 与 `80` 做或运算 `0F OR 80 = 8F` 也就是说 `8F` 就是雷的标记喽.. 我们可以看到内存中有 10 个 `8F` 的标记

下面我们试着找小红旗的标记,我们先在没有雷的地方单击鼠标右键设上小红旗.然后在有雷的地方在设一个在看看内存布局,结果有雷的地方变成了 `8F`,没有雷的地方是 `0E`。

好了以上的线索先保留,让我们在看看如何让扫雷自己去消灭小方格,现在要先借助一些游戏修改工具了(我相信大家都用过吧),目的是要找到鼠标当前点击方格的 `X Y` 坐标,找这个很简单,只要先点开一个方格然后输入其 `X` 或 `Y`,然后在任意点一个方格在搜索,直到搜索到为一个值时这个值就是需要的了.

我搜索的两个地址是 `1005118` 和 `100511C`

在这两个地方用 OD 下内存写入断点,断到了这里



### 0x01002068 - 0x010020B1 代码片段

```
01002068   |.   830D 18510001>or      dword ptr [1005118], FFFFFFFF   ;初始化变量  
0100206F   |.   830D 1C510001>or      dword ptr [100511C], FFFFFFFF   ;初始化变量  
01002076   |.   53             push    ebx  
01002077   |.   891D 40510001 mov     dword ptr [1005140], ebx  
0100207D   |.   E8 91080000    call    01002913  
01002082   |.   8B4D 14        mov     ecx, dword ptr [ebp+14]  
01002085   |>   393D 40510001 cmp     dword ptr [1005140], edi  
0100208B   |.   74 34          je      short 010020C1  
0100208D   |.   841D 00500001 test    byte ptr [1005000], bl  
01002093   |.^  0F84 54FFFFFF je      01001FED  
01002099   |.   8B45 14        mov     eax, dword ptr [ebp+14]  
0100209C   |.   C1E8 10        shr     eax, 10  
0100209F   |.   83E8 27        sub     eax, 27  
010020A2   |.   C1F8 04        sar     eax, 4  
010020A5   |.   50             push    eax                   ; /Arg2  
010020A6   |.   0FB745 14     movzx   eax, word ptr [ebp+14]     ; |  
010020AA   |.   83C0 04       add     eax, 4                 ; |  
010020AD   |.   C1F8 04       sar     eax, 4                 ; |  
010020B0   |.   50             push    eax                   ; |Arg1  
010020B1   |>   E8 1E110000   call    010031D4               ; \  
```

### 鼠标消息处理分支 (0x01001FDF)

在往上翻到 `01001FDF` 到鼠标消息处理分支

```
01001FDF   |>\  33FF          xor     edi, edi               ;   Cases 202 (WM_LBUTTONUP),205 (WM_RBUTTONUP),208 (WM_MBUTTONDOWN) of switch 01001F5F  
01001FE1   |.   393D 40510001 cmp     dword ptr [1005140], edi  
01001FE7   |.   0F84 BC010000 je      010021A9  
01001FED   |>   893D 40510001 mov     dword ptr [1005140], edi  
01001FF3   |.   FF15 D8100001 call    dword ptr [<&USER32.ReleaseCapture>]; [ReleaseCapture  
01001FF9   |.   841D 00500001 test    byte ptr [1005000], bl  
01001FFF   |.   0F84 B6000000 je      010020BB  
01002005   |.   E8 D7170000   call    010037E1               ;处理我们鼠标点击的函数  
0100200A   |.   E9 9A010000   jmp     010021A9  
```

在 `01001FDF` 下断点，跟到 `010037E1` 里去

### 处理鼠标点击的函数 (0x010037E1)

```
010037E1   /$   A1 18510001   mov     eax, dword ptr [1005118]       ;横坐标给 EAX  
010037E6   |.   85C0          test    eax, eax  
010037E8   |.   0F8E C8000000 jle     010038B6  
010037EE   |.   8B0D 1C510001 mov     ecx, dword ptr [100511C]       ;纵坐标给 ECX  
010037F4   |.   85C9          test    ecx, ecx  
010037F6   |.   0F8E BA000000 jle     010038B6  
010037FC   |.   3B05 34530001 cmp     eax, dword ptr [1005334]       ;判断是否越界   
01003802   |.   0F8F AE000000 jg      010038B6  
01003808   |.   3B0D 38530001 cmp     ecx, dword ptr [1005338]       ;判断是否越界   
0100380E   |.   0F8F A2000000 jg      010038B6  
01003814   |.   53             push    ebx  
01003815   |.   33DB          xor     ebx, ebx  
01003817   |.   43             inc     ebx  
01003818   |.   833D A4570001>cmp     dword ptr [10057A4], 0  
0100381F   |.   75 4A         jnz     short 0100386B  
01003821   |.   833D 9C570001>cmp     dword ptr [100579C], 0  
01003828   |.   75 41         jnz     short 0100386B  
0100382A   |.   53             push    ebx  
0100382B   |.   E8 BD000000   call    010038ED  
01003830   |.   FF05 9C570001 inc     dword ptr [100579C]  
01003836   |.   E8 7AF0FFFF   call    010028B5  
0100383B   |.   6A 00         push    0                   ; /Timerproc = NULL  
0100383D   |.   68 E8030000   push    3E8                 ; |Timeout = 1000. ms  
01003842   |.   53             push    ebx                 ; |TimerID  
01003843   |.   FF35 245B0001 push    dword ptr [1005B24]     ; |hWnd = 000B03F4 ('扫雷',class='扫雷')  
01003849   |.   891D 64510001 mov     dword ptr [1005164], ebx     ; |  
0100384F   |.   FF15 B4100001 call    dword ptr [<&USER32.SetTimer>]   ; \SetTimer  
01003855   |.   85C0          test    eax, eax  
01003857   |.   75 07         jnz     short 01003860  
01003859   |.   6A 04         push    4                   ; /Arg1 = 00000004  
0100385B   |.   E8 F0000000   call    01003950               ; \winmine.01003950  
01003860   |>   A1 18510001   mov     eax, dword ptr [1005118]  
01003865   |.   8B0D 1C510001 mov     ecx, dword ptr [100511C]  
0100386B   |>   841D 00500001 test    byte ptr [1005000], bl  
01003871   |.   5B             pop     ebx  
01003872   |.   75 10         jnz     short 01003884  
01003874   |.   6A FE         push    -2  
01003876   |.   59             pop     ecx  
01003877   |.   8BC1          mov     eax, ecx  
01003879   |.   890D 1C510001 mov     dword ptr [100511C], ecx  
0100387F   |.   A3 18510001   mov     dword ptr [1005118], eax  
01003884   |>   833D 44510001>cmp     dword ptr [1005144], 0  
0100388B   |.   74 09         je      short 01003896  
0100388D   |.   51             push    ecx  
0100388E   |.   50             push    eax  
0100388F   |.   E8 23FDFFFF   call    010035B7  
01003894   |.   EB 20         jmp     short 010038B6  
01003896   |>   8BD1          mov     edx, ecx  
01003898   |.   C1E2 05       shl     edx, 5  
0100389B   |.   8A9402 405300>mov     dl, byte ptr [edx+eax+1005340]  
010038A2   |.   F6C2 40       test    dl, 40  
010038A5   |.   75 0F         jnz     short 010038B6  
010038A7   |.   80E2 1F       and     dl, 1F  
010038AA   |.   80FA 0E       cmp     dl, 0E  
010038AD   |.   74 07         je      short 010038B6  
010038AF   |.   51             push    ecx                 ;   坐标 X  
010038B0   |.   50             push    eax                 ;   坐标 Y  
010038B1   |.   E8 5CFCFFFF   call    01003512               ;   选种方格  
010038B6   |>   FF35 60510001 push    dword ptr [1005160]  
010038BC   |.   E8 52F0FFFF   call    01002913  
010038C1   \.   C3             retn  
```

哈哈 选种方格的函数找到了。

现在我们来整理一下线索

### 找到的数据

- `0x8F` = 雷
- `0x8E` = 有雷 AND 小红旗
- `0x0E` = 无雷 AND 小红旗

### 找到的地址

- `0x1005334` 横向方格数
- `0x1005338` 纵向方格数

### 找到的函数

```
010038AF   |.   51             push    ecx                 ;   坐标 X  
010038B0   |.   50             push    eax                 ;   坐标 Y  
010038B1   |.   E8 5CFCFFFF   call    01003512               ;   选种方格  
```

### 实践思路

下面说下实践的思路,

- 功能一 "查雷": 把内存布局中的所有 `0x8F` 有雷 都换成 `0x8E` 小红旗
- 功能二 "排雷": 用一个 `DLL` 注入扫雷进程，然后帮它调用选种方格的函数，有雷的地方不选

#### 功能一代码

```c
DWORD x = 0x10056A8;
DWORD y = 0x10056AC;

HWND hwnd = ::FindWindow(NULL, "扫雷");
DWORD hProcessId;

::GetWindowThreadProcessId(hwnd, &hProcessId);
HANDLE Process = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, hProcessId);


int b = 0 , s = 0, nx = 0, ny = 0;
::ReadProcessMemory(Process, (LPCVOID)x, &nx, 1, NULL);    //获取横向方格长度
::ReadProcessMemory(Process, (LPCVOID)y, &ny, 1, NULL);    //获取纵向方格长度

for(int i = 0; i < nx * 32; i += 32)
{
  for(int j = 0; j < ny; j++)
  {
    ::ReadProcessMemory(Process, (LPCVOID)(addr + i+j), &b, 1, NULL);
    if (b == 0x8F)       //判断是否有雷
    {
      s = 0x8E;    //如果有雷就画上小红旗
      ::WriteProcessMemory(Process, (LPVOID)(addr + i+j), &s, 1, NULL);
    }
  }
}

::InvalidateRect(hwnd, NULL, TRUE);
::CloseHandle(Process);
```

#### 功能二代码

功能二 在注入的 `DLL` 中实现的,因为同一进程内才能调用同一进程的函数

```c
            DWORD addr = 0x1005361;
      DWORD x = 0x10056A8;
      DWORD y = 0x10056AC;
      

      HWND hwnd = ::FindWindow(NULL, "扫雷");
      DWORD hProcessId;
      ::GetWindowThreadProcessId(hwnd, &hProcessId);
      HANDLE Process = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, hProcessId);
      
```



```c
int b = 0 ,  nx = 0, ny = 0;  

DWORD s = 0;  
::ReadProcessMemory(Process, (LPCVOID)x, &nx, 1, NULL);  //获取横向方格长度  
::ReadProcessMemory(Process, (LPCVOID)y, &ny, 1, NULL);  //获取纵向方格长度  

DWORD xuanzong = 0x10037E1;        //选方格的函数地址  
DWORD x1 = 1;  
DWORD y1 = 1;  

int (*Rec)[900] = new int[nx][900];  

for(int i = 0; i < nx * 32; i += 32)  
{  
  for(int j = 0; j < ny; j++)  
  {  
    ::ReadProcessMemory(Process, (LPCVOID)(addr + i+j), &b, 1, NULL);  
    if (b == 0x8E || b == 0x8F)     //把内存布局写入自定义的数组  
     {  
      Rec[i/32][j] = 1;     
     }  
     else  
       Rec[i/32][j] = 0;  
  }  
}  

for(i = 0; i < nx; i ++)  
{     
  for(int j = 0; j < ny; j++)  
  {  
    x1 = i + 1;  
    y1 = j + 1;  
    if(Rec[i][j] != TRUE)           //选择没有雷的方格  
    {  
      _asm  
      {  
        push x1        //要选方格的 X  
        push y1        //要选方格的 Y  
        call xuanzong  //调用选方格函数  
      }  
    }  
  }  
}  
```

哈哈,这样一个扫雷的秒杀外挂就出来了！由于我水平有限,如有错误之处请跟贴指正！(附件暂时还不能用 5555555 有时间找个地址我会贴出来

---
**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

**引用**: 最初由 `CARY`发布
**【文章标题】**: 打造扫雷终极外挂
**【文章作者】**: `CARY`
**【作者邮箱】**: `caryzyu@hotmail.com`
**【软件名称】**: 扫雷 5.1.2600.0
**【下载地址】**: XP 自带
**【编写语言】**: `VC++ 6.0`

我把正确的代码贴出来，如下：

```c
void __stdcall saolei()  
{  
   DWORD addr = 0x1005361;  
   DWORD x    = 0x10056A8;  
   DWORD y    = 0x10056AC;  

  HWND hwnd = ::FindWindow(NULL, "扫雷");  
  DWORD hProcessId;  
  ::GetWindowThreadProcessId(hwnd, &hProcessId);  
  HANDLE Process = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, hProcessId);  

  int b = 0 ,  nx = 0, ny = 0;  

  DWORD s = 0;  
  ::ReadProcessMemory(Process, (LPCVOID)x, &nx, 1, NULL);  //获取横向方格长度  
  ::ReadProcessMemory(Process, (LPCVOID)y, &ny, 1, NULL);  //获取纵向方格长度  

  DWORD xuanzong = /*0x10037E1*/0x01003512;        //选方格的函数地址  
  DWORD x1 = 1;  
  DWORD y1 = 1;  

  int (*Rec)[900] = new int[nx][900];  

  for(int i = 0; i < nx * 32; i += 32)  
  {  
  for(int j = 0; j < ny; j++)  
  {  
    ::ReadProcessMemory(Process, (LPCVOID)(addr + i+j), &b, 1, NULL);  
    if (b == 0x8E || b == 0x8F)     //把内存布局写入自定义的数组  
    {  
      Rec[i/32][j] = 1;     
    }  
    else  
       Rec[i/32][j] = 0;  
  }  
  }  

  for(i = 0; i < nx; i ++)  
  {     
  for(int j = 0; j < ny; j++)  
  {  
      x1 = i + 1;  
      y1 = j + 1;  
      if(Rec[i][j] != 1)           //选择没有雷的方格  
      {  
        _asm  
        {  
           push x1        //要选方格的 X  
           push y1        //要选方格的 Y  
           call xuanzong  //调用选方格函数  
        }  
      }  
  }  
  }  
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
