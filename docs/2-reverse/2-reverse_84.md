

# CmdBar.dll BugFix

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-243.htm`](https://www.pediy.com/kssd/pediy09/pediy09-243.htm)

> ollydbg 的 CmdBar 插件中的 mac 宏执行命令一直都觉得是一个非常好用的功能。  
>   
> 该 Bug 是指，有些情况下明明我们已经在 macro.def 中定义了命令，可是 cmdbar 还是提示无效的宏命令。  
>   
> 看代码;  
> 
> 引用: int LoadMac(char \*answer,ulong parm) {  
>   const char \*nomore = "NO MORE";  
>   int  i,j;  
>   BOOL IsMacName;  
>   char key\[10\],line\[TEXTLEN\],buf\[TEXTLEN\];  
>   char \*pdest;  
>   PCOMMAND\_QUE pnew,p;  
>   
>   pdest = MacroNames;  
>   IsMacName = FALSE;  
>   while(\*pdest != '\\0') {  
>     if(!stricmp(string,pdest)) {  
>       IsMacName = TRUE;  
>       break;  
>     }  
>     pdest += lstrlen(MacroNames)+1;   //这里应该为 pdest += lstrlen(pdest)+1;   
>   }  
>   if(IsMacName == FALSE) {  
>     wsprintf(answer,"\\"%s\\" is invalid macro name!!",string);  
>     return(-1);  
>   } 
> 
>   
>   
> 
> 引用: 02B92A95    |> /53                  /PUSH EBX                               ; /Arg2 = 00000000  
> 02B92A96    |. |68 440BBA02         |PUSH CmdBar.02BA0B44                   ; |Arg1 = 02BA0B44 ASCII "dialog"  
> 02B92A9B    |. |E8 44230000         |CALL CmdBar.02B94DE4                   ; \\CmdBar.02B94DE4  
> 02B92AA0    |. |83C4 08             |ADD ESP, 8  
> 02B92AA3    |. |85C0                |TEST EAX, EAX  
> 02B92AA5    |. |75 07               |JNZ SHORT CmdBar.02B92AAE  
> 02B92AA7    |. |BE 01000000         |MOV ESI, 1  
> 02B92AAC    |. |EB 13               |JMP SHORT CmdBar.02B92AC1  
> 02B92AAE    |> |A1 A4E1B902         |MOV EAX, DWORD PTR DS:\[2B9E1A4\]  
> 02B92AB3        50                  PUSH EAX        //这里修改为 EBX  
> 02B92AB4    |. |E8 359A0000         |CALL <JMP.&KERNEL32.lstrlenA>          ; \\lstrlenA  
> 02B92AB9    |. |40                  |INC EAX  
> 02B92ABA    |. |03D8                |ADD EBX, EAX  
> 02B92ABC    |> |803B 00              CMP BYTE PTR DS:\[EBX\], 0  
> 02B92ABF    |.^\\75 D4               \\JNZ SHORT CmdBar.02B92A95 
> 
> cmdbar 有源代码，这里就不编译了，直接放一个修改过的二进制文件。 

 4dd466d0e1e0cc123afbe504da2462ca