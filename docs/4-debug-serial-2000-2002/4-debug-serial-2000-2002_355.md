
# HyperCam Version 1.20.04

> 原文：[`www.pediy.com/kssd/pediy03/forum698.htm`](https://www.pediy.com/kssd/pediy03/forum698.htm)

**破解程序:**

HyperCam Version 1.20.04 (for Intel processors)

**破解工具:**

`SoftIce 3.0` for Windows95 (或更高版本)

**破解者:**

`chcw`

1. 用 `WinICE` 载入 `HyperCam.exe`
2. 在 `HyperCam` 中选择 `License/Enter License Now`，弹出 `HyperCam License` 对话框。
3. 按 `Ctrl-D`，切换到 `SoftICE` 窗口，在 `SoftICE` 命令行设置断点 `BPX MessageBoxA`
4. 再按 `Ctrl-D`，回到执行程序 `HyperCam`。在 `Register` 和 `Key` 中随便填入注册码，如 `Register=chcw`，`Key=9530109`。并选择一种注册类型，如 `Single User - Number of Licensed 1`，添完上述内容后，选择 `'OK'`，触发 `SoftICE`。
5. `SoftICE` 的光带停留在 `MessageBoxA` 函数的入口处：

   ```
   USER32!MessageBoxA
       015F:BFF541BA  PUSH EBP
   ```

   将光标移到 `MessageBoxA` 函数的返回处

   ```
       015F:BFF541D    RET 0010
   ```

   并键入命令 `HERE`，程序执行到光标所在处。（由于函数 `MessageBoxA` 要调用函数 `MessageBoxExA`，所以在上述执行过程中会出现一个对话框，直接选 `'OK'` 或 `'确定'` 即可）

6. 连续按 `F10` 键，回到 `MessageBoxA` 函数被调用处：

   ```
       015F:00427881  CALL [USER32!MessageBoxA]
       015F:00427887  POP EDI
       015F:00427888  POP ESI
       015F:00427889  RET 000C
   ```

   这部分代码在程序 `HyperCam.exe` 中，但在程序调用 `MessageBoxA` 后，程序立即以子程序的方式返回，综合在调用 `MessageBoxA` 前的代码，可以看出这是程序 `HyperCam` 中的一个子程序(入口地址为 `00427853`)，其主要功能大致是显示一个信息框而已。因此，我们没有必要把注意力放在这里。继续按 `F10`，跳出这个子程序。返回到调用它的上一层程序。

   ```
       015F:0040A03B  CALL 00427853 (调用上述子程序 00427853)
   ```

   显然，既然 `CALL 00427853` 的功能是显示一个注册错误的信息框，那么在它被执行之前，一定有一些函数或语句是用来判断注册正确与否，并根据判断结果来决定是显示错误信息，还是提示注册成功（`HyperCam` 在用户输入正确的注册码时，没有任何提示信息）。从 `CALL 00427853` 语句向上找，在 `015F:0040A020` 处有一条语句 `JMP 0040A040`，跳转地址正好是 `CALL 00427853` 的下一条语句。由于这是一条无条件跳转语句，我们继续往上找能使程序流程执行到这条跳转语句的条件跳转语句（这里的条件，就是指注册码正确与否）。在 `015F:0040A00D` 和 `015F:00409FF2` 处各有一条条件跳转语句，但它们都不影响程序的主要流程。最后在 `015F:00409FD8` 处，有一个条件跳转：

   ```
       015F:00409FD1  CALL 004092E0    ;    调用 004092E0 检查注册码的有效性
       015F:00409FD6  TEST EAX, EAX    ;    若返回值为 0，则显示错误信息。
       015F:00409FD8  JE  0040A022    ;
   ```

   分析以上语句，不难看出，当调用函数 `004092E0` 的返回值为 `0` 时，将显示注册错误的对话框窗口。可见，函数 `4092E0` 应该是 `HyperCam` 检查注册码是否正确的函数。键入命令 `BC *`，清除原先的断点；并在 `015F:00409FD1` 处双击鼠标，设置执行断点。

7. 按 `Ctrl D` 回到 `HyperCam` 中，仍使用原先的注册码，再次选择 `'OK'` 后。触发 `SoftICE`，执行亮条停留在断点 `015F:00409FD1 CALL 004092E0` 处，先按 `F10`，`Step Out` 过这个函数的调用。此时，`EAX` 的值为 `0` 并显示高亮，显然函数 `004092E0` 改动了 `EAX`。在 `SoftICE` 的命令行下，键入 `r EAX=1`，造成注册成功的假象，再按 `Ctrl D`，继续执行程序，返回到 `HyperCam`。这时我们发现，提示用户输入注册信息的窗口 `HyperCam License` 已经关闭。但在 `License` 页中显示的信息还是未注册，这是怎么回事呢？

   我们可以作这样一个假设，`004092E0` 函数不仅通过返回值来表明注册信息正确与否，还通过传递指针参数来返回注册信息。调用 `004092E0` 的函数再进一步将这些注册信息记录在磁盘上。由于我们在上面程序的运行过程中没有填充这些信息，因而尽管程序的流程大致是正确的，但注册信息却没有被保存下来。

8. 如果要用 `Patch` 方法来破解 `HyperCam`，显然不能只修改 `015F:00409FD8` 处的条件判断，具体该如何破解，我没有仔细研究。我主要分析了获取有效注册码的方法。

9. 继续上面的破解过程，在 `015F:00409FD1` 处设置有执行断点。按 `'OK'` 后，触发 `SofeICE`，执行亮条停留在断点处，按 `F8`，进入函数 `004092E0` 中。

10. 接下来按 `F10` 键，到达函数体中的两个子函数调用子函数调用 `015F:00409313 CALL` 和 `0041F3AC`，`015F:00931C CALL 0041F3BF` 该函数的大致作用是判断时区和现在的时间，可能与注册方式中的临时性注册有关，由于我们选择单用户按份数注册的方式。因而可以直接跳过(`Step Out`)这两个函数。继续按 `F10` 键。

11. 接下来进入两个循环：

    ```
    015F:0040932E    MOV EDX, DWORD PTR [ESI]
            ...
    015F:00409348    CALL 0040EDB0     ;    字符串比较（stricmp）
    015F:0040934D    ADD ESP, 0000000C
    015F:00409350    TEST EAX, EAX     ;
    015F:00409352    JE 004096B5       ;    若 eax==0 则跳转
    015F:00409358    ADD ESI, 00000004 ;    指向下一个字符串
    015F:0040935B    CMP ESI, 00436EBC ;    字符串数组都比较完了吗？
    015F:00409361    JB 0040932E       ;    若未比较完，则继续比较。
    ```

    在循环中不断调用子程序 `0040EDB0`，该程序的功能相当于 `stricmp`，它将用户输入的姓名和系统内部的两个数组中的字符串进行比较。字符串数组如下：

    ```c
    char *s1[]={"hacker","ED!SON","tHATDUDE","super user","saltine","xygorf","Borin Thibault",
         "Alexander Chen","-M-O-A-","^SaTaNa","Michael Jackon"}
    char *s2[]={"abel", "Monkey", "jackkuo", "TRACY"}
    ```

    写到这里，也许有人会猜想，这可能是程序在比较用户输入的姓名是否和系统保留的几个字符串相同，若相同，则继续检验注册码的正确性，也就是说，这是系统留下的后门。当时我也这么想，那么剩下的只有注册码了。于是，我按 `Ctrl D`，回到 `HyperCam` 的注册窗口，将名字那一栏用上述数组中的某个字符串填写(如`"saltine"`),再按 `Ctrl D`,在命令行输入 `s ds:0 l ffffffff '9530109'`，找到了存放注册码的内存地址 `0030:80527d70`, 然后设置断点 `BPM 0030:80527d70`，再键入 `X`,运行程序。但是在拦截到对字符串 `'9530109'` 的所有访问中，却没有发现可能是检验注册码的语句。看来我们原先的猜想是错误的。再次分析上述的循环后发现。字符串数组 `s1`、`s2` 实际上是两个黑名单，当用户输入的注册名是数组中的某个字符串时，系统立即认为注册失败，不再检查注册码。而只要你输入的注册名不是上面数组中的某个字符串后，就可以跳过这两个循环，

12. 继续按 `F10` 键，执行到下列语句处：

    ```
    015F:00409395    PUSH ECX      ;    将注册码地址压栈
    015F:00409396    CALL 004098C0
    015F:0040939B    MOV EBX, EAX      ;    EBX <- EAX
    015F:0040939D    ADD ESP, 4
    015F:004093A0    TEST EBX, EBX     ;    返回值在 EBX 中
    015F:004093A2    JZ 004096B5       ;    若 EBX 为 0，则程序认为注册出错。
    ```

    由于子程序 `004098C0` 的参数为注册码，而该函数返回值决定注册是否成功，因而有必要分析该子程序。按 `F8`，进入该程序：

    ```
             ...
    015F:004098D6    CMP AL, 41      ;
    015F:004098D8    JL 004098E6     ;    对注册码进行检查，
    015F:004098DA    MOV AL, [ECX]      ;      将不在'A'-'Z'范围内的字符过滤掉。
    015F:004098DC    CMP AL, 5A
    015F:004098DE    JG 004098E6
             ...
    015F:00409903    CMP ECX, 00000080
    015F:0040990D    JGE 004099A5      ;    检查注册码的长度(存放在 ECX 中)，
    015F:00409913    CMP ECX, 03       ;    长度<=3 或>=128 的为非法。
    015F:00409916    JLE 004099A5
             ...
    015F:0040991C    MOVSX EBP, BYTE PTR [ESP+16]
    015F:00409921    SUB EBP, 00000041     ;    对注册码进行某种换算，
             ...
    015F:0040992C    MOV AL, BYTE PTR [ESP+EBX+14] ;    得到一个新的加密码 SecretCode。
             ...
    015F:00409947    CALL 00409A20
             ...
    015F:00409967    INC EBX
    015F:00409968    CMP EBX, ECX
    015F:0040996A    JL 0040992C
             ...
    015F:0040996C    MOVSX ESI, BYTE PTR [EDI+0043B77F] ;    将加密码分成前 n-1 个字符和
    015F:0040997B    MOV BYTE PTR [EDI+0043B77F], 00     ;    最后一个字符两部分，
    015F:00409982    CALL 004099C0             ;    分别作某种运算，
             ...
    015F:0040998C    CMP ESI, EAX              ;    比较其结果是否相同。
    015F:0040998E    SETNE CL              ;    若不同，则注册为非法。
             ...
    015F:00409992    AND ECX, 0043B780   ;    将加密码 SecretCode（去掉最后一个字符）的地址返回。
    015F:00409998    MOV EAX, ECX      ;
    ```

    子程序 `004098C0` 对注册码进行检查和判断，生成 `SecretCode`，并将结果放在 `EAX` 返回。

13. 从子程序 `004098C0` 返回后，继续按 `F10` 键，下面一部分程序将再次检查你的注册姓名是否在黑名单 `s2` 上（看来作者对数组 `s2` 中的人恨之入骨）跳过这段程序。到 `015F:0040945B    MOV EAX,[EBP+14]` 处。

14. 接下来，程序将对用户输入的注册名进行处理：

    ```
    015F:004094B9    LEA EDI, DWORD PTR [ESP+20]     ;    用户输入的注册名所存放的地址
                 ...
    015F:004094C1    LEA EDX, DWORD PTR [ESP+00000120] ;    系统处理后得到的新串 LName 存放的地址
    015F:004094C8    REPNZ SCASB
    015F:004094CA    NOT ECX                 ;    取得注册名的串长度
                 ...
    015F:004094DE    REPZ MOVSD              ;    将注册名复制到新串 LName 中
                 ...
    015F:004094F5    CMP ECX, 00000010       ;    检查用户输入的姓名是否小于 16 个字符，
    015F:004094F8    JL 004094B9             ;    如果是，则将姓名串不断重复复制到
                                ;    LName 处，直到超过 16 个字符为止。
    ```

    由于我们输入的是`"chcw"`, 因而系统处理后将得到串 `LName="CHCWCHCWCHCWCHCW"`。

15. 继续按 `F10`，直到下列语句处：

    ```
    015F:004096D2    MOV AL, [EBX]      ;    AL <- SecretCode[0]（EBX 中存放的是 SecretCode 的地址）
    015F:004096D4    MOV EBP, [ESP+00000238]
    015F:004096DB    CMP AL, 50      ;    若 AL=='P'，则为第一种注册方式(Single User)
    015F:004096DD     JZ 00409769      ;    跳转到第一种注册方式的处理语句处。
    015F:004096E3     CMP AL, 53      ;    若 AL=='S'，则为第二种注册方式(Unlimited Site License)
    015F:004096E5      JZ 0040974F      ;    跳转到第二种注册方式的处理语句处。
    015F:004096E7      CMP AL, 57      ;    若 AL=='P'，则为第三种注册方式(Unlimited Word-Wide License)
    ```



```
015F:004096E9    JNZ 00409707      ;    若三种都不是，则注册出错。   

015F:004096EB    MOV EDI, DWORD PTR [ESP+0000023C]   

015F:004096F2    CMP EDI, 00000003     ;    判断用户输入的注册方式是否为 3   

015F:004096F5    JNE 00409707      ;    若不是，则注册出错。   

015F:004096F7    CMP BYTE PTR [EBX+01], 4F  ;    判断 SecretCode[1]是否为'O'   

015F:004096FB    JNE 00409707      ;    若不是，则注册出错。   

015F:004096FD    CMP BYTE PTR [EBX+02], 52  ;    判断 SecretCode[2]是否为'R'   

015F:00409701    JE 00409787       ;    若不是，则注册出错。   

      ...   

015F:00409707    LEA ECX, DWORD PTR [ESP+00000230]  ;    注册出错处理   

      ...   

015F:0040974F    MOV EDI, DWORD PTR [ESP+0000023C]   

015F:00409756    CMP EDI, 00000002    ;    判断用户输入的注册方式是否为 2   

015F:00409759    JNE 00409707      ;    若不是，则注册出错。   

015F:0040975B    CMP BYTE PTR [EBX+01], 49  ;    判断 SecretCode[1]是否为'I'   

015F:0040975F    JNE 00409707      ;    若不是，则注册出错。   

015F:00409761    CMP BYTE PTR [EBX+02], 54  ;    判断 SecretCode[2]是否为'T'   

015F:00409765    JE 00409787       


015F:00409767    JMP 00409707      ;    若不是，则注册出错。   

015F:00409769    LEA EAX, DWORD PTR [EBX+01]  ;取得 SecretCode[1]的地址   

015F:0040976C    PUSH 00000002   

  015F:0040976E    PUSH EAX    ;    将 SecretCode[1]的地址压栈   

  015F:0040976F    CALL 00409A40    ;    根据字符 SecretCode[1]和字符 SecretCode[2]   

            ;计算注册的数目。由于采用的是 26 进制，因而最多可以表示 26x26 份拷贝。   

015F:00409774    MOV EDI, DWORD PTR [ESP+00000244]   

015F:0040977B    ADD ESP, 00000008   

015F:0040977E    CMP EDI, 00000001  ;    判断用户输入的注册方式是否为 2   

015F:00409781    JNE 00409707     ;    若不是，则注册出错。   

015F:00409783    CMP EBP, EAX    ;    比较计算所得得注册数是否与用户输入的注册数相同   

015F:00409785    JNE 00409707     ;    若不是，则注册出错。
```

从上面的程序段可以看出，该段程序的主要功能是对 `SecretCode` 进行检查，根据用户注册的方式不同，可分为三种情况：

1.  第一种注册方式(`Single User`)，`SecretCode` 的第一个字符必须是 `'P'`，接下来的两个字符应为注册数目的 26 进制值。
2.  第二种注册方式(`Unlimited Site License`)，该种情况下，`SecretCode` 的头三个字符必须是 `'S'`、`'I'`、`'T'`。
3.  第三种注册方式(`Unlimited World Wide License`)，该种情况下，`SecretCode` 的头三个字符必须是 `'W'`、`'O'`、`'R'`。

在 `SecretCode` 的前三个字符与用户的注册方式完全相符后，继续按 `F10` 键，到下列语句处：

```
015F:004097AB    PUSH 00436F68      ;    字符串"HCA"的地址压栈    
    015F:004097B0    PUSH EBX    ;    SecretCode[3]的地址压栈     
    015F:004097B1    CALL 0040ED70    ;    比较以 SecretCode[3]为启始地址的字符串的前三个字符是否为'HCA' 
015F:004097B6    ADD ESP, 0000000C   

015F:004097B9    TEST EAX, EAX    ;    若不是"HCA",则注册出错。   

015F:004097BB    JNE 00409707
```

该段程序检查 `SecretCode` 的第三到第六个字符是否为 `"HCA"`（就是 `HyperCam` 的缩写），如果不是，则注册出错。

通过上述检查后，继续按 `F10` 键，到以下语句处：

```
015F:004097C8    MOV DL, [EAX]  ;    DL <- LName[i]   

015F:004097CA    MOV BL, [ESI]  ;    BL <- Secret[6+j]   

015F:004097CC    MOV CL, DL   

015F:004097CE    CMP DL,    BL  ;    比较 DL，BL 是否相当。   

015F:004097D0    JNZ 004097F0  ;    若不相等，则注册出错   

015F:004097D2    TEST CL, CL  ;    判断是否已经到达字符串的末尾   

015F:004097D4    JZ 004097EC   

      ...   

015F:004097E2    ADD EAX, 02  ;    i+=2   

015F:004097E5    ADD ESI, 02  ;    j+=2   

015F:004097E8    TEST CL, CL  ;    判断是否已经到达字符串的末尾   

015F:004097EA    JNZ 004097C8  ;    若不是，则循环。
```

这部分程序段将 `SecretCode`（`Secret` 在第 14 步处调用程序 `004098C0` 时产生）剩余的字符所组成的字符串与字符串 `LName` 进行比较。若二者相同，则认为注册成功。

至此，我们已经分析完 `HyperCam` 的注册检验过程。在此过程中，程序不在内存中生成任何合法的注册码，而仅根据用户输入的注册码产生一个 `SecretCode`，并对此 `SecretCode` 进行后续的操作，这使得破解过程较为复杂。而且在整个注册检验过程中，程序还多次检验用户输入的注册名是否是常用的 `Cracker` 名字，这使破解的复杂程度进一步增加。因此，我们有理由相信，`HyperCam` 程序在编写过程中，曾经有意识的加入了反 `Crack` 的代码。

下面我们要为 `HyperCam` 写一个注册码生成器，通过对 `HyperCam` 注册检验过程的分析，我们已经知道，该注册码生成器的输入信息为：

1.  用户注册的姓名 `Name`。
2.  用户注册的方式 `Register type`。
3.  对于以第一种注册方式(`Single User`)注册的用户，还需要输入注册的数目 `Copy Number`.

根据输入的信息，可以生成中间代码 `SecretCode`，它由三部分组成，分别对应于下列信息：

1.  第一部分为 3 个字符，表示注册的方式代码，可以由输入 2、3 换算得到。
2.  第二部分为固定的三个字符 `"HCA"`。
3.  第三部分为 `LName`，可以由输入 `Name` 换算得到。

得到了 `SecretCode` 之后，我们可以进一步求取最终的注册码 `Register Code`，这是一个逆向的过程，由于 `SecretCode` 的每个字符是通过 `Register Code` 的当前字符以及 `SecretCode` 的前一个字符生成的，因此，只要注册码的前面部分是正确的，我们就可以采用穷举法变换 `Register Code` 当前的字符，并检验由此生成的 `SecretCode` 是否合法，从而能从左自右，试探出全部合法的注册字符。由于每个字符只能取大写字符，因此全部的工作量只有 26 X `strlen(Register Code)`。另外注册码 `Register Code` 除了要求能正确生成 `SecretCode` 外，其自身还要满足一定的条件（在第 12 步中已经指出），这可以通过对 `Register Code` 的最后一个字符进行检验来实现。

附: `HyperCam` 的注册码生成器 `KeyMaker.C`

```c
---------------------- KeyMaker.C -----------------------------

#include <stdio.h>   

#include <string.h>   

#include <malloc.h>   

#include <ctype.h>   

#include <process.h>   

int shiftsum(char *result, int len)   

{   

int fact,i,x,sum=0;   

char c;   

fact=1;   

if (len<fact) len=strlen(result);   

if (len<=0) return sum;   

for (i=0; i<len; i++) {   

  c=result[i];   

  x=c-'A';   

  x*=fact;   

  sum += x;   

  fact = 26*fact;   

}   

return sum;   

}   

char checksum(char *regcode)   

{   

int sum=0,val,i;   

char c,tmp;   

for (i=0;i<strlen(regcode);i++) {   

  c=regcode[i];   

  val=c-'A';   

  if (i & 1) val=val*2;   

  if (val>=26) val=val-25;   

  sum += val;   

}   

tmp=sum % 26;   

return tmp;   

}   

int genregcode(char *regcode, char *result, int *tail)   

{   

char *table="EHIOTBVPSFRNWYQCJAGKUZXDML";   

char tmpcode[27];   

int  i,j,x,y;   

long  len;   

char c;   

/* Filter characters that are not in 'A' to 'Z' */   

for (i=0,j=0; regcode[i]!='\0'; i++)   

  if (isupper(regcode[i])) tmpcode[j++]=regcode[i];   

tmpcode[j]='\0';   

/* Test the len of the regcode */   

len=strlen(tmpcode);   

if ((len>=128) || (len<=3)) {   

  printf("error: cannot accept a regcode with mislen");   

  return NULL;   

}   

i=j=0;   

y=tmpcode[2]-'A';   

for (i=0;i<len;i++)   

if (i!=2) {   

  c=tmpcode[i];   

  if (c<'A') return NULL;   

  if (tmpcode[j]>'Z') return NULL;   

  x=strchr(table, c)-table;   

  x=x-y;   

  x--;   

  if (x<0) x+=26;   

  result[j++]=x+'A';   

  y=c-'A';   

}   

result[j]='\0';   

*tail=result[j-1]-'A';   

result[j-1]='\0';   

return checksum(result);   

}   

void main()   

{   

char regcode[27];   

char secretcode[27],tmp[27];   

int i,j,len;   

char ch;   

char LName[33],Name[33];   

int rtype,copy;   

int chksum,tail;   

int again;   

for (i=0;i<26;i++)   

  regcode[i]='A';   

regcode[i]='\0';   

printf("\n\tHyper Cam Cracker\n\tWritten By Mr. Chcw\n\n");   

printf("\tPlease enter your name: ");   

gets(Name);   

strcpy(LName, Name);   

if (strlen(Name) >= 32) {   

  printf("Error : User Name is too long\n");   

  exit(1);   

}   

if (strlen(Name)<16) {   

  while (strlen(LName)<16)   

    strcat(LName, Name);   

  LName[16]='\0';   

}   

do  {   

  printf("\n\tPlease choose the license type:\n");   

  printf("\t1. Single User\n");   

  printf("\t2. Unlimited Site License\n");   

  printf("\t3. Unlimited World Wide License\n\t");   

  scanf("%d", &rtype);   

  again=0;   

  switch (rtype)  {   

    case 1:   

      printf("\n\tPlease enter number of lincensed: ");   

      scanf("%d", &copy);   

      secretcode[0] = 'P';   

      secretcode[1] = 'A' + copy % 26;   

      secretcode[2] = 'A' + copy / 26;   

      break;   

    case 2:   

      strcpy(secretcode, "SIT");   

      break;   

    case 3:   

      strcpy(secretcode, "WOR");   

      break;   

    default:   

      printf("\tIncorrect option, please enter a number in 1-3");   

      again=1;   

  }   

} while (again);   

secretcode[3]='\0';   

strcat(secretcode, "HCA");      // Add the  HCA mark
```



```
strcat(secretcode, LName);   // Add the  LName String   

l  4dd466d0e1e0cc123afbe504da2462ca
```
