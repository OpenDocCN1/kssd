

# 用于将反汇编代码分段的 Ida 脚本

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-98.htm`](https://www.pediy.com/kssd/pediy09/pediy09-98.htm)

> 当某个函数的逻辑关系比较复杂时，代码很不好看，所以写了这个脚本来辅助进行人工分析。  
> 下一步会尝试写一个程序来分析这个脚本的输出文件，用来减轻人工分析的工作量。  
>   
> 代码主要作用：  
>    将某个模块的代码以跳转指令作为分界线，将代码分成若干代码段，  
> 然后将代码段的信息输出到文件，并在给每个代码段的开始给代码段  
> 赋予一个 Part%d 的名称，有利于对代码进行人工分析。  
>   
>   
> #include <idc.idc>  
>   
> // Author : HenryShow.  
> // Version : V0.1  
> static main()  
> {  
>   // Declaration of vars  
>   auto CodeStart, CodeEnd, nIndex; // 分析的地址。  
>   auto PartInAddress, PartOutAddress1, PartOutAddress2; // 保存代码段的入口地址，以及出口地址  
>   auto nPartCount;  // 保存模块中代码段的个数  
>   auto Instruction, Operand1; // 保存分析出来的命令  
>   auto handle; // 文件句柄  
>   auto TmpString;  // 存放类似于 Part%d 的临时字串。  
>     
>   // 打开文件，用来写入函数中代码段的信息。  
>   handle=fopen("Output.txt","wt");  
>   // 得到当前光标所在的地址作为开始地址  
>   CodeStart=ScreenEA();  
>   // 让用户手动输入结束地址  
>   CodeEnd = AskAddr(0,"Please insert ending address:");  
>   // 如果结束地址比开始地址还小，则结束脚本。  
>    if (CodeEnd <= CodeStart){  
>      fclose(handle);  
>      return;  
>    }  
>      
>    // 代码段个数清零  
>   nPartCount = 0;   
>     
>   nIndex = CodeStart;  
>     
>   // 为代码段的入口地址设置初始值  
>   // 第一段的入口地址就是脚本分析的初地址  
>   PartInAddress = nIndex;  
>     
>   PartOutAddress1 = nIndex;  
>   PartOutAddress2 = nIndex;  
>     
>   // 开始循环判断  
>   while (nIndex < CodeEnd){  
>     if (nIndex == PartInAddress){  
>       // 如果当前地址是某个代码段的入口地址，则在该地址添加 Part%d 的注释，以便于根据人工分析。  
>       TmpString = "Part" + ltoa(nPartCount+1, 10);  
>       MakeComm(nIndex, TmpString);  
>     }  
>     else{  
>       // 否则将注释清除。  
>       // 如果不需要清除，则将此行注释掉。  
>       MakeComm(nIndex, "");  
>     }  
>       
>     // 得到当前分析地址的指令字符串。  
>     Instruction = GetMnem(nIndex);  
>       
>     if (substr(Instruction, 0, 1) == "j"){  
>         // 如果指令是一个以 j 开头的指令，则为跳转指令。  
>         // 那么指令的第一个操作数就是出口地址 1。  
>       fprintf(handle, "%d ", nPartCount);  
>       fprintf(handle, "%.8X ", PartInAddress);  
>       // 得到操作数对应的地址。  
>       PartOutAddress1 = LocByName(GetOpnd(nIndex, 0));  
>       if (PartOutAddress1 == -1){  
>         // 取不到地址说明为$+\*类型的操作数。  
>         Operand1 = GetOpnd(nIndex, 0);  
>         if (substr(Operand1, 0, 1) == "$"){  
>           if (substr(Operand1, 1, 2) == "+"){  
>             // $+\*  
>             PartOutAddress1 = nIndex + atol(substr(Operand1, 2, -1));  
>           }  
>           else{  
>             //$-\*  
>             PartOutAddress1 = nIndex - atol(substr(Operand1, 2, -1));  
>           }  
>         }  
>       }  
>       fprintf(handle, "%.8X ", PartOutAddress1);  
>         
>       if (substr(Instruction, 0, 3) != "jmp"){  
>         // 如果不是 jmp 指令，下一条指令的地址就是出口地址 2。  
>         fprintf(handle, "%.8X", nIndex + ItemSize(nIndex));  
>       }  
>       else{  
>         // 如果为 jmp 指令，则只有出口地址 1 有效。  
>         fprintf(handle, "%.8X", 0);  
>       }  
>       fprintf(handle, "\\n");  
>       // 下一个代码段的入口地址就是下一条需要分析的指令的地址。  
>       PartInAddress = nIndex + ItemSize(nIndex);  
>       nPartCount++;  
>     }  
>     else if (Name(nIndex + ItemSize(nIndex)) != ""){  
>       // 如果下一行有一个标号，那么从此地址开始创建下一个代码段。  
>       // 并且将上一个代码段的出口地址 1 设置为当前地址，而出口地址 2 为无效地址。  
>       fprintf(handle, "%d ", nPartCount);  
>       fprintf(handle, "%.8X ", PartInAddress);  
>       fprintf(handle, "%.8X ", nIndex + ItemSize(nIndex));  
>       fprintf(handle, "%.8X", 0);  
>       fprintf(handle, "\\n");  
>       PartInAddress = nIndex + ItemSize(nIndex);  
>       nPartCount++;  
>     }  
>         
>     // 继续分析下一条指令。  
>     nIndex = nIndex + ItemSize(nIndex);  
>   }  
>     
>   fprintf(handle, "%d ", nPartCount);  
>   fprintf(handle, "%.8X ", PartInAddress);  
>   fprintf(handle, "%.8X ", 0);  
>   fprintf(handle, "%.8X", 0);  
>   fprintf(handle, "\\n");  
>     
>    
>   // 关闭文件句柄。  
>   fclose(handle);  
>   return;  
> } 

 4dd466d0e1e0cc123afbe504da2462ca