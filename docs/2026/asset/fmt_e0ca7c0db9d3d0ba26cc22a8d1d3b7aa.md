
哈哈哈...成功啦吗???...别高兴的太早... 关掉WT...重新执行WT...哇勒!@#$%...还是有延迟画面... 而且看萤幕最上面还有 UNREGISERED !!! 这个死德国佬...还布有暗桩...!@#$%.....

10.关掉WT...用 S-ICE 的 Loader...载入`Wtex95.exe`... 先按F8一次...再一直按F10...直到如下...

```asm
XXXX:0046BD0D MOV EBX,EAX
XXXX:0046BD0F MOV EAX,EDI
XXXX:0046BD11 CALL 00402C10 ;这锅CALL就是延迟画面啦...
```

所以往上看...看哪里可以避开此`CALL`...如下: 11.

```asm
XXXX:0046BCE3 MOV EBX,00000001
XXXX:0046BCE8 MOV EAX,[0046E6B4]
XXXX:0046BCED CMP BYTE PTR [EAX+000001D0],00
```

嘿嘿...`[EAX+000001D0]`就是旗标值罗...

```asm
XXXX:0046BCF4 JNZ 0046BD16 ;若不相同就跳过延迟画面...
```

12.所以我们先 `BD *` 在下 `BPM XXXX:YYYYYYYY` (就是`EAX+000001D0`那锅位址)... 看看程式是哪里在改这锅旗标值... 按F5...关掉WT...重新执行WT... 拦下後...多按几次 `G`...直到如下...

13.

```asm
XXXX:0044D77D MOV EBX,EAX
XXXX:0044D77F MOV BYTE PTR [EBX+000001D0],00 ;嘿嘿...抓到罗...
XXXX:0044D786 MOV EAX,EBX ;暗桩一...
XXXX:0044D788 CALL 0044D1A4 ;暗桩二...
XXXX:0044D78D CMP BYTE PTR [EBX+000001D0],00 ;返回前最後比对处...
XXXX:0044D794 JZ 0044D7A0 ;若是0...就跳到失败的地方罗...
```

先别急著改`XXXX:0044D77F MOV BYTE PTR [EBX+000001D0],00`... 因为程式後面还有暗桩...还会将`[EBX+000001D0]`这锅位址填0... 所以到`0044D78D CMP BYTE PTR [EBX+000001D0],00`这行才改... 下 `CODE ON` 指令...此指令共 7 BYTES... 所以可以改成`MOV BYTE PTR [EBX+000001D0],01`... 而`0044D794 JZ 0044D7A0`...改成两锅 `NOP`... 哈哈哈... 这一次真的破啦...死德国佬...

14.重新整理一次: 改 `Wtex95.exe`

```text
FIND: 80 BB D0 01 00 00 00 74 0A
EDIT: C6 83 D0 01 00 00 01 90 90

name: yhq
code: 4694
```

---

## yhq 破解实战录 07 --- PrimaSoft AutoFTP v1.0 破解法 !!!

以下称此软体为AF...(不是GF啦...我快想疯罗!!!)

1.输入name: yhq

      code: 87654321 (先乱写)

2.Ctrl-D 切入S-ICE...

3.下`bpx hmemcpy`指令...

4.按F5切回AF...按OK...

5.S-ICE拦下後...`BD *`把拦中断的功能关掉 按几次F12...跳回AF领空...

6.然後一直按F10....直到下面:

```asm
XXXX:0045F87E MOV EAX,[EBP-08]
XXXX:0045F881 POP EDX
XXXX:0045F882 CALL 0045C6A8 ;要害CALL!!!
XXXX:0045F887 TEST AL,AL    ;嘿嘿...眼熟吧!!!
XXXX:0045F889 JZ 0045F8C0   ;跳到0045F8C0就GAME OVER啦!!!
XXXX:0045F88B MOV BYTE PTR [EBX+000001C4],01
```

你要是跳到`0045F8C0`...要按F10一阵子才会看到失败窗... 我为啥知道这锅跳跃点是关键... 其实你要是跳到`0045F8C0`...接下来会有一堆`RET`(返回)指令... 所以大概猜想离"死期"不远罗... 而且这锅跳跃离`0045F889`的下一锅指令 `0045F88B MOV BYTE PTR [EBX+000001C4],01`有一点距离... 所以大胆判断...这是关键跳跃点... 所以在 `0045F882 CALL 0045C6A8` 设断...重新`TRACE`一次...

7.进入`0045F882 CALL 0045C6A8`那锅`CALL`後...一直按F8... 期间你会看到程式运算注册码的过程... 在这因为弟没时间KEY-IN...故略过... 仅列出比对核心要害处... 一直按F8...直到如下:

```asm
XXXX:0040396B MOV ESI,EAX
XXXX:0040396D MOV EDI,EDX
XXXX:0040396F CMP EAX,EDX ;命中要害!!!比对核心罗!!!
XXXX:00403971 JZ 00403A06 ;若相同则跳到00403A06
```

执行到`0040396F CMP EAX,EDX` 下 `D EDI` 指令: 看到输入的注册码 87654321 下 `D ESI` 指令: 看到真正的注册码 987654820-968

8.再次整理一遍:

软体名: PrimaSoft AutoFTP v1.0
姓  名: yhq
注册码: 987654820-968

---

## yhq 破解实战录 08 --- HyperSnap-DX v3.00 破解法 !!!

以下称此软体为HS...

1.执行HS...出现延迟画面...

2.Ctrl-D 切入S-ICE...

3.下`bpx lockmytask`指令...

4.按F5切回HS...按OK...

5.S-ICE拦下後...`BD *`把拦中断的功能关掉 按几次F12...跳回HS领空(HSDX主程式模组)...

6.然後一直按F10....直到下面:

```asm
XXXX:0040F6FE PUSH 0046CC44
XXXX:0040F703 PUSH ECX
XXXX:0040F704 PUSH EAX
XXXX:0040F705 CALL [00490D54] ;用F10带过此行...
XXXX:0040F70B TEST EDI,EDI     延迟画面就跑出来啦...
XXXX:0040F70D JZ 0040F745
```

所以往上看...看哪里可以避开此`CALL`...如下: 7.

```asm
XXXX:0040F685 CALL 00426C10
XXXX:0040F68A CMP DWORD PTR [0048547C],00 ;比对旗标值
XXXX:0040F691 PUSH EBX
XXXX:0040F692 PUSH ESI
XXXX:0040F693 PUSH EDI
XXXX:0040F694 JNZ 0040F745 ;若[0048547C]之值等於00... 那就死定罗!!!
```

8.所以在`0040F685`那行设断...再重新执行一次... 拦下後...追进去... 但是...发觉...好像没有更动`[0048547C]`的指令出现... 不信邪... 下`BPM XXXX:YYYYYYYY` (即`[0048547C]`的位址)... 按下`G`...结果还是在`0040F691 PUSH EBX`拦下... 表示此行的上一行`0040F68A CMP DWORD PTR [0048547C],00` 有读写此位址...当然嘛...要比较...当然要去读咩... 除此之外就没读写那锅位址的指令... 我偏不信邪...`BPM XXXX:YYYYYYYY`仍然开著... 重新载入执行...哇勒...还是在老地方拦下... 但是重头载入执行...你会发觉 `0040F68A CMP DWORD PTR [0048547C],00`被栏两次(检查两次) 可见没有其他的指令读写那锅位址... 算了...那就不管啦... 执行到`0040F694 JNZ 0040F745` 下`CODE ON`...记下机械码... 然後下`A`... 改成`JMP 0040F745` 嗯...强迫跳过啦...:p

9.这一套破的蛮丑的... 还有Unregistered version的标记... 不管啦...用UltraEdit改掉就好啦... 刚试了一下(2000年)...功能都没问题... 应该ok啦...收工!!!

10.重新整理一次:

软体名: HyperSnap-DX v3.00

改 `HSDX.EXE`

```text
FIND: 83 3D 7C 54 48 00 00 75 2A 8D 45 0C 50
EDIT: -- -- -- -- -- -- -- EB -- -- -- -- --
--: 表不用修改之处。
```

---

## yhq 破解实战录 09 --- 汉Ｘ通 v3.0 破解法 !!!

以下称此软体为XXX...

1.执行XXX...并且到输入序号的画面...输入87654321(乱填) 此套软体是依据注册码(Registered No,)(每套应该不同) and一些其他因素(ex:时间)来算出序号(Serial No.)... 因为我第二次trace时...序号又跟上次不同(若我没记错) 所以hacker版上的序号无效...

2.Ctrl-D 切入S-ICE...

3.下`bpx hmemcpy`指令...

4.按F5切回XXX...按OK...

5.S-ICE拦下後...`BD *`把拦中断的功能关掉 按几次F12...跳回XXX领空(XXX主程式模组)...

6.然後一直按F10....直到下面:

```asm
XXXX:3A49 PUSH 5EDA
XXXX:3A4C PUSH 00
XXXX:3A4E CALL USER!MESSAGEBOX ;这是WIN95中USER模组的API...
                              用来显示讯息的...
                              用F10带过此CALL...
                              错误讯息就跑出来啦...
```

所以往上看...看哪里可以避开此`CALL`...如下: 7.

```asm
XXXX:39E5 REPNZ SCASB
XXXX:39E7 NOT CX
XXXX:39E9 SUB DI,CX
XXXX:39EB REPZ CMPSB ;要害!!!比对核心...
XXXX:39ED JZ 39F4    ;若相同则跳到39F4...即避开"死亡的界线"
```

所以在`XXXX:39EB REPZ CMPSB`设断...重新执行XXX... 拦下後... 下 `D SI` 会看到输入的注册码...87654321 下 `D DI` 会看到正确的注册码...XXXXXXXXX(每锅人不同)

---

## yhq 破解实战录 10 --- The Bologna Pony Express v2.25 破解法 !!!

以下称此软体为BPE...

1.先设定好BPE...上线...执行BPE...并且抓图抓至50个... 就会出现限制的视窗啦!!! 然後离线...省钱钱咩... 再试著抓一次(按BPE的Find Now键)...限制视窗就跑出来啦... 先按下ok...关掉限制视窗...

2.Ctrl-D 切入S-ICE...

3.下`bpx postmessage`指令... 我为啥知道这个限制视窗是用`postmessage`这个API来执行显示的... 因为我有试嘛...试很多开窗...显示讯息的API...才试出来的...

4.按F5切回BPE...按Find Now键...

5.S-ICE在此拦下:

```asm
USER!POSTMESSAGE
XXXX:0001 MOV BX,SP
XXXX:0003 CMP WORD PTR SS:[BX+0C],00
```

6.然後 `BD *` 把中断点先暂停... 一直按F12....跳回BPE领空(BPE主程式模组)... 此时...小心慢慢按F12...一直到跳出S-ICE并显示限制视窗为止... 按下OK键...又回到S-ICE... 可看到如下的程式码:

```asm
XXXX:0043424D CMP WORD PTR [0044D050],19
XXXX:00434255 JLE 0043426A ;若[0044D050]的资料小於等於19H
                          则跳到0043426A
XXXX:00434257 CALL 00446A70 ;限制视窗...死翘翘的地方...
XXXX:0043425C MOV WORD PTR [0044D030],FFFF ;S-ICE在此拦下...
XXXX:00434265 JMP 0043482D
XXXX:0043426A MOV EDX,[EBP+FFFFFF3C]
```

大胆推测一下...在`00434255 JLE 0043426A`设断... 重新`TRACE`一次...拦下後...`R IP`...将IP改成`0043426A`... 嘿嘿...限制视窗不见了...而且程式处於连线状态... 嘿嘿...成功了吗???...还得真正连线试试看... PS:其实这里的`19H`(等於10进位的25)... 你要是自己往下追...你就会知道...为啥啦... `25*2=50`...自己追追看...:P

7.先用工具(EX:ULTRAEDIT)将`00434255 JLE 0043426A`的机械码改成`JMP 0043426A`... 然後连线抓图...嘿嘿...发生啥事... 只抓了一锅图就出现限制视窗... 所以还有暗桩... 先按OK...关掉限制视窗...CTRL-D 切回S-ICE... 一样用`BPX POSTMESSAGE`...抓图...拦下... 一直F12到BPE的领空...换一直F10... 你会回到`0043424D CMP WORD PTR [0044D050],19` 然後就要抓改`[0044D050]`的要害罗... `BPM XXXX:YYYYYYYY` (`[0044D050]`的位址)... 然後慢慢下`G`指令... 直到如下:

```asm
XXXX:0044426E MOVSX EAX,WORD PTR [0045D050]
XXXX:00444275 MOV [EBO+FFFFFF08],EAX
```

这两行使`[EBO+FFFFFF08]=[0045D050]`

```asm
XXXX:0044427B MOV DWORD PTR [EBP-04],0000004D
XXXX:00444282 MOV ECX,[EBP+FFFFFF08]
```

将`[EBP+FFFFFF08]`的值搬到`ECX`...

```asm
XXXX:00444288 ADD ECX,01 ;嘿嘿...将ECX加一...
XXXX:0044428B MOV [EBP+FFFFFF08],ECX ;加一後在存回去
```

所以我们得转移目标啦... `BPM AAAA:BBBBBBBB` (`[EBP+FFFFFF08]`的位址)

8.设断後...开始下`G`指令...慢慢来喔... 直到如下:

```asm
XXXX:00444428 CMP DWORD PTR [EBP+FFFFFF08],31 ;嘿嘿...比对核心罗...
                                               31H=49(10进位)...
XXXX:0044442F JLE 0044444D ;若是[EBP+FFFFFF08]中的资料
                           比31H小就跳到0044444D...
XXXX:00444431 MOV DWORD PTR [EBP-04],00000053
XXXX:00444438 CALL 00446A70 ;死亡的界线 !!!
XXXX:0044443D MOV DWORD PTR [EBP-04],00000054
XXXX:00444444 NOV WORD PTR [EBP-04],FFFF
XXXX:0044444D MOV DWORD PTR [EBP-04],00000056
```

所以再将`0044442F JLE 0044444D`改成`JMP 0044444D` 就收工啦 !!!

9.再次整理一遍:

软体名: The Bologna Pony Express v2.25

修改码: 改 `BolognaPonyT.exe`

```text
FIND: 7E 13 E8 14 28 01 00
```

