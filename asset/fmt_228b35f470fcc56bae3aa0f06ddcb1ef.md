
我们的方法是让第一句`Licence.? = new Licence.?( );`执行完后，不进行任何判断，直接执行上面的两句代码。对了，前面还有一处，就是`RSA`判断处的`provider1.VerifyData`也要改掉，这样才能进行到下面。

修改方法，在结合`ILDASM`的反汇编代码，在`UltraEdit`中定位。

### 第一处

```
IL_00a4:  /* 2D   | 0E                 */ brtrue.s   IL_00b4
```

将`2D`改为`2C`（`brfalse.s`）。物理偏移在`0x3118h`处。

### 第二处

```
  IL_0000:  /* 73   | (06)00003B        */ newobj     instance void RedGate.Licensing.Client.Licence/*02000008*//'?'/*02000009*/::.ctor() /* 0600003B */
  IL_0005:  /* 0A   |                   */ stloc.0
```

在物理偏移的`0x2cb4`处。紧接着这两句修改。原文件为

```
00002cbah: 02 03 12 00 28 29 00 00                    ; ....()..
```

改为

```
00002cbah: 02 17 7D 38 00 00 04 17                    ; ..}8....
```

即`MSIL`代码

```
ldarg.0
ldc.i4.1
stfld      bool RedGate.Licensing.Client.Licence::'?'
ret
```

保存修改，运行。按格式输入注册码和激活信息，显示激活成功，不再提示有使用时间了。序列号生成器偶就不写了，很简单的算法。不知道什么时候才能搞定`RSA`。

附件里是`patch`过的`RedGate.Licensing.Client.DLL`，仅供参考！

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 08 月 24 日 0:39:39

`4dd466d0e1e0cc123afbe504da2462ca`
