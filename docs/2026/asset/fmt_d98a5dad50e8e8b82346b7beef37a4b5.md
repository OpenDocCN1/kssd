
## 汇编代码

```
:0050919B 8BD8                    mov ebx, eax
:0050919D 83FB01                  cmp ebx, 00000001
:005091A0 7C23                    jl 005091C5
```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005091C3(C)
|

```
:005091A2 8D45E8                  lea eax, dword ptr [ebp-18]
:005091A5 50                      push eax
:005091A6 B901000000              mov ecx, 00000001
:005091AB 8BD3                    mov edx, ebx
:005091AD 8B45F8                  mov eax, dword ptr [ebp-08]
:005091B0 E863B0EFFF              call 00404218
:005091B5 8B55E8                  mov edx, dword ptr [ebp-18]
:005091B8 8D45F4                  lea eax, dword ptr [ebp-0C]
:005091BB E858AEEFFF              call 00404018
:005091C0 4B                      dec ebx
:005091C1 85DB                    test ebx, ebx
:005091C3 75DD                    jne 005091A2
```
====> 这个循环将`548114883`倒序排列`388411845`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005091A0(C)
|

```
:005091C5 8B45FC                  mov eax, dword ptr [ebp-04]
:005091C8 8B55F4                  mov edx, dword ptr [ebp-0C]
```
====> `EDX=388411845`         这就是注册码！

```
:005091CB E814ACEFFF              call 00403DE4
:005091D0 33C0                    xor eax, eax
:005091D2 5A                      pop edx
:005091D3 59                      pop ecx
:005091D4 59                      pop ecx
:005091D5 648910                  mov dword ptr fs:[eax], edx
:005091D8 68F2915000              push 005091F2
```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005091F0(U)
|

```
:005091DD 8D45E8                  lea eax, dword ptr [ebp-18]
:005091E0 BA05000000              mov edx, 00000005
:005091E5 E8CAABEFFF              call 00403DB4
:005091EA C3                      ret
```
—————————————————————————————————

进入加法`CALL`：`0050917B`   `call`  `005091FC`

\* Referenced by a CALL at Address:
|:0050917B
|

```
:005091FC 53                      push ebx
:005091FD 56                      push esi
:005091FE 8BF1                    mov esi, ecx
:00509200 8BDA                    mov ebx, edx
:00509202 8B03                    mov eax, dword ptr [ebx]
```
====> `EAX=528094555`

```
:00509204 E8BB1AF0FF              call 0040ACC4
```
====> 转换成浮点数

```
:00509209 D80520925000            fadd dword ptr [00509220]
```
====> `ST=528094555.00000000000 + 20020328=548114883.00000000000`

```
:0050920F 83C4F4                  add esp, FFFFFFF4
:00509212 DB3C24                  fstp tbyte ptr [esp]
:00509215 9B                      wait
:00509216 8BC6                    mov eax, esi
:00509218 E89718F0FF              call 0040AAB4
:0050921D 5E                      pop esi
:0050921E 5B                      pop ebx
:0050921F C3                      ret
```
—————————————————————————————————

### 【算法总结】

1、取硬盘序列号的10进制值`555490825`作为机器码

2、倒序排列机器码：`528094555`

3、`528094555.00000000000 + 20020328=548114883.00000000000`

4、倒序排列`548114883`，得出`388411845`     就是注册码  ^O^  ^O^

—————————————————————————————————

### 【完美爆破】

`00508E3C A170F05000`              `mov eax, dword ptr [0050F070]`

改为：  `8B45C89090`              `mov eax, dword ptr [ebp-38]`  补2个`NOP`

与`00508E39`处相映成趣。

—————————————————————————————————

### 【KeyMake之{89th}内存注册机】

中断地址：`00508EFE`

中断次数：`1`

第一字节：`E8`

指令长度：`5`

内存方式：`EDX`

—————————————————————————————————

### 【注册信息保存】

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\RenYuanSoft\HYDIC]

"LocalM"="|||{€wy|棬汉径公"

"LocalID"="z{xx{|棬汉径公"
```
—————————————————————————————————

### 【整理】

机器码：`555490825`

注册码：`388411845`

—————————————————————————————————

,     \_/

/| \_.-~/            \\_     ,        青春都一饷

( /~   /              \~-.\_ |\

`\\  \_/                \   ~\ )          忍把浮名

\_-~~~-.)  )\_\_/;;,.          \\_  //'

/'\_,\   --~   \ ~~~-  ,;;\\_\_\_(  (.-~~~-.        换了破解轻狂

`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_`\

/~~//'   /' `~\         ) /--..\_, )\_  `~

"  `~"  "      `"      /~'`\    `\\~~\

"     "   "~'  ""

Cracked By 巢水工作坊——fly [OCN][FCG]

2003-05-31  23:00
