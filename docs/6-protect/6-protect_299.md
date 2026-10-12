
# 我的 VC 花指令方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-225.htm`](https://www.pediy.com/kssd/pediy07/pediy7-225.htm)

在 VC 加入花指令，我在几年前就尝试使用了，一点心得，大家交流：

-----------------------------------------------------------

1. 首先，定义各种花指令的宏，可以单独放到一个 `.h` 文件中，也可以直接在程序的首部定义，我建议放到 `.h` 文件中。

   注意，所有转移标号和 `call` 的地址用相对地址符号 `$` 表示，以楼上的问题为例，格式如下：

   ```c
   #define _WJQ_USED_FLOWER //此开关控制编译器是否使用花指令

   //
   //花指令定义(定义多少种，自己需要而定)
   //

   #ifndef _WJQ_USED_FLOWER
   #define __FLOWER_XX0 _asm nop
   #else
   #define __FLOWER_XX0 _asm \
   {\
   _asm jz $+5 /*本指令 2 bytes len*/\
   _asm jnz $+3 /*本指令 2 bytes len*/\
   _asm _emit 0e8h /*本指令 1 bytes len*/\
   /*这里是要转移到的地址*/\
   }
   #endif
   ```

2. 根据上述格式，编辑好各种花指令，例如我编辑的 10 种为：
   - `__FLOWER_XX0`
   - `__FLOWER_XX1`
   - `__FLOWER_XX2`
   - ...
   - `__FLOWER_XX9`

3. 在 VC 中的使用方法：

   ```c
   int WJQ_Function(char* pStr, DWORD nIndex)
   {
       __FLOWER_XX0
       char buffer[200];
       __FLOWER_XX9
       /*
        buffer 的赋值部分（略）
       */
       __FLOWER_XX7
       for (long i=0; i<100; i++)
       {
           __FLOWER_XX2
           pStr[i] = buffer[i];
           __FLOWER_XX5
       }
       __FLOWER_XX4
       return TRUE;
       __FLOWER_XX6
   }
   ```

   以上程序中，仅仅通过是否定义了 `_WJQ_USED_FLOWER` 来决定是否插入花指令。将 `#define _WJQ_USED_FLOWER` 注释掉，就可以无花指令调试程序，调试成功后，加入宏定义即可加入花指令。

   Spring.W
   2005.3.20

> 引用:
>
> 言之有理啊,我试了下用 `asm db $74,$04,$75,$02,$e8,$03 end;` 这样的嵌入方式可以实现简单的花,可还没有尝试出复杂些的,难道只能靠数量吗?

在 Delphi 中需要花的地方插入一段 `code`，而且这段 `code` 可以任意变形：

```delphi
代码:

asm
  jz @
  jnz @    
  db $e8  //这里到@之间可以插入任和混淆视觉的代码
@:
end;
```

实践一下，你会得到很多不同花指令组合。![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

4dd466d0e1e0cc123afbe504da2462ca
