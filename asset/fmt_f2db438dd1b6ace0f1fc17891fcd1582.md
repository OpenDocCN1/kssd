
```
.00417423: 8B8BA5726000         mov     ecx,[ebx][0006072A5]

.00417429: 8B83081F6F00         mov     eax,[ebx][0006F1F08]

.0041742F: 8D1401               lea     edx,[ecx][eax]

.00417432: 8B8D30FFFFFF         mov     ecx,[ebp][0FFFFFF30]

.00417438: 66837C0AFF7D         cmp     w,[edx][ecx][-0001],07D ;"}"

.0041743E: 7704                 ja      .000417444

.00417440: 33C0                 xor     eax,eax

.00417442: EB05                 jmps    .000417449

.00417444: B801000000           mov     eax,000000001 ;"    "

.00417449: 898570FFFFFF         mov     [ebp][0FFFFFF70],eax

.0041744F: EB47                 jmps    .000417498

.00417451: 8B93A5726000         mov     edx,[ebx][0006072A5]

.00417457: 8B8B081F6F00         mov     ecx,[ebx][0006F1F08]

.0041745D: 8D040A               lea     eax,[edx][ecx]

......

00460556: 8A0418                 mov     al,[eax][ebx]

.00460559: 3C20                 cmp     al,020 ;" "

.0046055B: 7204                 jb      .000460561 -------- (7)

.0046055D: 3C7D                 cmp     al,07D ;"}"

.0046055F: 7613                 jbe     .000460574

.00460561: 68DE3D4C00           push    0004C3DDE ;" L=?

.00460566: 8D95C0FEFFFF         lea     edx,[ebp][0FFFFFEC0]

.0046056C: 52                   push    edx

.0046056D: E8E8E60400           call    .0004AEC5A

.00460572: EB2D                 jmps    .0004605A1

.00460574: 6A01                 push    001
```

我试所有的类似`CMP xxxx,20`

`JB/JL xxxx`（该行用`90 90`替换）

经验证发现许多应该显示的字符串也没有显示，惨惨惨！问题出在比较的方式不一样。您可以用`HEXEDIT`打开一个`EXE`文件，所有被显示字符串末尾有一个字节`00H`，当读取字符串完毕后读入字符`00H`，此时`JB/JL`必然成立，必须正常跳转，不能被改成`NOP`，所以改思路：

`CMP WORD PTR [XXXX],20`

`JB/JL XXXX ==>90 90`

`CMP BYTE PTR [XXXX],20` {对照(3)(4) (7)处} `==>20`改`00`

`JB/JL XXXX ==>JB`改`JE`

完整记录下要修改处的`CS:IP`或16进制字串，用`UNASPACK`脱掉`W32DASM`的外壳，再用`HEXEDIT`（必须记录被修改处的16进制字串）或`HACKVIEW`修改，最后您去测试，中文信息是不是全显示出来了？

版权所有，请不要修改原内容 `4dd466d0e1e0cc123afbe504da2462ca`
