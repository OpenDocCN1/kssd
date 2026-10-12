
这是一段关于序列号验证逻辑的汇编代码片段，主要涉及 Delphi 运行时库（如 `System` 和 `SysUtils` 单元）中的字符串处理和数学运算。代码首先通过计算用户名相关值与序列号第二段的组合，生成一个临时结果，然后利用指数表示法（`FormatFloat`）进行处理，最后从序列号中提取第一段并进行浮点数转换，以完成后续的验证比较。

```asm
00489856    .   48                  dec eax                                   ; 第 2 个分隔字符在序列号中的位置-1  

00489857    .   50                  push eax                                  ; 入栈  

00489858    .   8D45 E0           lea eax,dword ptr ss:[ebp-20]  

0048985B    .   50                  push eax  

0048985C    .   8B55 F8           mov edx,dword ptr ss:[ebp-8]              ; SerialNo  

0048985F    .   B8 B89B4800       mov eax,UnPacked.00489BB8                 ; '+'  

00489864    .   E8 FBADF7FF       call UnPacked.00404664                    ; System.@LStrPos;  

00489869    .   40                  inc eax  

0048986A    .   50                  push eax  

0048986B    .   8B45 F8           mov eax,dword ptr ss:[ebp-8]              ; SerialNo  

0048986E    .   E8 ADAAF7FF       call UnPacked.00404320                    ; System.@LStrLen(String):Integer;  

00489873    .   8BC8                mov ecx,eax  

00489875    .   8B45 F8           mov eax,dword ptr ss:[ebp-8]              ; SerialNo  

00489878    .   5A                  pop edx  

00489879    .   E8 02ADF7FF       call UnPacked.00404580                    ; System.@LStrCopy;  

0048987E    .   8B45 E0           mov eax,dword ptr ss:[ebp-20]  

00489881    .   BA 01000000       mov edx,1  

00489886    .   59                  pop ecx  

00489887    .   E8 F4ACF7FF       call UnPacked.00404580                    ; System.@LStrCopy;  

0048988C    .   BB 01000000       mov ebx,1  

00489891    >   8D45 D8           lea eax,dword ptr ss:[ebp-28]  

00489894    .   50                  push eax  

00489895    .   B9 01000000       mov ecx,1  

0048989A    .   8BD3                mov edx,ebx  

0048989C    .   8B45 FC           mov eax,dword ptr ss:[ebp-4]              ; 用户名  

0048989F    .   E8 DCACF7FF       call UnPacked.00404580                    ; System.@LStrCopy;  

004898A4    .   8B45 D8           mov eax,dword ptr ss:[ebp-28]             ; Name[i]=Copy(Name,i,1)  

004898A7    .   0FB600            movzx eax,byte ptr ds:[eax]              ; Name[i]  

004898AA    .   F7EB                imul ebx                                ; i  

004898AC    .   8945 D4           mov dword ptr ss:[ebp-2C],eax             ; Eax=Name[i]*i  

004898AF    .   DB45 D4           fild dword ptr ss:[ebp-2C]               ; 装入到 st(0)  

004898B2    .   E8 DD91F7FF       call UnPacked.00402A94                    ; System.@ROUND;  

004898B7    .   8945 CC           mov dword ptr ss:[ebp-34],eax             ; Round(Name[i])  

004898BA    .   8955 D0           mov dword ptr ss:[ebp-30],edx  

004898BD    .   DF6D CC           fild qword ptr ss:[ebp-34]                ; 装入到 st(0)  

004898C0    .   83C4 F4           add esp,-0C  

004898C3    .   DB3C24            fstp tbyte ptr ss:[esp]                   ; |dest <- st(0)  

004898C6    .   9B                  wait                                      ; |  

004898C7    .   8D55 DC           lea edx,dword ptr ss:[ebp-24]             ; |  

004898CA    .   B8 C49B4800       mov eax,UnPacked.00489BC4                ; |  

004898CF    .   E8 9804F8FF       call UnPacked.00409D6C                    ; \SysUtils.FormatFloat(AnsiString;Extended):AnsiString;overload;  

004898D4    .   FF75 DC           push dword ptr ss:[ebp-24]               ; FormatFloat(Round(Name[i]))  

004898D7    .   8D55 C8           lea edx,dword ptr ss:[ebp-38]  

004898DA    .   8BC3                mov eax,ebx                             ; i  

004898DC    .   E8 13ECF7FF       call UnPacked.004084F4                    ; SysUtils.IntToStr(Integer):AnsiString;overload;  

004898E1    .   FF75 C8           push dword ptr ss:[ebp-38]               ; IntStr(i)  

004898E4    .   FF35 7C1F4900    push dword ptr ds:[491F7C]               ; 序列号第二段  

004898EA    .   8D45 EC           lea eax,dword ptr ss:[ebp-14]  

004898ED    .   BA 03000000       mov edx,3  

004898F2    .   E8 E9AAF7FF       call UnPacked.004043E0                    ; System.@LStrCatN;  

004898F7    .   8B45 EC           mov eax,dword ptr ss:[ebp-14]             ; FormatFloat(Round(Name[i]))+IntToStr(i)+注册码第 2 段,记为 Temp  

004898FA    .   E8 31EDF7FF       call UnPacked.00408630                    ; SysUtils.StrToInt(AnsiString):Integer;  

004898FF    .   8BF0                mov esi,eax                             ; EAX=StrToInt(Temp)  

00489901    .   8B45 EC           mov eax,dword ptr ss:[ebp-14]  

00489904    .   E8 27EDF7FF       call UnPacked.00408630                    ; SysUtils.StrToInt(AnsiString):Integer;  

00489909    .   03F0                add esi,eax                             ; ESI=EAX+EAX  

0048990B    .   8BC6                mov eax,esi  

0048990D    .   8D55 C4           lea edx,dword ptr ss:[ebp-3C]  

00489910    .   E8 DFEBF7FF       call UnPacked.004084F4                    ; SysUtils.IntToStr(Integer):AnsiString;overload;  

00489915    .   8B55 C4           mov edx,dword ptr ss:[ebp-3C]             ; IntToStr(Temp*2)  

00489918    .   8D45 EC           lea eax,dword ptr ss:[ebp-14]  

0048991B    .   E8 D8A7F7FF       call UnPacked.004040F8  

00489920    .   43                  inc ebx                                 ; i=i+1  

00489921    .   8B45 FC           mov eax,dword ptr ss:[ebp-4]              ; Name  

00489924    .   E8 F7A9F7FF       call UnPacked.00404320                    ; System.@LStrLen(String):Integer;  

00489929    .   40                  inc eax                                 ; Length(Name)+1  

0048992A    .   3BD8                cmp ebx,eax                             ; i  

0048992C    .^  0F85 5FFFFFFF     jnz UnPacked.00489891                     ; 用户名长度循环  

00489932    .   6A 16               push 16  

00489934    .   68 99BD5116       push 1651BD99  

00489939    .   8B45 EC           mov eax,dword ptr ss:[ebp-14]             ;  
 用户名与序列号第 2 段计算结果,实际上是(IntToStr(Name[Length]*Length)+IntToStr(Length)+'序列号第 2 段')*2  

0048993C    .   E8 EFECF7FF       call UnPacked.00408630                    ; SysUtils.StrToInt(AnsiString):Integer;  

00489941    .   99                  cdq  

00489942    .   E8 15B6F7FF       call UnPacked.00404F5C                    ; System.@_llmul; 64 位乘法运算  

00489947    .   8945 CC           mov dword ptr ss:[ebp-34],eax            ; StrToInt(计算结果)*161651BD99  

0048994A    .   8955 D0           mov dword ptr ss:[ebp-30],edx            ; EDX 保存高 32 位,EAX 保存低 32 位  

0048994D    .   DF6D CC           fild qword ptr ss:[ebp-34]  

00489950    .   83C4 F4           add esp,-0C  

00489953    .   DB3C24            fstp tbyte ptr ss:[esp]                   ; |  

00489956    .   9B                  wait                                      ; |  

00489957    .   8D55 C0           lea edx,dword ptr ss:[ebp-40]             ; |  

0048995A    .   B8 C49B4800       mov eax,UnPacked.00489BC4                ; |'#'  

0048995F    .   E8 0804F8FF       call UnPacked.00409D6C                    ; \SysUtils.FormatFloat(AnsiString;Extended):AnsiString;overload;  

00489964    .   8B55 C0           mov edx,dword ptr ss:[ebp-40]             ; 指数表示  

00489967    .   8D45 EC           lea eax,dword ptr ss:[ebp-14]  

0048996A    .   E8 89A7F7FF       call UnPacked.004040F8                    ; System.@LStrLAsg(void;void;void;void);  

0048996F    .   8D45 F0           lea eax,dword ptr ss:[ebp-10]  

00489972    .   50                  push eax  

00489973    .   8B55 F8           mov edx,dword ptr ss:[ebp-8]              ; 序列号  

00489976    .   B8 B89B4800       mov eax,UnPacked.00489BB8                 ; '+'  

0048997B    .   E8 E4ACF7FF       call UnPacked.00404664                    ; System.@LStrPos;  

00489980    .   8BC8                mov ecx,eax                             ; '+'在序列号中的位置  

00489982    .   49                  dec ecx                                 ; 位置-1  

00489983    .   BA 01000000       mov edx,1  

00489988    .   8B45 F8           mov eax,dword ptr ss:[ebp-8]              ; 序列号  

0048998B    .   E8 F0ABF7FF       call UnPacked.00404580                    ; System.@LStrCopy;  

00489990    .   8B45 F0           mov eax,dword ptr ss:[ebp-10]             ; 取得序列号第 1 段  

00489993    .   E8 2C04F8FF       call UnPacked.00409DC4                    ; SysUtils.StrToFloat(AnsiString):Extended;overload;  

00489998    .   DB7D B4           fstp tbyte ptr ss:[ebp-4C]  

0048999B    .   9B                  wait  

0048999C    .   8B45 EC           mov eax,dword ptr ss:[ebp-14]             ; 用户名与序列号第 2 段计算结果  

0048999F    .   E8 2004F8FF       call UnPacked.00409DC4                    ; SysUtils.StrToFloat(AnsiString):Extended;overload;  
```
