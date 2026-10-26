
### 代码对比

`007CF01B`.  `8B15 A81D8300`   `mov edx,dword ptr ds:[831DA8]`     ;  dumped(?0083689C

`007CF021`.  `8B12`            `mov edx,dword ptr ds:[edx]`        ;  `EDX`中存放我们输入的注册码111222333444555666777888999000

`007CF023`.  `A1 18188300`     `mov eax,dword ptr ds:[831818]`

`007CF028`.  `8B00`            `mov eax,dword ptr ds:[eax]`        ;  `EAX`中存放在007907CC处生成的注册码b417d593d4834

`007CF02A`.  `E8 2D63C3FF`     `call dumped(?0040535C`             ;  对真假注册码进行对比，我们在这里F7跟进，看看是怎么比较的。

`007CF02F`.  `85C0`            `test eax,eax`

`007CF031`      `/74 0D`           `je short dumped(?007CF040`         ;  通常，在这里是一个爆破点但是程序还在别的地方对注册码进行判断，所以，仅仅在这里实施爆破是不行的。还要在后面进行爆破。

下面是真正的算法，该算法根据机器码算出序列号，是我们关注的重点之一。

### 核心算法

```
007907CC   /$  55              push ebp
007907CD   |.  8BEC            mov ebp,esp
007907CF   |.  83C4 C0         add esp,-40
007907D2   |.  53              push ebx
007907D3   |.  56              push esi
007907D4   |.  33C9            xor ecx,ecx
007907D6   |.  894D C0         mov dword ptr ss:[ebp-40],ecx
007907D9   |.  894D C4         mov dword ptr ss:[ebp-3C],ecx
007907DC   |.  894D C8         mov dword ptr ss:[ebp-38],ecx
007907DF   |.  894D F4         mov dword ptr ss:[ebp-C],ecx
007907E2   |.  894D F0         mov dword ptr ss:[ebp-10],ecx
007907E5   |.  8955 F8         mov dword ptr ss:[ebp-8],edx
007907E8   |.  8945 FC         mov dword ptr ss:[ebp-4],eax
007907EB   |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]
007907EE   |.  E8 154AC7FF     call dumped(?00405208
007907F3   |.  8D75 DC         lea esi,dword ptr ss:[ebp-24]
007907F6   |.  33C0            xor eax,eax
007907F8   |.  55              push ebp
007907F9   |.  68 F70A7900     push dumped(?00790AF7
007907FE   |.  64:FF30         push dword ptr fs:[eax]
00790801   |.  64:8920         mov dword ptr fs:[eax],esp
00790804   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00790807   |.  8B55 FC         mov edx,dword ptr ss:[ebp-4]
0079080A   |.  E8 E145C7FF     call dumped(?00404DF0
0079080F   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00790812   |.  B9 01000000     mov ecx,1
00790817   |.  BA 01000000     mov edx,1
0079081C   |.  E8 974AC7FF     call dumped(?004052B8
00790821   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00790824   |.  B9 01000000     mov ecx,1
00790829   |.  BA 02000000     mov edx,2
0079082E   |.  E8 854AC7FF     call dumped(?004052B8
00790833   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00790836   |.  B9 01000000     mov ecx,1
0079083B   |.  BA 03000000     mov edx,3
00790840   |.  E8 734AC7FF     call dumped(?004052B8
00790845   |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]
00790848   |.  B9 01000000     mov ecx,1
0079084D   |.  BA 06000000     mov edx,6
00790852   |.  E8 614AC7FF     call dumped(?004052B8
00790857   |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]
0079085A   |.  E8 619CC7FF     call dumped(?0040A4C0
0079085F   |.  8BD8            mov ebx,eax                       ;  eax=e8242862，eax是由硬盘的序列号得来的
00790861   |.  F7D3            not ebx                           ;  ebx=17DBD79D，ebx是对eax取反
00790863   |.  81F3 22211276   xor ebx,76122122                  ;  ebx=61C9F6BF，ebx与76122122进行异或
```

以下部分是分别取出ebx中的4个byte

```
00790869   |.  8BC3            mov eax,ebx
0079086B   |.  25 000000FF     and eax,FF000000
00790870   |.  C1E8 18         shr eax,18                        ;  eax=61
00790873   |.  8906            mov dword ptr ds:[esi],eax
00790875   |.  8BC3            mov eax,ebx
00790877   |.  25 0000FF00     and eax,0FF0000
0079087C   |.  C1E8 10         shr eax,10
0079087F   |.  8946 04         mov dword ptr ds:[esi+4],eax      ;  eax = C9
00790882   |.  8BC3            mov eax,ebx
00790884   |.  25 00FF0000     and eax,0FF00
00790889   |.  C1E8 08         shr eax,8                         ;  eax=F6
0079088C   |.  8946 08         mov dword ptr ds:[esi+8],eax
0079088F   |.  8BC3            mov eax,ebx
00790891   |.  25 FF000000     and eax,0FF
00790896   |.  8946 0C         mov dword ptr ds:[esi+C],eax      ;  eax=BF
```

提取后的值放分别放在 `ds:[esi]`,`ds:[esi+4]`,`ds:[esi+8]`,`ds:[esi+C]`，下面分别对这四个值进行计算：

```
00790899   |.  8B16            mov edx,dword ptr ds:[esi]        ;  edx = 61
0079089B   |.  81E2 C0000000   and edx,0C0                       ;  edx = 40
007908A1   |.  8B4E 04         mov ecx,dword ptr ds:[esi+4]      ;  ecx=C9
007908A4   |.  81E1 C0000000   and ecx,0C0                       ;  ecx=C0
007908AA   |.  C1E9 02         shr ecx,2                         ;  ecx=30
007908AD   |.  03D1            add edx,ecx                       ;  edx=70
007908AF   |.  8B4E 08         mov ecx,dword ptr ds:[esi+8]      ;  ecx=f6
007908B2   |.  81E1 C0000000   and ecx,0C0                       ;  ecx=C0
007908B8   |.  C1E9 04         shr ecx,4                         ;  ecx=0C
007908BB   |.  03D1            add edx,ecx                       ;  edx=7C
007908BD   |.  25 C0000000     and eax,0C0                       ;  eax=80
007908C2   |.  C1E8 06         shr eax,6                         ;  eax=02
007908C5   |.  03D0            add edx,eax                       ;  edx=7E
007908C7   |.  8955 CC         mov dword ptr ss:[ebp-34],edx
007908CA   |.  8B06            mov eax,dword ptr ds:[esi]        ;  eax=61
007908CC   |.  83E0 30         and eax,30                        ;  eax=20
007908CF   |.  C1E0 02         shl eax,2                         ;  eax=80
007908D2   |.  8B56 04         mov edx,dword ptr ds:[esi+4]      ;  edx=C9
007908D5   |.  83E2 30         and edx,30                        ;  edx=00
007908D8   |.  03C2            add eax,edx                       ;  eax=80
007908DA   |.  8B56 08         mov edx,dword ptr ds:[esi+8]      ;  edx=F6
007908DD   |.  83E2 30         and edx,30                        ;  edx=30
007908E0   |.  C1EA 02         shr edx,2                         ;  edx=0C
007908E3   |.  03C2            add eax,edx                       ;  eax=8C
007908E5   |.  8B56 0C         mov edx,dword ptr ds:[esi+C]      ;  edx=BF
007908E8   |.  83E2 30         and edx,30                        ;  edx=30
007908EB   |.  C1EA 04         shr edx,4                         ;  edx=03
007908EE   |.  03C2            add eax,edx                       ;  eax=8F
007908F0   |.  8945 D0         mov dword ptr ss:[ebp-30],eax
007908F3   |.  8B06            mov eax,dword ptr ds:[esi]        ;  eax=61
007908F5   |.  83E0 0C         and eax,0C                        ;  eax=00
007908F8   |.  C1E0 04         shl eax,4                         ;  eax=00
007908FB   |.  8B56 04         mov edx,dword ptr ds:[esi+4]      ;  edx=C9
007908FE   |.  83E2 0C         and edx,0C                        ;  edx=08
00790901   |.  C1E2 02         shl edx,2                         ;  edx=20
00790904   |.  03C2            add eax,edx                       ;  eax=20
00790906   |.  8B56 08         mov edx,dword ptr ds:[esi+8]
00790909   |.  83E2 0C         and edx,0C
0079090C   |.  03C2            add eax,edx
0079090E   |.  8B56 0C         mov edx,dword ptr ds:[esi+C]
00790911   |.  83E2 0C         and edx,0C
00790914   |.  C1EA 02         shr edx,2
00790917   |.  03C2            add eax,edx
00790919   |.  8945 D4         mov dword ptr ss:[ebp-2C],eax     ;  eax=27
0079091C   |.  8B06            mov eax,dword ptr ds:[esi]
0079091E   |.  83E0 03         and eax,3
00790921   |.  C1E0 06         shl eax,6
00790924   |.  8B56 04         mov edx,dword ptr ds:[esi+4]
00790927   |.  83E2 03         and edx,3
0079092A   |.  C1E2 04         shl edx,4
0079092D   |.  03C2            add eax,edx
0079092F   |.  8B56 08         mov edx,dword ptr ds:[esi+8]
00790932   |.  83E2 03         and edx,3
00790935   |.  C1E2 02         shl edx,2
00790938   |.  03C2            add eax,edx
0079093A   |.  8B56 0C         mov edx,dword ptr ds:[esi+C]
0079093D   |.  83E2 03         and edx,3
00790940   |.  03C2            add eax,edx
```
