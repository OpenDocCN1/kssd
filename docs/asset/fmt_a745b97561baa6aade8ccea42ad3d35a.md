
## 汇编代码分析与补丁

### 初始代码片段

```
711E3B5E    52                 PUSH EDX
711E3B5F    8A1D 8E062171      MOV BL,BYTE PTR DS:[7121068E]
711E3B65    80FB 00            CMP BL,0
711E3B68    75 27              JNZ SHORT 711E3B91
711E3B6A    35 FFFFFFFF        XOR EAX,FFFFFFFF
711E3B6F    894424 34          MOV DWORD PTR SS:[ESP+34],EAX
711E3B73    83C4 10            ADD ESP,10
711E3B76    64:8F05 00000000   POP DWORD PTR FS:[0]
711E3B7D    58                 POP EAX
711E3B7E    25 00010000        AND EAX,100
711E3B83    3D 00010000        CMP EAX,100
711E3B88    74 27              JE SHORT 711E3BB1
711E3B8A    E8 1D89FCFF        CALL 711AC4AC        ; 进来后，ENTER 进入里面看看
711E3B8F    EB 20              JMP SHORT 711E3BB1
711E3B91    894424 34          MOV DWORD PTR SS:[ESP+34],EAX
```

再次进到这里后向下找就可以很快到 oep 的：

```
711AC56B    ^\\EB DE          JMP SHORT 711AC54B
711AC56D    64:FF35 00000000  PUSH DWORD PTR FS:[0]
711AC574    64:8925 00000000  MOV DWORD PTR FS:[0],ESP      ; 向下直接找到这里
711AC57B    CC                 INT3
711AC57C    90                 NOP
711AC57D    64:8F05 00000000  POP DWORD PTR FS:[0]
711AC584    C3                 RETN                ; 在这里下 f2 断点，然后 F9 运行中断后 F7 就会跳去 oep
711AC585    C3                 RETN
```

OK 到这里分析完毕，重来，把前面记录下的地方总结一下：

### CRC 检测一

```
711E38A2    0F95C0            SETNE AL        这里处理 CRC 的
711E38A5    5E                POP ESI
711E38A6    5B                POP EBX
711E38A7    C3                RETN
```

......

修改为：

```
711E38A2    B0 00             MOV AL,0
711E38A4    90                NOP
711E38A5    5E                POP ESI
711E38A6    5B                POP EBX
711E38A7    C3                RETN
```

### CRC 处理二

```
711E5459    0F95C0            SETNE AL
711E545C    5F                POP EDI
711E545D    5E                POP ESI
711E545E    5B                POP EBX
711E545F    C3                RETN
```

……

修改为：

```
711E5459    B0 00             MOV AL,0
711E545B    90                NOP
711E545C    5F                POP EDI
711E545D    5E                POP ESI
711E545E    5B                POP EBX
711E545F    C3                RETN
```

### IAT 处理一

```
711E37C5    334D FC           XOR ECX,DWORD PTR SS:[EBP-4]  ;写入正确的 API 函数处理
711E37C8    8908              MOV DWORD PTR DS:[EAX],ECX
711E37CA    8955 F8           MOV DWORD PTR SS:[EBP-8],EDX
```

......

修改成:

```
711E37C5    8B4D FC           MOV ECX,DWORD PTR SS:[EBP-4]
711E37C8    8908              MOV DWORD PTR DS:[EAX],ECX
711E37CA    894D F8           MOV DWORD PTR SS:[EBP-8],ECX
```

### IAT 处理二

```
711E4FEB    66:C700 FF25      MOV WORD PTR DS:[EAX],25FF
```

......

```
711E502A    66:C700 FF25      MOV WORD PTR DS:[EAX],25FF
```

修改成:

```
711E4FEB    E8 10090000       CALL 711E5900
```

......

```
711E502A    E8 D1080000       CALL 711E5900
```

......

```
711E5900    66:813B 90E8      CMP WORD PTR DS:[EBX],0E890     ; 判断是 jmp 还是 call
711E5905    75 06             JNZ SHORT 711E590D              ; 如果不为 call 则跳
711E5907    66:C703 FF15      MOV WORD PTR DS:[EBX],15FF
711E590C    C3                RETN
711E590D    66:C703 FF25      MOV WORD PTR DS:[EBX],25FF
711E5912    C3                RETN
```

### 替换代码处理

```
711E5146    66:8910           MOV WORD PTR DS:[EAX],DX
```

修改为:

```
711E5146    66:8913           MOV WORD PTR DS:[EBX],DX
```

### 替换代码处理

```
711E5134    66:8910           MOV WORD PTR DS:[EAX],DX
```

修改为:

```
711E5134    66:8913           MOV WORD PTR DS:[EBX],DX
```

### 替换代码处理

```
71236710    >   807C24 08 01  CMP BYTE PTR SS:[ESP+8],1
71236715        0F85 9D010000   JNZ 712368B8
```

修改为:

```
71236710    >   807C24 08 01  CMP BYTE PTR SS:[ESP+8],1
71236715       /0F85 AC010000   JNZ 712368C7
```

### 替换代码处理

这下面的代码 nop 掉

```
711E505C    8910              MOV DWORD PTR DS:[EAX],EDX
```

......

```
711E501E    8910              MOV DWORD PTR DS:[EAX],EDX
```

......

```
711E5161    66:C703 90E9      MOV WORD PTR DS:[EBX],0E990
```

......

```
711E518B    8910              MOV DWORD PTR DS:[EAX],EDX
```

### 跳过注册

```
711E2D8F    8378 0C 01        CMP DWORD PTR DS:[EAX+C],1      ;把[EAX+C]的值改为 0
711E2D93    74 0C             JE SHORT 711E2DA1
711E2D95    8B83 24030000     MOV EAX,DWORD PTR DS:[EBX+324]
711E2D9B    8378 30 64        CMP DWORD PTR DS:[EAX+30],64    ;把[EAX+30]的值改成 65
711E2D9F    75 0A             JNZ SHORT 711E2DAB
711E2DA1    8BC3              MOV EAX,EBX
```

全篇完!^\_^

Greetz:

Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!

By loveboom\[DFCG\]\[FCG\]\[US\]

`http://blog.csdn.net/bmd2chen`

Email:loveboom#163.com

Date:5/22/2005 4:59:00 PM

`4dd466d0e1e0cc123afbe504da2462ca`
