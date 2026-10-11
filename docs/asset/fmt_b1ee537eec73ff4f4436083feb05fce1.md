
```
:0001.85D5 1E push ds
:0001.85D6 68302E push 2E30
:0001.85D9 1E push ds

* Possible StringData Ref from Data Seg 002 ->"wslam.ini"

:0001.85DA 684A08 push 084A
:0001.85DD 9A375E0000 call KERNEL.WRITEPRIVATEPROFILESTRING
```

然后用 `trw2000` 拦截 `DIALOGBOX`，设断 `bpx DIALOGBOX`，试探各 `DIALOGBOX` 的作用。

```
:0001.0891 8BFA mov di, dx
:0001.0893 9ADE080000 call USER.DIALOGBOX 弹出菜单里的对话框.

:0001.85B8 8BFA mov di, dx
:0001.85BA 9A94080000 call USER.DIALOGBOX 弹出注册框.

:0001.08D9 89963CFF mov [bp+FF3C], dx
:0001.08DD 9AFFFF0000 call USER.DIALOGBOX 功能不明.
```

### 注册算法分析

注册码检测 `call` `845C` 如下:

（如需动态分析，`trw2000` 设断 `bpx GETDLGITEMTEXT`，`pmodule` 返回既是）

* Referenced by a CALL at Addresses:
  * `:0001.8530`, `:0001.8593`, `:0001.85C6`

三个地方调用了注册码检测 `call`：
* `:0001.8530`，作用为验证输入的注册码
* `:0001.8593`，程序启动时，用来判断是否弹出注册对话框
* `:0001.85C6`，第二次验证输入的注册码，类似双重保险。

```
:0001.845C C8220000 enter 0022, 00
:0001.8460 57 push di
:0001.8461 56 push si
:0001.8462 33D2 xor dx, dx
:0001.8464 8956FE mov [bp-02], dx ----bp-02 地址处双字节清零
:0001.8467 BE302E mov si, 2E30 -----2E30 处存放了注册码
:0001.846A 3814 cmp [si], dl ---判断注册码第一位是否为零
:0001.846C 7426 je 8494 ----为零则跳转
:0001.846E 8D7EDE lea di, [bp-22] ----bp-22 的有效值给 di

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0001.8492(C)

|-------下面是一个循环对注册码字符进行变换

:0001.8471 8D46FE lea ax, [bp-02] ----bp-02 的有效值给 ax
:0001.8474 3BF8 cmp di, ax ----比较 di, ax
:0001.8476 731C jnb 8494 ----di>=ax 则跳
:0001.8478 8A04 mov al , [si] ----注册码首字符 ASCII 码给 al
:0001.847A 98 cbw ---CBW 字节转换为字. (把 AL 中字节的符号扩展到 AH 中去)
:0001.847B 8BD8 mov bx, ax ----符号扩展后的注册码首字符给 bx
:0001.847D F687FD0804 test byte ptr [bx+08FD], 04
:0001.8482 740A je 848E
:0001.8484 98 cbw
:0001.8485 2D3000 sub ax, 0030 ----ax=ax-0030
:0001.8488 8905 mov [di], ax ----把 ax 赋给 di 所指地址处的值
:0001.848A 83C702 add di, 0002 ----di=di+0002
:0001.848D 42 inc dx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  |:0001.8482(C)

:0001.848E 46 inc si
:0001.848F 803C00 cmp byte ptr [si], 00 ----判断注册码处理完毕否
:0001.8492 75DD jne 8471 ----没完跳回继续循环

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0001.846C(C), :0001.8476(C)

:0001.8494 83FA10 cmp dx, 0010

----注册码位数与 10(hex)即 16(dec)比较，在这儿设断点 `bpx ****`(每次运行不同):`8494` 可拦下所有调用
:0001.8497 7C48 jl 84E1 ------<=10 则跳过下面一大段，直接返回.

------下面的 bp-****都是注册码(设为 a[i])经过变换后的相应值(设为 a[i]')

:0001.8499 8B46E0 mov ax, [bp-20] ----注册码第二位变换值 a[1]'给 ax
:0001.849C F76EEA imul word ptr [bp-16]---a[1]'=a[1]'*a[6]'
:0001.849F F76EF4 imul word ptr [bp-0C]---a[1]'=a[1]'*a[6]'*a[11]'
:0001.84A2 F76EF6 imul word ptr [bp-0A]---a[1]'=a[1]'*a[6]'*a[11]'*a[12]'
:0001.84A5 8BC8 mov cx, ax ---cx=ax=a[1]'*a[6]'*a[11]'*a[12]'
:0001.84A7 8B46E4 mov ax, [bp-1C] ----注册码第四位变换值 a[3]'给 ax
:0001.84AA F76EE6 imul word ptr [bp-1A]
:0001.84AD F76EF2 imul word ptr [bp-0E]
:0001.84B0 F76EF8 imul word ptr [bp-08]
:0001.84B3 03C8 add cx, ax

    ---cx=cx+ax=a[1]'*a[6]'*a[11]'*a[12]'+a[3]'*a[4]'*a[10]'*a[13]'

:0001.84B5 8B46EC mov ax, [bp-14] ----注册码第八位变换值 a[7]'给 ax
:0001.84B8 F76EEE imul word ptr [bp-12]
:0001.84BB F76EF0 imul word ptr [bp-10]
:0001.84BE F76EFC imul word ptr [bp-04]
:0001.84C1 03C8 add cx, ax

---cx=cx+ax=a[1]'*a[6]'*a[11]'*a[12]'+a[3]'*a[4]'*a[10]'*a[13]'+a[7]'*a[8]'*a[9]'*a[15]'

:0001.84C3 8B46E2 mov ax, [bp-1E] ----注册码第三位变换值 a[2]'给 ax
:0001.84C6 F76EE8 imul word ptr [bp-18]
:0001.84C9 F76EFA imul word ptr [bp-06]
:0001.84CC F76EDE imul word ptr [bp-22]
:0001.84CF 03C1 add ax, cx

---ax=cx+ax=a[1]'*a[6]'*a[11]'*a[12]'+a[3]'*a[4]'*a[10]'*a[13]'+a[7]'*a[8]'*a[9]'*a[15]'+a[2]'*a[5]'*a[14]'*a[0]'

* Possible Ref to Menu: `WSLAMMENU`, Item: "帮助(H)..."

:0001.84D1 B9F401 mov cx, 01F4 ----cx=01F4
:0001.84D4 99 cwd
:0001.84D5 F7F9 idiv cx

----------------------------------------------------
div 源（eax＝eax/源，edx＝余数）
IDIV 整数除法. 以上两条，结果回送:
    商回送 AL,余数回送 AH, (字节运算);
    或 商回送 AX,余数回送 DX, (字运算).
----------------------------------------------------

:0001.84D7 83FA7F cmp dx, 007F ----余数和 007F 比较
:0001.84DA 7505 jne 84E1 ----不相等则失败
关键比较，offset:`8AFA`,用 `Hiew32` 改为 `je`,OK 了!
:0001.84DC C746FE0100 mov word ptr [bp-02], 0001 ---注册标志位置入

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  |:0001.8497(C), :0001.84DA(C)

:0001.84E1 8B46FE mov ax, [bp-02] ----ax 存放注册标志位
:0001.84E4 5E pop si
:0001.84E5 5F pop di
:0001.84E6 C9 leave
:0001.84E7 C3 ret
```

### 总结

注册码必须为 16 位

#### 1. 处理过程

`[注册码 ASCII 码值+08FD]`处值与 `04` 比较，不同则注册码处理值=注册码 ASCII 码值-`30`(hex)

#### 2. 检验过程

简化 `bp-****=****`，所以 `bp-12` 下面写为 `12`

```
22    20    1E    1C    1A    18    16    14    内存中的处理值
1     2     3     4     5     6     7     8     对应第 i 位注册码
a[0]' a[1]' a[2]' a[3]' a[4]' a[5]' a[6]' a[7]' 对应注册码处理值

12    10    0E    0C    0A    08    06    04
9     10    11    12    13    14    15    16
a[8]' a[9]' a[10]' a[11]' a[12]' a[13]' a[14]' a[15]'
```

令 `sum=a[1]'*a[6]'*a[11]'*a[12]'+a[3]'*a[4]'*a[10]'*a[13]'+a[7]'*a[8]'*a[9]'*a[15]'+a[2]'*a[5]'*a[14]'*a[0]'`

`sum%01F4` 与 `007F` 比较，相等则注册成功！

喜欢做注册机的话，就试试看吧！

`4dd466d0e1e0cc123afbe504da2462ca`
