
```
ADD EAX,EAX  
SUB EAX,ECX { this will always clear the Z-flag ! }  
MOV ESI,\[EDI+EAX\*2-4\]  

@@exit:  
POP EDI  
end;  
```

大家再看看上面的注册验证代码中的几行代码:

--------------------------------------------------------------------------------------------------------------

```
005189D5 >  8B83 04030000   mov eax,dword ptr ds:\[ebx+304\]         ; \*Tfrmreg.btnget:TBitBtn  
005189DB    66:BE EBFF      mov si,0FFEB  
005189DF >  E8 0CB6EEFF     call MusicPoi.00403FF0                   ; ->System.@CallDynaInst  
```

跟进 `MusicPoi.00403FF0` (`E8 0CB6EEFF call MusicPoi.00403FF0`)

--------------------------------------------------------------------------------------------------------------

```
00403FF0    50              push eax  
00403FF1    51              push ecx  
00403FF2    8B00            mov eax,dword ptr ds:\[eax\]  
00403FF4    E8 C7FFFFFF     call MusicPoi.00403FC0        //这就是 GetDynaMethod 啦  
00403FF9    59              pop ecx  
00403FFA    58              pop eax  
00403FFB    74 02           je short MusicPoi.00403FFF  
00403FFD    FFE6            jmp esi                      //看牢这里，它可是指向动态方法入口地址哦  
00403FFF    59              pop ecx  
00404000  ^ E9 53ECFFFF     jmp MusicPoi.00402C58  
00404005    C3              retn  
```

--------------------------------------------------------------------------------------------------------------

和上面的理论一一对应了，继续……

```
0043CDB0    53              push ebx                      ; 动态方法入口地址  
0043CDB1    8BD8            mov ebx,eax  
0043CDB3    8A83 21020000   mov al,byte ptr ds:\[ebx+221\]  
0043CDB9    2C 03           sub al,3  
0043CDBB    74 1F           je short MusicPoi.0043CDDC  
0043CDBD    2C 03           sub al,3  
0043CDBF    75 52           jnz short MusicPoi.0043CE13  
0043CDC1    8BC3            mov eax,ebx  
0043CDC3    E8 80790300     call MusicPoi.00474748  
0043CDC8    85C0            test eax,eax  
0043CDCA    74 07           je short MusicPoi.0043CDD3  
0043CDCC    E8 83CF0300     call MusicPoi.00479D54  
0043CDD1    EB 47           jmp short MusicPoi.0043CE1A  
0043CDD3    8BC3            mov eax,ebx  
0043CDD5    E8 5E710100     call MusicPoi.00453F38  
0043CDDA    5B              pop ebx  
0043CDDB    C3              retn  
0043CDDC    8BC3            mov eax,ebx  
0043CDDE    EB 03           jmp short MusicPoi.0043CDE3  
0043CDE0    8B40 30         mov eax,dword ptr ds:\[eax+30\]  
0043CDE3    85C0            test eax,eax  
0043CDE5    74 09           je short MusicPoi.0043CDF0  
0043CDE7    83B8 58010000 0>cmp dword ptr ds:\[eax+158\],0  
0043CDEE  ^ 74 F0           je short MusicPoi.0043CDE0  
0043CDF0    85C0            test eax,eax  
0043CDF2    74 16           je short MusicPoi.0043CE0A  
0043CDF4    8B15 0C2C5200   mov edx,dword ptr ds:\[522C0C\]         ; MusicPoi.00523BF0  
0043CDFA    8B12            mov edx,dword ptr ds:\[edx\]  
0043CDFC    8B80 58010000   mov eax,dword ptr ds:\[eax+158\]  
0043CE02    92              xchg eax,edx  
0043CE03    E8 2C0C0400     call MusicPoi.0047DA34  
0043CE08    EB 10           jmp short MusicPoi.0043CE1A  
0043CE0A    8BC3            mov eax,ebx  
0043CE0C    E8 27710100     call MusicPoi.00453F38  
0043CE11    5B              pop ebx  
0043CE12    C3              retn  
0043CE13    8BC3            mov eax,ebx  
0043CE15    E8 1E710100     call MusicPoi.00453F38                 ; 注册算法，关键跟进  
0043CE1A    5B              pop ebx  
0043CE1B    C3              retn  
```

调用 `MusicPoi.00453F38` 函数，注册码关键算法:

--------------------------------------------------------------------------------------------------------------

```
005188D0 >  55              push ebp                      ; <-Tfrmreg@btngetClick  
005188D1    8BEC            mov ebp,esp  
005188D3    6A 00           push 0  
005188D5    6A 00           push 0  
005188D7    53              push ebx  
005188D8    56              push esi  
005188D9    57              push edi  
005188DA    8BD8            mov ebx,eax  
005188DC    33C0            xor eax,eax  
005188DE    55              push ebp  
005188DF    68 8B895100     push <MusicPoi.->System.@HandleFinally>  
005188E4    64:FF30         push dword ptr fs:\[eax\]  
005188E7    64:8920         mov dword ptr fs:\[eax\],esp  
005188EA    B8 3C415200     mov eax,MusicPoi.0052413C  
005188EF >  E8 50C3EEFF     call MusicPoi.00404C44                 ; ->System.@LStrClr(void;void);  
005188F4    8D55 FC         lea edx,dword ptr ss:\[ebp-4\]  
005188F7 >  8B83 F8020000   mov eax,dword ptr ds:\[ebx+2F8\]        ; \*Tfrmreg.Edit1:TLabeledEdit  
005188FD >  E8 7A4AF4FF     call MusicPoi.0045D37C                 ; ->Controls.TControl.GetText(TControl):TCaption;  
00518902    8B45 FC         mov eax,dword ptr ss:\[ebp-4\]          ; 机器码 MachineCode  
00518905 >  E8 FAC5EEFF     call MusicPoi.00404F04                 ; ->System.@LStrLen(String):Integer;<+>  
0051890A    8BF0            mov esi,eax                             ; 机器码长度 Length  
0051890C    85F6            test esi,esi  
0051890E    7E 4B           jle short MusicPoi.0051895B             ; 如果长度<=0 则 Over!  
00518910    BF 01000000     mov edi,1                              ; i=1  
00518915    8B45 FC         mov eax,dword ptr ss:\[ebp-4\]          ; 机器码  
00518918    33DB            xor ebx,ebx  
0051891A    8A5C38 FF       mov bl,byte ptr ds:\[eax+edi-1\]        ; MachineCode\[i\]  
0051891E    83FB 41         cmp ebx,41                              ; 假如  MachineCode\[i\]<'A' 则 MachineCode\[i\]+1E  
00518921    7C 14           jl short MusicPoi.00518937  
00518923    83FB 5A         cmp ebx,5A                              ; 假如  MachineCode\[i\]>'Z' 则 MachineCode\[i\]+1E  
00518926    7F 0F           jg short MusicPoi.00518937  
00518928    83FB 49         cmp ebx,49                              ; 假如  MachineCode\[i\]>='I' 则 MachineCode\[i\]-5  
0051892B    7D 05           jge short MusicPoi.00518932  
0051892D    83C3 07         add ebx,7                               ; 假如  MachineCode\[i\]<'I'  则 MachineCode\[i\]+7  
00518930    EB 08           jmp short MusicPoi.0051893A  
00518932    83EB 05         sub ebx,5  
00518935    EB 03           jmp short MusicPoi.0051893A  
00518937    83C3 1E         add ebx,1E  
0051893A    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
0051893D    8BD3            mov edx,ebx  
0051893F >  E8 E8C4EEFF     call MusicPoi.00404E2C                 ; ->System.@LStrFromChar(String;String;Char);<+>  
00518944    8B55 F8         mov edx,dword ptr ss:\[ebp-8\]  
00518947    B8 3C415200     mov eax,MusicPoi.0052413C  
0051894C    8B0D 3C415200   mov ecx,dword ptr ds:\[52413C\]  
00518952 >  E8 F9C5EEFF     call MusicPoi.00404F50                 ; 字符串的连接->System.@LStrCat3;注意是倒序  
00518957    47              inc edi  
00518958    4E              dec esi  
00518959  ^ 75 BA           jnz short MusicPoi.00518915             ; 循环  
0051895B    B8 3C415200     mov eax,MusicPoi.0052413C  
00518960    8B0D 3C415200   mov ecx,dword ptr ds:\[52413C\]         ; 连接后的字符串  
00518966    BA A4895100     mov edx,MusicPoi.005189A4               ; 字符 '8'  
0051896B >  E8 E0C5EEFF     call MusicPoi.00404F50                 ; ->System.@LStrCat3;  
00518970    33C0            xor eax,eax                             ; '8'+连接后的字符串,即为有效注册码  
00518972    5A              pop edx  
00518973    59              pop ecx  
00518974    59              pop ecx  
00518975    64:8910         mov dword ptr fs:\[eax\],edx  
00518978    68 92895100     push MusicPoi.00518992  
0051897D    8D45 F8         lea eax,dword ptr ss:\[ebp-8\]  
00518980    BA 02000000     mov edx,2  
00518985 >  E8 DEC2EEFF     call MusicPoi.00404C68                 ; ->System.@LStrArrayClr(void;void;Integer);  
```
