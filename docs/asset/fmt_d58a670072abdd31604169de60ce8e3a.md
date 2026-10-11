
# 反一段代码发现这段汇编，源代码有分支，而汇编看不到分支

> 原文：[`www.pediy.com/kssd/pediy10/63291.html`](https://www.pediy.com/kssd/pediy10/63291.html)

```
mov     eax, [edi+60h]  

mov     esi, [eax+8]  -----得一个值   

sub     esi, 14h       -----去掉 0x14 结果是 0 或 其他 ( >=0x14 CF 为 0  <0x14 CF 为 1) 

neg     esi            ----- 补 得 0x00000000 或 其他 

sbb     esi, esi       -----esi = esi - esi -cf   得到是(cf 为 0 时 是 0x00000000 为 1 时 是 0xffffffff ) 

and     esi, 0C000000Dh   -----& 结果是 0x00000000 或者 0x0C000000D
```

```
if ((IoGetCurrentIrpStackLocation(Irp)->Parameters.SetFile.FileInformationClass >=FileEndOfFileInformation))  

     {  

        Status = STATUS_SUCCESS;//0x00000000  

     }  

     else  

     {  

        Status = STATUS_INVALID_PARAMETER;//0x0C000000D  

     }
```

### 总结

条件判断语句 `if ( == )`常用 `sub`, `neg`, `sbb` 的组合，后续指令视情况而定。

如果是 `if ( == ) { = 0; } else { = !0; }`，后续指令可以是 `and`

感谢分享

`VC` 优化会有这样的代码生成。

如：

代码：

```
   if(FindWindow(NULL,"计算器"))
    return 1;
   else   
    return 5;
```

优化编译后：

代码：

```
00401000  push    406030                  ;/Title = "计算器"
00401005  push    0                       ;|Class = 0
00401007  call    dword ptr [40509C]      ;\\FindWindowA
0040100D  neg     eax
0040100F  sbb     eax, eax
00401011  and     al, 0FC
00401013  add     eax, 5
00401016  retn
```
