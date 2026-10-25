
# SuperCapture 3.04 破解历程---半小时 (5千字)

2002年1月10日破解

软件名称：SuperCapture 3.04
软件类型：抓图工具
限制类型：45天试用

1. 首先装入`Trw2000`

2. 运行SuperCapture 3.0x（？？刚才不是说3.04吗？怎么现在又变成了3.0x？哦这个问题・・・・・，3.02和3.04注册码能共用所以我才这样说的。）

3. 程序一开始就出现了“注册”和“试用”的选项，选择“注册”再点击输入注册码；注册的形式为`XXXX-XXXX-XXXX-XXXX`（算一算是19个字母16进制等于13h），这是因为跟踪过才知道的，等会再交待如何得知。

4. 来到输入注册码的窗口，Ctrl-N激活Trw2000，然后输入`Bpx GetDlgItemTextA`设置断点。按F5返回SuperCapture3.0x的注册窗口。第一行是SuperCapture3.0x给出的用户码，用户码为：`ATKZ-KJMH-MHCH-QPCB`。第二行是用户输入的注册码，随意输入：`AAAA-BBBB-CCCC-DDDD`，然后用你的Mouse点击“确认”。

5. Bingo!断点有效！开工啦（虽然是在新年开工，但是没有利钱的，休想・・・・・・・・）

指针中断在`User32!GetDlgItemTextA`。

```
0167:BFF51743  B1A1          MOV CL,A1
      ;此时EIP指针在这里

XXXX:XXXX1745  55            PUSH EBP

    ・
    ・<F10>单步运行，按N次
    ・

XXXX:XXXX1764  E8E76B0000    CALL KERNEL32!SUnMapLs\_IP\_EBP
XXXX:XXXX1769  C9            LEAVE
XXXX:XXXX176A  C21000        RET 10
      ;返回调用子程序的上一级程序中

    ・
    ・<F10>单步运行，按N次
    ・

016F:00492AB3  FF154C264B00  CALL USER32!GetItemTextA
            ;返回来就指针在这里，也就是说：是这段代码调用
      用子程序向用户要求输入注册码的

XXXX:XXXX2AB9  ????????????  JMP SHOTR 00492ACB
                ;返回后程序无条件跳至 cs:00492ACB

    ・
  ;?????????是我忘记抄下的机器码，在下实在是粗心

    ・<F10>单步运行，按N次
    ・
    ・

XXXX:XXXX2ACB  5D            POP EBP
XXXX:XXXX2ACC  20 0C 00      RET
        ;再一次返回上一级的程序 0167:0041FA45 那里。

    ・
    ・<F10>单步运行，按N次
    ・

0167:0041FA45  E853300700    CALL 00492A90
;从上面子程序返回在这里。

XXXX:XXXXFA4A  85C0          TEST EAX,EAX
    ;这时大家看看寄存器EAX=？ 13吧?这就是我说为什么
XXXX:XXXXFA4C  0F8486000000  JZ  NEAR  0041FAD8
                  ;要输入13(十六进制)=19(十进制)个字母的原因。

    ・
  ;这时我已输入了AAAA-BBBB-CCCC-DDDD加上减号共是
    ・
  ;19个字节即是13(十六进制)，所以程序没有跳转。
    ・<F10>单步运行，按N次
    ・

XXXX:XXXXFA52  68CCEB4D00    PUSH DWORD 0041FAD8
XXXX:XXXXFA57  8D4C240C      LEA ECX,[ESP+0C]
                    ;大家记得D命令吗?不妨看看ESP+0C存放着什么东东啊

    ・
  ;不看不知道，一看吓一跳！看见了！
    ・
  ;刚才我输入假注册码：AAAA-BBBB-CCCC-DDDD
    ・
    ・<F10>单步运行，按N次
    ・
    ・

XXXX:XXXXFA5E  8D9348020000  LEA EDX,[EBX+0248]
                ;那这个有必要要看吗？当然要啦，这个还有人问？
    ・
  ;看见什么了？看见用户码:ATKZ-KJMH-MHCH-QPCB
    ・
    ・
    ・<F10>单步运行，按N次
    ・
    ・

0167:0041FA67  E8F4ECFFFF    CALL 0041E760
;这是多番跟踪得出的――这是核心！即算出正确注册
    ・
  ;的主程序，有性趣的朋友不妨追踪到里面研究研究啊
    ・
  ;在这我就不得出来了，让有性趣的朋友算算（其实是
    ・
  ;我不会算，反正知道是算注册码的主程序）。
    ・
  ;现场听众起哄了・・・・开始有人扔臭鸡蛋了・・・
    ・
```

别再打我了，说就说吧。

```
在0167:0041FA67  E8F4ECFFFF    CALL 0041E760
                    ;这里同上面的一样按<F10>带过。
    ・
  ;然后把你的机器代码段窗口拉宽几行，上下看看。
    ・
  ;大家看见什么了？（听众，怒视望我）
    ・
  ;好了，上面存着如下字节串
    ・
  ;字节串一：ATKZKJMHMHCHQPCB
    ・
  ;字节串二：AAAABBBBCCCCDDDD
    ・
  ;字节串三：NLEKSCUEOHTPRXHL
    ・
  ;字节串四：AAAA-BBBB-CCCC-DDDD
    ・
  ;
    ・
  ;以上的字节串是用<F8>跟踪，
    ・
  ;进入 “CALL 0041E760 ”再用D命令查看ESP寄存器所得
```

整理：

破解历时30分钟

设过两个断点：

其一 `Bpx GetWindowTextA`

其二 `Bpx GetDlgItemTextA`

随便输入：`AAAA-BBBB-CCCC-DDDD` = `AAAABBBBCCCCDDDD`

取用户码：`ATKZKJMHMHCHQPCB`    = `ATKZ-KJMH-MHCH-QPCB`

正确密码：`NLEKSCUEOHTPRXHL`    = `NLEK-SCUE-OHTP-RXHL`
