
```asm
0047393C     FF15 68104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeObjList>]
00473942     83C4 0C              add esp,0C
00473945     8D4D B4              lea ecx,dword ptr ss:[ebp-4C]
00473948     FF15 2C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeVar>]
0047394E     8B4D EC              mov ecx,dword ptr ss:[ebp-14]
```

### 第12处：投注号码>>过滤缩水，选号：11 03 09 12 19

```asm
00473AC0     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
00473AC6     08F1                 or cl,dh
00473AC8     8D4D CC              lea ecx,dword ptr ss:[ebp-34]
00473AC0     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
00473AC6     8BD0                 mov edx,eax
00473AC8     8D4D CC              lea ecx,dword ptr ss:[ebp-34]
```

### 第13处：投注号码>>过滤缩水，选号：11 03 09 12 19

```asm
00472D2C     FF15 68104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeObjList>]
00472D32     C3                   retn
00472D33     66:46                inc si
00472D35     8D4D B4              lea ecx,dword ptr ss:[ebp-4C]
00472D2C     FF15 68104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeObjList>]
00472D32     83C4 0C              add esp,0C
00472D35     8D4D B4              lea ecx,dword ptr ss:[ebp-4C]
```

### 第14处：投注号码>>过滤缩水，选号：11 03 09 12 19

```asm
00471D14     FF15 1C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrI4>]
00471D1A     28E6                 sub dh,ah
00471D1C     8D4D D0              lea ecx,dword ptr ss:[ebp-30]
00471D14     FF15 1C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrI4>]
00471D1A     8BC6                 mov edx,eax
00471D1C     8D4D D0              lea ecx,dword ptr ss:[ebp-30]
```

## 退出：

### 第15处：

```asm
004ACE76     FF15 C4124000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeStrList>]
004ACE7C     A7                   cmps dword ptr ds:[esi],dword ptr es:[edi]
004ACE7D     52                   push edx
004ACE7E     F2:                  prefix repne:
004ACE7F     8D4D B4              lea ecx,dword ptr ss:[ebp-4C]
004ACE82     FF15 8C134000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeObj>]
004ACE76     FF15 C4124000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeStrList>]
004ACE7C     83C4 24              add esp,24
004ACE7F     8D4D B4              lea ecx,dword ptr ss:[ebp-4C]
004ACE82     FF15 8C134000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeObj>]
```

## 剩下的都是推测的：

### 第16处：`mov edx,eax`

```asm
0046ED35     FF15 1C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrI4>]
0046ED3B     F2:64:               prefix repne:
0046ED3D     8D4D A8              lea ecx,dword ptr ss:[ebp-58]
0046ED40     FF15 50134000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrMove>]
0046ED35     FF15 1C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrI4>]
0046ED3B     8BD0                 mov edx,eax
0046ED3D     8D4D A8              lea ecx,dword ptr ss:[ebp-58]
0046ED40     FF15 50134000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrMove>]
```

### 第17处：估计跟注册标志有关,`mov edx,eax`

```asm
0046ED47     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
0046ED4D     50                   push eax
0046ED4E     848D 4DA4FF15        test byte ptr ss:[ebp+15FFA44D],cl
0046ED54     50                   push eax
0046ED55     1340 00              adc eax,dword ptr ds:[eax]
0046ED58     50                   push eax
0046ED59     68 3C104400          push dumped-x.0044103C                         ; UNICODE " order by "
0046ED5E     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
0046ED47     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
0046ED4D     8BD0                 mov edx,eax
0046ED4F     8D4D A4              lea ecx,dword ptr ss:[ebp-5C]
0046ED52     FF15 50134000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrMove>]
0046ED58     50                   push eax
0046ED59     68 3C104400          push dumped-x.0044103C                         ; UNICODE " order by "
0046ED5E     FF15 94104000        call dword ptr ds:[<&msvbvm60.\_\_vbaStrCat>]
```

### 第18处：`add esp,0c`

```asm
0046EE26     FF15 C4124000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeStrList>]
0046EE2C     0F33                 rdpmc
0046EE2E     D28D 4D8CFF15        ror byte ptr ss:[ebp+15FF8C4D],cl
0046EE34     2C 10                sub al,10
0046EE36     40                   inc eax
0046EE37     00C7                 add bh,al
0046EE39     45                   inc ebp
0046EE3A     FC                   cld
0046EE3B     04 00                add al,0
0046EE3D     0000                 add byte ptr ds:[eax],al
0046EE3F     837D D0 00           cmp dword ptr ss:[ebp-30],0
0046EE43     75 1A                jnz short dumped-x.0046EE5F
0046EE04     FF15 AC104000        call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
0046EE0A     8985 F4FEFFFF        mov dword ptr ss:[ebp-10C],eax
0046EE10     EB 0A                jmp short dumped-x.0046EE1C
0046EE12     C785 F4FEFFFF 000000>mov dword ptr ss:[ebp-10C],0
0046EE1C     8D45 A4              lea eax,dword ptr ss:[ebp-5C]
0046EE1F     50                   push eax
0046EE20     8D4D A8              lea ecx,dword ptr ss:[ebp-58]
0046EE23     51                   push ecx
0046EE24     6A 02                push 2
0046EE26     FF15 C4124000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeStrList>]
0046EE2C     83C4 0C              add esp,0C            // 上面3个push
0046EE2F     8D4D 8C              lea ecx,dword ptr ss:[ebp-74]
0046EE32     FF15 2C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeVar>]
0046EE38     C745 FC 04000000     mov dword ptr ss:[ebp-4],4
0046EE3F     837D D0 00           cmp dword ptr ss:[ebp-30],0
0046EE43     75 1A                jnz short dumped-x.0046EE5F
```

这里根据堆栈平衡原理，`0046EE26`处的`call`上面有3个`push`，因此`add esp,0c`。

### 第19处：`mov dword ptr ss:[ebp-14C],eax`

```asm
0046F3D9     FF15 AC104000        call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
0046F3DF     82E7 B7              and bh,FFFFFFB7
0046F3E2     48                   dec eax
0046F3E3     AF                   scas dword ptr es:[edi]
0046F3E4     0BEB                 or ebp,ebx
0046F3E6     0AC7                 or al,bh
0046F3E8     85B4FE FFFF0000      test dword ptr ds:[esi+edi\*8+FFFF],esi
0046F3EF     0000                 add byte ptr ds:[eax],al
0046F3F1     8B55 9C              mov edx,dword ptr ss:[ebp-64]
0046F3F4     8995 30FFFFFF        mov dword ptr ss:[ebp-D0],edx
0046F3FA     8D45 8C              lea eax,dword ptr ss:[ebp-74]
0046F3D9     FF15 AC104000        call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
0046F3DF     8985 B4FEFFFF        mov dword ptr ss:[ebp-14C],eax
0046F3E5     EB 0A                jmp short dumped-x.0046F3F1
0046F3E7     C785 B4FEFFFF 000000>mov dword ptr ss:[ebp-14C],0
0046F3F1     8B55 9C              mov edx,dword ptr ss:[ebp-64]
0046F3F4     8995 30FFFFFF        mov dword ptr ss:[ebp-D0],edx
0046F3FA     8D45 8C              lea eax,dword ptr ss:[ebp-74]
```

这个是参考程序其他地方的值，都是类似的：

```asm
call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
mov dword ptr ss:[ebp-xxx],eax
jmp short xxxxxxxx
mov dword ptr ss:[ebp-xxx],0      // 根据这里的地址推测的
```

### 第20处：`mov dword ptr ss:[ebp-154],eax`

```asm
0046F51E     FF15 AC104000        call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
0046F524     4F                   dec edi
0046F525     AB                   stos dword ptr es:[edi]
0046F526     C5D0                 lds edx,eax
0046F528     8EE9                 mov gs,cx
0046F52A     EB 0A                jmp short dumped-x.0046F536
0046F52C     C785 ACFEFFFF 000000>mov dword ptr ss:[ebp-154],0
0046F51E     FF15 AC104000        call dword ptr ds:[<&msvbvm60.\_\_vbaHresultCheckObj>]
0046F524     8985 ACFEFFFF        mov dword ptr ss:[ebp-154],eax
0046F52A     EB 0A                jmp short dumped-x.0046F536
0046F52C     C785 ACFEFFFF 000000>mov dword ptr ss:[ebp-154],0
```

### 第21处：`add esp,0C`

```asm
0046F556     FF15 4C104000        call dword ptr ds:[<&msvbvm60.\_\_vbaFreeVarList>]
0046F55C   ^ E3 E8                jecxz short dumped-x.0046F546
```
