
```
004DAD72     C683 AC030000 00    mov byte ptr ds:[ebx+3AC],0
004DAD79     8B83 9C030000       mov eax,dword ptr ds:[ebx+39C]
004DAD7F     E8 E862F4FF         call Dump3.0042106C
004DAD84     EB 1E               jmp short Dump3.004DADA4
004DAD86     8B83 8C030000       mov eax,dword ptr ds:[ebx+38C]
004DAD8C     83C0 68             add eax,68
004DAD8F     BA CCAE4D00         mov edx,Dump3.004DAECC
004DAD94     E8 779DF2FF         call Dump3.00404B10
004DAD99     8B83 8C030000       mov eax,dword ptr ds:[ebx+38C]
004DAD9F     8B10                mov edx,dword ptr ds:[eax]
004DADA1     FF52 30             call dword ptr ds:[edx+30]
004DADA4     8B83 A8030000       mov eax,dword ptr ds:[ebx+3A8]
004DADAA     BA F8AE4D00         mov edx,Dump3.004DAEF8
004DADAF     E8 04A1F2FF         call Dump3.00404EB8
004DADB4     75 0A               jnz short Dump3.004DADC0
004DADB6     E8 69BBFFFF         call Dump3.004D6924
004DADBB     E9 86000000         jmp Dump3.004DAE46
004DADC0     E8 B30AFBFF         call Dump3.0048B878
004DADC5     83C4 F8             add esp,-8
004DADC8     DD1C24              fstp qword ptr ss:[esp]
004DADCB     9B                  wait
004DADCC     E8 DF0AFBFF         call Dump3.0048B8B0
004DADD1     8BF0                mov esi,eax
004DADD3     0FB7C6              movzx eax,si
004DADD6     B9 0D000000         mov ecx,0D
004DADDB     33D2                xor edx,edx
004DADDD     F7F1                div ecx
004DADDF     85D2                test edx,edx
004DADE1     75 63               jnz short Dump3.004DAE46
004DADE3     8B83 A0030000       mov eax,dword ptr ds:[ebx+3A0]
004DADE9     8B93 A4030000       mov edx,dword ptr ds:[ebx+3A4]
004DADEF     E8 C4A0F2FF         call Dump3.00404EB8
004DADF4     74 50               je short Dump3.004DAE46
004DADF6     B2 01               mov dl,1
004DADF8     A1 34CB4600         mov eax,dword ptr ds:[46CB34]
004DADFD     E8 321EF9FF         call Dump3.0046CC34
004DAE02     8BD8                mov ebx,eax
004DAE04     BA 01000080         mov edx,80000001
004DAE09     8BC3                mov eax,ebx
004DAE0B     E8 C41EF9FF         call Dump3.0046CCD4
004DAE10     33C9                xor ecx,ecx
004DAE12     BA 04AF4D00         mov edx,Dump3.004DAF04             ; ASCII "\Software\FixVideo\VideoFixer\"
004DAE17     8BC3                mov eax,ebx
004DAE19     E8 1A1FF9FF         call Dump3.0046CD38
004DAE1E     84C0                test al,al
004DAE20     74 18               je short Dump3.004DAE3A
004DAE22     B9 F8AE4D00         mov ecx,Dump3.004DAEF8
004DAE27     BA 2CAF4D00         mov edx,Dump3.004DAF2C             ; ASCII "Date"
004DAE2C     8BC3                mov eax,ebx
004DAE2E     E8 A120F9FF         call Dump3.0046CED4
004DAE33     8BC3                mov eax,ebx
004DAE35     E8 6A1EF9FF         call Dump3.0046CCA4
004DAE3A     8BC3                mov eax,ebx
004DAE3C     E8 6B8EF2FF         call Dump3.00403CAC
004DAE41     E8 DEBAFFFF         call Dump3.004D6924
004DAE46     60                  pushad  //摧毁自杀代码
004DAE46    /EB 3C               jmp short Dump3.004DAE84
004DAE47     60                  pushad
004DAE48     E8 00000000         call Dump3.004DAE4D
004DAE4D     5E                  pop esi
004DAE4E     83EE 06             sub esi,6
004DAE51     B9 AD010000         mov ecx,1AD
004DAE56     29CE                sub esi,ecx
004DAE58     BA 3081C79D         mov edx,9DC78130
004DAE5D     C1E9 02             shr ecx,2
004DAE60     83E9 02             sub ecx,2
004DAE63     83F9 00             cmp ecx,0
004DAE66     7C 1A               jl short Dump3.004DAE82
004DAE68     8B048E              mov eax,dword ptr ds:[esi+ecx\*4]
004DAE6B     8B5C8E 04           mov ebx,dword ptr ds:[esi+ecx\*4+4]
004DAE6F     2BC3                sub eax,ebx
004DAE71     C1C8 17             ror eax,17
004DAE74     33C2                xor eax,edx
004DAE76     81C2 4B354324       add edx,2443354B
004DAE7C     89048E              mov dword ptr ds:[esi+ecx\*4],eax
004DAE7F     49                  dec ecx
004DAE80   ^ EB E1               jmp short Dump3.004DAE63
004DAE82     61                  popad
004DAE83     61                  popad
004DAE84     33C0                xor eax,eax //结束。
```

..............................................................................

## 然后还是要保护

`004DAC97 4D dec ebp` 到 `004DAE84 33C0 xor eax,eax` 之间的代码不被覆盖。

## 重启OD，同样在点修复按钮前下Code段内存写入断点

```
004DAB41     8945 26             mov dword ptr ss:[ebp+26],eax      ; Dump3.004DAC9A
004DAB44     78 03               js short Dump3.004DAB49
004DAB46     79 01               jns short Dump3.004DAB49
```

...................................................................

## 往上一点使用花指令插件

```
004DAADE     55                  push ebp
004DAADF     68 9FAE4D00         push Dump3.004DAE9F
004DAAE4     64:FF30             push dword ptr fs:[eax]
004DAAE7     64:8920             mov dword ptr fs:[eax],esp
004DAAEA     60                  pushad
004DAAEB     FC                  cld
004DAAEC     1BCE                sbb ecx,esi
004DAAEE     C1E9 EA             shr ecx,0EA
004DAAF1     72 01               jb short Dump3.004DAAF4
004DAAF3     43                  inc ebx
004DAAF4     50                  push eax
004DAAF5     E8 01000000         call Dump3.004DAAFB
004DAAFA     EB 58               jmp short Dump3.004DAB54
004DAAFC     58                  pop eax
```

## 整理一下

```
004DAACA     53                  push ebx
004DAACB     56                  push esi
004DAACC     33C9                xor ecx,ecx
004DAACE     894D D8             mov dword ptr ss:[ebp-28],ecx
004DAAD1     894D D0             mov dword ptr ss:[ebp-30],ecx
004DAAD4     894D CC             mov dword ptr ss:[ebp-34],ecx
004DAAD7     894D D4             mov dword ptr ss:[ebp-2C],ecx
004DAADA     8BD8                mov ebx,eax
004DAADC     33C0                xor eax,eax
004DAADE     55                  push ebp
004DAADF     68 9FAE4D00         push Dump3.004DAE9F
004DAAE4     64:FF30             push dword ptr fs:[eax]
004DAAE7     64:8920             mov dword ptr fs:[eax],esp
004DAAEA     60                  pushad
004DAAEB     FC                  cld
004DAAEC     1BCE                sbb ecx,esi
004DAAEE     C1E9 EA             shr ecx,0EA
004DAAF1     72 01               jb short Dump3.004DAAF4
004DAAF3     43                  inc ebx
004DAAF4     90                  nop
004DAAF5     90                  nop
004DAAF6     90                  nop
004DAAF7     90                  nop
004DAAF8     90                  nop
004DAAF9     90                  nop
004DAAFA     90                  nop
004DAAFB     90                  nop
004DAAFC     90                  nop
004DAAFD     0F88 02000000       js Dump3.004DAB05
004DAB03     0BDD                or ebx,ebp
004DAB05     90                  nop
004DAB06     90                  nop
004DAB07     90                  nop
004DAB08     90                  nop
004DAB09     90                  nop
004DAB0A     90                  nop
004DAB0B     90                  nop
004DAB0C     90                  nop
004DAB0D     90                  nop
004DAB0E     F9                  stc
004DAB0F     90                  nop
004DAB10     90                  nop
004DAB11     90                  nop
004DAB12     90                  nop
004DAB13     90                  nop
004DAB14     F9                  stc
004DAB15     90                  nop
004DAB16     90                  nop
004DAB17     90                  nop
004DAB18     90                  nop
004DAB19     90                  nop
004DAB1A     90                  nop
004DAB1B     90                  nop
004DAB1C     90                  nop
004DAB1D     90                  nop
004DAB1E     90                  nop
004DAB1F     90                  nop
004DAB20     66:BD 2B85          mov bp,852B
004DAB24     90                  nop
```
