
# 如何给 DLL 文件重定位加 MessageBoxA 显示框

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-93.htm`](https://www.pediy.com/kssd/pediy09/pediy09-93.htm)

> `.exe` 文件加显示框是很容易的事,但 `DLL` 呢,存在重定位的问题!
>
> 某一 `DLL`,如其运行中显示 `EBX` 的值为:`0123456`;查看其输入表,有 `MessageBoxA` 函数,我希望用这个函数将 `EBX` 的值自动显示出来,于是改造文件为:

```
29912345   jmp 30012345
...
29912355   xor eax,eax
...

30012345   pushad
           push 0
           push 0
           push ebx
           push 0
           call [312f10]       ;地址[312f10]为 MessageBoxA 在 IAT 表中的位置
           popad
           jmp 29912355
```

> 程序在调试过程中能正常显示 `EBX` 的值,但由于 `dll` 每次在内存中加载不同的基地址,因此,调试的当时是可以的,但保存后再重启程序就直接报错了,主要是 `29912345`,及 `CALL [312f10]` 处不再是我修改的地址了,因此报错,请问各位大侠,如何让程序在修改,保存并重启后程序后仍然能指向正确的地址?

```
          call  @F
          @@:
          pop  ebx
          sub  ebx,offset @B
          pushad
          push 0
          push 0
          push eax
          push 0
          call [ebx + 312f10]
          popad
```

```
          call  @F
          @@:
          pop  ebx
          sub  ebx,offset @B
```

> 是经典的解决重定位代码
>
> 这样写

```
30012345   pushad
           call 3001234B
           pop eax
           sub eax, 3001234B
           add eax, 312f10
           push 0
           push 0
           push ebx
           push 0
           call dwword ptr [eax]
           popad
           jmp 29912355
```

`4dd466d0e1e0cc123afbe504da2462ca`
