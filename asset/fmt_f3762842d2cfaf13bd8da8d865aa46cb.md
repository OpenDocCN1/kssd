
# 菜鸟 \[第一阶段 第四题\] 分析

> 原文：[`www.pediy.com/kssd/pediy11/123550.html`](https://www.pediy.com/kssd/pediy11/123550.html)

## 1

首先把 `Od` 调试异常都钩掉，`Od` 载入断在：

```
006530D4  >  /EB 01           jmp         006530D7h
```

两次 `F7` 到：

```
006530D8    E8 00000000     call       006530DDh
```

记住 `Esp=0012FFA4`, 把这个值纪录下来，为了以后对 `esp` 下 `DWORD` 硬件访问断点到达 `oep`。

由于是双进程壳，就有两种执行流程，父进程执行流程和子进程执行流程，父进程创建并调试子进程，接受子进程调试事件，控制子进程的运行，两种执行流程，必然有个分界点，这里是 `CreateMutexA`，壳对 `CreateMutexA` 入口有断点检测，所以在 `CreateMutexA+0x51` 下断，这里下硬件断：

```
7C80EA2D    C2 0C00         retn       000Ch  //下硬件断点
```

返回到达：

```
00656DD6    8985 C66C4000   mov        dword ptr [ebp+00406CC6h] , eax
00656DDC    8D85 3A271F03   lea        eax , dword ptr [ebp+031F273Ah]
00656DE2    2D FCCEDE02     sub        eax , 02DECEFCh
00656DE7    FF10            call       dword ptr [eax]
00656DE9    BB CA7DB9FE     mov        ebx , FEB97DCAh //这里改 eax=B7,
```

使程序按照子进程的流程运行

继续单步到：

```
00656E3D    F1              int1  //产生异常，正常运行的话父进程接受这个异常，并通过 GetContextThread 和 SetContextThread 改变 eip，进而改变子进程的流程，这里改 eip= 0x00656E5C
```

然后运行断在：

```
0065364A    F3:AE           scasb   //异常，这里不处理
```

8 次 `Shift+F9` 忽略 8 个异常，运行到：

```
00657766    87DF            xchg       edi , ebx  //这里处理异常，和 int1 异常一样，改 eip，这里 eip= 00657791
00657791    C3              ret
```

### IAT 处理

```
00653F44    8B3B            mov        edi , dword ptr [ebx]  开始
00653F46    03BD 1E424000   add        edi , dword ptr [ebp+0040421Eh]
00653F4C    803F 44         cmp        byte ptr [edi] , 44h
...
...
...
00653F7A    3D 88BF46CF     cmp        eax , CF46BF88h  //比较 hash 值
00653F7F    75 20           jne        00653FA1h
00653F81    8B85 1A424000   mov        eax , dword ptr [ebp+0040421Ah]  //等于处理
00653F87    D1E1            shl        ecx , 1
00653F89    03C1            add        eax , ecx
00653F8B    0FB700          movzx      eax , word ptr [eax]
00653F8E    C1E0 02         shl        eax , 02h
00653F91    0385 26424000   add        eax , dword ptr [ebp+00404226h]
00653F97    8B00            mov        eax , dword ptr [eax]
00653F99    0385 1E424000   add        eax , dword ptr [ebp+0040421Eh]   //得到 API 地址，EAX 7C831EDD kernel32.DeleteFileA
```

`00653F4` 到 `00653F7A` 是取出 `API` 字符串名字，并运算得到一个函数值，然后比较是不是 `IAT` 表里的 `API`，是的话到：

```
00653D67    8B18            mov        ebx , dword ptr [eax]
```

这里可以看到开始取 `DeleteFileA` 入口内容，并开始做变形处理存放到

```
00DF0000   /EB 01           jmp        00DF0003h
```

```
00653E23    F3:A4           movsb
```

这里下断，断几次，会看到 `00DF0000` 里发生变化：

```
00DF0000   /EB 01           jmp        00DF0003h
00DF0003    8BFF            mov        edi , edi
00DF0005    55              push       ebp
00DF0006    8BEC            mov        ebp , esp
00DF0008    FF75 08         push       dword ptr [ebp+08h]
00DF000B    EB 07           jmp        00DF0014h
00DF0014  ^\\EB F8           jmp        00DF000Eh
00DF000E  - E9 D21EA47B     jmp        7C831EE5h     //跳到 DeleteFileA+8
```

在来到这里：

```
0065418A    8907            mov        dword ptr [edi] , eax   写壳的 IAT
0065418C    EB 02           jmp        00654190h
```

这里 `edi=00657E1F`，`eax=DF00000`

`[esp-3C]`-`[esp-38]`= `DeleteFileA`

这里可以打补丁修复 `IAT`, 我申请的内存地址：`E20000`

二进制代码：

```
8B 44 24 C4 2B 44 24 C8 89 07 E9 81 41 83 FF
```

```
00E20000    8B4424 C4       mov        eax , dword ptr [esp-3Ch]
00E20004    2B4424 C8       sub        eax , dword ptr [esp-38h]
00E20008    8907            mov        dword ptr [edi] , eax
00E2000A  - E9 814183FF     jmp        00654190h
00E2000F    90              nop
```

`0065418A` 修改为：

```
0065418A  - E9 71BE7C00     jmp        00E20000h
0065418F    90              nop
```

这时把开始记录的 `esp=0012FFA4`, 对 `0012FFA4` 下 `DWORD` 硬件访问断点，然后运行，直奔 stolen `OEP`：

```
00654CB5    F7D2    not edx                        ; ntdll.KiFastSystemCallRet
00654CB7    39C2            cmp        edx , eax
00654CB9    F7C0 74E7F921   test       eax , 21F9E774h
00654CBF    0FACC2 48       shrd       edx , eax , 00000048h
00654CC3    0FBDC8          bsr        ecx , eax
00654CC6    C7C2 2431C7CD   mov        edx , CDC73124h
00654CCC    85C0            test       eax , eax
00654CCE    0FBAEA 31       bts        edx , 31h
00654CD2    F7D2            not        edx
00654CD4    F7C1 25C4A65C   test       ecx , 5CA6C425h
00654CDA    3BD0            cmp        edx , eax
```

## 2

### 修复 Stolen Code

首先修复 Stolen `OEP`, 观察栈

```
00654CDF    55              push       ebp            //1
00654CE0    EB 01           jmp        00654CE3h
...
00654CE3    8BEC            mov        ebp , esp       //2
00654CE5    EB 01           jmp        00654CE8h
...
00654CE8    6A FF           push       FFh         //3
...
00654CED    68 683FA8D9     push       D9A83F68h
00654CF2   810424 009CB326  add  dword ptr [esp] , 26B39C00h    //4 push    005BDB68
00654CF9    68 97AB43E7     push       E743AB97h
00654CFE    810424 8181FE18 add  dword ptr [esp] , 18FE8181h  //5  push    00422D18
00654D05    64:A1 00000000  mov        eax , dword ptr fs:[00000000h]  //6
00654D0E    50              push       eax           //7
00654D12    64:8925 0000000>mov        dword ptr fs:[00000000h] , esp    //8
00654D19    EB 01           jmp        00654D1Ch
...
00654D1C    83C4 A4         add        esp , A4h          //9
...
00654D22    53              push       ebx            //10
00654D23    EB 01           jmp        00654D26h
...
00654D26    56              push       esi
00654D27    EB 01           jmp        00654D2Ah        //11
...
00654D2A    57              push       edi            //12
00654D2B    EB 01           jmp        00654D2Eh
...
00654D2E    8965 E8         mov        dword ptr [ebp-18h] , esp  //13
00654D31    EB 01           jmp        00654D34h
...
00654D34  FF15 21816500   call  dword  ptr [00658121h]                  ;kernel32.GetVersion          //14
...
00654D3D    A3 A4A75E00     mov        dword ptr [005EA7A4h] , eax  //15
00654D42    EB 01           jmp        00654D45h
...
00654D45    A1 A4A75E00     mov        eax , dword ptr [005EA7A4h]  //16
00654D4A    EB 01           jmp        00654D4Dh
...
00654D4D    C1E8 08         shr        eax , 08h        //17
00654D50    EB 01           jmp        00654D53h
...
00654D53    25 FF000000     and        eax , 000000FFh    //18
00654D58    EB 01           jmp        00654D5Bh
...
00654D5B    A3 B0A75E00     mov        dword ptr [005EA7B0h] , eax    //18
00654D60    EB 01           jmp        00654D63h        
...
00654D63    8B0D A4A75E00   mov        ecx , dword ptr [005EA7A4h]    //19
00654D69    EB 01           jmp        00654D6Ch
...
00654D6C    81E1 FF000000   and        ecx , 000000FFh        //20
00654D72    EB 01           jmp        00654D75h
00654D75  - E9 D58FDCFF     jmp        0041DD4Fh      //跳到程序代码段
```

在 `0041DD00` 到 `0041DD4F` 上下搬上面代码：

二进制代码：

```
55 8B EC 6A FF 68 68 DB 5B 00 68 18 2D 42 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 C4 A4
53 56 57 89 65 E8 FF 15 21 81 65 00 A3 A4 A7 5E 00 A1 A4 A7 5E 00 C1 E8 08 25 FF 00 00 00 A3 B0
A7 5E 00 8B 0D A4 A7 5E 00 81 E1 FF 00 00 00
```

修复后的 `oep`:

```
0041DD00    55              push       ebp
0041DD01    8BEC            mov        ebp , esp
0041DD03    6A FF           push       FFh
0041DD05    68 68DB5B00     push       005BDB68h
0041DD0A    68 182D4200     push       00422D18h
0041DD0F    64:A1 00000000  mov        eax , dword ptr fs:[00000000h]
0041DD15    50              push       eax
0041DD16    64:8925 0000000>mov        dword ptr fs:[00000000h] , esp
0041DD1D    83C4 A4         add        esp , A4h
0041DD20    53              push       ebx
0041DD21    56              push       esi
0041DD22    57              push       edi
0041DD23    8965 E8         mov        dword ptr [ebp-18h] , esp
```
