
# Duelist Crackme 4 破解

> 原文：[`www.pediy.com/kssd/pediy03/forum381.htm`](https://www.pediy.com/kssd/pediy03/forum381.htm)

**作者**：丁丁虾 `ddxia/[CCG]`  
**E\_mail**：`ddxia@263.net` (如果 Crackme 教学文章中有错，请来信:))  
**Web**：`http://go18.163.com/~ddxia`  
**目标**：`duelist crackme 4`  
**目标 URL**：`http://go18.163.com/~ddxia/crackme/DueList/Due-cm4.zip`  
**工具**：`SoftIce4.01 for NT`，`winapi.hlp`

据 `readme.txt` 说，用秘密表的形式来进行加密，解起来有些困难。同时要做一个 `KEYGEN`。

这次不用 `BPX MESSAGEBOXA` 换一下口味！:)

因为这次 `Crackme` 是要输入 `USER/CODE`，一定要经过变化才进行比较，所以在 `Crackme` 得到假的 `USER/CODE` 时，把它断下来。

```
bpx getwindowtexta  ------啊哦！！没有用！
bpx getdlgitem        ------与 DLG 操作有都断下来。
```

结果按下 `check` 后，就给断下来了，`F11` 然后再按 `F12` 返回 `Crackme` 的代码空间。

用 `SICE` 截下如下代码，`COME ON BABY`！！分析代码哦！！非常的有趣的活哦！:)

```
:u 401127 L13d

001B:00401127  6A00             PUSH        00
001B:00401129  6A00             PUSH        00
001B:0040112B  6A0E             PUSH        0E
001B:0040112D  6A03             PUSH        03
001B:0040112F  FF7508           PUSH        DWORD PTR [EBP+08]
001B:00401132  E841020000       CALL        USER32!SendDlgItemMessageA
```

`SendDlgItemMessage` 这个 API 调用了四次，来我们看一看它到底有什么用？

```c
LONG SendDlgItemMessage(
    HWND hDlg,             // 对话框的句柄
    int nIDDlgItem,        // 控件的 ID 号
    UINT Msg,              // 传送的消息
    WPARAM wParam,         // 第一个消息参数
    LPARAM lParam          // 第二个消息参数
);
```

从程序中我们可以知道当 `UINT Msg=E` 时是取输入字符的个数。  
当 `UINT Msg=D` 时是取输入字符的内容。

```
001B:00401137  A3AF214000       MOV         [004021AF],EAX
                                ^^^^^^^^^^^^^^^^^^^^^^^^---->取输入 USER 的个数

001B:0040113C  83F800          CMP         EAX,00
001B:0040113F  0F84D5000000    JZ          0040121A

001B:00401145  83F808          CMP         EAX,08
001B:00401148  0F8FCC000000    JG          0040121A

                                ^^^^^^^^^^^^^^^^^^^^^^^----->检查输入个数是否
                                    大于 0 小于 8

001B:0040114E  8BF0            MOV         ESI,EAX
                                ^^^^^^^^^^^^^^^^^^^^^^^----->注意这个

001B:00401150  6A00            PUSH        00
001B:00401152  6A00            PUSH        00
001B:00401154  6A0E            PUSH        0E
001B:00401156  6A04            PUSH        04
001B:00401158  FF7508          PUSH        DWORD PTR [EBP+08]
001B:0040115B  E818020000      CALL        USER32!SendDlgItemMessageA

001B:00401160  83F800          CMP         EAX,00
001B:00401163  0F84B1000000    JZ          0040121A

001B:00401169  3BF0            CMP         ESI,EAX
001B:0040116B  0F85A9000000    JNZ         0040121A

                                ^^^^^^^^^^^^^^^^^^^^^^^------>比较 user/code
                                     输入的个数是否相等
                                     相等就继续

001B:00401171  6860214000      PUSH        00402160
001B:00401176  6A08            PUSH        08
001B:00401178  6A0D            PUSH        0D
001B:0040117A  6A03            PUSH        03
001B:0040117C  FF7508          PUSH        DWORD PTR [EBP+08]
001B:0040117F  E8F4010000      CALL        USER32!SendDlgItemMessageA

                                ^^^^^^^^^^^^^^^^^^^^^^^^----->取的 USER 放入 402160

001B:00401184  6879214000      PUSH        00402179
001B:00401189  6A10            PUSH        10
001B:0040118B  6A0D            PUSH        0D
001B:0040118D  6A04            PUSH        04
001B:0040118F  FF7508          PUSH        DWORD PTR [EBP+08]
001B:00401192  E8E1010000      CALL        USER32!SendDlgItemMessageA

                                ^^^^^^^^^^^^^^^^^^^^^^^^----->取的 CODE 放入 402160

001B:00401197  B9FFFFFFFF      MOV         ECX,FFFFFFFF
001B:0040119C  41              INC         ECX
001B:0040119D  0FBE8160214000  MOVSX       EAX,BYTE PTR [ECX+00402160]
001B:004011A4  83F800          CMP         EAX,00
001B:004011A7  7432            JZ          004011DB

001B:004011A9  BEFFFFFFFF      MOV         ESI,FFFFFFFF
001B:004011AE  83F841          CMP         EAX,41
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->41='A'

001B:004011B1  7C67            JL          0040121A

001B:004011B3  83F87A          CMP         EAX,7A
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->7A='z'

001B:004011B6  7762            JA          0040121A

001B:004011B8  83F85A          CMP         EAX,5A
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->41='Z'

001B:004011BB  7C03            JL          004011C0
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->这几句代码是判断
                                    USER 是否输入了字符

001B:004011BD  83E820          SUB         EAX,20
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->全部转化为大写

001B:004011C0  46              INC         ESI
001B:004011C1  0FBE9617204000  MOVSX       EDX,BYTE PTR [ESI+00402017]
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->在 402017 表相对应的
                                    位置

001B:004011C8  3BC2            CMP         EAX,EDX
001B:004011CA  75F4            JNZ         004011C0
                                ^^^^^^^^^^^^^^^^^^^^^^^^----->表中找到后在去另一
                                    40203C 表去寻找对应
                                    的值。

001B:004011CC  0FBE863C204000  MOVSX       EAX,BYTE PTR [ESI+0040203C]
001B:004011D3  898194214000    MOV         [ECX+00402194],EAX
001B:004011D9  EBC1            JMP         0040119C

:d 402017

0023:00402017 41 31 4C 53 4B 32 44 4J-46 34 48 47 50 33 51 57  A1LSK2DJF4HGP3QW
0023:00402027 4F 35 45 49 52 36 55 54-59 5A 38 4D 58 4E 37 43  O5EIR6UTYZ8MXN7C
0023:00402037 42 56 39 00                  BV9.

:d 40203c

0023:0040203C 53 55 37 43 53 4A 4B 46-30 39 4E 43 53 44 4F 39  SU7CSJKF09NCSDO9
0023:0040204C 53 44 46 30 39 53 44 52-4C 56 4B 37 38 30 39 53  SDF09SDRLVK7809S
0023:0040205C 34 4E 46 00                  4NF.

001B:004011DB  FF35AF214000      PUSH        DWORD PTR [004021AF]
001B:004011E1  6894214000        PUSH        00402194
001B:004011E6  6879214000        PUSH        00402179
001B:004011EB  E854000000        CALL        00401244
                                ^^^^^^^^^^^^^^^^^^^^^^----->比较子程序
                                     转换后的值与 CODE 相比较

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

比较程序代码：（虽然简单，但是有漏洞哦！:) 呵呵~~~）

当 `USER` 为一个字母的时候，`CODE` 可以为任意字符 :)

可以试一下 `user:a /code:=` 是可以的！！

想一想，关键就在这个 `JCXZ 00401260` 的判断。

不知道是作者故意的，还是 BUG，自己去想吧 呵呵~~~~~~

:u 401244 L20

001B:00401244  C8000000          ENTER       0000,00
001B:00401248  B801000000        MOV         EAX,00000001
001B:0040124D  8B7D08            MOV         EDI,[EBP+08]
001B:00401250  8B750C            MOV         ESI,[EBP+0C]
001B:00401253  8B4D10            MOV         ECX,[EBP+10]
001B:00401256  F3A6              REPZ CMPSB
001B:00401258  67E305            JCXZ        00401260
001B:0040125B  B800000000        MOV         EAX,00000000
001B:00401260  C9                LEAVE
001B:00401261  C20C00            RET         000C

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

001B:004011F0  83F801            CMP         EAX,01
001B:004011F3  0F84DEFEFFFF      JZ          004010D7
001B:004011F9  EB1F              JMP         0040121A
001B:004011FB  837D1001          CMP         DWORD PTR [EBP+10],01
001B:004011FF  0F8422FFFFFF      JZ          00401127
001B:00401205  837D1002          CMP         DWORD PTR [EBP+10],02
001B:00401209  752F              JNZ         0040123A
001B:0040120B  E8B4000000        CALL        KERNEL32!ExitProcess
001B:00401210  B801000000        MOV         EAX,00000001
```



# 001B:00401215 E9FFFEFFFF JMP 00401119
`001B:00401215` `E9FFFEFFFF` `JMP` `00401119`

`001B:0040121A` `6800200000` `PUSH` `00002000`
`001B:0040121F` `6801204000` `PUSH` `00402001`
`001B:00401224` `68AE204000` `PUSH` `004020AE`
`001B:00401229` `6A00` `PUSH` `00`
`001B:0040122B` `E836010000` `CALL` `USER32!MessageBoxA`
`001B:00401230` `B800000000` `MOV` `EAX,00000000`
`001B:00401235` `E9DFFEFFFF` `JMP` `00401119`
`001B:0040123A` `B800000000` `MOV` `EAX,00000000`
`001B:0040123F` `E9D5FEFFFF` `JMP` `00401119`

时间：2000.10.19 20:99

## duelist crackme 4 破解（下）
*(希望转载的时候保持完整)*

作者：丁丁虾 `ddxia/\[CCG\]`

E\_mail：`ddxia@263.net` (如果 Crackme 教学文章中有错，请来信:))

Web：`http://go18.163.com/~ddxia`

目标：`duelist crackme 4`

目标 URL：`http://go18.163.com/~ddxia/crackme/DueList/Due-cm4.zip`

工具：`SoftIce4.01 for NT` `TOPO V1.2` `Hiew` `winapi.hlp`

下一步，如果按 Crackme 的要求是让我们写一个 `KEYGEN`，俺天性就是懒懒懒.......于是想出个馊主意，把增加代码把这个 Crackme 本身变成一个 `KEYGEN`，孙悟空七十二变 呵呵~~~~~~~~

步骤如下：
1、先用 `TOPO` 软件定位我们加入代码的位置
2、用 `HIEW` 来修改程序。

### 用 `TOPO V1.2` 来玩一玩！
首先在 `SCAN` 选项中选取 `EXEC ONLY`，是扫描代码段中有没有空余的地方，增加我们要加入的代码。`OPEN` 文件 `due-cm4.exe` 发现有 285 个字节可以利用。找个吉利的数 88，好！利用 88 个字节进行修改程序，程序定位在内存中位置为 `4014e3` 物理位置为 `AE3`。

我们想达到的目的是当程序判断我们的 `CODE` 是错误的时候，让程序本身跳出一个对话框来告诉大家正确的 `CODE` ：）

首先，我们确定那里是合适加入代码的地方？？很关键哦！我们找啊找，找到一个好朋友 呵呵~~~~~

比较程序代码：

`u 401244 L20`

```
001B:00401244 C8000000            ENTER   0000,00
001B:00401248 B801000000          MOV     EAX,00000001
001B:0040124D 8B7D08              MOV     EDI,[EBP+08]
001B:00401250 8B750C              MOV     ESI,[EBP+0C]
001B:00401253 8B4D10              MOV     ECX,[EBP+10]
001B:00401256 F3A6                REPZ CMPSB
001B:00401258 67E305              JCXZ    00401260
001B:0040125B B800000000          MOV     EAX,00000000
```

^^^^^^^^^^^^^^^^^^^^^^---->修改这句代码，

`jmp 4014e3`

而且非常的巧，代码长度一样

```
001B:00401260 C9                  LEAVE
001B:00401261 C20C00              RET     000C
```

ok let's go goo! 运行 `HIEW`（真是一个酷 B 的软件），打开 `due-cm4.exe`.

按 `F4` 可以改变显示模式，选择 `DECODE`。找到上面的代码

按 `F3` 可以修改代码，按 `F2` 为切换到 `ASM` 模式。输入 `JMP 0AE3`（注意了这里的跳转地址为物理地址）

按 `F9` 存盘哦！

按 `F5` 可以 `GOTO` 输入 `AE3`！！

在 `AE3` 处输入

| 内存地址 | 物理地址 | 代码 |
| :--- | :--- | :--- |
| `4014E3` | `AE3` | `mov esi,[ebp][0000C]` |
| `4014E6` | `AE6` | `push 0` |
| `4014E8` | `AE8` | `push 4014FF` |
| `4014ED` | `AED` | `push esi` |
| `4014EE` | `AEE` | `push 0` |
| `4014F0` | `AF0` | `call 966` |
| `4014F5` | `AF5` | `mov eax,0` |
| `4014FA` | `AFA` | `jmp 860` |

存盘

按 `F4` 切换到 `HEX` 模式下，输入

`4014FF` `AFF` `Correct Code: (Edit by ddxia) 00`

^^^-->注意在 `HEX` 方式下输入

赶紧去试一试:)

哎哟！差点忘了，再把以下这段消息

`Your registration info is invalid... Note that most.....`

改为：请输入小于等于 8 个字符的 `USER` 名(不能包括数字)`CODE` 的长度要和 `USER` 的一样 00

终于修改完了，好累啊！懒人经常走弯路。

啊！小子，没有完呢？想丢笔去 `HAPPY` 呵呵~~~~~

我们来弄明白这为什么要这样改？？？

| 内存地址 | 物理地址 | 代码 |
| :--- | :--- | :--- |
| `4014E3` | `AE3` | `mov esi,[ebp][0000C]` ；把正确的 `CODE` 放入 `ESI` |
| `4014E6` | `AE6` | `push 0` |
| `4014E8` | `AE8` | `push 4014FF` |
| `4014ED` | `AED` | `push esi` |
| `4014EE` | `AEE` | `push 0` |
| `4014F0` | `AF0` | `call 966` ； `CALL MESSAGEBOX` |
| `4014F5` | `AF5` | `mov eax,0` ； 恢复被我们修改的代码 |
| `4014FA` | `AFA` | `jmp 860` ； 跳回 `LEAVE` 的位置 |

看看 `WINAPI.HLP` 中的 `MESSAGEBOXa`，需要什么参数

```c
int MessageBox(
    HWND hWnd,        //  窗口句柄    0------>缺省
    LPCTSTR lpText,   //  内容的地址    ESI\---->CODE
    LPCTSTR lpCaption, // 标题的地址    4014FF->我们加进的一串字符
    UINT uType       //  对话框的类型  0------>普通类型，只有一个 OK 按钮
);
```

然后我们研究一下为什么是 `CALL 966` ？？这个嘛！呵呵~~~~ Let's see

`40122b` `call 401366` ---->当按 `F3` 它就会变成 `call 966` 知道为什么了吧 :)

`401366` `jmp messageboxa`

再来看一看 `jmp 860` 860 又什么东东？:) 这个卖个关子了 呵呵！！

（提示：按 `F5` 输入 `860`，回车，你就明白了 哈哈~~）

终于写完了，怎么还没有人 `CALL` 我的，看来今天晚没有时间去 `HAPPY` 了，要整理整理我的站点了 （遭到投诉了 呵呵~~~~~）

时间：2000.10.19 22:05

```cpp
#include <iostream.h>
#include <string.h>
#include <stdio.h>

void main(void)
{
    int len,i,n;
    char user[30],*upstr;
    char table1[]="A1LSK2DJF4HGP3QWO5EIR6UTYZ8MXN7CBV9";
    char table2[]="SU7CSJKF09NCSDO9SDF09SDRLVK7809S4NF";
    char code[10];
    cout<<"=========================\n";
    cout<<"KeyGen for due-cm4.\n";
    cout<<"Coded by ddxia[CCG].\n";
    cout<<"http://go18.163.com/~ddxia\n";
    cout<<"=========================\n\n";
    for(;;) {
        cout<<"Your name:(Lenth should less than 9 ,only character)\n" ;
        gets(user);
        len=strlen(user);
        if ((len>1)&(len<=8)) {
            upstr=strupr(user);
            for(i=0;i<len;i++,upstr++) {
                if (((char)*upstr>='A')&((char)*upstr<='Z')){
                    for(n=0;n<strlen(table1);n++)
                        if((char)*upstr==table1[n]){
                            code[i]=table2[n];
                            break;
                        }
                }
                else goto error;
            }
            break;
        }
error:
        cout<<"Your name is invalid!\n\n";
    }
    cout<<"Your Correct Code:";
    code[len]=NULL;
    cout<<code<<"\n\n";
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
