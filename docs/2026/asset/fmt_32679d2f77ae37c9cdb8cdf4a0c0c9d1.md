
```asm
C745FC68000000          mov [ebp-04], 00000068

:004583C4

668B950CFFFFFF          mov dx, word ptr [ebp+FFFFFF0C]

:004583CB

6683C201                add dx, 0001

:004583CF

0F804C060000            jo 00458A21

:004583D5

6689950CFFFFFF          mov word ptr [ebp+FFFFFF0C],

dx

:004583DC C745FC69000000          mov [ebp-04],

00000069

:004583E3 6683BD0CFFFFFF09        cmp word ptr

[ebp+FFFFFF0C], 0009

:004583EB 7510

       jne 004583FD

:004583ED C745FC6A000000

     mov [ebp-04], 0000006A

:004583F4 66C7850CFFFFFF0100

   mov word ptr [ebp+FFFFFF0C], 0001

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004583EB(C)

|

:004583FD

C745FC6C000000          mov [ebp-04], 0000006C

:00458404

E921FEFFFF              jmp 0045822A  <===从这里向上跳构成一个大循环，主要功能是将定数4C29AGE8BC2YCX3N进行变形处理，最后变形出来的值为(84

44 60 82 AF 96 76 76)

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00458253(C)

|

:00458409

C745FC6D000000          mov [ebp-04], 0000006D  <===跳出循环后，到这里

:00458410

668B45B4                mov ax, word ptr

[ebp-4C] <===AX=327 为前面注册名变形值相加总和

:00458414 660345A8

         add ax, word ptr [ebp-58] <===[ebp-58]为前面定值变形值相加总和5E0

:00458418

0F8003060000            jo 00458A21

:0045841E

6625FF01                and ax, 01FF

<===AX=907 AND 1FF=107(这个值很关键，备用)

:00458422 7908

           jns 0045842C

:00458424 6648

                 dec ax

:00458426

660D00FE                or ax, FE00

:0045842A

6640                    inc

ax

\* Referenced by a

(U)nconditional or (C)onditional Jump at Address:

|:00458422(C)

|

:0045842C

66898550FFFFFF          mov word ptr [ebp+FFFFFF50],

ax <===将107放入内存位置，后面用

:00458433 C745FC6E000000

 mov [ebp-04], 0000006E

:0045843A 66C7850CFFFFFF0100      mov

word ptr [ebp+FFFFFF0C], 0001

:00458443 C745FC6F000000

   mov [ebp-04], 0000006F

:0045844A 66C78510FFFFFF0100

 mov word ptr [ebp+FFFFFF10], 0001

:00458453 C745FC70000000

     mov [ebp-04], 00000070

:0045845A 668B4DB8

           mov cx, word ptr [ebp-48]

:0045845E

66898D24FEFFFF          mov word ptr [ebp+FFFFFE24],

cx

:00458465 66C78528FEFFFF0100      mov word ptr [ebp+FFFFFE28],

0001

:0045846E 66C78548FFFFFF0100      mov word ptr [ebp+FFFFFF48],

0001

:00458477 EB1B

   jmp 00458494

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004588D5(U)

           <===从这里开始循环

|

:00458479 668B9548FFFFFF

         mov dx, word ptr [ebp+FFFFFF48]

:00458480

66039528FEFFFF          add dx, word ptr [ebp+FFFFFE28]

:00458487

0F8094050000            jo 00458A21

:0045848D

66899548FFFFFF          mov word ptr [ebp+FFFFFF48],

dx

\* Referenced by a

(U)nconditional or (C)onditional Jump at Address:

|:00458477(U)

|

:00458494

668B8548FFFFFF          mov ax, word ptr [ebp+FFFFFF48]

:0045849B

663B8524FEFFFF          cmp ax, word ptr [ebp+FFFFFE24]

:004584A2

0F8F32040000            jg 004588DA   <===循环结束后，从这里跳出

\*\*\*\*\*略去一段花指令\*\*\*\*\*\*\*

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0045852B(U)

|

:00458539

8B8D70FEFFFF            mov ecx, dword ptr [ebp+FFFFFE70]

:0045853F 8B5580                  mov

edx, dword ptr [ebp-80]

   <===定数变形值84 44 60 82 AF 96 76 76(以注册名newlaos,7位长度的变形值)

:00458542

8B856CFEFFFF            mov eax, dword ptr [ebp+FFFFFE6C]

:00458548

8B75CC                  mov esi,

dword ptr [ebp-34]

   <===这里是一串不变值(2E 59 8E 3F E7 20 81 33

1C 61 F8 29 88 35 4E A4 47 5C),任何机器上都是一样的。在编注册机时可以直接拿来用了。

:0045854B 668B0C4A

               mov cx, word ptr [edx+2\*ecx]

:0045854F

66330C46                xor cx, word ptr

[esi+2\*eax]

   <===依次取出定数变形值与不变值进行异或运算，只要8位

:00458553 8B9574FEFFFF

           mov edx, dword ptr [ebp+FFFFFE74]

:00458559

8B4580                  mov eax,

dword ptr [ebp-80]

:0045855C 66890C50

   mov word ptr [eax+2\*edx], cx

   <===保存计算结果(依次是AA 1D EE

BD 48 B6 F7 45)，放回定数变形值的对应位置

:00458560 C745FC72000000

   mov [ebp-04], 00000072

\*\*\*\*\*\*略去一段花指令\*\*\*\*\*\*\*

:004585C3

8B8574FEFFFF            mov eax, dword ptr [ebp+FFFFFE74]

:004585C9

8B8D2CFFFFFF            mov ecx, dword ptr [ebp+FFFFFF2C]

  <===注册名处理变形后的值(依次是7C 77 65 7E 73 7D 61)，如果不足8位，用00补。超过8位，只取前8位

:004585CF

8B9570FEFFFF            mov edx, dword ptr [ebp+FFFFFE70]

:004585D5

8B7580                  mov esi,

dword ptr [ebp-80]<===上一步异或计算的结果(依次是AA 1D EE BD 48 B6 F7 45)

:004585D8 668B0441

               mov ax, word ptr [ecx+2\*eax]

:004585DC

66330456                xor ax, word ptr

[esi+2\*edx]

   <===再一次做异或运算(结果依次是D6 6A 8B C3 3B CB 96 45)

:004585E0

668BC8                  mov cx, ax

:004585E3

6681E1FF01              and cx, 01FF <===得出的结果分别与01FF做与运算，这里值没有变

:004585E8

7909                    jns

004585F3 <===跳走

:004585EA 6649

       dec cx

:004585EC 6681C900FE

       or cx, FE00

:004585F1 6641

             inc cx

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004585E8(C)

|

:004585F3

662B8D50FFFFFF          sub cx, word ptr [ebp+FFFFFF50]<===哈哈，[ebp+FFFFFF50]是什么，就是前面备用的107呀，计算得出负值

:004585FA

0F8021040000            jo 00458A21

\*

Reference To: MSVBVM60.\_\_vbaI2Abs, Ord:0000h

                       |

:00458600

FF1548104000            Call dword ptr [00401048]

<===这里调用的是VB里的ABS函数(求绝对值函数)，最终得出结果依次为31

9D 7C 44 CC 3C 71 C2(注册码出来了，就是319D-7C44-CC3C-71C2)

:00458606 66898514FFFFFF

         mov word ptr [ebp+FFFFFF14], ax

\*\*\*\*\*\*\*\*\*\*\*略去一段无关代码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*(这一段代码的作用，是如果上面的结果出来，就只取前面两位)

|

:004588CE

C745FC85000000          mov [ebp-04], 00000085

:004588D5

E99FFBFFFF              jmp 00458479

<===呵呵，这里向上跳构成一个大循环，每一循环计算出两位注册码

\*

Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004584A2(C)

|

:004588DA

C745FC86000000          mov [ebp-04], 00000086

\*\*\*\*\*\*\*\*\*\*\*略去一段无关代码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*

Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h

 |

:00458A01 FF15D8114000            Call

dword ptr [004011D8]

:00458A07 C3

         ret
```

-----------------------------------------------------------------------------------------

### 5、算法分析：`f(注册名)=注册码`

- a、将注册码取出每个字符的`ASC`值，分别与18做异或运算，成为新的一串值`C(8)`，在后面计算中不足8位用0补足。超过8位的，如：第9位就和第1位生成的结果再相加，生成新的第1位，第10位就和第2位生的结果再相加，生成新的第2位，以此类推生成新的只有8位的串值。最后再将所有数，只要是与18做过异或运算的结果都参与相加，得出一个总数`regsam`备用。

- b、将定数`4C29AGE8BC2YCX3N`进行变形处理(具体过程见上面)，得出一串数76,84,44,60,82,AF,96,76,这串数字与注册名的长度有关，产生一个移位后，再用于后面的运算。将这串值相加得出定数5E0(10进制1504)，备用

- c、程序又取出一串定值2E 59 8E 3F E7 20 81 33 1C 61 F8 29 88 35 4E A4 47 5C，但只用前面8位

- d、将总数`regsam`加上5E0，再与1FF做与运算，生成`lastsum`，备用

- e、取a步骤中的第n位值与b步骤中的第n位值做异或运算，再与c中的第n位值做异或运算，得出结果与1FF做与运算，再减去`lastsum`，得出的结果，取绝对值，这时得到的数构成注册码的两位(超过2位，只取2位；不足2位，在前面补0)，a、b、c串中共有8个数值，因此得出注册码的16位数值。

### 6、注册机

注册名可为英文、中文或中英文混合，但只要有中文输入后，长度不能大于8，纯英文(字符)的注册名不受长度限制

------------VB 注册机()------------------------

```vb
Private Sub Command1_Click()

Dim e As Integer

Dim
h As Integer

Dim i As Integer

Dim j As Integer

Dim k As Integer

Dim
l As Integer

Dim m As Integer

Dim reglen As Integer

Dim g As Long

Dim
t As Long

Dim regsam As Long

Dim lastsum As Long

Dim laststr As String

Dim
regname As String

Dim strtmp As String

Dim
A As Variant

Dim B As Variant

Dim C As Variant

regname
= Text1.Text    '取得注册名

reglen = Len(regname)   '取得注册名的长度，这里取决定数的变形情况

If
reglen = 0 Then

m = MsgBox("你还没有输入注册名，请重新输入", 0, "连注册名也不愿输入???")

Else

A
= Array(118, 132, 68, 96, 130, 175, 150, 118) '定义定数的数组，以0位长度注册名的变值作为初值(已经改为10进制)

B
= Array(46, 89, 142, 63, 231, 32, 129, 51) '定义固定数值的数组，只取有用的前8位(已经改为10进制)

C
= Array(0, 0, 0, 0, 0, 0, 0, 0)      '将注册名的数组初始化，只取8位，不足用0补

 For i = 1 To reglen   '对定数的数组，根据注册名长度进行移位处理

    numtmp
= A(7)       '把最后一个值先拿出来，数组值的定义与VC一样A(7)代表数组中第8个值，A(0)代表第1个

   For j = 1 To 7      '小循环，后面各数值向前移一位

A(8 - j) = A(7 - j)

    Next j

    A(0) = numtmp

 Next i

  l = 0

  k = 0

  h = 1

  Do

                   '完成注册名的变形处理，及相加运算

   e = Asc(Mid(regname, (l + 1), 1))

    g = CLng("&H"
+ Hex(e))   '这里针对中文特别设计必须用clng函数，用CINT函数会出错

    C(k) = (g +
C(k)) Xor 18  '后面这个C(k)只在注册名大于9的时候，才用到

    t = C(k)
```
