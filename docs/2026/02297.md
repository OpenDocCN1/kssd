
# Takagoraku v1.0 的破解，重新写了一边 (5千字)

一个小游戏，挺好玩的

下载 `http://sd.onlinedown.net/down/takagoraku.exe`

`bpx getwindowtext` 按 F10 一直到

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00409034(C)

|

:00409019 8B45E4
  mov eax, dword ptr [ebp-1C]

:0040901C 0FBE5405C0              movsx edx,
  byte ptr [ebp+eax-40]

:00409021 0155EC
  add dword ptr [ebp-14], edx<=累加用户名各字母的ASCII值

:00409024 FF45E4
  inc [ebp-1C]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00409017(U)

|

:00409027 8D4DC0
  lea ecx, dword ptr [ebp-40]

:0040902A 50
  push ecx

:0040902B E8C0EB0200              call 00437BF0

:00409030 59
  pop ecx

:00409031 3B45E4
  cmp eax, dword ptr [ebp-1C]

:00409034 77E3
  ja 00409019

:00409036 8B45E8
  mov eax, dword ptr [ebp-18]<=注册码前6位的16进制值

:00409039 2B45EC
  sub eax, dword ptr [ebp-14]<=减去用户名的累加值

:0040903C 50
  push eax<====================eax=0x69D55

:0040903D FF75F0
  push [ebp-10]

:00409040 8B5508
  mov edx, dword ptr [ebp+08]

:00409043 FF7219
  push [edx+19]

:00409046 E8AC680000              call 0040F8F7<==============核心计算，跟进去

=====================================================================================

  用户名：lancelot==>6C+61+6E+63+65+6C+6F+74=0x0352

  注册码：434343=0x6A0A7

          0x6A0A7-0x0352=0x69D55

=====================================================================================

\* Referenced by a CALL at Addresses:

|:00408024  , :00409046  , :0040F40C

|

:0040F8F7 55
  push ebp

:0040F8F8 8BEC
  mov ebp, esp

:0040F8FA 83C4F4
  add esp, FFFFFFF4

:0040F8FD 8B4510
  mov eax, dword ptr [ebp+10]<====eax=0x69D55

:0040F900 83C005
  add eax, 00000005<==============eax=0x69D5A

:0040F903 6BC064
  imul eax, 00000064<=============eax=0x02957728

:0040F906 0532FFFFFF              add eax,
  FFFFFF32<==============eax=0x0295765A

:0040F90B B91B000000              mov ecx,
  0000001B

:0040F910 99
  cdq

:0040F911 F7F9
  idiv ecx<=======================eax=0x187FA4

:0040F913 83C038
  add eax, 00000038<==============eax=0x187FDC

:0040F916 8D04C0
  lea eax, dword ptr [eax+8\*eax]<=eax=0xDC7EBC

:0040F919 8945F4
  mov dword ptr [ebp-0C], eax

:0040F91C DB45F4
  fild dword ptr [ebp-0C]<========14450364

:0040F91F D95DFC
  fstp dword ptr [ebp-04]

:0040F922 8B450C
  mov eax, dword ptr [ebp+0C]<====eax=0x3B77A183

:0040F925 83C0FB
  add eax, FFFFFFFB<==============eax=0x3B77A17E

:0040F928 B964000000              mov ecx,
  00000064

:0040F92D 99
  cdq

:0040F92E F7F9
  idiv ecx<=======================eax=0x983C7E

:0040F930 05CE000000              add eax,
  000000CE<==============eax=0x983D4C

:0040F935 8D0440
  lea eax, dword ptr [eax+2\*eax]<=eax=0x1C8B7E4

:0040F938 8D04C0
  lea eax, dword ptr [eax+8\*eax]<=eax=0x100E7704

:0040F93B 83C0C8
  add eax, FFFFFFC8<==============eax=0x100E76CC

:0040F93E B909000000              mov ecx,
  00000009

:0040F943 99
  cdq

:0040F944 F7F9
  idiv ecx<=======================eax=0x1C8B7DD

:0040F946 8945F4
  mov dword ptr [ebp-0C], eax

:0040F949 DB45F4
  fild dword ptr [ebp-0C]<========29931485

:0040F94C D95DF8
  fstp dword ptr [ebp-08]

:0040F94F D945F8
  fld dword ptr [ebp-08]

:0040F952 D8257BF94000            fsub dword ptr
  [0040F97B]

:0040F958 D85DFC
  fcomp dword ptr [ebp-04]

:0040F95B DFE0
  fstsw ax

:0040F95D 9E
  sahf

:0040F95E 7715
  ja 0040F975

:0040F960 D945F8
  fld dword ptr [ebp-08]

:0040F963 D8057BF94000            fadd dword ptr
  [0040F97B]

:0040F969 D85DFC
  fcomp dword ptr [ebp-04]

:0040F96C DFE0
  fstsw ax

:0040F96E 9E
  sahf

:0040F96F 7204
  jb 0040F975<===================不能跳

:0040F971 B001
  mov al, 01

:0040F973 EB02
  jmp 0040F977

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0040F95E(C), :0040F96F(C)

|

:0040F975 33C0
  xor eax, eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040F973(U)

|

:0040F977 8BE5
  mov esp, ebp

:0040F979 5D
  pop ebp

:0040F97A C3
  ret

============================================================================================
```

程序将`0x69D55`和`0x3B77A183`分别进行一序列运算后的值相比较，因此，我这样算：

```
0x3B77A183-5==0x3B77A17E

0x3B77A17E/0x64==0x983C7E

0x983C7E+0xCE==0x983D4C

0x983D4C\*3==0x1C8B7E4

0x1C8B7E4\*9==0x100E7704

0x100E7704-0x38==0x100E76CC

0x100E76CC/9==0x1C8B7DD<===========这是浮点运算前的值，我们略过浮点运算看行不行

0x1C8B7DD/9==0x32BF18<=============开始逆运算

0x32BF18-0x38==0x32BEE0

0x32BEE0\*0x1B==0x55A21A0

0x55A21A0+0xCE==0x55A226E

0x55A226E/0x64==0xDB38B

0xDB38B-5==0xDB386

0xDB386+0x352==0xDB6D8==898776<=======bingo!!! 这就是注册码啦!!
```

感谢小楼和6767[BCG]的帮助，由于程序重新安装后仍然是已注册版本，所以无法验证浮点运算部分了。

Crack by lancelot  2001.08.14
