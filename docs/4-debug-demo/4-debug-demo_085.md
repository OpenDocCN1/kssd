
# crackcode 代码分享

> 原文：[`www.pediy.com/kssd/pediy03/forum1217.htm`](https://www.pediy.com/kssd/pediy03/forum1217.htm)

## 笔记一

今天有些失眠，睡不着，所以闲着也是闲着。今天刚把 `crackcode` 下了，挺小的才 11.5k。

把它反汇编了。想慢慢的把程序从头读一遍，估计不会很难吧！：）今天还行，看了一点点，先和大家分享。我的汇编语言也学的马虎，如果你们觉注释还能看得过去的，就将就将！

```
:00401000 53                  push ebx
:00401001 55                  push ebp
:00401002 56                  push esi
:00401003 57                  push edi
:00401004 6A00                push 00000000

* Possible StringData Ref from Data Obj ->"Crackcode 2000 -- Author:Ru Feng "
                                   ->"(http:\\ocqpat.163.net)"
                                   |

:00401006 68AC624000          push 004062AC

* Possible StringData Ref from Data Obj ->"Thank you for using the Crackcode!Let "
                                   ->"us make the keygen so easy!"
                                   |

:0040100B 6868624000          push 00406268
:00401010 6A00                push 00000000

* Reference To: USER32.MessageBoxA, Ord:0000h
                                   |

:00401012 FF15C0504000        Call dword ptr [004050C0]
      ^^^^^^^^^^^^－－－》显示作者的堂堂大名。

:00401018 BE20A44000          mov esi, 0040A420
:0040101D BF04010000          mov edi, 00000104
:00401022 56                  push esi
:00401023 57                  push edi

* Reference To: KERNEL32.GetCurrentDirectoryA, Ord:0000h
                                   |

:00401024 FF1504504000        Call dword ptr [00405004]
      ^^^^^^^^^^^^－－－》取得当前的路径。

* Possible StringData Ref from Data Obj ->"CRACKCODE.INI"
                                   |

:0040102A 68DC604000          push 004060DC

* Possible StringData Ref from Data Obj ->"\\"
                                   |

:0040102F 6864624000          push 00406264
:00401034 56                  push esi
:00401035 E8360A0000          call 00401A70
:0040103A 59                  pop ecx
:0040103B 59                  pop ecx
:0040103C 50                  push eax
:0040103D E82E0A0000          call 00401A70
:00401042 59                  pop ecx
:00401043 BD50674000          mov ebp, 00406750
:00401048 59                  pop ecx
      ^^^^^^^^^^^^－－》以上这段代码实现一个函数
      `strcat()` 比如获得当前路径为 `C:\crackcode`；那么
      结果这段代码合并为：`c:\crackcode\crackcode.ini`
```

来看看 `00401A70` 中的代码：

```
:00401A70 8B4C2404            mov ecx, dword ptr [esp+04]
       ^^^^^^－－－》ECX 获得指向路径字符串
      `ECX=40A620`

:00401A74 57                  push edi
:00401A75 F7C103000000        test ecx, 00000003
:00401A7B 740F                je 00401A8C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401A8A(C)
|

:00401A7D 8A01                mov al, byte ptr [ecx]
:00401A7F 41                  inc ecx
:00401A80 84C0                test al, al
:00401A82 743B                je 00401ABF
:00401A84 F7C103000000        test ecx, 00000003
:00401A8A 75F1                jne 00401A7D

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401A7B(C), :00401AA2(C), :00401ABD(U)
|

:00401A8C 8B01                mov eax, dword ptr [ecx]
:00401A8E BAFFFEFE7E          mov edx, 7EFEFEFF
:00401A93 03D0                add edx, eax
:00401A95 83F0FF              xor eax, FFFFFFFF
:00401A98 33C2                xor eax, edx
:00401A9A 83C104              add ecx, 00000004
:00401A9D A900010181          test eax, 81010100
:00401AA2 74E8                je 00401A8C
       ^^^^^^^^^^^^^^－－》这段代码是循环取四个字符
       确定到那四个字符为结尾

:00401AA4 8B41FC              mov eax, dword ptr [ecx-04]
:00401AA7 84C0                test al, al
:00401AA9 7423                je 00401ACE
:00401AAB 84E4                test ah, ah
:00401AAD 741A                je 00401AC9
:00401AAF A90000FF00          test eax, 00FF0000
:00401AB4 740E                je 00401AC4
:00401AB6 A9000000FF          test eax, FF000000
:00401ABB 7402                je 00401ABF
:00401ABD EBCD                jmp 00401A8C
       ^^^^^^^^^^－－》这段代码为确定最后四个字符
       中有多少个字符。分别跳到下面相应的程序

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401A82(C), :00401ABB(C)
|

:00401ABF 8D79FF              lea edi, dword ptr [ecx-01]
:00401AC2 EB0D                jmp 00401AD1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AB4(C)
|

:00401AC4 8D79FE              lea edi, dword ptr [ecx-02]
:00401AC7 EB08                jmp 00401AD1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AAD(C)
|

:00401AC9 8D79FD              lea edi, dword ptr [ecx-03]
:00401ACC EB03                jmp 00401AD1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AA9(C)
|

＝＝＝＝＝＝＝＝＝＝＝＝＝＝下面的代码和也以上代码相似，不再作解释＝＝＝＝＝＝＝＝＝＝＝＝

:00401ACE 8D79FC              lea edi, dword ptr [ecx-04]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401A65(U), :00401AC2(U), :00401AC7(U), :00401ACC(U)
|

:00401AD1 8B4C240C            mov ecx, dword ptr [esp+0C]
:00401AD5 F7C103000000        test ecx, 00000003
:00401ADB 7419                je 00401AF6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401AED(C)
|

:00401ADD 8A11                mov dl, byte ptr [ecx]
:00401ADF 41                  inc ecx
:00401AE0 84D2                test dl, dl
:00401AE2 7464                je 00401B48
:00401AE4 8817                mov byte ptr [edi], dl
:00401AE6 47                  inc edi
:00401AE7 F7C103000000        test ecx, 00000003
:00401AED 75EE                jne 00401ADD
:00401AEF EB05                jmp 00401AF6

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401B0E(C), :00401B28(U)
|

:00401AF1 8917                mov dword ptr [edi], edx
:00401AF3 83C704              add edi, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401ADB(C), :00401AEF(U)
|

:00401AF6 BAFFFEFE7E          mov edx, 7EFEFEFF
:00401AFB 8B01                mov eax, dword ptr [ecx]
:00401AFD 03D0                add edx, eax
:00401AFF 83F0FF              xor eax, FFFFFFFF
:00401B02 33C2                xor eax, edx
:00401B04 8B11                mov edx, dword ptr [ecx]
:00401B06 83C104              add ecx, 00000004
:00401B09 A900010181          test eax, 81010100
:00401B0E 74E1                je 00401AF1
:00401B10 84D2                test dl, dl
:00401B12 7434                je 00401B48
:00401B14 84F6                test dh, dh
:00401B16 7427                je 00401B3F
:00401B18 F7C20000FF00        test edx, 00FF0000
:00401B1E 7412                je 00401B32
:00401B20 F7C2000000FF        test edx, FF000000
:00401B26 7402                je 00401B2A
:00401B28 EBC7                jmp 00401AF1
```

### 小结

希望大家能不能帮我解答，以上代码中不明白的地方。

1. `:00401A75 F7C103000000` `test ecx, 00000003` －－》为什么要 `TEST`?
2. 对以下代码中为什么要加上 `7EFEFEFF` 然后再 `XOR FFFFFFFF`，能不能告诉我原因？



:00401A8C `8B01`                     `mov eax, dword ptr [ecx]`

:00401A8E `BAFFFEFE7E`             `mov edx, 7EFEFEFF`

:00401A93 `03D0`                   `add edx, eax`

:00401A95 `83F0FF`                 `xor eax, FFFFFFFF`

:00401A98 `33C2`                   `xor eax, edx`

:00401A9A `83C104`                 `add ecx, 00000004`

:00401A9D `A900010181`             `test eax, 81010100`

:00401AA2 `74E8`                   `je 00401A8C`

谢谢大家的帮助！！

今天就到这里，就到这里。。。。。。  （待续）豆豆虾

2001.8.9 23：36 完成

历时：1 个小时。

以下是两位组织代号为`[BCG]`的回答：（感谢这两位热心的朋友）

`6767[BCG]`

1. `pop ecx` 应该是对应前面的那个 `push eax`，测试 `call` 的返回值；
2. 在许多程序中见过，可能与编译器有关

`冷雨飘心[BCG]`

1. `test ecx, 3` -> 判断是否从内存存储以 4 为节的地址开始，这可能是某种编译器所要求的存储方式，还有
2. 是一段经典程序，估计是 `VC` 生的代码，在于判断最后取到的四个字符是否全部为 0，加 `7EFEFEFF`
然后再 `XOR FFFFFFFF` 变反最后与 `81010100` 比较正是起到了这种作用，你可以自己运算一下就会明白了.

于是我用 `vc` 模拟 `crackcode`，通过 `strcat` 来实现该功能。发现用 `VC6` 编译出来，和 `crackcode` 的代码完全不同。估计是作者使用另一函数来实现。（原因在于如果人为的去写程序，估计不会用到 `TEST EAX, 3` 这段程序。）

```c
#include <afx.h>

#include<string.h>

int main()

{

char buffer[20];

char aaa[]="/";

char bbb[]="a xia";

    strcpy(buffer,"I am");

        strcat(buffer,aaa);

    strcat(buffer,bbb);

return 0 ;

}
```

`00401010 56`                        `push esi`

:00401011 `894C240C`                  `mov dword ptr [esp+0C], ecx`

:00401015 `57`                        `push edi`

:00401016 `668944240E`               `mov word ptr [esp+0E], ax`

:0040101B `83C9FF`                   `or ecx, FFFFFFFF`

\* Possible StringData Ref from Data Obj ->"I am"

:0040101E `BF30504000`               `mov edi, 00405030`

:00401023 `33C0`                     `xor eax, eax`

:00401025 `F2`                       `repnz`

:00401026 `AE`                       `scasb`

:00401027 `668B153C504000`          `mov dx, word ptr [0040503C]`

:0040102E `F7D1`                     `not ecx`

:00401030 `2BF9`                     `sub edi, ecx`

:00401032 `6689542414`              `mov word ptr [esp+14], dx`

:00401037 `8D542418`                `lea edx, dword ptr [esp+18]`

:0040103B `8BC1`                     `mov eax, ecx`

:0040103D `8BF7`                     `mov esi, edi`

:0040103F `8BFA`                     `mov edi, edx`

:00401041 `C1E902`                   `shr ecx, 02`

:00401044 `F3`                       `repz`

:00401045 `A5`                       `movsd`

:00401046 `8BC8`                     `mov ecx, eax`

:00401048 `33C0`                     `xor eax, eax`

:0040104A `83E103`                   `and ecx, 00000003`

:0040104D `8D542418`                `lea edx, dword ptr [esp+18]`

:00401051 `F3`                       `repz`

:00401052 `A4`                       `movsb`

:00401053 `8D7C240E`                `lea edi, dword ptr [esp+0E]`

:00401057 `83C9FF`                  `or ecx, FFFFFFFF`

:0040105A `F2`                       `repnz`

:0040105B `AE`                       `scasb`

:0040105C `F7D1`                     `not ecx`

:0040105E `2BF9`                     `sub edi, ecx`

:00401060 `8BF7`                     `mov esi, edi`

:00401062 `8BD9`                     `mov ebx, ecx`

:00401064 `8BFA`                     `mov edi, edx`

:00401066 `83C9FF`                  `or ecx, FFFFFFFF`

:00401069 `F2`                       `repnz`

:0040106A `AE`                       `scasb`

:0040106B `8BCB`                     `mov ecx, ebx`

:0040106D `4F`                       `dec edi`

:0040106E `C1E902`                   `shr ecx, 02`

:00401071 `F3`                       `repz`

:00401072 `A5`                       `movsd`

:00401073 `8BCB`                     `mov ecx, ebx`

:00401075 `8D542418`                `lea edx, dword ptr [esp+18]`

:00401079 `83E103`                   `and ecx, 00000003`

:0040107C `F3`                       `repz`

:0040107D `A4`                       `movsb`

:0040107E `8D7C2410`                `lea edi, dword ptr [esp+10]`

:00401082 `83C9FF`                  `or ecx, FFFFFFFF`

:00401085 `F2`                       `repnz`

:00401086 `AE`                       `scasb`

:00401087 `F7D1`                     `not ecx`

:00401089 `2BF9`                     `sub edi, ecx`

:0040108B `8BF7`                     `mov esi, edi`

:0040108D `8BD9`                     `mov ebx, ecx`

:0040108F `8BFA`                     `mov edi, edx`

:00401091 `83C9FF`                  `or ecx, FFFFFFFF`

:00401094 `F2`                       `repnz`

:00401095 `AE`                       `scasb`

:00401096 `8BCB`                     `mov ecx, ebx`

:00401098 `4F`                       `dec edi`

:00401099 `C1E902`                   `shr ecx, 02`

:0040109C `F3`                       `repz`

:0040109D `A5`                       `movsd`

:0040109E `8BCB`                     `mov ecx, ebx`

:004010A0 `83E103`                   `and ecx, 00000003`

:004010A3 `F3`                       `repz`

:004010A4 `A4`                       `movsb`

:004010A5 `5F`                       `pop edi`

:004010A6 `5E`                       `pop esi`

:004010A7 `5B`                       `pop ebx`

:004010A8 `83C420`                   `add esp, 00000020`

:004010AB `C3`                       `ret`

今天就到这里，就到这里。。。。。。    豆豆虾

2001.8.10 15：10 完成

历时：0.5 个小时。

感谢你们看我的笔记，能够互相交流提高！谁能告诉我上一篇留下的问题吗？？谢谢！！

接着上一回继续看代码。看代码之前我们先来认识一个 API 函数：

功能：获取 INI 文件中与指定关键字对应的参数（字符串性）并将此参数拷贝到 `lpReturnedString` 中。

```c
DWORD GetPrivateProfileString(
  LPCTSTR lpAppName,        // points to section name
  LPCTSTR lpKeyName,        // points to key name
  LPCTSTR lpDefault,        // points to default string
  LPTSTR lpReturnedString,  // points to destination buffer
  DWORD nSize,              // size of destination buffer
  LPCTSTR lpFileName        // points to initialization filename
);
```

参数

- `lpApplicationName` String 待寻找的关键字所在的段
- `lpKeyName` String 与参数相对应的关键字名
- `lpDefault` String 指定的关键字不存在时返回的缺省值
- `lpReturnedString` String 预先分配好的长度至少为 `nSize` 字节的字符串缓冲区
- `nSized` Integer 将要装入 `lpReturnedString` 缓冲区的最大字符数
- `lpFileName` String 初始化文件的名字

返回值 Integer 拷贝到 `lpReturnedString` 缓冲区的字符数

\* Possible StringData Ref from Data Obj ->"Options"

:00401049 `BB30604000`               `mov ebx, 00406030`

:0040104E `56`                       `push esi`    //==> INI FILENAME

:0040104F `57`                       `push edi`    //==>SIZE OF BUFFER

\* Reference To: `KERNEL32.GetPrivateProfileStringA`, Ord:0000h

:00401050 `8B3D60504000`            `mov edi, dword ptr [00405060]`

:00401056 `6824A54000`              `push 0040A524`  //==>POINT FOR BUFFER

:0040105B `55`                       `push ebp`    //==>DEFAULT STRING

\* Possible StringData Ref from Data Obj ->"CommandLine"

:0040105C `68D0604000`              `push 004060D0`   //==>KEY NAME

:00401061 `53`                       `push ebx`    //==>SECTION NAME

:00401062 `FFD7`                    `call edi`    //==>`GetPrivateProfileStringA`

:00401064 `56`                       `push esi`

:00401065 `6A02`                    `push 00000002`

:00401067 `6840A64000`              `push 0040A640`

:0040106C `55`                       `push ebp`

\* Possible StringData Ref from Data Obj ->"Mode"

:0040106D `685C624000`              `push 0040625C`

:00401072 `53`                       `push ebx`

:00401073 `FFD7`                    `call edi`

:00401075 `56`                       `push esi`

:00401076 `6A09`                    `push 00000009`

:00401078 `6888974000`              `push 00409788`

:0040107D `55`                       `push ebp`

\* Possible StringData Ref from Data Obj ->"First\_Break\_Address"

:0040107E `6848624000`              `push 00406248`

:00401083 `53`                       `push ebx`



```
:00401084 FFD7                   call edi   

:00401086 56                     push esi   

:00401087 6A03                   push 00000003   

:00401089 6884974000             push 00409784   

:0040108E 55                     push ebp   

* Possible StringData Ref from Data Obj -> "First_Break_Address_Code"   
                             |   

:0040108F 682C624000             push 0040622C   

:00401094 53                     push ebx   

:00401095 FFD7                   call edi   

:00401097 56                     push esi   

:00401098 6A03                   push 00000003   

:0040109A 6880974000             push 00409780   

:0040109F 55                     push ebp   

* Possible StringData Ref from Data Obj -> "First_Break_Address_Code_Lenth"   
                             |   

:004010A0 680C624000             push 0040620C   

:004010A5 53                     push ebx   

:004010A6 FFD7                   call edi   

:004010A8 56                     push esi   

:004010A9 6A09                   push 00000009   

:004010AB 6874974000             push 00409774   

:004010B0 55                     push ebp   

* Possible StringData Ref from Data Obj -> "Second_Break_Address"   
                             |   

:004010B1 68F4614000             push 004061F4   

:004010B6 53                     push ebx   

:004010B7 FFD7                   call edi   

:004010B9 56                     push esi   

:004010BA 6A03                   push 00000003   

:004010BC 6870974000             push 00409770   

:004010C1 55                     push ebp   

* Possible StringData Ref from Data Obj -> "Second_Break_Address_Code_Lenth"   
                             |   

:004010C2 68D4614000             push 004061D4   

:004010C7 53                     push ebx   

:004010C8 FFD7                   call edi   

:004010CA 56                     push esi   

:004010CB BE64974000             mov esi, 00409764   

:004010D0 6A09                   push 00000009   

:004010D2 56                     push esi   

:004010D3 55                     push ebp   

* Possible StringData Ref from Data Obj -> "Save_Code_Address"   
                             |   

:004010D4 68C0614000             push 004061C0   

:004010D9 53                     push ebx   

:004010DA FFD7                   call edi   

:004010DC 6888974000             push 00409788   
```

通过 `GetPrivateProfileStringA` 获得各个参数的值。也就是读出 `crackcode.ini` 的内容。

```
[Options]
CommandLine=winzip32.exe
Mode=2
First_Break_Address=407C1C
First_Break_Address_Code=BE
First_Break_Address_Code_Lenth=5
Second_Break_Address=457923
Second_Break_Address_Code_Lenth=2
Save_Code_Address=EDI
```

今天就到这里，就到这里。。。。。。          （待续）豆豆虾

2001.8.10 18：36 完成

历时：0.5 个小时。

来！继续 `crackcode` 的下一步代码分析：

```
:004010DC 6888974000             push 00409788  －－》point to "First_Break_Address"   

:004010E1 E8A8070000             call 0040188E  －－》把文本地址转换为 HEX   

:004010E6 A3A0984000             mov dword ptr [004098A0], eax －－》奖转换后的结果保存   

:004010EB C7042480974000         mov dword ptr [esp], 00409780  －－》point to "First_Break_Address_Code_Lenth"   

:004010F2 E897070000             call 0040188E   

:004010F7 8B0DA0984000           mov ecx, dword ptr [004098A0]   

:004010FD A350A64000             mov dword ptr [0040A650], eax   

:00401102 03C1                   add eax, ecx               －－》First_Break_Address+lenth   

:00401104 C7042474974000         mov dword ptr [esp], 00409774   

:0040110B A39C984000             mov dword ptr [0040989C], eax   

:00401110 E879070000             call 0040188E   

:00401115 A398984000             mov dword ptr [00409898], eax   

:0040111A C7042470974000         mov dword ptr [esp], 00409770 －－》"Second_Break_Address"   

:00401121 E868070000             call 0040188E   

:00401126 8B0D98984000           mov ecx, dword ptr [00409898] －－》"Second_Break_Address_Code_Lenth"   

:0040112C A354A64000             mov dword ptr [0040A654], eax   

:00401131 BF40A64000             mov edi, 0040A640   

:00401136 03C1                   add eax, ecx               －－》Second_Break_Address+lenth   

* Possible StringData Ref from Data Obj -> "0"   
                             |   

:00401138 C70424BC614000         mov dword ptr [esp], 004061BC   

:0040113F 57                     push edi                   －－》"MODE"   

:00401140 A394984000             mov dword ptr [00409894], eax   

:00401145 E886080000             call 004019D0                  

:0040114A 59                     pop ecx   

:0040114B 85C0                   test eax, eax   

:0040114D 59                     pop ecx   

:0040114E 7506                   jne 00401156   

:00401150 210544A64000           and dword ptr [0040A644], eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040114E(C)   

|   

* Possible StringData Ref from Data Obj -> "1"   
                             |   

:00401156 68B8614000             push 004061B8   

:0040115B 57                     push edi   

:0040115C E86F080000             call 004019D0   

:00401161 59                     pop ecx   

:00401162 85C0                   test eax, eax   

:00401164 59                     pop ecx   

:00401165 750A                   jne 00401171   

:00401167 C70544A6400001000000   mov dword ptr [0040A644], 00000001   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00401165(C)   

|   

* Possible StringData Ref from Data Obj -> "2"   
                             |   

:00401171 68B4614000             push 004061B4   

:00401176 57                     push edi   

:00401177 E854080000             call 004019D0   

:0040117C 59                     pop ecx   

:0040117D 85C0                   test eax, eax   

:0040117F 59                     pop ecx   

:00401180 750A                   jne 0040118C   

:00401182 C70544A6400002000000   mov dword ptr [0040A644], 00000002   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:00401180(C)   

|   

* Possible StringData Ref from Data Obj -> "3"   
                             |   

:0040118C 68B0614000             push 004061B0   

:00401191 57                     push edi   

:00401192 E839080000             call 004019D0   

:00401197 59                     pop ecx   

:00401198 85C0                   test eax, eax   

:0040119A 59                     pop ecx   

:0040119B 750A                   jne 004011A7   

:0040119D C70544A6400003000000   mov dword ptr [0040A644], 00000003   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040119B(C)   

|   

＝＝＝＝以上程序是对处理 mode 不同值，在[0040A644]地址中作标志＝＝＝＝＝＝＝＝＝＝＝   

相当于   

```
switch( mode )   

{   

    case '0':   

        mode=0;   

    case '1':   

        mode=1;   

    case '2':   

        mode=2;   

    case '3':   

        mode=3;   

}
```

* Possible StringData Ref from Data Obj -> "EAX"   
                             |   

:004011A7 68AC614000             push 004061AC   

:004011AC 56                     push esi      －－》"Save_Code_Address"   

:004011AD E81E080000             call 004019D0   

:004011B2 59                     pop ecx   

:004011B3 85C0                   test eax, eax   

:004011B5 59                     pop ecx   

:004011B6 7511                   jne 004011C9   

:004011B8 C605609740008B         mov byte ptr [00409760], 8B   

:004011BF C7054CA64000F0909090   mov dword ptr [0040A64C], 909090F0   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:004011B6(C)   

|   

* Possible StringData Ref from Data Obj -> "EBX"   
                             |



```
:004011C9 68A8614000       push 004061A8
:004011CE 56              push esi
:004011CF E8FC070000       call 004019D0
:004011D4 59              pop ecx
:004011D5 85C0            test eax, eax
:004011D7 59              pop ecx
:004011D8 7511            jne 004011EB
:004011DA C605609740008B    mov byte ptr [00409760], 8B
:004011E1 C7054CA64000F3909090 mov dword ptr [0040A64C], 909090F3
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004011D8(C)`

* Possible StringData Ref from Data Obj -> `"ECX"`

```
:004011EB 68A4614000       push 004061A4
:004011F0 56              push esi
:004011F1 E8DA070000       call 004019D0
:004011F6 59              pop ecx
:004011F7 85C0            test eax, eax
:004011F9 59              pop ecx
:004011FA 7511            jne 0040120D
:004011FC C605609740008B    mov byte ptr [00409760], 8B
:00401203 C7054CA64000F1909090 mov dword ptr [0040A64C], 909090F1
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004011FA(C)`

* Possible StringData Ref from Data Obj -> `"EDX"`

```
:0040120D 68A0614000       push 004061A0
:00401212 56              push esi
:00401213 E8B8070000       call 004019D0
:00401218 59              pop ecx
:00401219 85C0            test eax, eax
:0040121B 59              pop ecx
:0040121C 7511            jne 0040122F
:0040121E C605609740008B    mov byte ptr [00409760], 8B
:00401225 C7054CA64000F2909090 mov dword ptr [0040A64C], 909090F2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040121C(C)`

* Possible StringData Ref from Data Obj -> `"EDI"`

```
:0040122F 689C614000       push 0040619C
:00401234 56              push esi
:00401235 E896070000       call 004019D0
:0040123A 59              pop ecx
:0040123B 85C0            test eax, eax
:0040123D 59              pop ecx
:0040123E 7511            jne 00401251
:00401240 C605609740008B    mov byte ptr [00409760], 8B
:00401247 C7054CA64000F7909090 mov dword ptr [0040A64C], 909090F7
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040123E(C)`

* Possible StringData Ref from Data Obj -> `"ESI"`

```
:00401251 6898614000       push 00406198
:00401256 56              push esi
:00401257 E874070000       call 004019D0
:0040125C 59              pop ecx
:0040125D 85C0            test eax, eax
:0040125F 59              pop ecx
:00401260 7511            jne 00401273
:00401262 C6056097400090    mov byte ptr [00409760], 90
:00401269 C7054CA6400090909090 mov dword ptr [0040A64C], 90909090
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00401260(C)`

* Possible StringData Ref from Data Obj -> `"EBP"`

```
:00401273 6894614000       push 00406194
:00401278 56              push esi
:00401279 E852070000       call 004019D0
:0040127E 59              pop ecx
:0040127F 85C0            test eax, eax
:00401281 59              pop ecx
:00401282 7511            jne 00401295
:00401284 C605609740008B    mov byte ptr [00409760], 8B
:0040128B C7054CA64000F5909090 mov dword ptr [0040A64C], 909090F5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00401282(C)`

* Possible StringData Ref from Data Obj -> `"EBP"`

＝＝＝＝以上程序是对处理 `Save_Code_Address` 不同值，做相应的处理＝＝＝＝＝＝

相当于

```
switch( Save_Code_Address )
{
    case 'EAX':
        code1=8B;
        code2=909090F0；
    case 'EBX':
        code1=8B;
        code2=909090F0；
    case 'EC':
        code1=8B;
        code2=909090F0；
    case 'EDX':
        code1=8B;
        code2=909090F0；
    case 'EDI':
        code1=8B;
        code2=909090F0；
    case 'ESI':
        code1=90;
        code2=909090F0；
    case 'EBP':
        code1=8B;
        code2=909090F0；
}
```

小结： 希望大家能帮我解答。为什么不同的寄存器会变成这样不同的数字？

它到底是代表什么？？

今天就到这里，就到这里。。。。。。（待续）豆豆虾

2001.8.11 17：17 完成

历时：1.8 个小时。

Go on..... 各位看官： （上一讲中提到的问题，不知道谁知道？能否赐教！谢了！）

```
:00401295 833D4CA6400000      cmp dword ptr [0040A64C], 00000000 －》40A64C 的值看上一讲
:0040129C 7513              jne 004012B1 －－》比较是否设定需要读取的寄存器；
                            如果设定了值，转向 4012B1.否则继续。
:0040129E 56              push esi
:0040129F C60560974000BE    mov byte ptr [00409760], BE
:004012A6 E8E3050000       call 0040188E
:004012AB 59              pop ecx
:004012AC A34CA64000       mov dword ptr [0040A64C], eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040129C(C)`

```
:004012B1 6824A54000       push 0040A524  －－》指向被跟踪的程序名
:004012B6 E813000000       call 004012CE  －－》创建该程序进程
:004012BB 59              pop ecx
:004012BC 5F              pop edi
:004012BD 5E              pop esi
:004012BE 5D              pop ebp
:004012BF 85C0            test eax, eax
:004012C1 5B              pop ebx
:004012C2 7405            je 004012C9
:004012C4 E841000000       call 0040130A
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004012C2(C)`

```
:004012C9 33C0            xor eax, eax
:004012CB C21000          ret 0010
```

下面看 `call 004012CE` 子程序，前我们来研究一个 API〔`CreatProcess`〕它的参数说明

可是一大摞。

```c
BOOL CreateProcess(
    LPCTSTR lpApplicationName,                 // pointer to name of executable module
    LPTSTR lpCommandLine,                      // pointer to command line string
    LPSECURITY_ATTRIBUTES lpProcessAttributes, // process security attributes
    LPSECURITY_ATTRIBUTES lpThreadAttributes,  // thread security attributes
    BOOL bInheritHandles,                      // handle inheritance flag
    DWORD dwCreationFlags,                     // creation flags
    LPVOID lpEnvironment,                      // pointer to new environment block
    LPCTSTR lpCurrentDirectory,                // pointer to current directory name
    LPSTARTUPINFO lpStartupInfo,               // pointer to STARTUPINFO
    LPPROCESS_INFORMATION lpProcessInformation // pointer to PROCESS_INFORMATION
);
```

说明

创建一个新进程

返回值

`Long`，非零表示成功，零表示失败。会设置 `GetLastError`

参数表

| 参数 | 类型及说明 |
| :--- | :--- |
| `lpApplicationName` | `String`，要执行的应用程序的名字。可设为 `vbNullString`；在这种情况下，应用程序的名字应在 `lpCommandLine` 参数的起始处出现 |
| `lpCommandLine` | `String`，要执行的命令行。可用 `GetCommandLine` 函数取得一个进程使用的命令行。windows 会尽可能地根据下述搜索顺序来查找执行文件：<br>（1）包含了父进程执行文件的目录<br>（2）父进程当前的目录<br>（3）由 `GetSystemDirectory` 返回的系统目录<br>（4）仅适于 windows NT：16 位系统目录<br>（5）由 `GetSystemDirectory` 返回的 windows 目录<br>（6）由 `PATH` 环境变量指定的目录 |
| `lpProcessAttributes` | `SECURITY_ATTRIBUTES`，指定一个 `SECURITY_ATTRIBUTES` 结构，或传递零值表示采用不允许继承的默认描述符。该参数定义进程的安全特性 |
| `lpThreadAttributes` | `SECURITY_ATTRIBUTES`，指定一个 `SECURITY_ATTRIBUTES` 结构，或传递零值表示采用不允许继承的默认描述符。该参数定义进程之主进程的安全特性 |
| `bInheritHandles` | `Long`，`TRUE` 表示允许当前进程中的所有句柄都由新建的子进程继承 |
| `dwCreationFlags` | `Long`，来自 `API32.TXT` 文件的一个或多个下述常数之一，它们都带有前缀 `CREATE_`。下面这些用于 VB 程序员：<br>`CREATE_SEPARATE_WOW_VDM`（仅适用于 NT）：启动一个 16 位的 windows 应用程序时，强迫它在自己的内存空间运行<br>`CREATE_SHARED_WOW_VDM`（仅适用于 NT）：启动一个 16 位的 windows 应用程序时，强迫它在共享的 16 位虚拟机内运行<br>`CREATE_SUSPENDED`：立即挂起新进程。除非调用 `ResumeThread` 函数。<br>`IDLE_PRIORITY_CLASS`：新进程应该有非常低的优先级，只有在系统空闲的时候才能运行。基本值是 4<br>`HIGH_PRIORITY_CLASS`：新进程应该有非常高的优先级，基本值是 13 |



# 进程参数说明

-   `NORMAL_PRIORITY_CLASS`：标准优先级。如进程位于前台，则基本值是 `9`；如在后台，则优先值是 `7`
-   `lpEnvironment`：任何类型，指向一个环境块的指针
-   `lpCurrentDriectory`：`String`，新进程的当前目录路径
-   `lpStartupInfo`：`STARTUPINFO`，指定一个 `STARTUPINFO` 结构，其中包含了创建进程时使用的附加信息
-   `lpProcessInformation`：`PROCESS_INFORMATION`，该结构用于容纳新进程的进程和线程标识。大多数情况下，一旦这个函数返回，父应用程序都会关闭两个句柄。

## 使用体验

可怕吗？？我好害怕怕哦！！呵呵～～～ 其实你使用起来，是非常的简单的 ：）

因为许多的参数都可以用 `0` 来表示。

比如：

```
CreateProcess(0, serail.exe,0,0,FALSE,
    CREATE_SUSPENDED, 0,0,&startupInfo, &ProcessInformation);
```

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

这个例子是不是挺可爱的。40 多行的说明，用起来不过两行而已。在这里强调一下

有一部分的 crack 工具都使用这个程序哦！！比如 内存补丁程序，还有现在这个 crackcode

还有新版的 DEDE 也用到。你说是不是值得去学习它呢？？？

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

## 汇编代码示例

=======================call `004012CE` 子程序========================

\* Referenced by a CALL at Address:

|:004012B6

|

:004012CE

```
:004012CE 55                     push ebp
:004012CF 8BEC                   mov ebp, esp
:004012D1 83EC44                 sub esp, 00000044
:004012D4 56                     push esi
:004012D5 33F6                   xor esi, esi
:004012D7 6A44                   push 00000044
:004012D9 8D45BC                 lea eax, dword ptr [ebp-44]
:004012DC 56                     push esi
:004012DD 50                     push eax
:004012DE E86D080000             call 00401B50
:004012E3 83C40C                 add esp, 0000000C
:004012E6 8D45BC                 lea eax, dword ptr [ebp-44]
:004012E9 C745BC44000000         mov [ebp-44], 00000044
:004012F0 6810A44000             push 0040A410
:004012F5 50                     push eax
:004012F6 56                     push esi
:004012F7 56                     push esi
:004012F8 6A02                   push 00000002
:004012FA 56                     push esi
:004012FB 56                     push esi
:004012FC 56                     push esi
:004012FD FF7508                 push [ebp+08]
:00401300 56                     push esi

\* Reference To: KERNEL32.CreateProcessA, Ord:0000h

:00401301 FF1508504000           Call dword ptr [00405008]
:00401307 5E                     pop esi
:00401308 C9                     leave
:00401309 C3                     ret
```

=======================call `004012CE` 子程序========================

## 结语

估计大伙可能看到这会不会一头雾水呢？？为了更好理解它。

建议：下载 `R!SC's Process Patcher` 原代码进行研究。

大伙是不是觉得这样写下去，挺没有劲的！对吗？这个 CRACK 论坛不是中国最火的吗？

为什么没有人愿意解答我的问题？？只是希望得到结果（突破 `25` 位的限制）！：（

好失望！！原以为先有付出，才会有收获，唉！！ 我去了！ 这是最后的一篇文章。

byebye 曾经给过我一些希望的两位 `[FCG]` 朋友。

（依依惜别篇）豆豆虾

2001.8.12 19:10 完成

历时：`2.1` 个小时

---

`4dd466d0e1e0cc123afbe504da2462ca`
