

# TRW2000 1.06

> 原文：[`www.pediy.com/kssd/pediy01/forum011.htm`](https://www.pediy.com/kssd/pediy01/forum011.htm)

解除 TRW2000 1.06 的时间限制   
工具：   
Procdump1.6.2   
Hiew6.1   
W32dasm8.93   
  
  首先用 W32dasm 载入 TRW2000.EXE，但将告诉你“W32dasm 执行了非法程序”，它崩溃了。   
  在 Procdump 中选择 PEeditor\\指定文件 TRW2000.EXE\\SECTIONS, 你会发现 .data 的 characteristics 值为 C0000040,将之改为 60000020,保存后再用 W32dasm 载入,OK!   
  反汇编后选 refs/string data references,找到"The vention is too old, please",双击后见到如下：   
\* Possible StringData Ref from Data Obj ->"This version is too old , please "   
                                        ->"visit http://trw2000.yeah.net "   
                                        ->"to get update."   
                                  |   
:004018CD 682CF14100              push 0041F12C   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004016CA(U)   
|   
:004018D2 53                      push ebx   
  
向上找见   
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040164D(C), :00401675(C)   
  ~~~~~~~~      ~~~~~~~~   
|   
:004018C6 6A10                    push 00000010   
  
\* Possible StringData Ref from Data Obj ->"ERROR"   
                                  |   
:004018C8 6828F24100              push 0041F228           
  
就到:0040164D(C)及:00401675(C)   
:00401643 056C070000              add eax, 0000076C   
:00401648 3DD0070000              cmp eax, 000007D0  <----1   
:0040164D 0F8F73020000            jg 004018C6        \*\*\*2   
:00401653 53                      push ebx   
:00401654 8D4DE8                  lea ecx, dword ptr \[ebp\-18\]   
:00401657 E8AFDC0000              call 0040F30B   
:0040165C 8B4010                  mov eax, dword ptr \[eax+10\]   
:0040165F 40                      inc eax   
:00401660 83F802                  cmp eax, 00000002  <----3   
:00401663 7E16                    jle 0040167B        \*\*\*4   
  
猜到了吗?   
1 :00401648 3DD0070000              cmp eax, 000007D0  比较年份--2000 年(000007D0)   
2 :0040164D 0F8F73020000            jg 004018C6        若大于 2000 年则过期   
3 :00401660 83F802                  cmp eax, 00000002  比较月份--2 月(00000002)   
4 :00401663 7E16                    jle 0040167B      若不大于 2 月就跳过前面的过期提示   
  
知道如何作了?   
1 改 3DD0070000   
  为 3D34080000  可以用到 2100 年(那时可就干不动这活喽!)   
2 改 7E16   
  为 EB16        直接跳到开始   
    
完成了吗?运行 TRW2000,它告诉你执行了非法程序。WHY？别忘了我们前面用过 Procdump，只要用它将 TRW2000.EXE\\SECTIONS\\ .data 的 characteristics 值由 60000020 改回 C0000040 ,保存后运行 TRW2000,OK!   
  
附: 1 TRW2000 确实是一个好程序,不象 TRW0.75 版本动不动就死机。目前它的升级非常快。劝各位还是尽量用最新的版本吧，不要象我没事找使干。写这篇文章的目的是告诉大家：如何改动程序来使 W32dasm 工作。   
    2 TRW2000 版的注册费要六、七十元，我不愿花这么多钱来注册。谁有注册版本，能否 E-MAIL 我，在此先行谢过。   
                                  孙悟空之七十二变