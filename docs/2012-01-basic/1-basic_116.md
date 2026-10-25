
# 编写自己的反汇编引擎之用法举例

> 原文：[`www.pediy.com/kssd/pediy12/128431.html`](https://www.pediy.com/kssd/pediy12/128431.html)

首先声明本帖首发于赏金论坛，希望大家能多多关注这个正在成长的论坛 www.sgoldcn.com
本来打算接着帮大家分解我的代码，但暂时先停一下，给大家举个例子，看看如何使用我的反汇编引擎。大家不应该有个误区说，反引擎只是为了把代码翻译出来给人看，其实他还有更多的用途，比如反病毒引擎、虚拟机等都需要有自己的反汇编引擎，他们把程序的代码翻译出来后都是给机器看的，而不是给人来看，拿例子说话吧。

给大家演示的是一个抓病毒变种的方法（我会不会不小心把免杀方法给透露了？呵呵，没关系，兵来将挡，水来土掩）。

给大家提供几个病毒变种，记住，不要在真实机器上玩火。

![](img/819a2483f2bffaae8bec361d109f1dab.jpg)

可以看到这些样本都是同一个病毒的变种，他们的大小以及代码都有所不同。但是要知道，既然是变种，他们的代码肯定是会有相同的部分，那好，哪里相同我们就检测哪里。其实这些病毒变种都是 DLL 文件，我这里改成 EXE 是为了给大家看一下他们有很多图标是一样的，至少可以在视觉上让大家相信他们确实是同一病毒。

直接 OD 载入，来到 OEP:

![](img/80ccabd6e2bd4cf6a123f8b70717cd4b.jpg)

来到这个 call 或者 jmp 处:

![](img/6da65035bae1c664dc295fd0b8399a39.jpg)

![](img/2de923418051e36e49dcc515aca83b14.jpg)

那好，知道了上面这些特征，我们不必再往下看就可以想到一个检测此种病毒变种的方法了。

1. 检测 OEP 处的第一条指令是否为 `cmp dword ptr[esp+8],1`，有则跟进，无则退出检测，说明这个检测的文件不是我们要找的病毒变种。
2. 检测从 OEP 开始处到 50 个字节内是否有 `call` 或 `jmp`，有则跟进，无则退出检测。
3. 跟进之后检测在 100 个字节内是否有 `MOV disp32,reg/Ib/Iz`，有则进行统计。
4. 检查统计数据，大于 20 个垃圾代码则说明这就是我们要找的变种，杀之。

一百个人可能会想出一百种办法来检测这个病毒，我只是提供了一种思路。好了，现在来用我的引擎查杀此种病毒变种。

/========================================================================================================================================

```
                            INSTRUCTION Instruction={0};  
                            int iOpcodeLen=0;  
                            DWORD dwCount89C6C7 = 0;  
                            DWORD JmpOrCallAddr = 0;  
                            iOpcodeLen = Dissassemble((BYTE\*)OepVa,&Instruction);   //dissassemble  
                            //OEP 处的第一行指令必须是 : cmp dword ptr \[esp+8\], 1  
                            //这里也可以使用 Instruction. InstructionBuf\[\]来直接比较，更方便  
                            if (Instruction.Opcode1 == 0x83 && Instruction.Modrm == 0x7C && Instruction.SIB == 0x24 && Instruction.Displacement.DispByte == 0x08 && Instruction.Immediate.ImmByte == 0x01)        //837C24 08 01    cmp     dword ptr \[esp+8\], 1  
                            {  
                                     for (int i = 0;i <= 50; i++)  //oep --> oep + 50  
                                     {  
                                               OepVa+= iOpcodeLen;  
                                               iOpcodeLen = Dissassemble((BYTE\*)OepVa,&Instruction);   //dissassemble  
                                               if ((Instruction.Opcode1 == 0x89 || Instruction.Opcode1 == 0xC6 || Instruction.Opcode1 == 0xC7)&& Instruction.Modrm & 0x05)    // rm == 101,mov disp32,reg/Ib/Iz  
                                               {  
                                                        dwCount89C6C7++;  
                                               }  
                                               if (Instruction.Opcode1 == 0xE8 || Instruction.Opcode1 == 0xE9) //call/jmp xxxxxxxx  
                                               {  
                                                        OepVa += iOpcodeLen;  
                                                        JmpOrCallAddr = OepVa + Instruction.Immediate.ImmDword; //跳到 call/jmp  
                                                        for (int i = 0;i <= 100; i++)  
                                                        {  
                                                                 iOpcodeLen = Dissassemble((BYTE\*)JmpOrCallAddr,&Instruction);      //dissassemble  
                                                                 if ((Instruction.Opcode1 == 0x89 || Instruction.Opcode1 == 0xC6 || Instruction.Opcode1 == 0xC7)&& Instruction.Modrm & 0x05)     // rm == 101,mov disp32,reg/Ib/Iz  
                                                                 {  
                                                                           dwCount89C6C7++;  
                                                                 }  
                                                                 if (dwCount89C6C7 >= 20)//大于 20 说明是变种  
                                                                 {  
                                                                           printf("%s Detectedn",FileName);  
                                                                           dwCountDetectedFiles++;  
                                                                           goto \_FindNext;  
                                                                 }  
                                                                 JmpOrCallAddr += iOpcodeLen;  
                                                        }  
                                               }  
                                     }  
                            }  
                            else  //OEP 处指令不是: cmp dword ptr \[esp+8\],1，则检查下一文件  
                            {  
                                     goto \_FindNext;  
                            }
```

//========================================================================================================================================

来看看最终效果:

![](img/0b573d9c81dc45f02e760af5b3be067b.jpg)

在此仅举一例，引擎的更多用途还待大家发掘。

病毒样本.rar (348 K)

Test_Detect_Virus.rar (2212 K)

您正在看的文章来自赏金论坛 http://www.sgoldcn.com,原文地址:http://www.sgoldcn.com/read.php?tid=922.如有转载,请误删除此信息
