
# 浮点运算指令一例(用海伦公式计算三角形的面积)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-677.htm`](https://www.pediy.com/kssd/pediy08/pediy8-677.htm)

下面是关于浮点运算指令方面的一例，程序对三角形的面积和周长进行了计算。

```asm
.386
.model flat, stdcall  ;32 bit memory model
option casemap :none  ;case sensitive
include m1.inc
.code

start:
  invoke GetModuleHandle, NULL
  mov    hInstance, eax
  invoke InitCommonControls
  invoke DialogBoxParam, hInstance, IDD_DIALOG1, NULL, addr DlgProc, NULL
  invoke ExitProcess, 0

;########################################################################

DlgProc proc hWin:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
  mov    eax, uMsg
  .if eax==WM_INITDIALOG
  .elseif eax==WM_COMMAND
            .if wParam==IDC_OK
              invoke GetDlgItemText, hWin, IDC_A, addr lpA, sizeof lpA ;得到边长 a
              invoke FpuAtoFL, addr lpA, 0, DEST_FPU;转化为数值
              invoke GetDlgItemText, hWin, IDC_B, addr lpB, sizeof lpB
              invoke FpuAtoFL, addr lpB, 0, DEST_FPU
              invoke GetDlgItemText, hWin, IDC_C, addr lpC, sizeof lpC
              invoke FpuAtoFL, addr lpC, 0, DEST_FPU

              fld st(0)    ;判定所输入数据是否是三角形
              fld st(2)
              fadd
              fcomp st(3)
              fstsw  ax       ; 将状态字保存
              and eax, 100000000B  ；;测试状态字的第 9 位，若为 1 则说明两边之和小于了第三边
              jnz ErrorHandle           fld st(0)
              fld st(3)
              fadd
              fcomp st(2)
              fstsw  ax
              and eax, 100000000B
              jnz ErrorHandle
              fld st(1)
              fld st(3)
              fadd
              fcomp st(1)
              fstsw  ax
              and eax, 100000000B
              jnz ErrorHandle     ;判定结束
              fadd ;边长相加
              fadd ;边长相加
              invoke FpuFLtoA, 0, 4, addr lpL, SRC1_FPU or SRC2_DIMM
              invoke SetDlgItemText, hWin, IDC_L, addr lpL
              fild LTWO
              fdiv
              fld st(0)
              invoke FpuAtoFL, addr lpA, 0, DEST_FPU
              fsub
              fld st(1)
              invoke FpuAtoFL, addr lpB, 0, DEST_FPU
              fsub
              fld st(2)
              invoke FpuAtoFL, addr lpC, 0, DEST_FPU
              fsub
              fmul
              fmul
              fmul
              fsqrt
              invoke FpuFLtoA, 0, 4, addr lpS, SRC1_FPU or SRC2_DIMM
              invoke SetDlgItemText, hWin, IDC_S, addr lpS
              ret

              ErrorHandle:
              invoke MessageBox, hWin, addr lpMessage, NULL, MB_OK and MB_ICONERROR
              ret

            .endif
  .elseif eax==WM_CLOSE
    invoke EndDialog, hWin, 0
  .else
    mov    eax, FALSE
    ret
  .endif
  mov    eax, TRUE
  ret

DlgProc endp
end start
```

下面是 inc 文件的内容。

```asm
include windows.inc
include kernel32.inc
include user32.inc
include Comctl32.inc
include shell32.inc
include fpu.inc

includelib fpu.lib
includelib kernel32.lib
includelib user32.lib
includelib Comctl32.lib
includelib shell32.lib

DlgProc       PROTO  :HWND,:UINT,:WPARAM,:LPARAM

.const

IDD_DIALOG1    equ 101
IDC_A          equ 1003
IDC_B          equ 1004
IDC_OK         equ 1005
IDC_C          equ 1006
IDC_S          equ 1008
IDC_L          equ 1010

LTWO dw 2
lpMessage db "不是三角形",0

;#########################################################################

.data?

hInstance      dd ?
lpA db 10 dup(?)
lpB db 10 dup(?)

;#########################################################################

lpC db 10 dup(?)
lpL db 10 dup(?)
lpS db 10 dup(?)
```

附件是全部的源程序：`m1.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
