

# FlexNet 10.8 20 位 Sign 揭秘

> 原文：[`www.pediy.com/kssd/pediy10/99373.html`](https://www.pediy.com/kssd/pediy10/99373.html)

软件名称：保密  
加密方式:FlexNet 10.8.50  
工具：od，ida  
  
本文适用于目前所有的 Flexlm7.2 以上版本  
Flexlm(现改名为 FlexNet) 10.8 生成的 license 文件，如果使用了 ECC，就是 20 位。  
例如：  
FEATURE sample EXAMPLE 10.0 30-jan-2010 uncounted \\  
  8D3AE17F5FA71E6B86AD VENDOR\_STRING=Demo \\  
  HOSTID=0015287ec8ea ck=16   
sign 为 8D3AE17F5FA71E6B86AD  
       ^ ^ ^ ^  
请注意看，下面带^号的四个数字 DA1F（也就是第 2、4、6、8 位数）实际上是个时间，具体是什么时间下面具体分析。  
剩余的 16 位数 83E75FA71E6B86AD 这是真正的 Sign。由此产生了 2 个问题：  
1 如何算出 16 位 sign  
2 另外 4 位数是什么，起什么作用。  
一、16 位 sign 的计算  
其实很简单，先找出该程序的 Flexlm 版本号，用 lmtools.exe 可以找出来，或者“lmutil.exe lmver 目标 exe.exe”。  
然后用 ida 反编译该 exe，用对应版本的 sig 文件找出所有的 flexlm 函数。找到\_l\_string\_key 函数，这里面是对  
伪 sign 和真实的 sign 进行比对的地方。btw，请用 od 调试跟踪，在\_l\_string\_key 出设断，具体的我就不举例了。  
论坛上应该很多例子，大家自己练练手。如果\_l\_string\_key 里面有自定义算法，那要自己分析，再逆向出来。  
如果用 ida 找不到\_l\_string\_key 函数，还有一个办法，用 od 搜索 55 8B EC 81 EC DC 01 00 00 53，会找到 2 个  
函数，一个是\_l\_string\_key（一般找到的第一个就是），另一个是\_xuA00pGqD7WK\_0。  
二、另外 4 位 sign，DA1F 是什么  
这是个 starting date（开始日期），也就是你的 license 的开始日期。我碰到的这个软件（版权问题，名称保密）  
里面这个非常重要。我找到了这个算法，请看下面的机器代码：  
00781CA4    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  ;\[EBP-C\]=0000DA1F  
00781CA7    C1F8 09         SAR EAX,9        ;eax=6d  
00781CAA    83E0 7F         AND EAX,7F  
00781CAD    8945 F8         MOV DWORD PTR SS:\[EBP-8\],EAX  
00781CB0  |.  837D F8 63    CMP DWORD PTR SS:\[EBP-8\],63  
00781CB4  |.  7E 0C         JLE SHORT Sam.00781CC2  
00781CB6  |.  8B4D F8       MOV ECX,DWORD PTR SS:\[EBP-8\]  
00781CB9  |.  81C1 6C070000 ADD ECX,76C        ;eax=76c+6d=7d9=2009 年  
00781CBF  |.  894D F8       MOV DWORD PTR SS:\[EBP-8\],ECX  
  
  
00781CC2    8B55 F4         MOV EDX,DWORD PTR SS:\[EBP-C\]  ;\[EBP-C\]=0000DA1f  
00781CC5    C1FA 05         SAR EDX,5  
00781CC8    83E2 0F         AND EDX,0F        ;edx=month-1  
  
  
  
00781CE7    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]  ;\[EBP-C\]=0000DA1f  
00781CEA    83E0 1F         AND EAX,1F        ;=day  
  
是不是很简单？  
总结一下：  
年=(DA1F >> 9 & 7F )+76c  
月=DA1F >> 5 & 0F+1  (加 1 的原因里 Flexlm 把 1 月当成 0 月，以此类推)  
日=DA1F & 1F  
  
因此 DA1F 就是 2009 年 1 月 31 日。  
反过来，当我们要做 license 的时候，就需要自己设定起始日期，然后转换成 Flexlm 的日期数，  
公式为：日期数=日+(月份-1)\*0x20+(年-0x76c)\*0x200  
  
收工。