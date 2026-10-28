
# 注册机与特洛依 DLL 分析

## 汇编代码片段

```
0040ABF6   |8A11            mov     dl, [ecx]
0040ABF8   |41              inc     ecx
0040ABF9   |84D2            test    dl, dl
0040ABFB  ^|75 F9           jnz     short 0040ABF6
0040ABFD   |2BCE            sub     ecx, esi
0040ABFF   |8BD1            mov     edx, ecx
0040AC01   |C1E9 02         shr     ecx, 2
0040AC04   |8BF0            mov     esi, eax
0040AC06   |BF 846A5A00     mov     edi, 005A6A84                               ; ASCII "E507C88FADD96090C6283299DD8E64C5"
0040AC0B   |F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
0040AC0D   |8BCA            mov     ecx, edx
0040AC0F   |83E1 03         and     ecx, 3
0040AC12   |F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]
0040AC14   |B9 08000000     mov     ecx, 8
0040AC19   |BE 846A5A00     mov     esi, 005A6A84                               ; ASCII "E507C88FADD96090C6283299DD8E64C5"
0040AC1E   |BF 546A5A00     mov     edi, 005A6A54                               ; ASCII "E507C88FADD96090C6283299DD8E64C5"
0040AC23   |F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
0040AC25   |A4              movs    byte ptr es:[edi], byte ptr [esi]
0040AC26   \B9 08000000     mov     ecx, 8
0040AC2B    8D7C24 1C       lea     edi, [esp+1C]
0040AC2F    BE 846A5A00     mov     esi, 005A6A84                               ; ASCII "E507C88FADD96090C6283299DD8E64C5"
0040AC34    33C0            xor     eax, eax
0040AC36    F3:A7           repe    cmps dword ptr es:[edi], dword ptr [esi]    ;比较由注册文件运算出的硬件信息与先前取硬盘系列号生成的是否一致。
0040AC38    75 0B           jnz     short 0040AC45
```

如果相等，`al`＝1（成功的标志），否则跳到`0040AC45`,由`040AC47  32C0   xor  al, al`可知不成功返回0

```
0040AC3A    B0 01           mov     al, 1       ;置al为1
0040AC3C    5F              pop     edi
0040AC3D    5E              pop     esi
0040AC3E    5B              pop     ebx
0040AC3F    8BE5            mov     esp, ebp
0040AC41    5D              pop     ebp
0040AC42    C2 0400         retn    4
```

比较不相等的处理:

```
0040AC45    5F              pop     edi
0040AC46    5E              pop     esi
0040AC47    32C0            xor     al, al       ;置al 为 0
0040AC49    5B              pop     ebx
0040AC4A    8BE5            mov     esp, ebp
0040AC4C    5D              pop     ebp
0040AC4D    C2 0400         retn    4
```

大致就是这样。上面说得比较乱。做个总结。

## 注册机算法

注册机的算法如下：

比如说你要注册的名称为 `ABC`,你的硬件信息为`1234`（实际为32个字节，此处假设为4个字节） 则生成注册码的算法为:

`ABC`--> `414243`

`1234`--> `3031323334`

`'\_'`--->`ASCII`值为`5F`

进行如下运算 `X =( 4142435F3031323334) ^ D  % N` 。`D`是作者手中的私钥。`N`就是上面给出的。

`X`转换为串就是`key.dat`中值，也就是注册码了。

验证部分就是上面的算法了。`Y= ( X ^1C883) %N`   理论上用正确的注册码和对应的硬件信息生成的 `Y`值就是`4142435F3031323334`转换为串就是

`ABC\_1234` 作者以`\_`为标志，分出注册名称与硬件信息部分。然后验证`1234`与当前硬盘物理系列号生成的值是不是一致。一致认为注册成功(一机一码)。

## 特洛依 DLL 的破解原理

由于程序在载入`DLL`时,首先会尝试从当前的程序文件路径装载,如果没找到,则在系统目录下查找. 利用这个特点,我们可以让程序在调用系统`DLL`中某个函数时先调用自己提供的`DLL`中的同名函数(提供与系统`DLL`相同的名字,函数名也相同,放在程序的目录下面),在完成我们某个目的后,再通过自己的函数取出系统`DLL`该同名函数的地址,跳到该地址执行原函数功能.

下面是特洛依`DLL`的分析.

该`DLL`是通过挂钩`WSAStartup`实现`PATCH`功能的.看看该特洛依`DLL`提供的`WSAStartup`代码:

```
int __stdcall WSAStartup(WORD wVersionRequested,LPWSADATA lpWSAData)
.text:10001A10                 public WSAStartup
.text:10001A10 WSAStartup      proc near
.text:10001A10                 call    sub_10001100                   ;一开始就进入PATCH代码函数.
.text:10001A15                 push    offset aWsastartup ; "WSAStartup"
.text:10001A1A                 call    sub_10001140       ;这个call主要工作就是通过传入的串,也就是函数名,取到系统提供的
                                                            ;ws2_32.dll该函数的地址,放到EAX中去.
.text:10001A1F                 jmp     eax                ;跳到系统中真正的WSAStartup函数.(不能是CALL,只能是JMP,跟栈中参数有关)
.text:10001A1F WSAStartup      endp
```

再看下`10001100`处这个函数的代码:

```
.text:10001100 sub_10001100    proc near               ; CODE XREF: WSAStartupp
.text:10001100                 mov     eax, dword_10008B38            ;一个全局变量,记为x8B38吧
.text:10001105                 push    esi
.text:10001106                 inc     eax                            ;将x8B38加一
.text:10001107                 mov     esi, 5592D0h                   ;5592d0是IAT中的一个地址,存放RtlGetLastWin32Error的地址
.text:1000110C                 cmp     eax, 2                         ;x8B38变量是不是为2
.text:1000110F                 mov     dword_10008B38, eax
.text:10001114                 jnz     short loc_1000113E             ;不为2不PATCH.
```

这地方为什么要用一个全局变量,并判断该变量是不是为2 ?

主要原因是第一次调用这个函数时,壳还没有将代码段解压出来,不是`PATCH`时机.第二次才是时机.

; 下面这个`cmp`检测是不是要修改的目标程序,只有地址在`040AC2B`处值为`1C247C8D`才是目标程序
;否则你不小心把这个特洛依`DLL`放在某个文件下下面,而这个文件夹下面有程序正好调用系统的`ws2_32.dll`就麻烦了.

```
.text:10001116                 cmp     dword ptr ds:40AC2Bh, 1C247C8Dh
.text:10001120                 jnz     short loc_1000113E
.text:10001122                 call    sub_10001040
```

;上面的`call`是创建一个`key.dat`文件,并写入256个0,让程序执行到注册算法验证部分.没这文件或大小不对,直接跳到其它地方了.

```
.text:10001127                 mov     eax, [esi]              ; 取RtlGetLastWin32Error(见.text:10001107 处说明)
.text:10001129                 mov      dword_10008B34, eax    ;将RtlGetLastWin32Error地址放到一个全局变量处,因为做完我们想做的事后,还得真正调用它
.text:1000112E                 mov     dword ptr [esi], offset loc_10001000 ;
```

上面一句代码就是将原`iat`中`RtlGetLastWin32Error`地址换为地址`10001000` 即当程序调用`RtlGetLastWin32Error`时不是调用`RtlGetLastWin32Error`,而是跑到`10001000` 处.

```
.text:10001134                 mov     dword ptr ds:5594CCh, offset loc_10001030 ;
```

这句代码同上,处理`GetTickCount`地址,既当调用`GetTickCount`时,跑到了`10001030`处
(题外话,用这种方法破解,并不需要处理`GetTickCount`函数的.我当初写`LAODER`是因为有`SLEEP`,会被原程序检测到的,不知大牛对此有何看法?)

```
.text:1000113E loc_1000113E:                           ; CODE XREF: sub_10001100+14j
.text:1000113E                                         ; sub_10001100+20j
.text:1000113E                 pop     esi
.text:1000113F                 retn
.text:1000113F sub_10001100    endp
```

下面看看`10001000`处的代码,即调用`RtlGetLastWin32Error`时所要执行的代码:

```
.text:10001000                 mov     eax, 40ABAFh
.text:10001005                 cmp     [ebp+4], eax
.text:10001008                 jnz     short loc_10001026 ;
```

这个判断很重要,不是对每次调用`RtlGetLastWin32Error`都要`patch`的.只有`eax`符合上面的条件时才`patch`的.

联想一下前面注册算法分析部分.我们提到`string1`,如果提供一个正确的`sting1`,就可以让程序认为注册成功.

前面的注册算法分析部分有这么一行:

```
0040ABAA    E8 77980400     call    00454426           ;从解密的明文中取出用户名（以下划线'_'做为标记）（这个CALL中调用了RtlGetLastWin32Error,patch工作就是用到这个函数的） 
                                                         ;特洛依DLL修改了RtlGetLastWin32Error地址,当调用RtlGetLastWin32Error时执行了我们的代码,patch了string1.从而让程序认为正确注册了,后话
```

这个后话就是指现在了.因为在`454426`这个`call`中,调用了`RtlGetLastWin32Error`.刚才的那个判断就是判断`RtlGetLastWin32Error`是不是来自于这个地方.

如果来自于这个对方,就构造该`sting1`串.只在来自于该处的`RtlGetLastWin32Error`才能修改内存

下面的代码就是构造`string1`串了.

```
.text:1000100A                 pusha
.text:1000100B                 mov     edi, [ebp+8]
.text:1000100E                 mov     ecx, 5
.text:10001013                 lea     esi, aZ_        ; "学习\_"   (也就是用户名+ "\_")
.text:10001019                 rep movsb
.text:1000101B                 lea     esi, [ebp+5Ch]  ;ebp+5ch处是硬件信息
.text:1000101E                 mov     ecx, 20h        ;此处0x20是硬件信息的长度,32个字节
.text:10001023                 rep movsb               ; 移硬件信息。
.text:10001025                 popa
.text:10001026
.text:10001026 loc_10001026:                           ; CODE XREF: .text:10001008j
```

跳到真正的`RtlGetLastWin32Error`执行其代码.

```
.text:10001026                 jmp     dword_10008B34  ; 上面提到的保存RtlGetLastWin32Error的变量
```

至于为什么要处理`GetTickCount`,跟本主题没关系.不多说了.

系统提供的`ws2_32.dll`还有许多其它导出函数.所有特洛依`DLL`也要提供相应的导出函数.其做法类似上面`WSAStartup`函数.

```
.text:10001A15                 push    offset aWsastartup ; "WSAStartup"
.text:10001A1A                 call    sub_10001140       ;这个call主要工作就是通过传入的串,也就是函数名,取到系统提供的
                                                            ;ws2_32.dll该函数的地址,放到EAX中去.
.text:10001A1F                 jmp     eax
```

只是上面的串`WSAStartup`要改为其它导出函数名就行了.

至此这个特洛依`DLL`分析工作就差不多结束了.

## 经验总结
