

# VMProtect SDK+ASProtect SDK 混合编程\[代码演示\]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-527.htm`](https://www.pediy.com/kssd/pediy08/pediy8-527.htm)

刚看到 VMProtect 这个东东的时候真的不知道如何去使用  
就用了最笨的方法...OD 跟踪～跟到保护的地方然后添加地址^\_^～～  
  
说正题好了  
程序具体保护实例  
VMProtect SDK+ASProtect SDK 保护  
工具:  
VMProtect 1.12  
ASProtect 1.33\[Cr\]  
相关工具可以到 www.pediy.com 下载到  
  
具体保护例子:  
简单的字符串判断例子,附件下载  
内部使用  
VMProtect SDK 将验证代码保护起来  
外部使用 ASProtect 保护  
具体实例看代码  
  
关于使用 VMProtect 的保护代码的一点说明  
想简单的确定自身保护位置可以使用两种方法  
以 Delphi 为例子  
1.  
在工程编译选项中:  
Project->Options->Linker  
选项卡中的 Map File 选项中～将单选框选择 Detailed  
然后在 VMProtect 添加保护代码中会看到函数地址  
2.就是直接使用 SDK 代码  
asm                              //使用 VMProtect 的 SDK  
  db $EB,$10,'VMProtect begin',0  
end;  
  
asm  
  db $EB,$0E,'VMProtect end',0  
end;  
//---the end---  
VMProtect 使用说明\[俄文版\]  
http://www.polytech.ural.ru/vmprotect.htm  
  
附件中有全部代码和 SDK:  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> program Test;
> {$APPTYPE CONSOLE}
> uses
>   Windows;
> 
> function AGenKey:string;      //数组动态生成免得别别人从内存中看出来
> const
>   Aint : array\[0..9\] of Char = ('0', '1', '2', '3', '4', '5', '6', '7', '8', '9');
>   Achr  : array\[0..25\] of Char = ('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J',
>                                   'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T',
>                                   'U', 'V', 'W', 'X', 'Y', 'Z');
> begin
>   {key=H8E8A4R8T}
>   asm                              //使用 VMProtect 的 SDK
>     db $EB,$10,'VMProtect begin',0
>   end;
>   Result := Concat(Achr\[7\],Aint\[8\],Achr\[4\],Aint\[8\],Achr\[0\],Aint\[4\],Achr\[17\],Aint\[8\],Achr\[19\]);
>   asm
>     db $EB,$0E,'VMProtect end',0
>   end;
> end;
> 
> var
>   StrSN:String;
> begin
>   Writeln('//////////////////////////////////////////');
>   Writeln('//    VMProtect SDK+ASProtect SDK Example');
>   Writeln('//    Coded By:Anskya');
>   Writeln('//    Email:Anskya@Gmail.com');
>   Writeln('//    Web:Www.Anskya.Net');
>   Writeln('//////////////////////////////////////////');
>   Writeln('');
>   Writeln('');
> 
>   Write('输入注册码:');
>   Readln(StrSN);
> 
>   {$I reg\_crypt\_begin.inc}         //使用 ASProtect 的 SDK
>   asm                              //使用 VMProtect 的 SDK
>     db $EB,$10,'VMProtect begin',0
>   end;
> 
>   if StrSN=AGenKey then
>   begin
>     Writeln('恭喜注册成功');
>   end else
>   begin
>     Writeln('^\_^注册失败!');
>   end;
> 
>   asm
>     db $EB,$0E,'VMProtect end',0
>   end;
>   {$I reg\_crypt\_end.inc}
> 
>   Readln;
> end.
> 
> * * *
> 
> 
> ```
> 
>     

 4dd466d0e1e0cc123afbe504da2462ca