
我们注意到`static`块里调用了`System.load`来加载`foo`库（参看【1】）。这个`app`还在`OnCreate`函数里的第一行就调用了`this.init()`，而这个函数被声明为`native`函数（参看【2】），所以它应该是`foo`的一部分。

让我们来看看这个`foo`库。在`radare2`中打开这个库（你会在`lib`文件夹里看到几个不同的架构，我这里用的是`lib/x86_64`），分析它并列出它的输出。

```
michael@sixtyseven:~/Development/UnCrackable2/lib/x86_64$ r2 libfoo.so 

 -- Don't look at the code. Don't look.

[0x000007a0]> aaa

[x] Analyze all flags starting with sym. and entry0 (aa)

[x] Analyze len bytes of instructions for references (aar)

[x] Analyze function calls (aac)

[ ] [*] Use -AA or aaaa to perform additional experimental analysis.

[x] Constructing a function name for fcn.* and sym.func.* functions (aan))

[0x000007a0]> iE

[Exports]

vaddr=0x00001060 paddr=0x00001060 ord=004 fwd=NONE sz=183 bind=GLOBAL type=FUNC name=Java_sg_vantagepoint_uncrackable2_CodeCheck_bar

vaddr=0x00001050 paddr=0x00001050 ord=006 fwd=NONE sz=15 bind=GLOBAL type=FUNC name=Java_sg_vantagepoint_uncrackable2_MainActivity_init

vaddr=0x00004008 paddr=0x00003008 ord=014 fwd=NONE sz=0 bind=GLOBAL type=NOTYPE name=__bss_start

vaddr=0x00004008 paddr=0x00003008 ord=015 fwd=NONE sz=0 bind=GLOBAL type=NOTYPE name=__bss_start

vaddr=0x0000400d paddr=0x0000400d ord=016 fwd=NONE sz=0 bind=GLOBAL type=NOTYPE name=_end

5 exports

[0x000007a0]>
```

我们看到这个库导出了两个很有意思的函数：`Java_sg_vantagepoint_uncrackable2_MainActivity_init` 和 `Java_sg_vantagepoint_uncrackable2_CodeCheck_bar`（关于这些函数的命名，请查看 [Java nativ interface JNI](https://bbs.kanxue.com/elink@314K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3L8X3c8C8i4K6u0r3M7$3q4E0M7r3I4W2M7#2)9J5c8Y4y4S2L8i4m8D9k6g2)9#2k6X3S2W2L8r3I4G2K9X3&6A6i4K6u0W2K9s2c8E0L8l9`.`.)），我们要看的是：

`Java_sg_vantagepoint_uncrackable2_MainActivity_init`

```
[0x000007a0]> s 0x00001050

[0x00001050]> V
```

这是一个蛮短的函数：

```
[0x00001050 29% 848 libfoo.so]> pd $r @ sym.Java_sg_vantagepoint_uncrackable2_MainActivity_init                                                                                                     

/ (fcn) sym.Java_sg_vantagepoint_uncrackable2_MainActivity_init 15                                                                                                                                  

|   sym.Java_sg_vantagepoint_uncrackable2_MainActivity_init ();                                                                                                                                     

|           0x00001050      50             push rax                                                                                                                                                 

|           0x00001051      e8caf7ffff     call sub.fork_820           ;[1]                                                                                                                         

|           0x00001056      c605af2f0000.  mov byte [0x0000400c], 1    ; [0x400c:1]=58 ; ": (GNU) 4.9.x 20150123 (prerelease)"                                                                      

|           0x0000105d      58             pop rax                                                                                                                                                  

\           0x0000105e      c3             ret                                                                                                                                                      

            0x0000105f      90             nop
```

它调用了另一个函数`sub.fork_820`，这个要做的事就比较多了：
