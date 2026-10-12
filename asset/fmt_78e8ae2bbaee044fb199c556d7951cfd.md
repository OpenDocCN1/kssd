
### 第三处

```
0089C723    894D AC         mov dword ptr ss:[ebp-54],ecx  
0089C726    8D95 64FFFFFF   lea edx,dword ptr ss:[ebp-9C]  
0089C72C    8D4D B4         lea ecx,dword ptr ss:[ebp-4C]  
0089C72F    8945 94         mov dword ptr ss:[ebp-6C],eax  
0089C732    8945 A4         mov dword ptr ss:[ebp-5C],eax  
0089C735    C785 6CFFFFFF F>mov dword ptr ss:[ebp-94],驾照考试.004068F4  
0089C73F    C785 64FFFFFF 0>mov dword ptr ss:[ebp-9C],8  
0089C749    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089C74F    8D95 74FFFFFF   lea edx,dword ptr ss:[ebp-8C]  
0089C755    8D4D C4         lea ecx,dword ptr ss:[ebp-3C]  
0089C758    C785 7CFFFFFF D>mov dword ptr ss:[ebp-84],驾照考试.004070DC  
0089C762    C785 74FFFFFF 0>mov dword ptr ss:[ebp-8C],8  
0089C76C    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089C772    8D45 94         lea eax,dword ptr ss:[ebp-6C]  
0089C775    8D4D A4         lea ecx,dword ptr ss:[ebp-5C]  
0089C778    50              push eax  
0089C779    8D55 B4         lea edx,dword ptr ss:[ebp-4C]  
0089C77C    51              push ecx  
0089C77D    52              push edx  
0089C77E    8D45 C4         lea eax,dword ptr ss:[ebp-3C]  
0089C781    6A 40           push 40                                       //40 道题做完后就压栈  
0089C783    50              push eax  
0089C784    FF15 7C104000   call dword ptr ds:[<&MSVBVM60.#595>]        ; MSVBVM60.rtcMsgBox   //非注册用户只能做前 40 道题！  
0089C78A    8BC8            mov ecx,eax  
```

### 第四处

```
0089CAE2    66:833D 28408B0>cmp word ptr ds:[8B4028],0FFFF        //验证无误后，无限制做题，变量范围 65535  
0089CAEA    75 12           jnz short 驾照考试.0089CAFE           //爆破点 C’  
0089CAEC    66:C786 8800000>mov word ptr ds:[esi+88],0E2  
0089CAF5    66:C786 8A00000>mov word ptr ds:[esi+8A],15F  
0089CAFE    66:833D 28408B0>cmp word ptr ds:[8B4028],0            //验证失败后，弹出提示框，变量范围 0  
0089CB06    0F85 AF000000   jnz 驾照考试.0089CBBB                 //爆破点 C  
0089CB0C    B9 04000280     mov ecx,80020004  
0089CB11    B8 0A000000     mov eax,0A  
0089CB16    894D 9C         mov dword ptr ss:[ebp-64],ecx  
0089CB19    894D AC         mov dword ptr ss:[ebp-54],ecx  
0089CB1C    8D95 64FFFFFF   lea edx,dword ptr ss:[ebp-9C]  
0089CB22    8D4D B4         lea ecx,dword ptr ss:[ebp-4C]  
0089CB25    8945 94         mov dword ptr ss:[ebp-6C],eax  
0089CB28    8945 A4         mov dword ptr ss:[ebp-5C],eax  
0089CB2B    C785 6CFFFFFF F>mov dword ptr ss:[ebp-94],驾照考试.004068F4  
0089CB35    C785 64FFFFFF 0>mov dword ptr ss:[ebp-9C],8  
0089CB3F    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089CB45    8D95 74FFFFFF   lea edx,dword ptr ss:[ebp-8C]  
0089CB4B    8D4D C4         lea ecx,dword ptr ss:[ebp-3C]  
0089CB4E    C785 7CFFFFFF D>mov dword ptr ss:[ebp-84],驾照考试.004070DC  
0089CB58    C785 74FFFFFF 0>mov dword ptr ss:[ebp-8C],8  
0089CB62    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089CB68    8D45 94         lea eax,dword ptr ss:[ebp-6C]  
0089CB6B    8D4D A4         lea ecx,dword ptr ss:[ebp-5C]  
0089CB6E    50              push eax  
0089CB6F    8D55 B4         lea edx,dword ptr ss:[ebp-4C]  
0089CB72    51              push ecx  
0089CB73    52              push edx  
0089CB74    8D45 C4         lea eax,dword ptr ss:[ebp-3C]  
0089CB77    6A 40           push 40                                       //40 道题做完后就压栈  
0089CB79    50              push eax  
0089CB7A    FF15 7C104000   call dword ptr ds:[<&MSVBVM60.#595>]        ; MSVBVM60.rtcMsgBox   //非注册用户只能做前 40 道题！  
0089CB80    8BC8            mov ecx,eax  
```

### 第五处

```
0089CED8    66:833D 28408B0>cmp word ptr ds:[8B4028],0FFFF        //验证无误后，无限制做题，变量范围 65535  
0089CEE0    75 12           jnz short 驾照考试.0089CEF4           //爆破点 D’  
0089CEE2    66:C786 8800000>mov word ptr ds:[esi+88],160  
0089CEEB    66:C786 8A00000>mov word ptr ds:[esi+8A],1A6  
0089CEF4    66:833D 28408B0>cmp word ptr ds:[8B4028],0            //验证失败后，弹出提示框，变量范围 0  
0089CEFC    0F85 AF000000   jnz 驾照考试.0089CFB1                 //爆破点 D  
0089CF02    B9 04000280     mov ecx,80020004  
0089CF07    B8 0A000000     mov eax,0A  
0089CF0C    894D 9C         mov dword ptr ss:[ebp-64],ecx  
0089CF0F    894D AC         mov dword ptr ss:[ebp-54],ecx  
0089CF12    8D95 64FFFFFF   lea edx,dword ptr ss:[ebp-9C]  
0089CF18    8D4D B4         lea ecx,dword ptr ss:[ebp-4C]  
0089CF1B    8945 94         mov dword ptr ss:[ebp-6C],eax  
0089CF1E    8945 A4         mov dword ptr ss:[ebp-5C],eax  
0089CF21    C785 6CFFFFFF F>mov dword ptr ss:[ebp-94],驾照考试.004068F4  
0089CF2B    C785 64FFFFFF 0>mov dword ptr ss:[ebp-9C],8  
0089CF35    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089CF3B    8D95 74FFFFFF   lea edx,dword ptr ss:[ebp-8C]  
0089CF41    8D4D C4         lea ecx,dword ptr ss:[ebp-3C]  
0089CF44    C785 7CFFFFFF D>mov dword ptr ss:[ebp-84],驾照考试.004070DC  
0089CF4E    C785 74FFFFFF 0>mov dword ptr ss:[ebp-8C],8  
0089CF58    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089CF5E    8D45 94         lea eax,dword ptr ss:[ebp-6C]  
0089CF61    8D4D A4         lea ecx,dword ptr ss:[ebp-5C]  
0089CF64    50              push eax  
0089CF65    8D55 B4         lea edx,dword ptr ss:[ebp-4C]  
0089CF68    51              push ecx  
0089CF69    52              push edx  
0089CF6A    8D45 C4         lea eax,dword ptr ss:[ebp-3C]  
0089CF6D    6A 40           push 40                                       //40 道题做完后就压栈  
0089CF6F    50              push eax  
0089CF70    FF15 7C104000   call dword ptr ds:[<&MSVBVM60.#595>]        ; MSVBVM60.rtcMsgBox   //非注册用户只能做前 40 道题！  
0089CF76    8BC8            mov ecx,eax  
```

### 第六处

```
0089D92C    66:833D 28408B0>cmp word ptr ds:[8B4028],0FFFF        //验证无误后，无限制做题，变量范围 65535  
0089D934    75 12           jnz short 驾照考试.0089D948           //爆破点 E’  
0089D936    66:C786 8800000>mov word ptr ds:[esi+88],1D1  
0089D93F    66:C786 8A00000>mov word ptr ds:[esi+8A],200  
0089D948    66:833D 28408B0>cmp word ptr ds:[8B4028],0            //验证失败后，弹出提示框，变量范围 0  
0089D950    0F85 AF000000   jnz 驾照考试.0089DA05                 //爆破点 E  
0089D956    B9 04000280     mov ecx,80020004  
0089D95B    B8 0A000000     mov eax,0A  
0089D960    894D 9C         mov dword ptr ss:[ebp-64],ecx  
0089D963    894D AC         mov dword ptr ss:[ebp-54],ecx  
0089D966    8D95 64FFFFFF   lea edx,dword ptr ss:[ebp-9C]  
0089D96C    8D4D B4         lea ecx,dword ptr ss:[ebp-4C]  
0089D96F    8945 94         mov dword ptr ss:[ebp-6C],eax  
0089D972    8945 A4         mov dword ptr ss:[ebp-5C],eax  
0089D975    C785 6CFFFFFF F>mov dword ptr ss:[ebp-94],驾照考试.004068F4  
0089D97F    C785 64FFFFFF 0>mov dword ptr ss:[ebp-9C],8  
0089D989    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089D98F    8D95 74FFFFFF   lea edx,dword ptr ss:[ebp-8C]  
0089D995    8D4D C4         lea ecx,dword ptr ss:[ebp-3C]  
0089D998    C785 7CFFFFFF D>mov dword ptr ss:[ebp-84],驾照考试.004070DC  
0089D9A2    C785 74FFFFFF 0>mov dword ptr ss:[ebp-8C],8  
0089D9AC    FF15 88114000   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>] ; MSVBVM60.__vbaVarDup  
0089D9B2    8D45 94         lea eax,dword ptr ss:[ebp-6C]  
0089D9B5    8D4D A4         lea ecx,dword ptr ss:[ebp-5C]  
0089D9B8    50              push eax  
0089D9B9    8D55 B4         lea edx,dword ptr ss:[ebp-4C]  
0089D9BC    51              push ecx  
0089D9BD    52              push edx  
0089D9BE    8D45 C4         lea eax,dword ptr ss:[ebp-3C]  
```
