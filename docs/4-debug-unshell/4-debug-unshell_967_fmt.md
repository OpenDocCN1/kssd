
# safeguard1.01 版主程序脱壳完全 Script

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-574.htm`](https://www.pediy.com/kssd/pediy07/pediy7-574.htm)

safeguard1.01 版主程序脱壳完全 Script

具体分析就不发表了

```c
/*
safeguard v1.01 版主程序脱壳脚本

windowsxp sp1  Ollydbg v1.10 CHS  OllyScript v0.92

注意：异常选项中不忽略\[除零异常\]

完成功能：

1.从双进程到单进程的转换

2.自动修复父进程参与的代码解码

3.自动完成输入表的解密和还原道 rdata 段中，修正输入表调用

4.自动完成 stolen code

                    by fxyang

                    2005.6.19
*/
```

```
#log

dbh

var index

var address

//////////////////////////////////////////////////////////////////

//anti OutputDebugStringA 修复

//////////////////////////////////////////////////////////////////

var setc1

gpa "OutputDebugStringA","kernel32.dll"

mov address,0

mov setc1,$RESULT

BPRM  setc1,1

mov address,0

eob setcode1

esto

setcode1:

inc address

cmp address,2

je Otp

esto

Otp:

mov address,esp

add address,4

mov [address],#00000000#

gpa "OutputDebugStringA","kernel32.dll"

mov setc1,$RESULT

BPRM  setc1,1

eob setcode2

bp 00415458

//eob int31:

mov index,0

esto

//防止飞到第二个 OutputDebugStringA anti

setcode2:

cmp index,0

je int31

inc index

mov address,esp

add address,4

mov [address],#00000000#

//pause

run

//下面 Script 完成双进程到单进程的转换，壳是在 int3 异常中处理的

//第一个 int3 处理

int31:

bc eip

mov eip,0041547E

bp 00415863

eob int32

esto

//第二个 int3 处理

int32:

bc eip

mov [eip],#90#

mov [00415875],#5C3F3F5C433A5C57494E444F57535C73797374656D33325C77696E6C6F676F6E2E65786500000000#

mov eip,00415989

bp 00415B04

eob jump0

esto

jump0:

bc eip

bp 00415B63

eob jump2

esto

jump2:

mov eax,1

bp 004165BD

eob jump1

esto

jump1:

bc eip

mov eip,0041c470

gpa "OutputDebugStringA","kernel32.dll"

mov setc1,$RESULT

BPRM  setc1,1

eob setcode3

esto

setcode3:

mov address,esp

add address,4

mov [address],#00000000#

bp 0041D025

eob tmp1

esto

//正确的解密种子 AH

tmp1:

bc eip

mov eax,CD17544C

bp 0041D160

eob tmp2

esto
```

```c
/*
到异常解码的地方：

0041DD22    8985 9C854000   MOV DWORD PTR SS:[EBP+40859C],EAX

0041DD28    EB 22           JMP SHORT safeguar.0041DD4C

0041DD2A    EB 47           JMP SHORT safeguar.0041DD73

0041DD2C    DF69 4E         FILD QWORD PTR DS:[ECX+4E]

0041DD2F    58             POP EAX

0041DD30    DF59 74         FISTP WORD PTR DS:[ECX+74]

0041DD33    EE             OUT DX,AL                                             ; I/O 命令

0041DD34    EB 01           JMP SHORT safeguar.0041DD37

0041DD36    DF75 E9         FBSTP TBYTE PTR SS:[EBP-17]

0041DD39    0F599C81 C1E5FF>MULPS XMM3,DQWORD PTR DS:[ECX+EAX*4-1A3F]

0041DD41    FF9D FFE1EB51   CALL FAR FWORD PTR SS:[EBP+51EBE1FF]                 ; 远距呼叫

0041DD47    E8 EEFFFFFF     CALL safeguar.0041DD3A

0041DD4C    CC             INT3

0041DD4D    90             NOP
*/
```

```
tmp2:

bp 0041DD28

eob tmp3

esto

//修复长度为 1B84 的父进程参与解码

tmp3:
```

```c
/*
0041DD10    60             PUSHAD

0041DD11    9C             PUSHFD

0041DD12    B8 4EDD4100    MOV EAX,safeguar.0041DD4E

0041DD17    33D2          XOR EDX,EDX

0041DD19    BB 73737373    MOV EBX,73737373

0041DD1E    33C9          XOR ECX,ECX

0041DD20    3118          XOR DWORD PTR DS:[EAX],EBX

0041DD22    8D40 04        LEA EAX,DWORD PTR DS:[EAX+4]

0041DD25    83C1 04        ADD ECX,4

0041DD28    83C2 04        ADD EDX,4

0041DD2B    81F9 04010000  CMP ECX,104

0041DD31    74 0A         JE SHORT safeguar.0041DD3D

0041DD33    81FA 841B0000  CMP EDX,1B84

0041DD39    74 0A         JE SHORT safeguar.0041DD45

0041DD3B  ^ EB E3         JMP SHORT safeguar.0041DD20

0041DD3D    81C3 01010101  ADD EBX,1010101

0041DD43  ^ EB D9         JMP SHORT safeguar.0041DD1E

0041DD45    9D             POPFD

0041DD46    61             POPAD

0041DD47    EB 05         JMP SHORT safeguar.0041DD4E

0041DD49    90             NOP

0041DD4A    90             NOP

0041DD4B    90             NOP

60 9C B8 4E DD 41 00 33 D2 BB 73 73 73 73 33 C9 31 18 8D 40 04 83 C1 04 83 C2 04 81 F9 04 01 00

00 74 0A 81 FA 84 1B 00 00 74 0A EB E3 81 C3 01 01 01 01 EB D9 9D 61 EB 05
*/
```

```
bc eip

mov eip,0041DD10

mov [eip],#609CB84EDD410033D2BB7373737333C931188D400483C10483C20481F904010000740A81FA841B0000740AEBE381C301010101EBD99D61EB05#

//bp 0041DE3D

bp 0041BD13

eob iatbiao

esto

//下面是处理输入表的 Script

iatbiao:

bc eip
```

```c
/*
修改壳的处理代码：

0041BD13    55             PUSH EBP

0041BD14    8BEC          MOV EBP,ESP

0041BD16    60             PUSHAD

0041BD17    8B7D 08        MOV EDI,DWORD PTR SS:[EBP+8]               ; 003A0000

0041BD1A    8B75 0C        MOV ESI,DWORD PTR SS:[EBP+C]               ; Stack SS:[0012FF9C]=77E5B285 (kernel32.GetProcAddress)

0041BD1D    8B1D 20134100  MOV EBX,DWORD PTR DS:[411320]             ; 00411650 /rdata 中的存放地址

0041BD23    8933          MOV DWORD PTR DS:[EBX],ESI

0041BD25    66:C707 FF25  MOV WORD PTR DS:[EDI],25FF

0041BD2A    47             INC EDI

0041BD2B    47             INC EDI

0041BD2C    891F          MOV DWORD PTR DS:[EDI],EBX

0041BD2E    83C7 04       ADD EDI,4

0041BD31    83C3 04       ADD EBX,4

0041BD34    891D 20134100 MOV DWORD PTR DS:[411320],EBX

0041BD3A    897C24 FC     MOV DWORD PTR SS:[ESP-4],EDI

0041BD3E    90             NOP

0041BD3F    90             NOP

0041BD40    90             NOP

0041BD41    90             NOP

0041BD42    90             NOP

0041BD43    E9 88040000    JMP safeguar.0041C1D0
*/
```

```
mov [411320],00411650

mov [0041BD1D],#8B1D20134100893366C707FF254747891F83C70483C304891D20134100897C24FC9090909090E98804000090#

//0041BD1D    8B1D 20134100   MOV EBX,DWORD PTR DS:[411320]             ; safeguar.00411650 中断在这里

bp 0041BD1D

mov index,0

log esi

eob setiat

esto

//下面用于模块的分割 Script

setiat:

inc index

cmp index,16

je setiat1

cmp index,1f

je setiat1

cmp index,20

je setiat1

cmp index,23

je setiat2

esto

setiat1:

mov address,[411320]

add address,4

mov [411320],address

run

setiat2:

bc 041BD1D

mov address,[411320]

add address,4

mov [411320],address
```

```c
/*
由于 Script 花费在处理输入表的时间比较长,所以下面这个 time anti 要修改

0041F251    3D D0070000    CMP EAX,7D0

0041F256    EB 50         JMP SHORT safeguar.0041F2A8
*/
```

```
bp 0041F256

eob time1

run

time1:

mov !ZF,1

gpa "GetTickCount","kernel32.dll"

bp $RESULT

mov index,0

eob temp

run

temp:

inc index

cmp index,2

je temp2

run

temp2:

bc eip

mov index,0

eoe seteoe1

run

seteoe1:

//pause
```

```c
/*
004208CF   /EB 14         JMP SHORT safeguar.004208E5

004208E5    68 00000000   PUSH 0

004208EA    EB 03         JMP SHORT safeguar.004208EF

004208EC    FD             STD

004208ED    50             PUSH EAX

004208EE    FB             STI

004208EF    E8 00000000    CALL safeguar.004208F4

004208F4    830424 0A      ADD DWORD PTR SS:[ESP],0A

004208F8    68 38F44000    PUSH safeguar.0040F438

stolen code
*/
```

```
inc index

cmp index,31

je ep

esto

ep:

mov eip, 004208CF

bprm 004208E5,2

eob oep

run

oep:
```

```c
/*
伪 OEP

0040F407    FF35 62204100  PUSH DWORD PTR DS:[412062]                 ; safeguar.00400000

0040F40D    E8 7A000000    CALL safeguar.0040F48C
*/
```

```
bp 0040F407

eob setiatadd

run

setiatadd:

bc eip
```

```c
/*
0040F3F0    60             PUSHAD

0040F3F1    B8 1CF44000    MOV EAX,safeguar.0040F41C

0040F3F6    8B18          MOV EBX,DWORD PTR DS:[EAX]

0040F3F8    8B1B          MOV EBX,DWORD PTR DS:[EBX]

0040F3FA    8B5B 02       MOV EBX,DWORD PTR DS:[EBX+2]
*/
```



```
0040F3FD    8918            MOV DWORD PTR DS:[EAX],EBX  

0040F3FF    83C0 06         ADD EAX,6  

0040F402    8078 03 00      CMP BYTE PTR DS:[EAX+3],0  

0040F406    74 EE           JE SHORT safeguar.0040F3F6  

0040F408    61              POPAD  

0040F409    90              NOP  

60 B8 1C F4 40 00 8B 18 8B 1B 8B 5B 02 89 18 83 C0 06 80 78 03 00 74 EE 61 90  
```

\*/

//重建 iat 调用地址

`mov eip,0040F3F0`

`mov [eip],#60B81CF440008B188B1B8B5B02891883C0068078030074EE6190#`

//pause

`bp 0040F409`

`eob setep`

`run`

//处理 stolen code

`setep:`

`bc eip`

`mov eip,0040F3F0`

`mov [eip],#6A00E841000000A3622041006A006847F240006A006A65FF3562204100E87A0000006A00E813000000#`

`msg "safeguard v1.01 脱壳完成:-), 感谢 simonzh !"`

`ret`

```
fxyang
2005.6.19
```

希望这个脚本对`simonzh`有所帮助，特别是对`iat`的处理，虽然加密函数很复杂，但是传递给这个函数参数时漏洞太大：

```
0041E195    E8 79DBFFFF     CALL safeguar.0041BD13  

0012FF98    003A004D  |Arg1 = 003A004D  

0012FF9C    77E5D8B4  |Arg2 = 77E5D8B4  

0012FFA0    00000000  \Arg3 = 00000000  
```

```
EAX 003A004D  

ECX 77E5D8B4  kernel32.LoadLibraryA  

EDX 7FFE0304  

EBX 00413000  safeguar.00413000  

ESP 0012FF98  

EBP 00013000  

ESI 00416633  safeguar.00416633  

EDI 0072C794  

EIP 0041E195  safeguar.0041E195  
```

修改`iat`处理代码：

```
0041BD13    55              PUSH EBP  

0041BD14    8BEC            MOV EBP,ESP  

0041BD16    60              PUSHAD  

0041BD17    8B7D 08         MOV EDI,DWORD PTR SS:[EBP+8]         ; 003A0000  

0041BD1A    8B75 0C         MOV ESI,DWORD PTR SS:[EBP+C]         ; Stack SS:[0012FF9C]=77E5B285 (kernel32.GetProcAddress)  
```

很容易修改成自动恢复的

`4dd466d0e1e0cc123afbe504da2462ca`
