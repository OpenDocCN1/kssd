
# 禁用注册表之暴力破解法（4千字）

## 背景

不知各位有没有浏览过恶意网站，有没有被禁用注册表编辑器？如果你被禁用的话，你可以用许多方法恢复。那你有没有试过用破解的方法呢？下面是我的破解方法：

## 使用 HWND 获取窗口句柄

运行注册表编辑器 `regedit.exe`，弹出禁用注册表的信息。按 `Ctrl+D` 进入 ICE，输入命令 `HWND`，看到这些东东：

```
Window Handle  hQueue  SZ  QOwner    Class Name
          Window Procedure

0080(0)        206F    32  MSGSRV32
#32769 (Desktop)      1787:00005F92
03D8(1)        220F    32  EXPLORER
BaseBar              136F:0000028A
  03DC(2)      220F    32  EXPLORER
MenuSite              136F:0000028A
  0420(3)      220F    32  EXPLORER
SysPager              136F:00000140
    0424(4)    220F    32  EXPLORER
ToolbarWindow32      136F:000002B6
03B0(1)        220F    32  EXPLORER
BaseBar              136F:0000028A
  03B4(2)      220F    32  EXPLORER
MenuSite              136F:0000028A
```

略去一些。

```
0540(1)        410F    32  REGEDIT  IME
                175F:00000000
053C(1)        410F    32  REGEDIT
#32770 (Dialog)      175F:00007720
  0544(2)      410F    32  REGEDIT
Button                175F:000035CC
  0548(2)      410F    32  REGEDIT
Static                175F:0000828E
  054C(2)      410F    32  REGEDIT
Static                175F:0000828E
```

注意：`REGEDIT Button` 对应的 `Window Handle` 为 `544`。

输入命令 `bmsg 544 wm_lbuttonup`（`544` 这个数字是不确定的，所以要用 `HWND` 来查看）。

这个命令的意思是当放开鼠标左键的消息传到 `Button` 时引发中断。

## 反汇编分析

退出 ICE，用鼠标按确定键后引发中断，再按 `F12` 键 n 次后，来到这里：

```
017F:0040CD11  57                  PUSH    EDI
017F:0040CD12  52                  PUSH    EDX
017F:0040CD13  57                  PUSH    EDI
017F:0040CD14  57                  PUSH    EDI
017F:0040CD15  51                  PUSH    ECX
017F:0040CD16  6800050000          PUSH    00000500
017F:0040CD1B  FF150C954100        CALL    [0041950C]
017F:0040CD21  897DF8              MOV     [EBP-08],EDI
017F:0040CD24  85C0                TEST    EAX,EAX
017F:0040CD26  742A                JZ      0040CD52
017F:0040CD28  397DFC              CMP     [EBP-04],EDI
017F:0040CD2B  7425                JZ      0040CD52
017F:0040CD2D  8B4518              MOV     EAX,[EBP+18]
017F:0040CD30  0D00000100          OR      EAX,00010000
017F:0040CD35  50                  PUSH    EAX
017F:0040CD36  56                  PUSH    ESI
017F:0040CD37  FF75FC              PUSH    DWORD PTR [EBP-04]
017F:0040CD3A  FF750C              PUSH    DWORD PTR [EBP+0C]
017F:0040CD3D  FF1504964100        CALL    [00419604] ; 这个 CALL 是弹出禁用窗口的
017F:0040CD43  FF75FC              PUSH    DWORD PTR [EBP-04] ; 注意，EIP 指向这里
017F:0040CD46  8BF0                MOV     ESI,EAX
017F:0040CD48  FF15BC944100        CALL    [004194BC]
017F:0040CD4E  8BC6                MOV     EAX,ESI
017F:0040CD50  EB05                JMP     0040CD57
017F:0040CD52  B8FFFFFFFF          MOV     EAX,FFFFFFFF
017F:0040CD57  5F                  POP     EDI
017F:0040CD58  5E                  POP     ESI
017F:0040CD59  8BE5                MOV     ESP,EBP
017F:0040CD5B  5D                  POP     EBP
017F:0040CD5C  C3                  RET
```

经过两次运行，发现 `0040CD24` 和 `0040CD28` 都不是判断是否禁用注册表的地方，所以按 `F10` 键 n 次，返回上一层 `CALL`，来到这里：

```
017F:0040C7B6  E9DD000000          JMP     0040C898
017F:0040C7BB  E808020000          CALL    0040C9C8
017F:0040C7C0  85C0                TEST    EAX,EAX ; 这里是判断是否禁用的地方
017F:0040C7C2  741B                JZ      0040C7DF
017F:0040C7C4  6A10                PUSH    10
017F:0040C7C6  A100884100          MOV     EAX,[00418800]
017F:0040C7CB  6A10                PUSH    10
017F:0040C7CD  6A28                PUSH    28
017F:0040C7CF  6A00                PUSH    00
017F:0040C7D1  50                  PUSH    EAX
017F:0040C7D2  E8D1040000          CALL    0040CCA8 ; 这个 CALL 是弹出禁用窗口的。
017F:0040C7D7  83C414              ADD     ESP,14 ; 注意，EIP 指向这里
017F:0040C7DA  E9B9000000          JMP     0040C898
```

下中断 `bpx 40c7c2`，然后退出 ICE，再运行注册表编辑器，当执行到 `40c7c2` 时中断，修改 `EIP` 为 `40c7df`，按 `F5`，运行正常。

## 修改程序文件

用二进制编辑器打开 `regedit.exe`，查找 `74 1b 6a 10 a1 00`，把 `74` 改为 `eb`，破解完成。
