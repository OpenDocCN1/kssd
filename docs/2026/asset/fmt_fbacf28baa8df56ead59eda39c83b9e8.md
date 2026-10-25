
# crackme破解教程(续) (高手莫入) (2千字)

## crackme破解教程(续)(高手莫入)

又贴4篇,有一点点难度,破出则功力大涨

### ------------------(1)-----------------------------

`fireworx.12`破解教程(编号:80)(a little difficult)无出错提示

`http://go3.163.com/~gfcrack/crackme/fireworx.12.zip`

输入`guodong`  `duba`  `7654321`

`bpx hmemcpy`

`g`

点鼠标,被拦

`bc\*`

`pmodule`

按F10直到下面

```
0167:004554AC  MOV      EAX,[EBP-1C] \*\*\*
0167:004554AF  MOV      EDX,[EBP-14] \*\*\*
0167:004554B2  CALL    00403B74    \*\*\*
0167:004554B7  JNZ      004554DA    \*\*\*
0167:004554B9  PUSH    BYTE +00
0167:004554BB  LEA      EDX,[EBP-1C]
```

`d edx`  (`edx=5971622`)

`d eax`  (`eax=7654321`)

整理 `guodong`  `duba`  `5971622`

### -----------------(2)---------------------------

`cabeca`破解教程(初级)(编号:21)

`http://go3.163.com/~gfcrack/crackme/Cabeca.zip`

依次输入 `guodong` `111111`  `222222`

`ctrl+n`激活`trw2000`

`bpx hmemcpy`

`go`

点Try,被拦

`bc \*`

`pmodule`

不停按F10,直到下面`42d4a5`处

```
0167:0042D4A1  MOV      EDX,[EBP-04]  \*
0167:0042D4A4  POP      EAX
    \*
0167:0042D4A5  CALL    004038D0      \*\*\*\*
0167:0042D4AA  JNZ      0042D4E5      \*
=>出错(1)
0167:0042D4AC  LEA      EDX,[EBP-10]
0167:0042D4AF  MOV      EAX,[0042F718]
0167:0042D4B4  CALL    00406550
0167:0042D4B9  MOV      EAX,[EBP-10]
0167:0042D4BC  PUSH    EAX
0167:0042D4BD  LEA      EDX,[EBP-04]
0167:0042D4C0  MOV      EAX,[EBX+01EC]
0167:0042D4C6  CALL    00419DE0
0167:0042D4CB  MOV      EDX,[EBP-04]  \*
0167:0042D4CE  POP      EAX
    \*
0167:0042D4CF  CALL    004038D0      \*\*\*\*
0167:0042D4D4  JNZ      0042D4E5      \*=>出错(2)
0167:0042D4D6  MOV      EAX,0042D5E8
```

`42d4a5`处 `d eax`    `eax=27692`

`d edx`

`edx=111111`

按F9

重新输入`guodong` `27692` `222222`

点Try,被拦

按F10,直到下面`42d4cf`处

`d eax`  `eax=15895`

`d edx`  `edx=222222`

总结 `guodong` `27692` `15895`

### ------------------(3)-----------------------------

`fireworx.6`破解教程(easy)(编号:75)

`http://go3.163.com/~gfcrack/crackme/fireworx.6.zip`

输入`guodong`  `7654321`

`ctrl+n`激活`trw2000`

`bpx hmemcpy`

`g`

点确定,被拦

`bc\*`

`pmodule`

不停按F10,直到下面(1)处

```
0167:004417F8  CALL    004416F8
0167:004417FD  MOV      EDX,[EBP-08]  \*\*
0167:00441800  POP      EAX
\*\*
0167:00441801  CALL    00403B44    \*\*\*(1)
0167:00441806  JNZ      00441822    \*\*
```

(1)  `d edx`真码(`71A1-01CC-4EB2-38C5`)

    `d eax`假码(`7654321`)

整理 `guodong`  `71A1-01CC-4EB2-38C5`

### --------------------(4)----------------------------

`DueList.4`破解教程(编号:49)(a little difficult)

`http://go3.163.com/~gfcrack/crackme/DueList.4.zip`

破解:风飘雪

运行`trw2000`,输入姓名: `guodong`  注册码: `7654321`

`bpx hmemcpy`

`go` 被拦

`bc\*`

`pmodule`

此时,光条停在`401184`行

`s 401189,ffffff '7654321'`

得 `402179`

这时,向上看,内存窗口看到`7654321`

                下一行  `CDSKS0C`(倒数第二位是零)

总结:  姓名: `guodong`  注册码: `CDSKS0C`

风飘雪

`http://duba.126.com`

`gd1@yeah.net`
